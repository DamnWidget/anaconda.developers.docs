# Anaconda Plugable Architecture

Since version [v1.3.0](https://github.com/DamnWidget/anaconda/releases/tag/v1.3.0) anaconda supports a plugable interface that makes possible to use the anaconda asynchronous client-server architecture with other languages, the reference implementation is the [anaconda_php](https://github.com/DamnWidget/anaconda_php) plugin that adds linting, complexity checker and mess detector for PHP using anaconda as platform so it benefits of all the anaconda's performance characteristics.

# How it works?

Sublime Text 3 python runtime environment allows any plugin to import any other plugin as a library so any plugin can in theory import anaconda components and use them in it's own context, in the practice this is not that easy but it works.

Anaconda compatible plugins instantiate the anaconda's `Worker` system so they can communicate themselves with the standalone JsonServer to perform actions and fire commands. The JsonServer has to be able to load and incorporate new handlers for the anaconda compatible plugins so it adds a mechanism to register handlers dynamically when the JsonServer is being used.

For this to work, the plugin has to implement an specific packages hierarchy and it has to be named `anaconda_whatever`.

## Plugin Packages Template

The mandatory directory hierarchy of an anaconda compatible plugin is as follows:

```
├── plugin
│   └── handlers_<name>
└── templates
```

Obiously, the plugin can add whatever it needs on it owns to this hierarchy, but anaconda will look for whatever `handlers_<name>` package inside the `plugin` directory to import it and register it into the anaconda handlers registry, this operation is perform by a metaclass that `AnacondaHandler` inherits from.

## The plugin anaconda version compatibility

Each plugin may need an specific version of anaconda to work, this can (and must) be specified comparing the `anaconda_version` variable that contains a tuple of three elements (major, minor, patch) as value with whatever version you are expecting e.g.

```python
from Anaconda.version import version as anaconda_version

if anaconda_version < (1,3,4):
    raise RuntimeError("Anaconda v1.3.4 or better is required!")
```

## Anaconda vs anaconda imports mess

Anaconda can be installed trough GitHub by just cloning the repository into the Sublime Text 3 `Packages` directory or using the `Command Palette` as human that I am, I make mistakes and probably the bigest one was to name the plugin as `Anaconda` in the Packages Control packages channel so anyone that install the plugin using the `Command Palette` ends with a package called `Anaconda` in their `Packages` directory instead of the `anaconda` that get installed when using git.

Now is not that easy to change that so the solution is to make a wrapper around the imports just to make sure that if the anaconda plugin is installed from whatever source, the imports works fine. An example of this is the `anaconda_lib.anaconda_plugin` module in the `anaconda_php` plugin

```python
# Copyright (C) 2014 - Oscar Campos <oscar.campos@member.fsf.org>
# This program is Free Software see LICENSE file for details

ANACONDA_PLUGIN_AVAILABLE = False
try:
    from anaconda.listeners import linting
    from anaconda.anaconda_lib.worker import Worker
    from anaconda.anaconda_lib.helpers import is_code
    from anaconda.anaconda_lib.callback import Callback
    from anaconda.version import version as anaconda_version
    from anaconda.anaconda_lib.progress_bar import ProgressBar
    from anaconda.anaconda_lib import helpers as anaconda_helpers
    from anaconda.anaconda_lib.linting import sublime as anaconda_sublime
except ImportError:
    try:
        from Anaconda.listeners import linting
        from Anaconda.anaconda_lib.worker import Worker
        from Anaconda.anaconda_lib.helpers import is_code
        from Anaconda.anaconda_lib.callback import Callback
        from Anaconda.version import version as anaconda_version
        from Anaconda.anaconda_lib.progress_bar import ProgressBar
        from Anaconda.anaconda_lib import helpers as anaconda_helpers
        from Anaconda.anaconda_lib.linting import sublime as anaconda_sublime
    except ImportError as error:
        print(str(error))
        raise RuntimeError('Anaconda plugin is not installed!')
    else:
        ANACONDA_PLUGIN_AVAILABLE = True
else:
    ANACONDA_PLUGIN_AVAILABLE = True


__all__ = ['ANACONDA_PLUGIN_AVAILABLE']

if ANACONDA_PLUGIN_AVAILABLE:
    __all__ += [
        'Worker', 'Callback', 'ProgressBar', 'anaconda_sublime', 'is_code',
        'anaconda_version', 'linting', 'anaconda_helpers'
    ]
```

**note**: I am very sorry about that

## Warnings and workarounds

Maybe you expect to use the `get_settings` function as it is very handy in your compatible plugin but it would not work as `get_settings` always look into the `Anaconda.sublime-settings` file and is not going to look for options into your own file, a template to archieve the same result is as follows.

```python
def get_settings(view, name, default=None):
    """Get settings
    """

    if view is None:
        return None

    plugin_settings = sublime.load_settings('<YourPluginName>.sublime-settings')
    return view.settings().get(name, plugin_settings.get(name, default))
```

## Architecture and Design

You have freedom to design your plugin as you want as far as you decouple command calls and process of results from the process of the command and the generation of the result itself but I hardly recommend to approach an architecture and design similar to [anaconda_php](https://github.com/DamnWidget/anaconda_php) as it is after all a reference implementation of an anaconda compatible plugin.

For more information about how to decouple this, refer to the `anaconda_php` code and to the `Anaconda Architectire and Design` section in this same documentation.
