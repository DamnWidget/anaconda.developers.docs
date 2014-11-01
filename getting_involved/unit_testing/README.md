# Unit Testing

Anaconda has two independent but related parts that work together to offer the user the best experience
possible

1. The Anaconda Sublime Text  plugin that interacts with the Sublime Text GUI and with the user
2. The Anaconda's JsonServer that executes the linting, completions, checks and analysis of the code (this maybe a separate and standalone project in the future as it would be used in other editors not  into Sublime Text 3 only)

Unit test a Sublime Text plugin is not easy as it has to run into the embedded Sublime Text python
interpreter as the `sublime` and `sublime_plugin` modules will not work otherwise. There are a few
of projects that aims to help the plugin developers to unit test their projects in Sublime Text, we
think only one of them offer real capabilities to do so and it's this one that we use (and we also
contribute with).

This one is the Randy Lai's [UnitTesting](https://github.com/randy3k/UnitTesting) Sublime Text
plugin.

## First Step

The first step to can run the anaconda's test suite is to install the plugin `UnitTesting` in your
Sublime Text 3 using the `Command Palette`

## How to run tests

Anaconda has two different test suites that need to be run by separate. The first one is the
part of anaconda that is related to the Sublime Text 3 runtime environment, the second one is
related with the anaconda's JsonServer component.

### Run the GUI related tests

The GUI related tests suite must be run using the UnitTesting `Command Palette` action
`UnitTesting: Run current project tests suite`.

This will try to run the Sublime Text 3 GUI related tests and give you the result embedded
into a Sublime Text bottom panel.

### Run the JsonServer related tests

The JsonServer related tests can be run using the anaconda's tests runner directly so you just
have to execute the `Command Palette` action `Anaconda: Run Project Tests`.

**note**: we are assuming that you are using the [anaconda's development project configuration](/using_github/README.html#anacondas-development-project-configuration) already, otherwise the previous related step will not work as anaconda try to use `nosetests` by default.

## Adding new tests

You never add a new module, feature or bug fix to anaconda without adding a batch of unit tests for it.
You never commit any code **until you make sure** that all the tests pass, so that means all the
tests should be Ok when you run the tests suites in the project.

If you commit code that breaks the unit test suite you can be *hunted down* by other developers
so make sure your code follow the guidelines and pass the tests.

You have to add unit tests for your code because in this way other developers can check if their
own changes to other parts of the code affects in any way your contribution just running the
tests suite.

Tests go into dedicated test packages, normally `anaconda_server/test/` and `gui_test` directories
(relative to the root path). Tests should be named as `test_module.py`. If the module is part of
a core library like `anaconda_lib.helpers` you can add that test directly into `plugin_test/test_helpers.py` file.


