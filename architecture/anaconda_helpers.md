# Anaconda Helpers

Helpers is a small library of plugin utilites that are common to tedious actions that you have to repeat and repeat again and again that are related to the Sublime Text 3 API itself. They are quite useful as you don't have to copy and paste the same lines from method to method (remember that DRY is one of our mantras) all around. Let me walk you trough it.

## is_code

The `is_code` function determines if whatever is under the cursor in the active buffer in the moment that the function is called is code in the given language (`python` by default).

Signature:
```python
is_code(view, lang='python', ignore_comments=False, ignore_repl=False)
```

The `view` parameter is just the view that we are working on and **must** be passed always as it is the only mandatory argument.

If `lang` is different than `python` then, the check will be performed to that language, this was introduced when anaconda was turn into a plugable architecture platform, for example, this is used by `anaconda_php` plugin passing `php` as language instead of `python`

If the `ignore_comments` is `True` then the check will return a positive result even if what is under the cursor is a comment, this is useful to things like the `PEP257` linter that lints docstrings.

Finally if `ignore_repl` is set as `True`, anaconda will return postive result inside `SublimeREPL` buffers, this may or may not be desirable depending on the feature that you are implementing.

## is_python

This is exactly as the `is_code` function but is used only in anaconda related actions itself (as anaconda as plugin works only with Python), and it's just a covenience method.

## check_linting

This method checks if the linting process has to be fired depending on several conditions, this method is useful to linter implementors.

Signature:
```python
check_linting(view, mask, code='python')
```

The `view` parameter is the view that we are working on, the `mask` parameter is a bitmask of conditions to check, they are pretty self explanatory

```python
ONLY_CODE = 0x01
NOT_SCRATCH = 0x02
LINTING_ENABLED = 0x04
```

Finally the `code` parameter is the language that we want to make the checks for, `python` by default.

## check_linting_behaviour

This function is only useful to check if the configured linting behaviour is listed into the given behaviours list.

Signature:
```python
check_linting_behaviour(view, behaviours)
```

## create_subprocess

The `create_subprocess` function is a convenience method to run a subprocess in whatever platform in an easy way, it is used to start the JsonServer using the configured `python_interpreter`.

Signature:
```python
create_subprocess(args, **kwargs)
```

The `args` is just a list of arguments to pass to the `subprocess.Popen`. The `kwargs` is a dictionary containing useful data like the current directory or the startupinfo in Windows platform, if none is passed, `create_subprocess` uses defaults.

The function returns a new instance of the `subprocess.Popen` class or give you an error in case that the process can't be spawned.

## get_settings

The `get_settings` function, looks for a given key into the active project configuration (if any) and if the key is not found (or there is no project configuration), then it look for it into the global anaconda settings. In case that the key is not found there neither, it can use a default value.

Signature:
```python
get_settings(view, name, default=None)
```

The `view` is the view that we are working on as usual, the `name` is the key that we want to lookup, finally the `default` is the default value that we want to return back in case that the key doesn't exists in any of the configurations.

This function is really useful and is widely used in anaconda.

## active_view

This method is just a shortcut for `sublime.active_window().active_view()`

## prepare_send_data

We already seen this helper in the GUI commands and listeners examples. It prepares a dictionary that has to be send as JSON trough the JsonClient asynchronous socket to the JsonServer.

Signature:
```python
prepare_send_data(location, method, handler)
```

The `location` parameter is our good friend the location of the cursor in the current buffer, the `method` is just the remote method that we want to call and finally the handler is the remote handler where the method is defined.

## project_name

The `project_name` is a convenience function that generates a valid project name for the active window, this name is used to store the Jedi cache and anaconda logs in the local filesystem.

Signature:
```python
project_name()
```

If we don't have a project file we just use the first folder name in the window's folders as name, if we don't have any folders in the window we just use the `window.window_id`

## get_traceback

This just return back a string representation of the last traceback.

## get_view

The `get_view` looks for a given view id in the given window opened buffers, if nothing is found, `None` is returned back.

Signature:
```python
get_view(window, vid)
```

The `window` is (obviously) the window to look from and the `vid` is the view id.

## get_window_view

The `get_window_view` looks for the given view id in all the opened windows

Signature:
```python
get_window_view(vid)
```

Where `vid` is the view id to look for.

## valid_languages

The is a cached function that return a list of valid languages for anaconda plugins.
