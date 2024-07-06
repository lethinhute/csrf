# Task 1: normal transaction with CRSF vulnerability

## 1.1: Login, Check balance

## 1.2: Doing the transaction

## 1.3: Tranfer money illegitimately

# Task 2: CSRF Countermeasure implementation

## 2.1: Solution 1: Create a CSRF token generator
This is the simplest method to prevent csrf attack. In the python code base, we create a function to generate random token when the user is logged in:

``` 

def generate_csrf_token(): 
        return secrets.token_hex(16)

```

And when the login is successful, we also add to the cookie the generated token with the following line:
```

 resp.set_cookie('csrf_token', generate_csrf_token()) 

```

After that, we also have to make the change to login to include such csrf token invalid detection, using the following:

```

 def login():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        submitted_csrf_token = request.form['csrf_token']  # Get the submitted token
        
        # Verify CSRF token
        if submitted_csrf_token != session.get('csrf_token'):
            return "Invalid CSRF token", 403  # Forbidden
        
        if username in user_accounts and user_accounts[username]['password'] == password:
            resp = make_response(f"Logged in as {username}")
            resp.set_cookie('user_session', json.dumps({'username': username}))
            resp.set_cookie('csrf_token', generate_csrf_token())  # Regenerate token
            return resp
        else:
            return "Invalid credentials", 401 

```

## 2.2: Solution 2:
