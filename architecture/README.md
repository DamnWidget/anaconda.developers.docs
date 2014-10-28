# Anaconda Architecture and Design

Anaconda is a decoupled asynchornous client-server implementation where the presentation logic (tied directly with the Sublime Text 3 GUI) and the plugin logic is completely decoupled in a way that it makes possible to archieve the following goals:

1. Make possible the plugin to work with interpreters different than the Sublime Text 3 embedded one (that in the moment of writing those lines is Python 3.3.3)
2. Make possible to run every operation perfomed by the plugin in totally and real asynchronous way non blocking the Global Intepreter Lock (GIL) into the Sublime Text 3's python embedded intrepreter
3. Make possible to use Vagrant boxes
4. Make possible to work with remote servers
5. Make possible to work with PyPy, Stackless Python, Jython, IronPython and others
6. Make possible an independient plugable architecture that can be used to offer anaconda as asynchornous high performance framework for other plugins

This architecture has it's handicaps as well, the most important are:

1. Harder to maintain
2. Harder to test
3. More points of failure as the complexity increases

# The Anaconda Components

The plugin is effectively splitted up in two different components and in a shared library that is used by both components

### GUI Related Component

This component depends on the Sublime Text 3 runtime and it runs inside it's embedded Python Interpreter.

### The JsonServer (anaconda_server)

The JsonServer is a standalone standard Python library asynchronous server that executes all the heavy processment including the auto-completion, linting, complexity check, and code analysis.

# How this works?

When the plugin is loaded, the Sublime Text 3 plugin system calls the `plugin_loaded` hook that is present in the `anaconda.py` main plugin entry point. This hook prepare the `Main.sublime-menu` template and run the anaconda `IO loop` in a separate execution thread if it's not running yet.

When you open or focus a buffer that contains Python code, anaconda will then execute any of it's listeners hooks (probably autocomplete or linting) that will trigger the `Worker` system in order to process the operation in the standalone JsonServer. At this point, there is no JsonServer still running.

The `Worker` system is smart enough to be aware that there is no JsonServer running and start a new one using the configured python interpreter that the user has configured (if any). Anaconda will use the first port that is available in the system and register the Sublime Text 3 window that contains the Python code into the `IO Loop`. From now the plugin will send new requests from this Sublime Text 3 window to this new configured JsonServer.

If the user opens another window that contains Python code, another isolated JsonServer will be started to handle requests from this window. In this way we maintain every window isolated (so we ca use different Python interpreters in all of them) and we can use more than one core of the processor in the machine as they are independent processes.
