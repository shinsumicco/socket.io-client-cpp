# Socket.IO C++ Client
[![Build Status](https://travis-ci.org/shinsumicco/socket.io-client-cpp.svg)](https://travis-ci.org/shinsumicco/socket.io-client-cpp)

By virtue of being written in C++, this client works in several different platforms.  
The [examples](https://github.com/shinsumicco/socket.io-client-cpp/tree/master/examples) folder contains an iPhone, QT and Console example chat client!  
It depends on [websocket++](https://github.com/zaphoyd/websocketpp) and is inspired by [socket.io-clientpp](https://github.com/ebshimizu/socket.io-clientpp).

[![Clients with iPhone, QT, Console and web](https://cldup.com/ukvVVZmvYV.png)](https://github.com/socketio/socket.io-client-cpp/tree/master/examples)

## Features

- 100% written in modern C++11
- Compatible with socket.io 1.0+ protocol
- Binary support
- Automatic JSON encoding
- Multiplex support
- Similar API to the Socket.IO JS client
- Cross platform

## Quickstart

**[Full overview of API can be seen here](./API.md)**

The APIs are similar to the JS client.

#### Connect to a server

```cpp
sio::client h;
h.connect("http://127.0.0.1:3000");
```

#### Emit an event

```cpp
// emit event name only:
h.socket()->emit("login");

// emit text
h.socket()->emit("add user", username);

// emit binary
char buf[100];
h.socket()->emit("add user", std::make_shared<std::string>(buf,100));

// emit message object with lambda ack handler
h.socket()->emit("add user", string_message::create(username), [&](message::list const& msg) {});

// emit multiple arguments
message::list li("sports");
li.push(string_message::create("economics"));
socket->emit("categories", li);
```

Items in `message::list` will be expanded in server side event callback function as function arguments.

#### Bind an event

##### Bind with function pointer

```cpp
void OnMessage(sio::event &) {
    // handle login message
    // post to UI thread if any UI updating.
}

h.socket()->on("new message", &OnMessage);
```

##### Bind with lambda

```cpp
h.socket()->on("login", [&](sio::event& ev) {
    // handle login message
    // post to UI thread if any UI updating.
});
```

##### Bind with member function

```cpp
class MessageHandler {
public:
    void OnMessage(sio::event &);
};

MessageHandler mh;
h.socket()->on("new message",std::bind( &MessageHandler::OnMessage,&mh,std::placeholders::_1));
```

#### Using namespace
```cpp
h.socket("/chat")->emit("add user", username);
```
**[Full overview of API can be seen here](./API.md)**

## License

MIT
