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

Allows the Python script/program to execute shell commands on the underlying OS. Below is a basic example of how this fcuntion() can be used.


