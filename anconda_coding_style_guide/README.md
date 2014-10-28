# Anconda Coding Style Guide

Like other Python projects, anaconda follows the [PEP-8](http://www.python.org/dev/peps/pep-0008/) and
[Sphinx](http://sphinx-doc.org/) as docstrings conventions. Make sure to read those documents if you
intent to contribute to anaconda.

## Naming Conventions

* Package and module names **must** be all lower-case, words may be separated with underscores. No exceptions.
* Class names are CamelCase e.g. `AnacondaGetDocumentation`
* The function, variables and class member names **must** be all lower-case, with words separated by underscores. No exceptions.
* Internal (private) methods and members are prefixed with a single underscore e.g. `_execute`

## Style rules

* **Lines shouldn't exceed 79 characters length**
* Tabs **must** be replaced by four blank spaced, **never merge** tabs and blank spaces
* **Never** use multiple statements in the same line, e.g. `if check is True: a = 0` with the only exception for [conditional expressions](http://docs.python.org/3/reference/expressions.html#conditional-expressions)
* Complehensions are preferred to the built-in functions `filter` and `map` when appropiate
* Only use a `global` declaration in a function if that function actually modifies the global variable
* Use the `format` method to format strings instead of the *semi-deprecated* old `'%s' % (data,)` way
* Use single quotes `'` to enclose your strings instead of the double quotes `"`
* **Never** use a print statement, we always use a `print` function instead (even in the JsonServer) as we support Python 3
* You **never** use print for logging purposes. Use the `logging` module instead.

## Principles

1. Never violate [DRY](http://programmer.97things.oreilly.com/wiki/index.php/Don%27t_Repeat_Yourself)
2. Any code change **must** pass unit tests and shouldn't break any other test in the system
3. No commit should break the master build
4. No change should break user code silently, deprecations and incompatibilities must be always a kown (and well documented) matter.
