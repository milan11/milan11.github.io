---
layout: post
title: Git - rebasing a tree
---

In Git, rebasing can be used instead of merging to integrate changes from another branch.

## Simple example of merge and rebase

Here, we integrated branch *other* to *main* by merging it:

![Simple merge]({{ site.baseurl }}/images/git-rebase/merge.svg)

And here by rebasing it:

![Simple rebase]({{ site.baseurl }}/images/git-rebase/rebase.svg)

## Rebasing a tree?

But, what if the branch *other* was something more complicated? Let's observe this:

![Tree rebase]({{ site.baseurl }}/images/git-rebase/rebase_tree.svg)

Here, branch *other* is a tree of branches which have been branched off and later merged. As you can see, the rebasing happened and applied every single commit from that complicated tree, effectively flattening all the history to *main*.

Notice the order of the new (rebased) commits (7' to 10'). They do not follow the order in time of the previous commits (the previous commits are ordered in time sequantially from 5 to 10). Instead, they follow some breadth-first search ordering.

Assume there were some conflicts when we merged *a* to *other* and *b* to *other*. The conflicts needed to be resolved manually - so the merge commits *Ma* and *Mb* include some manually made changes. Note that while rebasing, some conflicts occur again, so we need to manually resolve them again. Since the commits are applied (rebased) one by one, these conflicts do not look the same as the conflicts we had to resolve in *Ma* and *Mb* - in fact, more conflicts can happen here and they can be harder to resolve. So, be cautious about such rebase.  In addition, all changes we made in *Ma* and *Mb* disappeared. This is another reason why we should do only conflict resolving changes (not arbitrary changes) in the merge commits.

You can try it yourself using [this script](https://gist.github.com/milan11/2550b31382acd71674fe#file-git-rebase-sh).

## More complicated tree

What if the branch *other* was branched off from *main* later, thus *main* having some of the commits already included? Here, commit 3 is already included in *main*, but when we follow the path back through e.g. a (commits 8 and 7), it's not so obvious. The next graph differs from the previous only in the marked arrow.

![Other tree rebase]({{ site.baseurl }}/images/git-rebase/rebase_tree_2.svg)

Yes, Git knows about it and does not include these commits again.

You can try it yourself using [this script](https://gist.github.com/milan11/2550b31382acd71674fe#file-git-rebase-2-sh).

In fact, this situation can happen if:

- you pull using rebase (by calling ```pull --rebase``` or setting ```pull.rebase``` to ```true``` in config)
- and you decided to use local branches when doing your changes

Imagine that:

- *main* is *origin/master*
- *other* is *master*
- *a* and *b* are your arbitrary branches

And so it happened (follow the last picture, but with these new branch names):

- *origin/master* and *master* were synchronized at the beginning, at commit 2
- we decided to branch off our work, creating branches *a* and *b*
- we worked on these branches, doing commits 7, 8, 9, 10
- optionally, we pulled master, thus moving both *origin/master* and *master* to commit 3
- optionally, we did some commits to the *master* branch - 5, 6
- we merged *a* and *b* to *master*, creating *Ma* and *Mb*
- we pulled *origin/master*, which:
(1) fetches it, so it is now at commit 4
(2) rebases *master* to *origin/master*, so commits 7' to 10' are created
- only for completeness: next push will move *origin/master* to *master* - commit 10'

Maybe this happened (will happen) to you by accident and you will wonder how it's possible that it worked and you ended up with a correct state. I think it's because the rebase action can handle such trees correctly.

## Conclusion

I was fascinated how powerful the rebasing actually is. However, I do not state that the resulting flat history (you get by using rebasing instead of merging) is always an advantage.

Moreover, such combining of rebasing and merging as described here can be quite dangerous, because you have to do some merges again (while the rebasing happens) which increases the risk of doing them wrong and getting wrong resulting state.
