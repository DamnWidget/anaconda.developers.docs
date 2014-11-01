# The Anaconda Worker System

Anaconda uses `Worker` instances in order to run commands in the JsonServer and register callbacks that are fired when a response from the JsonServer is available to be consumed.

The `Worker` system consist in two files that lives in the `anaconda_lib` directory on the plugin's root directory

* `anaconda_lib/worker.py` implements the Local and Remote workers classes
* `anaconda_lib/jsonclient.py` is an implementation of the anaconda's `ioloop.EventHandler` asynchronous event driven socket

## The JsonClient

The JsonClient is just an implementation of the anaconda's `ioloop.Eventhandler` class. In the earlier days of the plugin, the client was using the standard library `asyncore` and `asynchat` implementations but they are not much than a toy and Windows support was totally broken. As there were no other real options @damnwidget implemented a small and simple I/O event driven socket that uses the widely accessible `select` mechanism to implement an asynchronous TCP socket client that was good enough to support the plugin needs.

The anaconda's `ioloop` is a classical asynchronous handler implementation to use it, user code has to implement a handler class that inherits from `ioloop.EventHandler` and implements at least the `handle_read` and `process_message` methods.

An example of a client implementation is as follows:

```python
import ioloop

class TestClient(ioloop.EventHandler):
    '''Client for test
    '''

    def __init__(self, host, port):
        ioloop.EventHandler.__init__(self, (host, port))
        self.message = []

    def ready_to_write(self):
        return True if self.outbuffer else False

    def handle_read(self, data):
        self.message.append(data)

    def process_message(self):
        print(b''.join(self.message))
        self.message = []
```

<blockquote>**note**: select is not a real good choice for environments that needs to handle lots of connections concurrently, for an asynchronous client that is not going to try to perform more than five or six operations concurrently is just fine.</blockquote>

The JsonClient knows how to communicate with the remote asynchronous socket of the JsonServer as well as register callbacks to handle the JsonServer responses when they are available.

Each JsonServer has a hashmap where the key is a hexed unique id and the content is a callback. Before to send a request to the JsonServer, a JsonClient instance generate a hexed unique id for that request and register a callback on it, then it send the hexed unique id to the JsonServer that will send it back with it results all together.

When the JsonClient handle a response from the JsonServer, it lookup in it's hexed unique ids hash map for the hex id and fire the registered callback with the data that the JsonServer returned. Then the callback is removed form the hash map.

The callback can be an anaconda `Callback` object or whatever other callable object.

## The Worker

The `Worker` is one of the most important parts of the GUI related code (if not the most), it's responsible of start a JsonServer when no server is running (because we just started up our Sublime Text 3 or due inactivity for long period of time).

It register each window in a self managed workers list for each Sublime Text 3 window that contains Python code, the worker can be local or remote so it make us able to use JsonServers in remote systems or in vagrant boxes for example.

When a worker is established and a JsonServe is started, it register a JsonClient instance and connects it to the JsonServer and start registering callbacks.

The worker checks for JsonSever healthy in every moment reloading or starting new servers as required, it is responsible of kill old JsonServers and start new ones on project or `python_interpeter` switches.

There is nothing special about the worker, it is a classical watcher wrapper around a socket.
