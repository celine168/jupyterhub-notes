# Git
## Problem
I did `git reset --hard -1` in an attempt to commit a change I made (git
didn't think I made any changes even though I staged them).

So I lost my Dockerfile.

## Solution
[Wonderful git documentation](https://git-scm.com/book/en/v2/Git-Internals-Maintenance-and-Data-Recovery)

`git fsck --full` gives all objects that have nothing pointed to it.

```
$ git fsck --full
Checking object directories: 100% (256/256), done.
dangling tree ec953c8b7bcdf0a022e0611c5f1dbc4fd7b59336

$ git cat-file -p ec953c8b7bcdf0a022e0611c5f1dbc4fd7b59336
100644 blob 04fd8d0f65744752d362f0dfb8d424a0ceb4b5b7    Dockerfile
100644 blob b90601e7c9075a3e76f1c8a3acaeab8e581d1751    README.md
100644 blob 3380dc6e291a03d10e730f0eb7cc8003263e7027    environment.yml

$ git cat-file -p  04fd8d0f65744752d362f0dfb8d424a0ceb4b5b7
```
The last command outputted the contents of my Dockerfile.

