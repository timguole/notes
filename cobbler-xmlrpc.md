# Cobbler XMLRPC Note

## XMLRPC in Python

### Client side

**`xmlrpc.client.ServerProxy:`** the object to connect to XMLRPC Server and do the RPC.

**`xmlrpc.client.Server:`** the alias for xmlrpc.client.ServerProxy

### Server side

**`xmlrpc.server.SimpleXMLRPCServer:`** the object listen and accept RPC requests

**`xmlrpc.SimpleXMLRPCRequestHandler:`** the request handler

#### Methods in server object

**`register_function():`** register function(the procedure).

**`register_instance():`** register an object which contains the 'procedures'.

**`serve_forever():`** run the server

## Cobbler XMLRPC Server

### Objects

**`CobblerXMLRPCInterface:`** xmlrpc interface (wrapper) for CobblerAPI.

**`ProxiedXMLRPCInterface:`** proxy for CobblerXMLRPCInterface. It has the method '_dispatch()' which xmlrpc server object uses to dispatch rpc requests.

**`CobblerAPI:`** the api implementation

**`CobblerXMLRPCServer:`** the sublclass of xmlrpc.server.SimpleXMLRPCServer.

### Cobblerd

```python
In cobblerd.py
Start from core()

In core()
	setup logging
    read settings
    call do_xmlrpc_rw()

In do_xmlrpc_rw()
	create xinterface = ProxiedXMLRPCInterface(cobbler_api, remote.CobblerXMLRPCInterface)
    create server = remote.CobblerXMLRPCServer(("127.0.0.1", port))
    register: server.register_instance(xinterface)
        start running: server.serve_forever()
```

### remote.py

ProxiedXMLRPCInterface._dispatch() dispatches rpc requests to methods in CobblerXMLRPCInterface. So, all non-private method in CobblerXMLRPCInterface can be called via XMLRPC.

## Cobbler XMLRPC example

This script demostrates how to make xmlrpc in Python from Cobbler host.

```python
#!/usr/bin/python3
# Cobbler xmlrpc api demo (localhost xmlrpc)

import os
import sys
import xmlrpc.client
from cobbler import utils

if __name__ == "__main__":
    url_cobbler_xmlrpc = utils.local_get_cobbler_xmlrpc_url()
    shared_secret = utils.get_shared_secret()
    proxy = xmlrpc.client.Server(url_cobbler_xmlrpc)

    try:
        proxy.ping()
    except Exception as e:
        print("cobblerd does not appear to be running/accessible: %s" % repr(e), file=sys.stderr)
        exit(1)

    token = proxy.login("", shared_secret)

	# find system by an attribute. The attribute can be: name, mac_address, ip_address, etc.
	# Return value is a list of system names
    systems = proxy.find_system({'name': 'Foobar'})
    for s in systems:
        print(s)
```

Other API methods

```pytho
get_systems() # a list of systems with full attributes info
find_system({'attr': 'value'}) # find a system with the attr being value
remove_system(s, token) # remove a system
new_system(token) # create a system, return the object id
modify_system(oid, key, value, token) # modify attr of a system
save_system(oid, token) # save a system to disk file
sync(token) # make a sync
```

