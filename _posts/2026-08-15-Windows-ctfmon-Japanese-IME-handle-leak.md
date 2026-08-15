---
layout: post
title: Windows 10 Japanese IME Bug leading to Denial of Service
published: true
categories: [Research, Windows]
excerpt: I traced severe desktop lag to a Japanese IME Registry handle leak that left ctfmon.exe holding more than 104,000 handles. A zero-capability AppContainer reproduced it on Windows 10, and MSRC classified the finding as a product reliability bug.
---

My Windows desktop was lagging like hell. I closed Chrome and checked the usual memory suspects. Chrome was using around 3 GB on a machine with 64 GB of RAM. The whole desktop and text input kept randomly freezing.

The actual problem was `ctfmon.exe`, which had somehow accumulated more than 104,000 handles. Almost all of them pointed to one Japanese IME Registry key.

The interesting part was that a zero-capability AppContainer could trigger persistent handle allocation in a process outside its sandbox. I sent it to MSRC as a denial-of-service candidate. Microsoft summarized the technical result and classified it as a product bug because the demonstrated impact stayed within the same user session.

Fair enough. It is still a pretty ridiculous Windows bug, so here is the writeup.

---

### Affected version

Windows 10 Pro 22H2 x64, build 19045.6466

## The 104,000-handle ctfmon.exe

---

The machine had been running for approximately 261 hours when I caught it. Task Manager's RAM view pointed nowhere useful. `ctfmon.exe` had a completely abnormal handle count.

An elevated Sysinternals Handle snapshot showed:

```text
ctfmon.exe total handles: 104,670
Registry Key handles:     103,887
```

Grouping the named handles made the problem obvious:

```text
102,459  HKCU\SOFTWARE\Microsoft\IME\15.0\IMEJP\MSIME\AutoCharWidth
  1,386  HKCU\SOFTWARE\Microsoft\Spelling\Dictionaries
```

After restarting only `ctfmon.exe`, the process returned to:

```text
Total handles: 802
Key handles:   107
```

The desktop stopped lagging immediately. The Windows text-input broker had been carrying around more than 100,000 live Registry handles.

You can check the current count from PowerShell:

```powershell
Get-Process ctfmon | Select-Object Id, HandleCount, CPU,
    @{Name='WorkingSetMB'; Expression={[math]::Round($_.WorkingSet64 / 1MB, 1)}}
```

If the count is already out of control, signing out or restarting Windows will reset it. Restarting `ctfmon.exe` alone also worked for me. The shell needs sufficient integrity as the broker is elevated.

## Reducing it to an edit-client focus bug

The leaked path contains `IMEJP`, so Japanese IME was the obvious suspect. The necessary state turned out to be:

1. Microsoft Japanese IME is installed.
2. The Japanese keyboard layout is selected.
3. Conversion mode is active and the input indicator shows `あ`.
4. Focus moves between an edit client and a non-editing window.

A tiny WinForms program with two top-level windows was enough to reproduce the bug. One window contained a multiline `TextBox`; the other contained only a `Label`. A timer alternated activation between them.

The important part of the payload was basically this:

```csharp
int transitions = 0;
int targetTransitions = cycles * 2;

timer.Tick += delegate
{
    if ((transitions % 2) == 0)
    {
        editorForm.Activate();
        textBox.Focus();
    }
    else
    {
        plainForm.Activate();
    }

    transitions++;
    if (transitions >= targetTransitions)
    {
        timer.Stop();
        plainForm.Close();
        editorForm.Close();
    }
};
```

Fifty cycles in an ordinary process produced:

```text
Cycles:         50
Handles before: 820
Handles after:  927
Growth:         107
Growth/cycle:   2.14
```

The handles remained after the triggering process exited.

This also reproduced when activating native Notepad++ edit controls. Codex and Claude made the real-world leak much faster because their UIs generate a lot of edit-client activation. The minimal WinForms program established Japanese IME edit-client activation as the root cause.

## Confirming the Registry leak

I captured eight ordinary edit-client activation cycles in Process Monitor. `ctfmon.exe` performed exactly 32 successful `RegOpenKey` operations against:

```text
HKCU\SOFTWARE\Microsoft\IME\15.0\IMEJP\MSIME\AutoCharWidth
```

Its handle count increased by exactly 32 over the same run. Each successful open persisted as a live handle.

This is the observable root cause: edit-client activation in Japanese conversion mode opens handles to `AutoCharWidth` and leaves them alive in `ctfmon.exe`.

The closest CWE is [CWE-775](https://cwe.mitre.org/data/definitions/775.html), Missing Release of File Descriptor or Handle after Effective Lifetime.

## The AppContainer experiment

Once the standalone reproduction worked, I wanted to know whether a sandboxed process could cause the same allocation in the out-of-container broker.

I created a temporary AppContainer profile, gave it zero capabilities, copied the WinForms payload into its profile directory, and launched it using `PROC_THREAD_ATTRIBUTE_SECURITY_CAPABILITIES`. The launcher verified the sandbox through the child token.

The verified child properties were:

```text
TokenIsAppContainer: true
Integrity SID:       S-1-16-4096
Capabilities:        zero
Child exit code:     0
```

After 50 focus cycles:

```text
Caller sandbox:   AppContainer, zero capabilities
Caller integrity: Low (S-1-16-4096)
Cycles:           50
Handles before:   1,349
Handles after:    1,453
Total growth:     104
Key growth:       104
```

The child exited normally, and I deleted its temporary AppContainer profile. The 104 additional handles stayed in `ctfmon.exe`.

A separate restricted low-integrity process produced the same result:

```text
Cycles:         50
Handles before: 1,037
Handles after:  1,141
Growth:         104
Key growth:     104
```

The tested Windows 10 environment was:

```text
Windows 10 Pro 22H2 x64, build 19045.6466
ctfmon.exe:            10.0.19041.4522
MSCTF.dll:             10.0.19041.4522
JpnInputRouter.dll:    10.0.19041.5794
InputLocaleManager.dll: 10.0.19041.6456
```

The test requires Japanese conversion mode to be active before the AppContainer launches.

## MSRC classification

Microsoft publicly lists the AppContainer sandbox as a serviced security boundary. Its stated goal is isolation of code and data outside the sandbox according to the container's capabilities.

That made the result look boundary-relevant: a zero-capability AppContainer was causing persistent kernel-object allocation in an external Windows broker. I reported the persistent resource allocation to MSRC as an availability issue.

MSRC summarized the impact as progressive degradation of the same user session, gated by an already-active Japanese conversion mode, with recovery through restarting `ctfmon.exe`, signing out, or restarting Windows. Their security bar focuses on availability impact across another user, session, security principal, or the system.

A local application already has several direct ways to consume its user's resources. This primitive reaches an external broker and produces same-user degradation. MSRC therefore routed it to Windows product feedback.

The AppContainer-to-broker behavior is still technically interesting. The rejection was annoying after all that work, but the boundary logic makes sense. The result is a Windows reliability bug with a clean resource-leak primitive.

## Afterthoughts

This investigation started because I was annoyed that my expensive PC was lagging despite having 64 GB of RAM. It ended with a minimal WinForms reproducer, Process Monitor traces, an AppContainer launcher, and an MSRC report.

The main lesson for me was to separate component-boundary activity from security impact. A sandboxed process making an external broker allocate something looks suspicious. The deciding question is the new capability gained against another security principal.

This finding demonstrated same-user session degradation. The bug was real, the leak was real, and the 104,000 handles were definitely real. Even so, MSRC's servicing criteria placed it in product feedback.

At least now, if somebody searches why `ctfmon.exe` has over 100,000 Registry handles and their desktop is dying, there is an answer other than "restart your computer".

### Timeline

**2026-08-13 JST** — Desktop lag led to discovery of more than 104,000 handles in `ctfmon.exe`  
**2026-08-14 JST** — Reduced the issue to Japanese IME edit-client activation and reproduced it from a zero-capability AppContainer  
**2026-08-14 06:29 JST** — Submitted the denial-of-service candidate to MSRC  
**2026-08-14 16:12 JST** — MSRC classified it as a same-user product reliability bug  
**2026-08-15 JST** — Writeup published

### Links

- [Microsoft Security Servicing Criteria for Windows](https://www.microsoft.com/en-us/msrc/windows-security-servicing-criteria)
- [Microsoft AppContainer isolation documentation](https://learn.microsoft.com/en-us/windows/win32/secauthz/appcontainer-isolation)
- [CWE-775: Missing Release of File Descriptor or Handle after Effective Lifetime](https://cwe.mitre.org/data/definitions/775.html)
- [Microsoft Feedback Hub](https://support.microsoft.com/en-US/Windows/Apps/send-feedback-to-microsoft-with-the-feedback-hub-app)

***
