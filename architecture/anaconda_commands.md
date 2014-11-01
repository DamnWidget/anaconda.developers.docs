# Anaconda GUI Commands

Anaconda GUI commands lives into the `commands` package of the anaconda plugin root directory. Each one is an isolated piece of code that normally depends in some way of the `anaconda_lib` package where common functionality is implemented.

**note**: this is true with the exception of the Vagrant related commands that are all contained into the `commands/vagrant.py` file and the tests runner related commands that are contained into the `commands/anaconda_test_runner.py` files because it made sense to put commands that are feature related into the same file.

## Anatomy of an anaconda GUI command

Usually, anaconda GUI commands surprises new developers because their simplicity, that is because anaconda commands are really dumb and they know nothing about how to execute any of the actions related to them.

The only that an anaconda GUI command has to know how to do is to capture where the cursor is located into a buffer, the buffer itself and the name of the file being edited and pass that information to the anaconda's callback system that will handle the command trigger and delegate all the heavy process to the JsonServer while register a callback to process the response when it is ready to be delivered.

Every anaconda command has to know how to process the results from the callback system and how to present this information to the end user.

The usual design of an anaconda GUI command is as follows:

```python
# Copyright (C) 2013 ~ 2014 - Oscar Campos <oscar.campos@member.fsf.org>
# This program is Free Software see LICENSE file for details

import sublime
import sublime_plugin

from ..anaconda_lib.worker import Worker
from ..anaconda_lib.callback import Callback
from ..anaconda_lib.helpers import prepare_send_data, is_python


class AnacondaWhatever(sublime_plugin.TextCommand):
    """Clear docstring about what this command do
    """

    data = None  # this will be filled and used later

    def run(self, edit):
        """This is triggered by the Sublime Text 3 plugin system
        """

        if self.data is None:
            try:
                loc = self.view.rowcol(self.view.sel()[0].begin())
                jserver_command = 'whatever'
                jserver_handler = 'jedi'
                data = prepare_send_data(
                    loc, jserver_command, jserver_handler
                )
                Worker().execute(
                    Callback(on_success=self.prepare_data), **data)
            except Exception as error:
                logging.err(error)
        else:
            self.print_data(edit)

    def is_enabled(self):
        """Determine if this command is enabled or not
        """

        return is_python(self.view)

    def prepare_data(self, data):
        """
        This command is passed as callback so it's called when
        a result has been delivered from the JsonServer after
        process the command `whatever` request
        """

        if data['success'] is True:
            self.data = data['data']
            if self.data is None or self.data == '':
                self._show_status()
            else:
                sublime.active_window().run_command(self.name())
        else:
            self._show_status()

    def print_data(self, edit):
        """Dummy func that shows how to print data in a panel
        """

        data_panel = self.view.window().create_output_panel(
            'anaconda_data'
        )
        data_panel.set_readonly(False)
        region = sublime.Region(0, data_panel.size())
        data_panel.erase(edit, region)
        data_panel.insert(edit, 0, self.data)
        self.data = None
        data_panel.set_read_only(True)
        data_panel.show(0)
        self.view.window().run_command(
            'show_panel', {'panel': 'output.anaconda_data'}
        )

    def _show_status(self):
        """If something goes wrong, just inform the user
        """

        self.view.set_status(
            'anaconda_data', 'Anaconda: Something went wrong!'
        )
        sublime.set_timeout_async(
            lambda: self.view.erase_status('anaconda_data'), 5000
        )
```

Let me walk you into the code block by block

```python
# Copyright (C) 2013 ~ 2014 - Oscar Campos <oscar.campos@member.fsf.org>
# This program is Free Software see LICENSE file for details

import sublime
import sublime_plugin

from ..anaconda_lib.worker import Worker
from ..anaconda_lib.callback import Callback
from ..anaconda_lib.helpers import prepare_send_data, is_python
```

Here we import the `Worker`, `Calback` and some helpers being the `prepare_send_data` the most important one. `Worker` class instances knows how to communicate themselves with the JsonServer and execute commands on it as `Workers` know how to speak Json with the `anconda_server`.

We use a `Callback` instance to register the command callback that has to be fired when the JsonServer returns back some data and the command is success (`AnacondaWhatever.prepare_data` in this case).

The `prepeare_send_data` helper is commonly used in all the commands and listeners to maintain the code clean and don't repeat code all around (remember, we have to maintain the code as DRY as possible always that it makes sense as it improves it's maintainability).

The `is_python` helper is used to activate this command and make it available under `Contextual Menu` and `Command Palette` (or even to don't execute it if is called from the Sublime Text 3 console) only in Python files.

**note**: is a requirement that all the anaconda commands include a method to deactivate or activate them depending on the context, that is mandatory.

```python
class AnacondaWhatever(sublime_plugin.TextCommand):
    """Clear docstring about what this command do
    """

    data = None  # this will be filled and used later
```

In this case, we need to inherit our class from the `sublime_plugin.TextCommand` class as we need to write into a panel in the GUI (for detailed information about how to write code for Sublime Text 3, refer to it's [API reference documentation](https://www.sublimetext.com/docs/3/api_reference.html)).

We define a class level property called `data` that will be used later.

```python
def run(self, edit):
        """This is triggered by the Sublime Text 3 plugin system
        """

        if self.data is None:
```
We use the `data` property to know when we have to fire the command execution into the JsonServer or process it results when they become available reusing the Sublime Text 3 plugin call functionality itself.

```python
            try:
                loc = self.view.rowcol(self.view.sel()[0].begin())
```

We always enclose the code into a `try except` block just to don't fail miserably in case that something goes wrong. As I already noted before, we capture the current location of the cursor in the current buffer and store it in the `loc` variable.

```python
                jserver_command = 'whatever'
                jserver_handler = 'jedi'
                data = prepare_send_data(
                    loc, jserver_command, jserver_handler
                )
```

The `jserver_command` is the command that we want to execute in the JsonServer context, this command is supposed to be handled by the `jedi` handler (that's not true as this command does not exists but this is useful for our illustrative purposes) so this is translated to
<blockquote>Hey JsonServer, use the jedi handler to execute the whatever command</blockquote>

Then we prepare the data that we are going to send to the JsonSever using the `prepare_send_data` command that just prepare a common JSON structure for us represented as a Python dictionary.

```python
                Worker().execute(
                    Callback(on_success=self.prepare_data), **data)
```

This is an important piece of code here. We instantiate a new instance of the `Worker` class and call it's execute method with two arguments.

1. An instance of the Callback class with the `on_success` callback set as `self.prepare_data`
2. The data dictionary that was prepared in the previous block unpacked

```python
            except Exception as error:
                logging.err(error)
        else:
            self.print_data(edit)
```

If there is some exception we just logging the error in the Sublime Text 3 console. The else correspond to the first conditional check, this is hit when we have a result coming back from the JsonServer and our function is called again by the callback that we passed to the `Callback` instance.

```python
    def prepare_data(self, data):
        ...
        if data['success'] is True:
            self.data = data['data']
            if self.data is None or self.data == '':
                self._show_status()
            else:
                sublime.active_window().run_command(self.name())
        else:
            self._show_status()
```

This piece of code is pretty straightforward, this method is called when there is a result available in the `Callback` mechanism with a single parameters that is whatever response that came from the JsonServer after process our request.

We check that the request was successful and then set the value of the `AnacondaWhatever.data` property with the data that the JsonServer returned back. If there is no data at all, we call the method `self._show_status()` that just display an error message to the user in the status bar.

Otherwise we call the `AnacondaWhatever` method again using the internal Sublime Text 3 API call, as the `AnacondaWhatever.data` property is set, it will hit the `else` conditional at the beginning of the `run` method and then the `print_data` dummy method will be called.

**note**: the specific data structure that is returned back from the JsonServer will be explained in subsequent sections

```python
def print_data(self, edit):
        """Dummy func that shows how to print data in a panel
        """

        data_panel = self.view.window().create_output_panel(
            'anaconda_data'
        )
        data_panel.set_readonly(False)
        region = sublime.Region(0, data_panel.size())
        data_panel.erase(edit, region)
        data_panel.insert(edit, 0, self.data)
        self.data = None
        data_panel.set_read_only(True)
        data_panel.show(0)
        self.view.window().run_command(
            'show_panel', {'panel': 'output.anaconda_data'}
        )
```

When we write the data that came back from the JsonServer into the panel we set the `AnacondaWhatever.data` as `None` so subsequent calls to the `AnacondaWhatever` command will trigger the call to the JsonServer, otherwise we would get the same panel again and again. This is how we reuse the built-in Sublime Text 3 plugin call mechanism in a way that allow us to be 100% asynchronous for real.

The `is_enabled` and `_show_status` methods doesn't deserve an explanation as they are common methods with no special code on them.

## How are the commands injected into the plugin?

The anaconda main entry point (`anaconda.py` file in the root directory) just includes a line that import all the symbols in the `commands` package. The exported symbols in the package are controlled by the `__init__.py` script into the `commands` directory so when you add a new command, you have to make sure that it is being part of the `__all__` list.

```python
# Copyright (C) 2013 - Oscar Campos <oscar.campos@member.fsf.org>
# This program is Free Software see LICENSE file for details

from .doc import AnacondaDoc
from .goto import AnacondaGoto
from .rename import AnacondaRename
from .mccabe import AnacondaMcCabe
from .get_lines import AnacondaGetLines
from .autoimport import AnacondaAutoImport
from .autoformat import AnacondaAutoFormat
from .find_usages import AnacondaFindUsages
from .enable_linting import AnacondaEnableLinting
from .next_lint_error import AnacondaNextLintError
from .disable_linting import AnacondaDisableLinting
from .complete_func_args import AnacondaCompleteFuncargs
from .set_python_interpreter import AnacondaSetPythonInterpreter
from .test_runner import (
    AnacondaRunCurrentFileTests, AnacondaRunProjectTests,
    AnacondaRunCurrentTest, AnacondaRunLastTest
)
from .vagrant import (
    AnacondaVagrantEnable, AnacondaVagrantInit, AnacondaVagrantStatus,
    AnacondaVagrantUp, AnacondaVagrantReload, AnacondaVagrantSsh
)

__all__ = [
    'AnacondaDoc',
    'AnacondaGoto',
    'AnacondaRename',
    'AnacondaMcCabe',
    'AnacondaGetLines',
    'AnacondaVagrantUp',
    'AnacondaVagrantSsh',
    'AnacondaAutoImport',
    'AnacondaAutoFormat',
    'AnacondaFindUsages',
    'AnacondaVagrantInit',
    'AnacondaRunLastTest',
    'AnacondaEnableLinting',
    'AnacondaNextLintError',
    'AnacondaVagrantEnable',
    'AnacondaVagrantStatus',
    'AnacondaVagrantReload',
    'AnacondaRunCurrentTest',
    'AnacondaDisableLinting',
    'AnacondaRunProjectTests',
    'AnacondaCompleteFuncargs',
    'AnacondaSetPythonInterpreter',
    'AnacondaRunCurrentFileTests',
]
```

Anything that is not part of the `__all__` list of exported objects will not be available in the plugin.
