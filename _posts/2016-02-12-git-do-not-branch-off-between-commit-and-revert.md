---
layout: post
title: Git - do not branch off between commit and revert
---

Committed to *master* branch by mistake? Do not solve it by branching off a *feature* branch and then reverting that commit on *master*. You will lose changes in that commit - they will not be merged back from the *feature* branch.

Suppose we have done a commit (*2*) by mistake to *master* (it should have been on a *feature* branch to be merged to *master* later):

![Commit to master by mistake]({{ site.baseurl }}/images/git-do-not-branch-off-between-commit-and-revert/commit_branch_revert_a.svg)

We can smartly solve it by branching off *feature* branch now and then create a commit reverting commit *2* on *master*:
{% highlight bash %}
git branch feature
git revert HEAD
{% endhighlight %}

This is how it looks like now:

![Branch and then revert]({{ site.baseurl }}/images/git-do-not-branch-off-between-commit-and-revert/commit_branch_revert_b.svg)

In fact, this is what we want. The branch *feature* contains the changes in commit *2*. The branch *master* does not contain them, because they were reverted by commit *2'*.

Now, let's work on that branches. Create some commit on *master* (*3*) and some other unrelated commit on *feature* (*4*):

![Branch and then revert]({{ site.baseurl }}/images/git-do-not-branch-off-between-commit-and-revert/commit_branch_revert_c.svg)

Later it's time to merge *feature* back to *master*, because we finished the work on *feature*:
{% highlight bash %}
git checkout master
git merge feature
{% endhighlight %}

The result will look like this:

![Branch and then revert]({{ site.baseurl }}/images/git-do-not-branch-off-between-commit-and-revert/commit_branch_revert_d.svg)

This looks fine. However, we do not have the changes made in commit *2*. Git knows that *2* is an ancestor of *master* and thus is already included in *master*. Of course, the change is not included, because we reverted it afterwards. So, we have lost the change. If many other changes have been done on *feature*, we can easily overlook this.

In fact, this happens even if doing rebase instead of merge:

{% highlight bash %}
git checkout feature
git rebase master
{% endhighlight %}

Here is a whole script that shows this situation (note that at the end, the file *f* does not contain the line "*some change*" - added in that reverted commit):

{% highlight bash %}
#!/bin/bash

set -e
set -u

mkdir test
cd test
git init

# A

touch f
git add f
echo "initial" >> f
git commit -am "1_initial"

echo "some change" >> f
git commit -am "2_some change"

# B

git branch feature
git revert HEAD

# C

echo "other change on master" >> f_master
git add f_master
git commit -am "3_other change on master"

git checkout feature

echo "other change on feature" >> f_feature
git add f_feature
git commit -am "4_other change on feature"

# D

git checkout master
git merge feature
#git checkout feature
#git rebase master
{% endhighlight %}


