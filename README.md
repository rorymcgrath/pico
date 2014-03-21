##Install
`easy_install pico`
or
`pip install pico`


##Write a Python module:
```python
# example.py
import pico

def hello(name="World"):
    return "Hello " + name

```

## Start the server:
`python -m pico.server`

Or run behind Apache with mod_wsgi

##Call your Python functions from Javascript:

```html
<!DOCTYPE HTML>
<html>
<head>
  <title>Pico Example</title>
    <script src="/pico/client.js"></script>
    <script>
        pico.load("example");
    </script>
</head>
<body>
  <p id="message"></p>
  <script>
  example.hello("Fergal", function(response){
    document.getElementById('message').innerHTML = response;  
  });
  </script>
</body>
</html>

```

## What? How?

* Pico is a very small web application framework for Python & Javascript.
* Pico allows you to write Python modules, classes and functions that can be imported and called directly from Javascript.
* Pico is not a Python to Javascript compiler 
   - Pico is a bridge between server side Python and client side Javascript.
* Pico is a server, a Python libary and a Javascript library! The server is a WSGI application which can be run standalone or behind Apache with mod_wsgi.
* Pico is a Remote Procedure Call (RPC) library for Python without any of the hassle usually associated with RPC. Literally add one line of code (``import pico``) to your Python module to turn it into a web service that is accessible through the Javascript (and Python) Pico client libararies.


## So I have an Amazon EC2 Server. How do I install pico.
For this we are going to install pico to run with gevent (https://github.com/surfly/gevent). 
This is a concurrency library for python that includes a wsgi server. It's more 
efficient than apache mod_wsgi in many ways.

Firstly lets ssh into your machine. We now need to install some dependencies.
Lets start by installing greenlet.

`sudo easy_install greenlet`

Next we need to install cython.

`sudo apt-get install cython`

Now we can install the gevent module, this may take several minutes.

```
wget https://github.com/surfly/gevent/archive/1.0rc2.tar.gz
tar -xf 1.0rc2.tar.gz
cd gevent-1.0rc2/
python setup.py build
sudo python setup.py install
```


Now we need to make the pico server and the modules directory. 
For the purposes of this tutorial we will place the both of these in home directory.

`touch ~/pico_server`

`mkdir ~/modules`


Now using your preferred editor we add the following code to the pico_server script:

```python
#!/usr/bin/python
import gevent
import gevent.pywsgi
import pico.server
import sys
sys.stdout = sys.stderr # sys.stdout access restricted by mod_wsgi
path = '/home/ubuntu/modules/' # the modules you want to be usable by Pico
if path not in sys.path:
    sys.path.insert(0, path)
pico.server.STREAMING = True
print("Serving on port 8800")
server = gevent.pywsgi.WSGIServer(('0.0.0.0', 8800), pico.server.wsgi_app)
server.serve_forever()
```

This server needs to executable, so lets change the permissions for that now.

`chmod +x /home/ubuntu/pico_server`

Now we want to make a start stop script for the pico service.

`sudo touch /etc/init.d/pico`

Again using your prefered editor set the contents of this file to be:

```bash
#!/bin/bash
#/etc/init.d/pico
#

case "$1" in
  start)
    /home/ubuntu/pico_server &> /var/log/pico.log &
    ;;
  stop)
    killall pico_server
    ;;
  *)
    echo "Usage: /etc/init.d/pico {start|stop}"
    exit 1
    ;;
esac

exit 0
```

Again this file needs to be executable so lets change the premissions:

`sudo chmod +x  /etc/init.d/pico`

Now lets start the pico server
`sudo service pico start`

Now we just need to set some configurations for the apache server. You should have
apache 2 installed on your machine by the way. Here we create or edit the /etc/apache2/httpd.conf file.

`sudo touch /etc/apache2/httpd.conf`

And using your prefered editor (vim) set the contents of it to be:

```
ProxyPass /pico/ http://localhost:8800/pico/ retry=5
ProxyPassReverse /pico/ http://localhost:8800/pico/
```

Remove any wsgi references to pico if they exist.

The final step now is to enable the mod_proxy in apache so that the configuration 
file we just created works.

`sudo a2enmod proxy_http`
`sudo apache2ctl restart`

Now take the html for the example above and place it in your /var/www/ folder.
Take the python code from the example and place it in your ~/modules folder.
Navigate to your severs address in your browser and say hello to Fergal.





The Pico protocal is very simple so it is also easy to communicate with Pico web services from other languages (e.g. Java, Objective-C for mobile applications). See the client.py for a reference implementation.

See the [wiki](https://github.com/fergalwalsh/pico/wiki) for more information.


![](https://nojsstats.appspot.com/UA-34240929-1/github.com)
