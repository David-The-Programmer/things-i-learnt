# Transferring files from SSH client to SSH host and vice-versa

Let's say you read [Setting Up SSH](./setting-up-ssh.md) and now have SSH setup. Next thing that might be useful to know is how to transfer files between your local computer (SSH client) and the remote computer (SSH server/host).

There are 2 popular tools used to do so, namely `scp` and `rsync`. Secure Copy or `scp` is basically like the `cp` command, but to copy files from local to remote and vice-versa over a SSH connection.
Remote Sync or `rsync` synchronises the deltas or differences in contents of folders between local and remote.

## How to use `scp` to transfer files from SSH client to SSH host

## Copying files

Copying files using `scp` follows this format, just like `cp`:

```
scp <path/of/file/in/source/folder> <path/of/destination/folder>
```

Copying files using `scp` from SSH client to SSH host takes the format shown below.

```
scp <path/to/file/in/client> <user>@<host>:<path/to/host/folder>
```

E.g, copying foo.txt from SSH client to SSH host
```
scp ~/Desktop/foo.txt host@192.0.0.1:~Downloads/
```

Similarly, copying from SSH host to SSH client is the same, it just involves switching the paths.
```
scp <user>@<host>:<path/to/file/in/host> <path/to/client/folder>
```

Copying foo.txt from SSH host to SSH client

```
scp host@192.0.0.1:~Downloads/bar.txt ~/Desktop/
```

If you want the file copied to be renamed something else at the destination folder, specify the full path of the copied file instead of the destination folder. An example would be:

```
scp ~/Desktop/foo.txt host@192.0.0.1:~Downloads/bar.txt
```
Thus, `foo.txt` would be copied from the client and renamed as `bar.txt` in the host.

### Copying folders

If you want to copy all the contents of a folder, including the folder itself, use the `-r` flag. For example:

```
scp -r ~/Desktop/foo/ host@192.0.0.1:~Downloads/
```

If you want to copy all the contents of a folder, but without the folder itself, ensure to put the trailing slash and asterisk `/*` to the path of the source folder.

```
scp -r ~/Desktop/foo/* host@192.0.0.1:~Downloads/
```
Thus, if folder `foo` contained files `bar.txt` and `baz.txt`, then the path of the copied files would be `~/Downloads/bar.txt` and `~/Downloads/baz.txt` respectively in the SSH host.

### Copying files using `rsync`

Copying files using `rsync` is very similar to `scp`. **Note: ensure that `rsync` is installed on both the SSH client and SSH host**.

```
rsync -avzP <path/to/file/from/source> <user>@<host>:<path/to/destination/folder/>
```

- The flag `-a` means `--archive`. 
    - This flag tells `rsync` to preserve most non regular files such as symlinks. It also enables recursive copying.
- The flag `-v` means `--verbose`. 
    - This flag tells `rsync` to show whatever files that are being transferred and a brief summary of that at the end of the transfer.
- The flag `-z` means `--compress`
    - This flags tells `rsync` to compress the file data as it is being sent to the destination folder. This reduces the amount of data transmitted during the transfer, helping to reduce transfer time when doing so over a slow connection.
- The flag `-P` means `--partial --progress`. 
    - The `--partial` flag tells `rsync` to keep partially transferred files during the transfer, to allow resumption of transfer should it be disrupted by a network fault.
    - The `--progress` flag tells `rsync` to show the progression of the transfers taking place, basically a progress bar.

### Copying folders using `rsync`

Copying folders with `rsync` is similar to `scp`, but with one key difference. When copying the contents of a folder without the folder itself, only a slash `/` is needed to be put at the end of the source folder path, unlike `scp` which requires `/*`.

**NEED TO FIGURE OUT IF THE TOOL OVERWRITES EXISTING FOLDERS**

## Pros and Cons of each tool
Pros of `scp`
- Uses secure SSH connection to copy files

Cons of `scp`
- No way of resuming copying of files if any disruption occurs
- Copies files wholesale from local to remote and vice versa

Pros of `rsync`
- Able to resume copying of files if any disruption occurs
- Syncs deltas of files (differences in the binaries of the files), allowing for more efficient synchronisation of large size files compared to `scp`
- Able to be used for local only file transfers as well, thus being an enhanced `cp`

Cons of `rsync`
- By default overrides entire contents of folder that you are copying to

## Useful Links

https://fedoramagazine.org/scp-users-migration-guide-to-rsync/

https://medium.com/axcess-io/understanding-rsync-and-scp-46bcb20791d0

https://unix.stackexchange.com/questions/39718/is-there-ever-a-reason-to-use-scp-instead-of-rsync/303084#303084

https://www.freecodecamp.org/news/rsync-examples-rsync-options-and-how-to-copy-files-over-ssh/
