Here's a general rebase flow.

You have a branch `main`, and a long-lived feature branch `next`.

First things first, you will want to spend time squashing commits in both branches, making each commit atomic, meaningful, specific, etc.

I'm not sure commits can be too small, but imo, the smaller the better.
For example:
```
message: Use @typescript-eslint parser
.eslintrc.js:
- parser: '@babel/eslint-parser',
+ parser: '@typescript-eslint',
```

Keeping a commit log clean is best done _as you are completing work_. If you currently don't have a clean commit log, then you are well behind the 8 ball.

Here's how to keep a commit log clean as you are developing:
1. Commit frequently, with well-labeled commit message.
 - Given your commit message and original diff, any developer should be able to:
 - A: Easily understand the change.
 - B: See the commit message accurately and completely describes the change.
2. When you find a bug with a previous commit:
 - A: Using gitlens in vscode (or git blame in general) ctrl+c copy the commit message for the line your screwed up
 - A.1: In vscode side by side red/green diff view, you can pull this from the red/left side.
 - B: Fix the lines of code that need fixing, stage _just_ those lines.
 - C: Commit as `squash! <ctrl+v paste commit message>`
 - Run `git rebase -i <hash from before commit message you are modifying> --autosquash` [There's also a .gitconfig setting to _always_ autosquash]
3. Frequently review your commits:
 - A: Accurate commit messages, that completely describe a change
 - B: Use interactive rebase to `edit` a commit, and break it into smaller commits when necessary.
 - C: Any given commit message should typically be less than a half page of text, ideally you don't even need an extended message body.
 - D: If a commit has "extra" changes unrelated to the commit message, it's of course cleaner to break those into their own (possibly one-line) commit

Rebase a long-lived feature branch onto latest master:
1. It is best to do this frequently.
2. Once you are totally confident in your ability to resolve rebase conflicts deterministically, without error, you should enable `rerere` (search my personal .gitconfig, in this repo)

Ok, how to actually rebase:
```
git fetch origin master
git rebase\
  --interactive\
  --onto origin/master 8c0065007 # old master commit, you'll discard everything older than this
  --reapply-cherry-picks --reschedule-failed-exec\
  --empty=drop
```
Ok, now you'll edit the `git-rebase-todo` file.
I recommend starting with git-lens's rebase editor, but you may want to ctrl+f search stuff, and I'm not sure if eamodio has fix the ctrl+f behavior yet.

Example:
```
pick 120498r Foo bar, its fixed now
pick 0y841rf Baz baz baz, bad comit msg
pick 0y83r2f Run `yarn add xstate-devtools --dev`
```
The `Run` commit should be modified like so:
```
pick 120498r Foo bar, its fixed now
pick 0y841rf Baz baz baz, bad comit msg
drop 0y83r2f Run `yarn add xstate-devtools --dev`
exec yarn add xstate-devtools --dev && git commit -am 'Run `yarn add xstate-devtools --dev`' --no-verify
```

This way you avoid some conflicts on `yarn.lock`.

You can't directly reword commits in this file, despite the fact that you can... edit this file... which contains your commit messages.

You CAN do this:
1. `reword` a commit, AND edit the commit message you see in `git-rebase-todo`
2. When you get tossed into an editor to reword that given commit message, if you scan through the comments git includes in that COMMIT_MSG file, you'll see something like:
```
Command we are completing now:
reword 1342rd Newly worded commit message
```
And can copy-paste your new commit message from there.


Keeping a clean commit history is really 65% of the work involved with keeping a long-lived feature branch sane.
And BOTH master and next need clean commit histories.

Anyway, resolving conflicts!
Let's imagine you see this in your editor:
```
<<<< HEAD(?)
let foo = 'bar'
||||| (parent of your next commit)
let Foo = 'bar'
======
let Foo = 'bar + bar'
>>>> (your next commit)
```
I think this 3-hunk conflict format requires activating a certain git setting. See my .gitconfig file.

Compare the 2nd and 3rd conflict hunks. Obviously, we added `+ bar` to the string.

How do we resolve?

Well, upstream master/main, has decided we don't want to capitalize `Foo`.

So the _CORRECT_ resolution here...
What do you think it would be? Do we capitalize `foo` or not?

Quote "(your next commit)" is not concerned with the capitalization of `foo`. It simply wants to add `+ bar`.
So all you need to do, is add `+ bar` in the new context. The new context is the first conflict hunk (code in master).

A trick, is to click "+" in vscode, (despite existing merge conflicts), and then unstage the changes, and view the added code.
You can see the code in master (first hunk) is unchanged.
You can take the middle hunk, cut it, paste it onto the first hunk, and see how the two bases have diverged, but you don't actually want those changes.

Sorry if it got a bit dry there.

An alternative approach, is to run `git rebase --show-current-patch`, read the commit's message, see the original diff, and basically re-apply those changes in the new context.

DO NOT simply "make the code look right"

The time travel gods are commanding you to not mess up the flow of history:
You should ONLY do what the commit message says!
This is a rebase conflict, i.e. telling git how to apply a specific commit.

Enough for now :) 
