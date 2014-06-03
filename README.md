# Flask-ini

Flask-ini is a Flask extension that allows your application to be
configured with configparser ini files.

Reasons you may want to do this include:

* Your Flask app is part of a larger application which is configured
  using ini files.
* You have a templated rollout system that plays ball better with ini
  files.  
* You don't like the idea of your config file containing executable
  Python code.
* You just like the format of ini files more.

## Installation

Install the extension with the following commands:

    $ pip install Flask-ini
    
## Usage

The FlaskIni object is a subclass of
[configparser.SafeConfigParser](https://docs.python.org/3/library/configparser.html)
and all of its methods are available.

When you use the `read()` or `readfp()` method, you must be in your
application's context, because these methods modify your application
config.

```python
from flask import Flask
from flask_ini import FlaskIni

app = Flask(__name__)
with app.app_context():
    app.iniconfig = FlaskIni()
    app.iniconfig.read('/etc/myapp.conf')
```

If your config file contains a section called `flask`, it will be used
to update Flask's own config object, according to the following rules:

1. Variable names will be transformed from lower to upper case.
2. If the variable exists in Flask's `default_config` dictionary, it
   will be assumed to have the same type as the default value.
3. If the variable does not exist in Flask's `default_config`
   dictionary, it will be assumed to have a string type.
   
Please note the following:

* One variable in the `default_config` has the type
  `datetime.timedelta`. Flask-ini can read this variable from your ini
  file and it will be interpreted as an integer number of days. If you
  want a more specfic delta it is recommended that you configure this
  variable another way.
* ConfigParser uses the words `true` and `false` for booleans, rather
  than Python's `True` and `False`. Likewise, strings must not be
  quoted.
  
## Example

### Config file

```ini
[flask]
; this is a comment
debug = true
secret_key = sesame1
session_cookie_name = foobar

[backend]
endpoint = http://www.example.net/rest
timeout = 360
```

### Application

```python
from flask import Flask
from flask_ini import FlaskIni

app = Flask(__name__)
app.iniconfig = FlaskIni()
with app.app_context():
    app.iniconfig.read('/etc/myapp.conf')

@app.route('/')
def example():
    if app.debug: # True
        endpoint = app.iniconfig.get('backend', 'endpoint')
        timeout  = app.iniconfig.getint('backend', 'timeout', fallback=300)
        return "Contacting endpoint %s for %d seconds" % (endpoint, timeout)
    else:
        return "Won't get here, thanks to the config file"
        
if __name__ == '__main__':
    app.run()
```
