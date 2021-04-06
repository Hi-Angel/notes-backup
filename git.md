# Misc

* `HEAD~n` syntax: works reliably only in absence of merge commits. If those are present, then it only counts the merge commit, entirely ignoring the commits that were brought by it.
* resetting bisect without causing it to checkout somewhere: `git bisect reset HEAD`
* bisecting specific subset of commits: e.g. you might know your problem is likely in `FOO` subsystem of the project. Bisection can be limited to just it by starting bisect explicitly with a list of paths, like `git bisect start -- path1/FOO path2`. Known problem: it isn't able to dive into between commits modifying those paths if turned out to be the problem is not in one of them. So you'll have to reset bisect at this point, and restart it again by using last known good/bad commits pair.
