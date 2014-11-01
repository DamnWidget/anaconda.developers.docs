# Anaconda GUI Listeners

Anaconda GUI listeners lives into the `listeners` package in the plugin root directory they are no more complex than the GUI commands, in fact the smaller of them is ten lines long being four of them docstring and other two blank lines so the real code longitude is just four lines of code as it' shown below

```python
class AnacondaAutoformatPEP8EventListener(sublime_plugin.EventListener):
    """Anaconda AutoPEP8 formatter event listener class
    """

    def on_pre_save(self, view):
        """Called just before the file is going to be saved
        """

        if is_python(view) and get_settings(view, 'auto_formatting'):
            view.run_command('anaconda_auto_format')
```

## Anatomy of a GUI Listener

The biggest (and probably most complex) is the one responsible of the linting process, and it's just one hundred thirty eight lines long, as you can see the anaconda architecture makes the code pretty easy to maintain.

Listeners work in a very similar way as the GUI commands do, the most obvious difference is that they are triggered by GUI events like open or save a file, focus a file or start/stop writing. Obviously, depending on the feature that they implement they are more or less complex.

In the case of the `AnacondaAutoformatPEP8EventListener` it just runs the GUI command `AnacondaAutoFormat` as soon as a Python buffer is being saved so it's interaction with the JsonServer is delegated to the GUI command.

A more complex example that interacts itself with the JsonServer but simple enough to don't get lost in implementation details is the `AnacondaSignaturesEventListener` that is used to display the signature of whatever function that is under the cursor in any moment in a completely asynchronous way and with a perfect performance that no any other plugin can replicate.

```python
# Copyright (C) 2013 - Oscar Campos <oscar.campos@member.fsf.org>
# This program is Free Software see LICENSE file for details

import sublime_plugin

from functools import partial

from ..anaconda_lib.worker import Worker
from ..anaconda_lib.helpers import prepare_send_data, is_python, get_settings


class AnacondaSignaturesEventListener(sublime_plugin.EventListener):
    """Signatures on status bar event listener class
    """

    documentation = None
    exclude = (
        'None', 'str', 'int', 'float', 'True', 'False', 'in', 'or', 'and')

    def on_modified(self, view):
        """Called after changes has been made to a view
        """

        if not is_python(view) or not get_settings(view, 'display_signatures'):
            return

        try:
            location = view.rowcol(view.sel()[0].begin())
            if view.substr(view.sel()[0].begin()) in ['(', ')']:
                location = (location[0], location[1] - 1)

            data = prepare_send_data(location, 'doc', 'jedi')
            Worker().execute(partial(self.prepare_data, view), **data)
        except Exception as error:
            print(error)

    def prepare_data(self, view, data):
        """Prepare the returned data
        """

        if data['success'] and 'No docstring' not in data['doc']:
            self.documentation = data['doc'].splitlines()[2]
            if self.documentation.split('(')[0] not in self.exclude:
                if self.documentation is not None and self.documentation != '':
                    self._show_status(view)
                    return

        view.erase_status('anaconda_doc')

    def _show_status(self, view):
        """Show message in the view status bar
        """

        view.set_status(
            'anaconda_doc', 'Anaconda: {}'.format(self.documentation)
        )

```

Let's walk trough the code

```python
import sublime_plugin

from functools import partial

from ..anaconda_lib.worker import Worker
from ..anaconda_lib.helpers import prepare_send_data, is_python, get_settings
```

We import the needed `sublime_plugin` module as we have to inherit our class from the `sublime_pugin.EventListener` class. We also import here our already known `Worker` class as well as some handy helpers, the only new one is `get_settings` that we will review later.

```python
    documentation = None
    exclude = (
        'None', 'str', 'int', 'float', 'True', 'False', 'in', 'or', 'and')
```

As with the GUI commands, we define a class variable named `documentation` that we are going to use to store the docstrings that we need to display our signatures. We define a list of excluded terms that will be ignored.

```python
    def on_modified(self, view):
        """Called after changes has been made to a view
        """

        if not is_python(view) or not get_settings(view, 'display_signatures'):
            return
```

The `on_modified` method is fired from the Sublime Text 3 plugin system as soon as the buffer is modified. We check if the buffer is python and if the `display_signatures` option is set as `True` if any of those are `False` we return and do nothing.

**note**: The `get_settings` helper method is a handy function that allow us to check if a given configuration option is set in the global `Anaconda.sublime-settings` configuration file or in the Project configuration file and in case that is not set anywhere just return a default value, in he last lines the default will be `None` that evaluates as `False` in the context of the example.

```python
        try:
            location = view.rowcol(view.sel()[0].begin())
            if view.substr(view.sel()[0].begin()) in ['(', ')']:
                location = (location[0], location[1] - 1)

            data = prepare_send_data(location, 'doc', 'jedi')
            Worker().execute(partial(self.prepare_data, view), **data)
        except Exception as error:
            print(error)
```

We enclose the code into a `try except` block, as usual we capture the location of the cursor in the current buffer and set it as value for the semantic variable `location`. The `(`, `)` conditional is not illustrative and it will be ignored as it just an implementation detail of the specific feature.

As with the GUI commands, we prepare the JSON data to be sent as a Python dictionary using the helper `prepare_send_data` and set the value of the variable `data` with its returned value. Then we instantiate a new instance of the `Worker` class and call it's `execute` method passing this time a standard `functools` library `partial` object that will be called back when a response from the JsonServer is available.

If there is some exception we just log them into the Sublime Text 3 console

```python
    def prepare_data(self, view, data):
        """Prepare the returned data
        """

        if data['success'] and 'No docstring' not in data['doc']:
            self.documentation = data['doc'].splitlines()[2]
            if self.documentation.split('(')[0] not in self.exclude:
                if self.documentation is not None and self.documentation != '':
                    self._show_status(view)
                    return

        view.erase_status('anaconda_doc')
```

The `prepare_data` method will be called by the `partial` wrapper with the `view` object and the `data` that came from the JsonServer. We check that the `success` element of the returned `data` dictionary is `True` and that the string `'No docstring'` is not present in the `doc` element of the dictionary.

Then we process the returned data to extract the method signature from the documentation that the JsonServer just returned back to us and show it to the user using the `_show_status` method that just use the Sublime Text 3 status bar to display the information.

Note than we don't set the `AnacondaSignaturesEventListener.documentation` as `None` as this class is not recursive and it doesn't call itself again to display results. This is what we call a continuous syncing class.

Finally we call the `erase_status` method from the Sublime Text 3 `view` object if there is nothing to show

**note**: we need to use the `partial` object because we need a way to reference the `view` that made the call to the JsonServer in a clean and reliable way


## How are the listeners injected into the plugin?

As with the GUI commands, the listeners are injected into the plugin using the `__all__` exported objects list in the `listeners/__init__.py` file.

```python
# Copyright (C) 2013 - Oscar Campos <oscar.campos@member.fsf.org>
# This program is Free Software see LICENSE file for details

from .linting import BackgroundLinter
from .completion import AnacondaComletionEventListener
from .signatures import AnacondaSignaturesEventListener
from .autopep8 import AnacondaAutoformatPEP8EventListener


__all__ = [
    'BackgroundLinter',
    'AnacondaComletionEventListener',
    'AnacondaSignaturesEventListener',
    'AnacondaAutoformatPEP8EventListener'
]
```

Again, whatever that is not part of the `__all__` exported objects list will not be loaded by the plugin and will not be available in runtime.
