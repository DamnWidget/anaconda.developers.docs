# Anaconda guide for contributors

Anaconda uses GitHub as project tracker as well as repository and development/contribution mechanism, make sure that you get familiar with it and the way that anaconda uses GitHub for the development workflow.

# Anaconda's 12 steps contribution workflow

This is a resumed description of the general process for commiting code into the master branch of the anaconda project. There are exceptions to this general workflow like for example, security updates or critical bug fixes, minor changes to documentation or typo fixes and of course critical fixes on broken builds.

## The Development Workflow

All the develoment workflow is performed in the [Anaconda's GitHub](https://github.com/DamnWidget/anaconda) project site.

1. When you are going to start to work in a fix or create a new feature you have to make sure of
    1. Make sure there is already an open issue related with the code that you are going to submit. If that is not the case, create it yourself.
    2. The issue has been discussed and approved by the anaconda's development team
    3. If the fix or feature affects the anaconda's JsonServer or core, make sure [@damnwidget](https://github.com/DamnWidget) is involved in the fix or feature and is going to help you with the integration of your code into anaconda
2. You should work in your own [fork](https://help.github.com/articles/fork-a-repo)
3. You have to do it in a new [branch](http://git-scm.com/book/en/Git-Branching-What-a-Branch-Is)
4. Make sure your fork is [synced](https://help.github.com/articles/syncing-a-fork) with the last in-development version and your branch is [rebased](http://git-scm.com/book/en/Git-Branching-Rebasing/) if needed.
5. Make sure that you add an entry about the new fix/feature that you introduced into anaconda in the [relnotes index](https://github.com/DamnWidget/anaconda/blob/master/templates/relnotes.tpl)
6. Make sure that you add whatever documentation that is needed in order to use your feature (if applicable) forking and requesting a pull in the [anaconda's documentation repository](https://github.com/DamnWidget/anaconda.github.io)
7. Make sure that you reference the issue or issues that your changes affects in your [commit message](https://help.github.com/articles/closing-issues-via-commit-messages) so we can just track the code from the issues panel.
8. When your work is complete you should send a [pull request](https://help.github.com/articles/using-pull-requests) on GitHub.
9. Your pull request will be then reviewed by [@damnwidget](https://github.com/DamnWidget) and other contributors
10. If there is any issue with your code in the review, you should fix them and push your changes back for a new review (make sure you add a new comment into the pull request issue as GitHub doesn't alert anyone about new commits being pushed into a pull request) rebasing and squashing your branch if is needed to maintain a logic log
11. When the review finishes, [@damnwidget](https://github.com/DamnWidget) will merge your contribution into the master branch
12. Goto step 1

# Anaconda's virtual environments

Anaconda is developed to support both Python2 and Python3. The JsonServer is developed in a
way that it support both versions of the interpreter you should have at least two different virtual environments with Python2 and Python3 or test your changes with those interpreters if you have both installed in your machine.

# Anaconda's development project configuration

If you intend to contribute with anaconda you should use the following as your `.sublime-project`
file depending on which platform are you developing on (remember to change the virtual environment
path depending if you are testing python2 or python3).

##### Mac OS X
```python
{
	"build_systems":
	[
		{
			"name": "Anaconda Python Builder",
			"selector": "source.python",
			"shell_cmd": "python -u \"$file\""
		}
	],
	"folders":
	[
		{
			"path": "."
		}
	],
	"settings":
	{
		// Environment
		"python_interpreter": "~/.virtualenvs/anaconda/bin/python",
		"extra_paths": ["/Applications/Sublime Text.app/Contents/MacOS/"],
		"test_command": "python -m unittest discover",
		"test_virtualenv": "~/.virtualenvs/anaconda",
		// Utilities
		"display_signatures": true,
		// JsonServer debug
		"jsonserver_debug": false,
		"jonserver_debug_port": 9999
	}
}

```

##### GNU/Linux

```python
{
	"build_systems":
	[
		{
			"name": "Anaconda Python Builder",
			"selector": "source.python",
			"shell_cmd": "python -u \"$file\""
		}
	],
	"folders":
	[
		{
			"path": "."
		}
	],
	"settings":
	{
		// Environment
		"python_interpreter": "~/.virtualenvs/anaconda/bin/python",
		"extra_paths": ["/opt/sublime_text_3"],
		"test_command": "python -m unittest discover",
		"test_virtualenv": "~/.virtualenvs/anaconda",
		// Utilities
		"display_signatures": true,
		// JsonServer debug
		"jsonserver_debug": false,
		"jonserver_debug_port": 9999
	}
}
```

##### Windows

```python
{
	"build_systems":
	[
		{
			"name": "Anaconda Python Builder",
			"selector": "source.python",
			"shell_cmd": "python -u \"$file\""
		}
	],
	"folders":
	[
		{
			"path": "."
		}
	],
	"settings":
	{
		// Environment
		"python_interpreter": "~/anaconda/bin/python",
		"extra_paths": ["C:\Program Files\Sublime Text 3\"],
		"test_command": "python -m unittest discover",
		"test_virtualenv": "~/anaconda",
		// Utilities
		"display_signatures": true,
		// JsonServer debug
		"jsonserver_debug": false,
		"jonserver_debug_port": 9999
	}
}
```
