# RestApi-Flask-Python-Simple

this repository will teach you how to create simple rest Api and make it more secure with Python using Flask Framework 

## Step 1 :
install the libraries we need using terminal

```bash
pip install Flask
pip install Flask-HTTPAuth
```

## Step 2 :

### create our flask app
```python
from flask import Flask

# create app
app = Flask(__name__)


# run our app from here
if __name__ == "__main__":
    app.run(debug=True)
```

## Step 3 :
### get function
create our functions to get and send the data with our Api <br/><br/>
read [this](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods) if you need to know more about request methods

```python
# we need to import jsonify here
from flask import Flask, jsonify

# create app
app = Flask(__name__)

# we set the method of the request is post 
# to keep the parameter hidden from the hackers
@app.route("/get", methods=["POST"])
def get():
    
    # for example we have this data from our database
    employees = [
            ("ali", 18, "engineer"),
            ("ahmed", 25, "designer")
           ]
    
    json_data = {}
    data = []
    
    # this is not data structure class kkkkkkk ok? 🤡
    for employee in employees:
        
        json_data["name"] = employee[0]
        json_data["age"] = employee[1]
        json_data["title"] = employee[2]
        
        # we use copy method here because if we don't use it 
        # it will return one dictionary  test it without copy method 😏😏
        data.append(json_data.copy())
        
        
    # we have many ways to return the data from our Api 
    # but i will teach you how to return it as json
    return jsonify({"data": data})
    
    
# run our app from here
if __name__ == "__main__":
    app.run(debug=True)
```
#Example :
this part get the parameters from our Api url
for example we will use localhost server to 
make request to our Api
```python
from requests import request

url = 'http://127.0.0.1:5000/get'
r = request(method="POST", url=url)

print(r.text)

```

```output:```
```text
{
    data:
        [
           {
               "name":"ali",
               "age": 18,
               "title":"engineer"
           },
           {
               "name":"ahmed",
               "age": 25,
               "title":"designer"
           },
       ]
}
```



## Step 4 :
### push function

```python
# to get parameters form the request
# we need to import request here 😱😱
from flask import Flask, request

# create app
app = Flask(__name__)

# I will not explain why we using POST method 
# you should know from the past example
@app.route("/add", methods=["POST"])
def push():
    # get parameter from the request
    data = request.args.get("data")
    
    # this should be the output of the next example
    print("data is : ", data)
    
    
    
# run our app from here
if __name__ == "__main__":
    app.run(debug=True)
```

# Example :

```python
from requests import request

url = 'http://127.0.0.1:5000/get'

data = "hello world 😎😎"

r = request(method="POST", url=url, params={"data":data})

r.close()

```

```output on the server:```

```text
data is : hello world 😎😎
```

# Note:
```text
you can use the push function to update or insert data on you database
i will show you how to do that in the next example.
```

# Example :
you can make another function and give it any name you like and update your data throw it

```python
# to get parameters form the request
# we need to import request here 😱😱
from flask import Flask, request

# create app
app = Flask(__name__)

# I will not explain why we using POST method 
# you should know from the past example
@app.route("/add", methods=["POST"])
def push():
    # get parameter from the request
    data = request.args.get("data")
    
    # here you need to make connection with your database
    con =  #your connection
    cur =  #your cursor
    
    cur.execute('''INSERT INTO YOURTABLE(word) VALUES(%s)''', (data,))
    con.commit()
    
    # you must close your connection
    # because if you don't close it 
    # you will get an error i'm idiot in English 🤨
    con.close()
    
    
    
# run our app from here
if __name__ == "__main__":
    app.run(debug=True)

```

# Security
If you want to make your API you must sure that you are the only one who can reach it
so I will show you two methods to add security to your API


#First Method

you can use auth in your API so when someone sent request to your API 
he/she can't get receive or send data through your API if he/she does not  have auth information.
let's write our code 😎

```python
# this is our code from the last example
from flask import Flask, request

# we need to import basic auth from the Flask-HTTPAuth library 
from flask_httpauth import HTTPBasicAuth


# create app
app = Flask(__name__)

# we need to make object from the HTTPBasicAuth class
auth = HTTPBasicAuth()


# we need to set password and user name 
Login = {"This is the username": "this is the password"}


# set our url and set the method of the request
@app.route("/add", methods=["POST"])
# we need here to set our auth to check the 
# username and password before call the function push
@auth.login_required
def push():
    # get parameter from the request
    data = request.args.get("data")
    
    # here you need to make connection with your database
    con =  #your connection
    cur =  #your cursor
    
    cur.execute('''INSERT INTO YOURTABLE(word) VALUES(%s)''', (data,))
    con.commit()
    
    # you must close your connection
    # because if you don't close it 
    # you will get an error i'm idiot in English 🤨
    con.close()
    
# here we need to make our function to check 
# the username and password
@auth.verify_password
def verify(username, password):
    if not(username and password):
        return False
    return Login.get(username) == password

    
    
# run our app from here
if __name__ == "__main__":
    app.run(debug=True)

```

# Example:
Make request to our Api

```python
from requests import request
# we need to import basic auth 
from requests.auth import HTTPBasicAuth

url = 'http://127.0.0.1:5000/get'

data = "hello world 😎😎"
username = "This is the username"
password = "this is the password"

auth = HTTPBasicAuth(username, password)

r = request(method="POST", url=url, params={"data":data}, auth=auth)

r.close()

```



# Second Method

This method is similar to a trick because you will enter the password
or any other information as parameter in order to verify the request.

```python
# this is our code from the last example
from flask import Flask, request

# create app
app = Flask(__name__)

# we need to set password and user name 
Login = {"This is the username": "this is the password"}


def verify(username, password):
    if not(username and password):
        return False
    return Login.get(username) == password


# set our url and set the method of the request
@app.route("/add", methods=["POST"])
def push():
    # get parameter from the request
    password = request.args.get("password")
    user = request.args.get("user")
    
    if verify(user, password):
    
        # here you need to make connection with your database
        con =  #your connection
        cur =  #your cursor
        
        cur.execute('''INSERT INTO YOURTABLE(word) VALUES(%s)''', (data,))
        con.commit()
        
        # you must close your connection
        # because if you don't close it 
        # you will get an error i'm idiot in English 🤨
        con.close()
    
    else:
        return "kkkk you can't access🙄🙄"
    
    
# run our app from here
if __name__ == "__main__":
    app.run(debug=True)
```


# Example:
Make request to our Api

 ```python
from requests import request

url = 'http://127.0.0.1:5000/get'

data = "hello world 😎😎"
username = "This is the username"
password = "this is the password"


parameters = {"data":data, 'user':username, 'password':password}
r = request(method="POST", url=url, params=parameters)

r.close()


```

In the end you can combine the two methods , and there is other methods of Authentication 
you can read it [here](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication)
to make your Api more secure

### I hope this explanation is useful to you 😍



## License

kkk there is no license you can copy it as you like 😃😎


