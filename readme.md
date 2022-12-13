This [repository](https://github.com/MrDesjardins/demo_submodule_parent) uses a submodule Git [repository](https://github.com/MrDesjardins/demo_submodule_child).

# Steps to Create the SubModule Relationship

The first step is to add the submodule to this project.

```sh
git submodule add https://github.com/MrDesjardins/demo_submodule_child ./packages/demo_submodule_child
```

The second step is to modify the `tsconfig.json` to have an alias. It is not _required_ but make the code from the `src` folder of the parent repository to access any submodule package easily.

```json
"paths": {
  "@demo_submodule_child": ["packages/demo_submodule_child/src"],
  "@demo_submodule_child/*": ["packages/demo_submodule_child/src/*"]
}
```

The third step is to add two registries to Node. The following command compiles Typescript into JavaScript and then it calls Node with the two registrys following with the JavaScript entry file.

```json
"start": "npx tsc && node -r ts-node/register/transpile-only -r tsconfig-paths/register build/src/index.js"
```

# How to Modify the SubModule Code

There are several patterns. You can have in a complete other folder the repository and do a change, then push the change. Then come back to the consumer folder and pull the changes. However, that is a lot of steps.

A more straightforward pattern is to modify the Submodule directly into the consuming (parent) project. You can push changes from that SubModule and the changes will then be available to every one using the child repository.


# Good to Know

## GitModule File
Within your parent repository, when the `git submodule` was performed, it created an hidden file called `.gitmodules`. In our example, it contains:

```
[submodule "packages/demo_submodule_child"]
	path = packages/demo_submodule_child
	url = https://github.com/MrDesjardins/demo_submodule_child
```

## Pushing Changes From SubModule

When performing changes into a SubModule folder (our `package/demo_submodule_child`) you will notice that the `git status` at the root of the parent repository shows the changes **with** your repository changes. For example, here I modified the `readme.md` from the parent and changed the content of the `index.ts` of the child:

```sh
❯ git status
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
  (commit or discard the untracked or modified content in submodules)
        modified:   packages/demo_submodule_child (modified content)
        modified:   readme.md

no changes added to commit (use "git add" and/or "git commit -a")

```

Performing at the root of the parent project `git add .` **only** adds the file from the parent. Performing `git status` after the add result to:

```sh
❯ git status
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   readme.md

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
  (commit or discard the untracked or modified content in submodules)
        modified:   packages/demo_submodule_child (modified content)
```

To commit and push the change for the Submodule, you need to go to the Submodule folder and perform the git commands desired.

```sh
cd packages/demo_submodule_child 
git status
```
As the output:

```sh
 /mnt/c/code/demo_submodule_parent  master ⇡1 +1 ─────────────────────────────────────────────────── base  
❯ cd packages/demo_submodule_child 

/mnt/c/code/demo_submodule_parent/packages/demo_submodule_child  master !1 ───────────────────────── base   
❯ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   src/index.ts

no changes added to commit (use "git add" and/or "git commit -a")
```
You can see that when we performed `cd` that the prompt changed with awareness that we were _leaving_ the parent git repository. Here you can `git add .` and `git push origin master` to have the change available to everyone.


# Version
Each commit in the submodule is tracked with the parent. Because the commit version is part of the submodule system, it allows a parent to refer to a previous version. Hence, there is no obligation to be on the latest version of the submodule.

For example, if there is two commits on the submodule and the parent is still on the first one. A `git log` in the submodule folder gives:

```
commit d9024dbcc10578f18abb1c29fa08f43929f80f6c (HEAD -> master, origin/master, origin/HEAD)

    Small change from the parent repository to the submodule

commit 795569b2fa0111b1db23424722793795a6f9ec4a

    First commit
```

Moving to the root of the project (parent) we notice that because the change was performed within the parent project that the parent knows that he is pointing to a new version (commit). A diff shows:

```diff
diff --git a/packages/demo_submodule_child b/packages/demo_submodule_child
index 795569b..d9024db 160000
--- a/packages/demo_submodule_child
+++ b/packages/demo_submodule_child
@@ -1 +1 @@
-Subproject commit 795569b2fa0111b1db23424722793795a6f9ec4a
+Subproject commit d9024dbcc10578f18abb1c29fa08f43929f80f6c
```

However, any other project that use the submodule is still refering to `795569b2fa0111b1db23424722793795a6f9ec4a` which allows to have your project free to update at any time.

# Advantages and Inconvenients

The main advantages are how easy is to modify the code without having to rely on `NPM`. Working on another submodule is like working with code that is local to your project. A big advantage is that if you rely on VsCode to debug your code that it will work flawlessly becaue for VsCode debugger that code is part of your project. Another advantage is that it uses the dependencies of your project. While it is a good practice to being able to compile the submodule independently and having a set of package file, it is easier to handle versionning if the submodule is used between projects you own. There is no need to rely on `npm link` which can be brittle and require compiling between changes with a lot of requirement like the same NPM version.

The main disadvantanges are that it works well at some scale. I would not recommend to develop a huge system with a lot of dependencies. However, if you have a couple of Node projects that need shared files it is easy to setup and does not require you to have a private NPM repository. Hence, this solution works well on a small scale with private intention.

Relying on NPM to have a well tested, isolated and atomic library is ideal but requires to have a bundler, a private NPM repository and tools that facilitate code modification and debugging that the Git submodule solution does not need.