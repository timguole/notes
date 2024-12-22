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

