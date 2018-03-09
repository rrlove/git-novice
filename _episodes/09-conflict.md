---
title: Conflicts
teaching: 15
exercises: 15
questions:
- "What do I do when my changes conflict with someone else's?"
objectives:
- "Explain what conflicts are and when they can occur."
- "Resolve conflicts resulting from a merge."
keypoints:
- "Conflicts occur when two or more people change the same file(s) at the same time."
- "The version control system does not allow people to overwrite each other's changes blindly, but highlights conflicts so that they can be resolved."
---

As soon as people can work in parallel, it's likely someone's going to step on someone
else's toes.  This will even happen with a single person: if we are working on
a piece of software on both our laptop and a server in the lab, we could make
different changes to each copy.  Version control helps us manage these
[conflicts]({{ page.root }}/reference/#conflicts) by giving us tools to
[resolve]({{ page.root }}/reference/#resolve) overlapping changes.

To see how we can resolve conflicts, we must first create one.  The file
`ingredients.txt` currently looks like this in both partners' copies of our `ingredients.txt`
repository:

~~~
$ cat ingredients.txt
~~~
{: .bash}

~~~
4 avocados
salt
1/2 onion
cilantro
pepper
1/2 tomato
~~~
{: .output}

Let's add a new ingredient to one partner's copy only:

~~~
$ nano ingredients.txt
$ cat ingredients.txt
~~~
{: .bash}

~~~
4 avocados
salt
1/2 onion
cilantro
pepper
1/2 tomato
peas
~~~
{: .output}

and then push the change to GitHub:

~~~
$ git add ingredients.txt
$ git commit -m "The New York Times said so"
~~~
{: .bash}

~~~
[master e3814e4] The New York Times said so
 1 file changed, 1 insertion(+)
~~~
{: .output}

~~~
$ git push
~~~
{: .bash}

~~~
Counting objects: 5, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 342 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To https://github.com/guacmaster/guac.git
   7fb6b7e..e3814e4  master -> master
~~~
{: .output}

Now let's have the other partner
make a different change to their copy
*without* updating from GitHub:

~~~
$ nano ingredients.txt
$ cat ingredients.txt
~~~
{: .bash}

~~~
4 avocados
salt
1/2 onion
cilantro
pepper
1/2 tomato
lime juice
~~~
{: .output}

We can commit the change locally:

~~~
$ git add ingredients.txt
$ git commit -m "Add citrus"
~~~
{: .bash}

~~~
[master 07ebc69] Add citrus
 1 file changed, 5 insertion(+)
~~~
{: .output}

but Git won't let us push it to GitHub:

~~~
$ git push
~~~
{: .bash}

~~~
To https://github.com/guacmaster/guac.git
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'https://github.com/guacmaster/guac.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
~~~
{: .output}

![The Conflicting Changes](../fig/conflict.svg)

Git detects that the changes made in one copy overlap with those made in the other
and stops us from trampling on our previous work.
What we have to do is pull the changes from GitHub,
[merge]({{ page.root }}/reference/#merge) them into the copy we're currently working in,
and then push that.
Let's start by pulling:

~~~
$ git pull
~~~
{: .bash}

~~~
remote: Counting objects: 3, done.
remote: Compressing objects: 100% (1/1), done.
remote: Total 3 (delta 2), reused 3 (delta 2), pack-reused 0
Unpacking objects: 100% (3/3), done.
From https://github.com/guacmaster/guac
   12d2b5e..5fa9f4b  master     -> origin/master
Auto-merging ingredients.txt
CONFLICT (content): Merge conflict in ingredients.txt
Automatic merge failed; fix conflicts and then commit the result.
~~~
{: .output}

`git pull` tells us there's a conflict,
and marks that conflict in the affected file:

~~~
$ cat ingredients.txt
~~~
{: .bash}

~~~
4 avocados
salt
1/2 onion
cilantro
pepper
1/2 tomato
<<<<<<< HEAD
peas
=======
lime juice
>>>>>>> 5fa9f4bbc64cfea4231f462afb1147b8e4703aaa
~~~
{: .output}

Our change (the one in `HEAD`) is preceded by `<<<<<<<`.
Git has then inserted `=======` as a separator between the conflicting changes
and marked the end of the content downloaded from GitHub with `>>>>>>>`.
(The string of letters and digits after that marker
identifies the commit we've just downloaded.)

It is now up to us to edit this file to remove these markers
and reconcile the changes.
We can do anything we want: keep the change made in the local repository, keep
the change made in the remote repository, write something new to replace both,
or get rid of the change entirely.

~~~
$ nano ingredients.txt
$ cat ingredients.txt
~~~
{: .bash}

~~~
4 avocados
salt
1/2 onion
cilantro
pepper
1/2 tomato
lime juice
~~~
{: .output}

To finish merging,
we add `ingredients.txt` to the changes being made by the merge
and then commit:

~~~
$ git add ingredients.txt
$ git status
~~~
{: .bash}

~~~
On branch master
Your branch and 'origin/master' have diverged,
and have 1 and 1 different commits each, respectively.
  (use "git pull" to merge the remote branch into yours)
All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)

Changes to be committed:

	modified:   ingredients.txt
~~~
{: .output}

~~~
$ git commit -m "peas have no place in guacamole"
~~~
{: .bash}

~~~
[master 2abf2b1] Merging changes from GitHub
~~~
{: .output}

Now we can push our changes to GitHub:

~~~
$ git push
~~~
{: .bash}

~~~
Counting objects: 6, done.
Delta compression using up to 24 threads.
Compressing objects: 100% (6/6), done.
Writing objects: 100% (6/6), 591 bytes | 0 bytes/s, done.
Total 6 (delta 4), reused 0 (delta 0)
remote: Resolving deltas: 100% (4/4), completed with 2 local objects.
To https://github.com/guacmaster/guac.git
   5fa9f4b..3fec3a2  master -> master
~~~
{: .output}

Git keeps track of what we've merged with what,
so we don't have to fix things by hand again
when the collaborator who made the first change pulls again:

~~~
$ git pull
~~~
{: .bash}

~~~
remote: Counting objects: 6, done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 6 (delta 4), reused 6 (delta 4), pack-reused 0
Unpacking objects: 100% (6/6), done.
From https://github.com/guacmaster/conversions
   5fa9f4b..3fec3a2  master     -> origin/master
Updating 5fa9f4b..3fec3a2
Fast-forward
 conversion.py | 6 ++++++
 1 file changed, 6 insertions(+)
~~~
{: .output}

We get the edited file:

~~~
$ cat ingredients
~~~
{: .bash}

~~~
4 avocados
salt
1/2 onion
cilantro
pepper
1/2 tomato
lime juice
~~~
{: .output}

We don't need to merge again because Git knows someone has already done that.

Version control's ability to merge conflicting changes
 is another reason users tend to divide their programs and papers into
 multiple files instead of storing everything in one large file.
 There's another benefit too: whenever there are repeated conflicts
 in a particular file, the version control system is essentially trying
 to tell its users that they ought to clarify who's responsible for
 what, or find a way to divide the work up differently.

> ## Solving Conflicts that You Create
>
> Each partner edit methods.txt to create another conflict. Use the same steps as above to
> resolve the conflict
{: .challenge}

> ## Conflicts on Non-textual files
>
> What does Git do when there is a conflict in an image or some other 
> non-textual file that is stored in version control?
>
> > ## Hint
> >
> > Try it! Commit a small binary image file (jpg or png) and create a
> > conflict
> {: .solution}
>
> > ## Solution
> > 
> > You must choose one version or the other, because it's impractical
> > to edit the actual binary differences
> {: .solution}
{: .challenge}

> ## A Typical Work Session
>
> You sit down at your computer to work on a shared project that is tracked in a
> remote Git repository. During your work session, you take the following
> actions, but not in this order:
>
> - *Make changes* by appending the number `100` to a text file `numbers.txt`
> - *Update remote* repository to match the local repository
> - *Celebrate* your success
> - *Update local* repository to match the remote repository
> - *Stage changes* to be committed
> - *Commit changes* to the local repository
>
> In what order should you perform these actions to minimize the chances of
> conflicts? Put the commands above in order in the *action* column of the table
> below. When you have the order right, see if you can write the corresponding
> commands in the *command* column. A few steps are populated to get you
> started.
>
> |order|action . . . . . . . . . . |command . . . . . . . . . . |
> |-----|---------------------------|----------------------------|
> |1    |                           |                            |
> |2    |                           | `echo 100 >> numbers.txt`  |
> |3    |                           |                            |
> |4    |                           |                            |
> |5    |                           |                            |
> |6    | Celebrate!                | `AFK`                      |
>
> > ## Solution
> > 
> > |order|action . . . . . . . . . . |command . . . . . . . . . . |
> > |-----|---------------------------|----------------------------|
> > |1    | *Update local*            | `git pull`                 |
> > |2    | *Make changes*            | `echo 100 >> numbers.txt`  |
> > |3    | *Stage changes*           | `git add numbers.txt`      |
> > |4    | *Commit changes*          | `git commit -m "add 100"`  |
> > |5    | *Update remote*           | `git push`                 |
> > |6    | *Celebrate!*              | `AFK`                      |
> {: .solution}
{: .challenge}
