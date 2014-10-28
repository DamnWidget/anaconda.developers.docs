# GUI Related Design and Architecture

The most important thing to remember about the code that runs into the embedded Python interpreter of Sublime Text 3 is that you **never** use the *asynchronous* versions of the Sublime Text 3 API as Sublime Text 3 **will not clean up your memory** when the anaconda's callback system is used.

Sublime Text 3 embedded python interpreter is not able to clean up memory allocated in local frames of threads created with the standard python threading library when the *thread safe* asynchronous versions of the Sublime Text 3 API runs the Python code of the plugin, I am not sure about why this happens but I guess is not a good idea to mess with C++ threads executing the Python embedded intrepreter and the GIL as Sublime Text 3 does on those methods.

For more information about that, you can [review this issue](https://github.com/DamnWidget/anaconda/issues/97#issuecomment-31624102) in the anaconda project site.

Anyway, there is no need at all to use the `async` metods in the Sublime Text 3 API as they don't really solve any problem of GUI freeze related with the code in the plugins as there is a GIL in the embedded intereter and if it gets lock everything get locked doesn't matters if the code is running from a different C++ thread or not.

Anaconda resolve this problem using it's decoupled architecture so every method that is called by the Sublime Text 3 plugin system returns inmediatly non blocking the embedded interpreter GIL but registering a callback for the action in the anaconda's callback system so use anaconda offers the most smooth and responsive experience that you are never going to feel using Sulime Text 3 plugins.

# How the code is distributed?

At the very beginning, anaconda code was distributed as any other Sulime Text plugin, everything in the root directory with no apparent order with big files that include all the commands and logic all together. This was pretty unmaintenable and in early stages the code was completely refactored to improve it's quality and make it much more maintenable (but harder to test, sorry about that).

The anaconda plugin directory hierarchy is a follows:

```
anaconda
    ├── anaconda_lib
    │   ├── autopep
    │   │   └── autopep8_lib
    │   │       └── lib2to3
    │   │           ├── fixes
    │   │           └── pgen2
    │   ├── builder
    │   ├── jedi
    │   │   ├── api
    │   │   ├── evaluate
    │   │   │   └── compiled
    │   │   │       └── fake
    │   │   └── parser
    │   └── linting
    │       ├── gutter_mark_themes
    │       └── pyflakes
    ├── anaconda_server
    │   ├── commands
    │   ├── handlers
    │   └── lib
    │       └── compat
    ├── commands
    |── listeners
    ├── messages
    └── templates
```

The `anconda_lib` package is shared between the GUI related and the JsonServer parts of anaconda plugin. The GUI related code uses the modules that exists directly in the `anaconda_lib` package while the JsonServer uses the `autopep`, `jedi` and `linting` modules.

The directories are pretty semantic and descriptives themselves, e.g. the `commands` directory contains all the plugin Sublime Text 3 commands while the `listeners` directory contains the plugin`EventListener` hooks.

Each command is contained in it self module that is named `<command_name>.py` e.g. `autoformat.py` that contains the command to trigger the `AutoPEP8` formatter in the JsonServer. In this way, when we want to add a new command to the plugin, we know that we have to add it in a new file into the `commands` directory so we can refer to that file later to fix any bug related with this specific command. Each plugin GUI component is isolated so you know exactly which files to look at when you are trying to solve a bug or add a new functionallity.

This architecture help us as well to be able to replace a component for another one in easy way without affect the rest of the plugin in any way. This improves code quality and is less error prone.
