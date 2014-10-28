# Submiting Code

Code submition in anaconda is done trough the [GitHub Pull Request System](https://help.github.com/articles/using-pull-requests)
so you should be familiarized with it before trying to contribute your code into anconda.

## Contributing with a bug fix

You found a bug and you are planning to fix it and share the fix with the rest of the community?.
First of all, thank you very much, contributions like that make anaconda better and maintains it alive.

Before start working in yor fix, you have to go to the [anaconda issues](https://github.com/DanWidget/anaconda/issues)
and look for the bug that you've found because maybe some other person is working on fix it already.

If you don't find any open issue related to the bug that you found, make sure that the bug is not
already fixed in a more up-to-date version of anaconda (or into a development branch). You can do
that just checking the code in the last commit of the GitHub repo or just looking for a closed
issue related with the bug.

Another useful place were to find information related with the bug is in the `releases messages`
where the developers add information about bug fixes and other changes for the incoming new release
version.

Of course your bug fix **must** follow the [anaconda coding style guide]() and [Unit Testing]()
guidelines.

### My bug is listed as a closed issue

That means that the bug that you found is **already fixed** by another developer and is added already
to a more up-to-date version of anaconda or is fixed in the in-development version or in a
fix branch and it's waiting for the release of the new version of the plugin.

#### What can I do then?

You can pull the in-development version of the plugin (that is always the last commit in the
master branch) or the bugfix branch where the bug was fixed on and test if your bug is gone. If is
not, just re-open the issue and **explain what the problem is and why the fix doesn't work**. You may
also want to work with the person that wrote the first fix.

### My bug is listed as an open issue

Then this is a known issue, just read the issue discussion thread to discover if someone else is already working on a fix for that, if someone is already working to fix this problem, you can just contact her to work together in solving the issue.

If no other person is working on fix the issue, just write a new comment to the issue discussion
thread informing that you are going to work on solve the issue actively and ask other developers
about guidance or ideas.

### My bug is not listed anywhere

Then you found an unknown anaconda issue. Create a new issue in the GitHub repo and tag it a a **bug**,
explain that you are starting to work on fix it and ask for any help or guidance if needed.

Depending on the importance of the bug that you found, [@damnwidget](https://github.com/DamnWidget) may like to assist you or even
solve himself the issue as fast as possible with your help.

## Contributing with a new feature

You added a killer feature to your own anaconda fork and you decided to share it with the community?
That's really great!! Thank you.

New features **may or may not** been always welcome, that depends of the nature of the new feature
and the impact in the plugin and how developers use it. If you are planning to develop and share a
new feature for anaconda, or you already developed it and what you want is just include it as part
of the project, create a new issue in the GitHub project and tag it as **enhancement** or just send
a new **pull request** explaining about your new feature and why it should be added into the framework.

**note**: Remember that all the code that you submit to the project **must** include unit tests.

## Licensing the code

All the code that get into anaconda plugin **must** be licensed under the **GPLv3** (or a later
version on your choice) license.
