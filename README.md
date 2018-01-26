# git-objects-sync

git push based on objects, instead of commits.

## Issue

`git push` updates remote references like branches and builds a pack file based on missing commits on the remote side.
If you have several files identical in two completel separate branches (no shared commit history), git will upload objects twice.

See [Git push new branch with same files, uploads all files again](https://stackoverflow.com/questions/48228425/git-push-new-branch-with-same-files-uploads-all-files-again) at Stackoverflow for more information.

## Solution

This little Python script simply syncs missing objects - not based on commits.

What it does concretely:

1. Builds a list of objects (blobs, tree, commit) that are related to one commit (only one, latest, commit support for the moment)
2. Sends the list of object SHAs to the server using the command `git-cat-file-check.py $gitDir` via SSH. That command returns object SHAs that do not exist on the remote.
3. We pack all objects via `git pack-objects` to stdout and send it directly via SSH `git-unpack-objects.py $gitDir`. It simply unpacks the objects, so it is immediately available on the remote.
4. You can optional update remote references like branches so it points now to the uploaded commit.

### Scripts needed on the server

`git-cat-file-check.sh` is a simple script available in the $PATH of the server to build a list of missing object SHAs,
receives a list of object SHA per line.

```
#!/bin/sh
# git-cat-file-check.sh

git --git-dir=$1 cat-file --batch-check <&0 | grep ' missing' | cut -d' ' -f 1
```


`git-unpack-objects.sh` is a simple script available in the $PATH of the server to receive a pack and extracts it 
in the target Git repository.

```
#!/bin/sh
# git-unpack-objects.sh
git --git-dir=$1 unpack-objects <&0
```

