# ICP - Infinite communication prototcol

ICP is communication protocol which is focused on extensibility. ICP is created for communication between client and server (notice sometimes client of server can be also server, look to ReverseProxy extension).
ICP is based on extensions for example:

- Methods - for calling methods
- Variables - for getting/setting/subcribing variables
- Topics - for subcribing things
- Sessions - for working with sessions
- ReverseProxy - for give ability client for creating endpoint on server
- And many other, there are no limit for your ideas

## Local and remote

Because ICP is created for communicating beetwen client and server, each extension implements two parts:

- Local for server
- Remote for client
  For example if it is Variable extension:
- To create this variable on server you should create `LocalVariable`
- To work with this variable on client you should create `RemoteVariable`
  Often both local and remote implements same methods for example:

```
# Pseudo code

# Creating variable
localVariable = new LocalVariable(initialValue)
remoteVariable = new RemoteVariable()

# Getting it value
localVariable.get()
remoteVariable.get()

# Setting it value
localVariable.set(newValue)
remoteVariable.set(newValue)
```

But in reality value of this Variable is stored in **Local** and when you trying to get it from Remote it **sends request to get value**.
Also example with methods when you call `localMethod.call()` method is executed on this machine, but if you call `remoteMethod.call()` method will be executed on server

## Server and connection

As noted earlier there are two parts Local and Remote. For creating something Local you should give it server, and for creating sommething Remote you should give it connection. Connection and server describes transport, for example:

- TCP on port 32
- UPD on 432
- WS on address `/icp`
- Other BCP connection for ReverseProxy
- Serial port between two arduinos
- BLE
- Anything you want

For example over TCP

```
# Server, address is 1.2.3.4
server = new TCPServer(port=32)
method = new LocalMethod(sever, function(){
  # Here is our method
  pritn("Callen")
})

# Cient
client = new TCPClient(address="1.2.3.4", port=32)
method = new RemoteMethod(client)

method.call()
```

This code will connect to server and call method

# Endpoint

Endpoint is like path to file, it is need for creating lots of methods, variable and other things on one server. It is recomended to use unix-like path for endpointFor example lets say our server should have methods for getting time on server, variable with random value got from client and variable that increments when client connect, so we will have next endpoints:

- `/time` - method for getting time
- `/rand` - variable with random value from client
- `/clients` - variable which contains
  End example in pseudocode:

```
# Server
server = new TCPServer(port=32)
time = new LocalMethod(sever, "/time",function(){
  return getTime()
})

rand = new LocalVariable(sever, "/rand", 0)
clients = new LocalVariable(sever, "/clients", 0)


# Cient
client = new TCPClient(address="1.2.3.4", port=32)

time = new RemoteMethod(client, "/time")
rand = new RemoteVariable(client, "/rand")
clients = new RemoteVariable(client, "/clients")

# Getting time on server
time.call()

# Set random
rand.set(getRandom())

# Set clients
clients.set(clients.get() += 1)
```

Also it is able to use wildcards in path for example if you create LocalVariable with endpoint `/users/:id/followers` will proccess requests to
`/user/1/followers` and `/user/2/followers` and other `/user/smth/followers` and return followers **different for each user**. Sending endpoint required only from client to server

## Message id

For identification replies from server there is id. Id can be anything: array, object, number, string. For easy id generation you can just choose random number from 0 to 0xfffffff. Server **must** reply with id same as request, for example if client send request with id 124, server must reply with id 124

## Message type

And last parameter is type. Type is needed for extensions for example: variable what you can do with, you can get/set/subcribeChanges, so variable provide types get/set/subcribeChanges. For replying message it is recomended to use error/ok/publish. Type can be only string

## Message encoding

Each message is JSON-encoded object which must have:

- id - anything
- type - type for extension
- endpoint (if from client to server) - endpoint to call to
  It can also have other parameters provided by extensions.
  Example messages

```json
{"id":"Smth", "type": "get", "endpoint":"/var"}
{"id":089, "type": "ok"}
{"id":[0121,122], "type": "call", "endpoint":"/method"}
```

## Example communication

Here is example communication between client and server, they are using Method and Variable extension
And it is code for client and server

```
# Server
server = new TCPServer(port=32)
time = new LocalMethod(sever, "/time",function(){
  return getTime()
})

rand = new LocalVariable(sever, "/rand", 0)
clients = new LocalVariable(sever, "/clients", 0)


# Cient
client = new TCPClient(address="1.2.3.4", port=32)

time = new RemoteMethod(client, "/time")
rand = new RemoteVariable(client, "/rand")
clients = new RemoteVariable(client, "/clients")

# Getting time on server
time.call()

# Set random
rand.set(getRandom())

# Set clients
clients.set(clients.get() += 1)
```

And communication:

```json
// -> To server
// <- From server
-> {"id":54689, "type": "call", "endpoint": "/time"}
<- {"id":54689, "type": "ok", "return": 12890409}
-> {"id": 23998, "type": "set", "value": 0.89, "endpoint": "/rand"}
<- {"id": 23998, "type": "ok"}
-> {"id": 43908, "type": "get", "endpoint": "/clients"}
<- {"id": 43908, "type": "ok", "value": 4}
-> {"id": 23998, "type": "set", "value": 0.89, "endpoint": "/clients"}
<- {"id": 23998, "type": "ok"}
```

## Auto-generated documentation

ICP implementation should have method for getting documentation in next format:

```json
[
  // Array of endpoints
  {
    "endpoint": "/path/to/endpoint",
    "type": "typeOfEndpoint",
    "specificationOfType": "http://url.to/specification/ofTypeOfEndpoint",
    "description": "Here is documentation of endpoint"
  }
]
```

And here is example of documentation:

```
# Pseudo code of server
server = new WSServer(port=8032)
testVar = new LocalVariable(server, "/test/var", 0)
testMethods = new LocalMethod(server, "/test/method", function test(){
  print("smth"
})

testVar.documentate("Test variable")
testMethods.documentate("Test method")

server.getDocumentation() // return documentation
```

And here is generated docs:

```json
[
  {
    "endpoint": "/test/var",
    "type": "variable",
    "specificationOfType": "http://github.com/maximmasterr/ICP/tree/master/docs/variable.md",
    "description": "Test variable"
  },
  {
    "endpoint": "/test/method",
    "type": "method",
    "specificationOfType": "http://github.com/maximmasterr/ICP/tree/master/docs/method.md",
    "description": "Test method"
  }
]
```

## Implementing ICP Base in your language

First of all choose json encoder/decoder for your implementation. Then implement base of server and client, after that implement basic real using server and client, for example:

- TCP server/client
- WS server/client
- Serial server/client
- Input/output server/client

Notice you should it is highly recomended to give user ability to load only modules what they need

Then implement some extensions, it is highly recomend to implement Method and Variable extensions.

And the last step add your implementation to github and package manager.

### Recomended structure of implementation

Ok, if your language supports OOP use it. Ok let's start from implementing base of server, what methods should base server have:

- subcribe - which accepts path to endpoint and function which will called when there is message to this endpoint
- documentate - which accepts object that should be pushed to documentation
- getDocumentation - which returns serialized json of documentation
- handle - which should be called when server acceptes message, message and function to send reply to client (both json-decoded)

To subcribe callback function will be passed message and function that which accept json decoded msg that need to send to client as reply
Also note about handling messages you should iterate for each endpoint until will be find endpoint with path that matches to requested, if no endpoints no endpoints found send client `{"type": "error", "msg":"Endpoint '/test' not found", "code": -1}`
Here is an example of base server in pseudocode

```
class ICPServer {
  endpoints = new Array
  documentation = new Array

  subcribe(endpoint, callback){
    endpoints.push({endpoint, callback})
  }

  documentate(doc){
    documentation.push(doc)
  }

  getDocumentation(){
	return encodeJSON(documentation)
  }

  handle(msg, write){
    find = false
    for(i = 0; (i < endpoints.length) and not find; i++){
	  if(wildcardMatch(pattern=endpoints[i].endpoint, target=msg["endpoint"]){
		endpoints[i].callback(msg, function(reply){
		  reply["id"] = msg["id"]
		  write(reply)
		})

	  find = true
	  }
    }
    if(!find){
	  reply = decodeJSON("{\"type\": \"error\", \"msg\":\"Endpoint '" + msg["endpoint"] + "' not found\", \"code\": -1}")
	  reply["id"] = msg["id"]
	  write(reply)
    }
  }
}
```

Let's continue with writing base connection, what methods it should have:

- send - this method accept json decoded message, add random ID, encode it and send to server, also accepts callback which should be called when
- \_\_write - this method is used to send encoded message to server, will be replaced with real connection
- handle - should be called when client get message from server

Here is example of connection in pseudocode:

```
class ICPConnection {
  handlers = new Map # map is object that stores key-value
  send(msg, callback){
    id = random(from=0 to=0xffffff)
    msg["id"] = id
    handlers[id] = callback
    __write(encodeJSON(msg))
  }

  __write(msg){}

  handle(msg) {
    if(handlers.has(msg["id"]){
      handlers.get(msg["id"])(msg)
      handlers.delete(msg["id"])
	}
  }
}
```

Now we are going to implement real server/connection, I am going to use TCP so let's look how server should work:

- Server waiting for connection, when it gets connection it registers to get messages from it
- When server got message from client server parsing it char by char and adds each char to buf, when parses meets new line it parses json and call handle function

Here is implementation in pseudocode:

```
class TcpICPServer extends ICPServer {
  tcp = new TCPServer(port)
  tcp.onConnection(function(socket){
    buf = ""
    socket.onMessage(function(msg){
      for(char in msg){
        if(char == "\n"){
          handle(decodeJSON(buf), function(msg){
            socket.write(encodeJSON(msg) + "\n")
          })
        }else{
          buf+=char
        }
      }
    })
  })
}
```

Now let's implement TCP connection, what it should do:

- Replace default \_\_write function with function that should send message to server
- When it get message from server, connection should parse it char by char and when meets new line call handler function
  Here is example realization in pseudocode:

```
class TcpICPConnection extends ICPConnection {
  socket = new TCPConnection(address, port)
  buf = ""
  socket.onMessage(function(msg){
    for(char in msg){
      if(char == "\n"){
        handle(decodeJSON(buf))
      }else{
        buf+=char
      }
    }
  })

  __write(msg){
    socket.write(msg)
  }
}
```

Ok, that is all look out specification for Variable and Methods extensions
