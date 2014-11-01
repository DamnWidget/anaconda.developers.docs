# Integrate a Third Party Tool

Integrate a third party into anaconda is not more difficult than integrate it into any other plugin or application, is just decoupled.

## Integrating the fake "MegaPyLinter" tool

We are going to walk trough the process of integrate fake python linter into the plugin just for illustrative purposes, this tool is called `MegaPyLinter` as is developed completely in Python so even if it's a tool, we can use it from anaconda as a library with a feww modifications.

<blockquote>**note**: you should use third party tools as libraries always that is possible to do it, even if they are not designed in that way and it doesn't require to modify the third party tool itself</blockquote>

### First step, integrate the code into `anconda_lib`

First of all we need to integrate the code into the `anaconda_lib` package. We can just download a tarball file of the third party tool or make a git (or whatever code version system it uses) clone. Then we just have to clean up it to make sure that we don't push unnecessary files into the anaconda repository (like Makefiles, setup.py.
