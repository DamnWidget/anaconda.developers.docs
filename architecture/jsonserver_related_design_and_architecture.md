# JsonServer Related Design and Architecture

The JsonServer is an standalone standard library Python asynchronous server that understand how to speak a basic text based JSON protocol with whatever client that connects to it. The first thing to note about the JsonServer is that it **is not** an HTTP server so it doesn't know how to speak HTTP (and doesn't have all the overhead related with the HTTP protocol neither), it **is not** a JSON-RPC server neither, it is a custom text based protocol that doesn't add unnecessary overhead as the operations that it performs are simple enough to don't require of a weight control protocol.

# Protocol definition

The protocol is really simple, the server start listening bytes from the client, when some data is available to be read, it stores it into a read buffer. Every time that new data is received, the server checks if there is a terminator (specific symbol that indicates that the message is complete and has to be processed), this operation is performed internally by the `asynchat.async_chat` class and is handled by the `found_terminator` method of the `JSONHander` class in the `anaconda_server/jsonserver.py` module that inherits from the `asynchat.async_chat` that defines the compound symbol `\r\n` as terminator for the socket.

When the terminator is found, all the buffer content is copied into a local variable wisdomly named `message` and the read buffer is reset. Then the server enter in the `json_decode` context passing the `message` variable as parameter and decoding the JSON string that came from the socket into a variable named `data`.

Then the server checks if there is some data to be processed, if there is no data, it log it and returns creating a orphan callback (and possibly a leak if there is no timeout defined in the callback) in the client that sent the request, this has to be improved obviously.

Then the server extract the method, uid, vid and handler_type from the data and process it with the handler that the client specified.

## Anatomy of a JSON message

```python
{
    "method": "complete",
    "handler": "jedi",
    "uid": "e78f9e051df844a48fabcf850e4f44b1",
    "vid": 113,
    "line": 7,
    "offset": 11,
    "filename": "documentation.py",
    "source": "import os
import sys


class Example(object):
    def __init__(self):
        os."
}
```

The JSON data structure can include setting options and other arbitrary information depending on the command or feature that is being implemented but there is always some mandatory fields, they are.

* **method**: this is the method to be called in the remote handler
* **handler**: this is the remote handler that implements the method that we want to call
* **uid**: this is the unique hexed id that the JsonClient generates for each request
* **vid**: this is the view id that generated the Sublime Text 3 GUI event that fired the action (and isgoing to be updated with the result even if is not active when the information is available)

The command is handled by a `JSONHandler` method named `handle_command` that send the command to a registered anaconda handler that can be native into the anaconda plugin or registered by other plugin that uses anaconda as framework (like `anaconda_php`).

# How the code is ditributed?

The `anconda_server` was splitted up in the first (and uniq) anaconda code refactor to isolate it from the GUI specific code. Common code that is shared with the GUI specific code is stored into the `anaconda_lib` library.

The `anaconda_server` directory hierarchy is as follows:

```
├── commands
├── handlers
└── lib
    └── compat
```

Yes, there is a `commands` package but it has nothing to do with the GUI commands one.

## The handlers package

Every action that is performed by the JsonServer is performed trough a handler, actually, there are four handlers

1. **autoformat_handler**: this handler is being used by the AutoPEP8 format system
2. **jedi_handler**: this handler is used by any action that is related to the Jedi library
3. **python_lint_handler**: this handler is used by any action that perform any type of linting
4. **qa_handler**: this handler is used by any action related with *Quality Assurance* or code quality analysis

<blockquote>**note**: Plugins compatible with anaconda use their own handlers.</blockquote>

All the handlers **must** inherit from the base `anaconda_server.lib.anaconda_handler.AnacondaHandler` class, a part from that, they doesn't need to implement any specific method (but they can re-implement the `run` method that is invoqued from the `handle_command` method in the `jsonserver.py`).

Handlers implement the methods that are being called from the GUI commands and listeners so they will be named in the same way. That means that if you want to implement a GUI command that call a `winter_is_coming` method in the `got` handler, the `got` handler has to implement a `winter_is_coming` method on it.

Anaconda handlers are registered into the global `ANACONDA_HANDLERS` in the `__init__.py` file of the `handlers` package in programatically way. Plugins that use anaconda as platform, register their own handlers in dynamic way when the JsonServer is started and firstly used.

The handler methods can implement directly whatever functionality that is needed and return back a result but do that will convert them in big files hard to maintain. This is why finally there is a `commands` package, we implement on it each method that the handlers need to implement this make the code **smaller**, **easier to maintain** and **highly componentized** so any of the commands can be swapped with a more up to date version or a totally new one if needed with no or minor modification of any other part of the whole system.

# What is needed to add a new action to the plugin?

That depends if there is a handler already defined where you can attach the new action or you have to create a new one yourself.

### In case that there is a handler already

Just add a new method in the handler that will call your new action (that will be implemented inside the `commands` package of the `anaconda_server`) assign all the required parameters and making whatever pre-data processment or post processment that your action requires.

### In case that there isn't any handler

You have to create a a new handler into the `handlers` package and then follow the step described above
