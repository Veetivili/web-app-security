# Web Application Security

**Student number:** AB0197  
**Name:** Veeti Hakala    
**Group:** TIC21S  
**Time management:** Approximately 10 hours


## Week 02  

### JWT Tokens:

#### Old Wasdat - Curl - Change password using curl

**Title:** Change password feature doesn't require old password. Password can be changed via `CURL`.

**Description:** Web application has a vulnerable change password feature. Authentication with old password is not implemented in the application.

**Steps to produce:**  

1. Navigate to `https://wasdat.fi/8080`.
2. Create a user & login in to the website.
3. Open developer tools and head to user settings.
4. Change password in a browser and copy http request cURL. (Adding something in to `bio` field is required first.)
4. Modify curl statement:  
    > Remove browser header and set password to `JAMK2023` -> `cbcde1454599deb370203eaff81c2ba3ed01805e`  

    ```
    curl -i 'http://wasdat.fi:8080/api/user' \
    -X 'PUT' \
    -H 'Accept: application/json, text/plain, */*' \
    -H 'Accept-Language: en-US,en;q=0.9' \
    -H 'Authorization: Token eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJpYXQiOjE2OTQ0MjQ0NzgsIm5iZiI6MTY5NDQyNDQ3OCwianRpIjoiMzNhOTljODItMmJhMS00NmUzLWFlZmUtOGMwYWQ5NmUzZDI1IiwiZXhwIjo4ODA5NDQyNDQ3OCwiaWRlbnRpdHkiOjEsImZyZXNoIjp0cnVlLCJ0eXBlIjoiYWNjZXNzIiwid2FzIjp0cnVlfQ.crl_EFfcJn_GY0sWv5fQb75yjHMyVqZYAY8vloWoP9U' \
    -H 'Content-Type: application/json;charset=UTF-8' \
    -H 'Cookie: sessionid=4e5ruzr7hol8h73xk5p1m6mch2cr4w7l; csrftoken=W1e4kc6ldN7o6XAew1tiJQYmOIdgmaqm' \
    -H 'Origin: http://wasdat.fi:8080' \
    -H 'Proxy-Connection: keep-alive' \
    -H 'Referer: http://wasdat.fi:8080/' \
    --data-raw '{"user":{"email":"attacker@example.com","username":"attacker","bio":"attacker bio","image":null,"password":"cbcde1454599deb370203eaff81c2ba3ed01805e"}}' \
    --compressed \
    --insecure
    ```  
<img src="../images/WAS6.PNG" width="500" />



5. Run command from Kali CLI and password is set to `JAMK2023`.
6. You also get the flag: CurlFlagEarned: WasFlag4_1{PasswordSetWithCurl}`.


* Impact estimation:
    * Low Severity. User can make harm for other users such as add or remove items but can't access any
* Mitigation:
    * Use more complex identifiers, random generated UUIDs for example.
        * See ![4-Web_Application_Security_Testing/05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References)
    * Check two parameters: buid and authenticate for which user buid belongs to.  


---

#### Old wasdat - Craft JWT token with alg=none and change user's password

**Title:** Encoding algorithm can be bypassed with setting alg=none `JWT Token`.

**Description:** Web Application has misconfiguration in JWT Token authentication. Application has secret_key but it can be bypassed with setting encryption alg=none. Basically it sends and empty signature and application accepts it as valid signature.

**Steps to produce:**  

1. Navigate to `https://wasdat.fi/8080`.
2. Create a user & login in to the website.
3. Open developer tools and head to user settings.
4. Change password in a browser and copy http request cURL. (Adding something in to `bio` field is required first.)
4. Modify curl statement:  
    > Remove browser header and set password to `JAMK2023`.
    > Take first part of the `JWT Token` which is `eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9`. Decode in `https://jwt.io`.
5. You will see that payloads is: `{"typ":"JWT","alg":"HS256"}` change it to: `{"typ":"JWT","alg":"none"}`
6. Decode it again and replace old value in the Authentication Token field.
7. Run curl command from kali cli.
8. You'll get flag: `JWTFagEarned: WasFlag4_2{AlgNoneShouldBeDead}`


<img src="../images/WAS6.PNG" width="500" />


* Impact estimation:
    * Low Severity. User can access other users order history and view their address information.
* Mitigation:
    * Use more complex identifiers, random generated UUIDs for example.
        * See ![4-Web_Application_Security_Testing/05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References)
    * Check two parameters: buid and authenticate for which user buid belongs to.  
    * Don't include user id in url parameter.

---


#### Old wasdat - Craft JWT token with known secret and impersonate to be the victim :

**Title:** JWT Token `secret` has leaked, which is librarys default value.

**Description:** Web application has vulnerability in the JWT Token `secret`. It is using default value. Any user can impersonate other users because authentication is always validated if secret is known and is used in the `http payload`.

**Steps to produce:**  

1. Navigate to `https://wasdat.fi/8080`.
2. Create a user & login in to the website.
3. Open developer tools and head to user settings.
4. Change password in a browser and copy http request cURL. (Adding something in to `bio` field is required first.)
4. Modify curl statement:  
    > Remove browser header and set password to `JAMK2023`.
    > Copy the Authentication `JWT Token`. Decode in `https://jwt.io`.
5. You will see that payloads is:  

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
) 
``` 
6. Add `secret-key` to signature:

  
```
    HMACSHA256(
    base64UrlEncode(header) + "." +
    base64UrlEncode(payload),
    secret-key
    ) 
``` 

7. Decode it again and replace old value in the Authentication Token field.
7. Run curl command from kali cli.
8. You'll get flag: `JWTFagEarned: WasFlag4_3{AchievementUnlocked_MasterOfTokens}`

<img src="../images/WAS7.PNG" width="500" />


* Impact estimation:
    * Low Severity. User can make harm for other users by changing their usernames.
* Mitigation:
    * Require http headers not to allow requests from from other origins.
        * See: ![Cross-Site Request Forgery preventation](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html#verifying-origin-with-standard-headers)
---

