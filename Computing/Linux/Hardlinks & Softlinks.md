Just a few notes on the differences between hardlinks and softlinks (i.e. **symbolic links**, or **symlinks**).

To create a hardlink:

```Bash
ln target newlink
```

To create a softlink:

```Bash
ln -s target newlink
```

[This](https://stackoverflow.com/questions/185899/what-is-the-difference-between-a-symbolic-link-and-a-hard-link) Stack Overflow page contains a useful description of the difference between the two. The key idea is that a hardlink like `ln target newlink`  creates a file called `target` that points to the same inode that `target` points to.
Creating a symlink `ln -s target newlink` creates a file called `newlink` that points to the **file** called `target`.

This diagram illustrates it well:

![|500](_attachments/Screenshot%202023-01-10%20at%2009.53.38.png)

This gives hardlinks & softlinks a few differences in practice:

* Deleting, renaming or moving the original file won't affect a hardlink. But it will break a symlink.
* Hardlinks can't cross file systems. Hence hardlinks can only be made on the same device.

### Hardlinks vs Copying
What's the difference between creating a hard link, and copying a file?
When you copy a file, you are creating a new file, **inode** and **new data blocks on disk**. The new file will have its own metadata, permissions, timestamps, etc. It's a completely different file, with its own identity.

We can see this illustrated as follows:

```Shell
touch file.txt
ln file.txt file_hardlink.txt
cp file.txt file_copy.txt
ls -i
> 14028218584894682500 file_copy.txt    182693612220106237 file_hardlink.txt    182693612220106237 file.txt
```

The original file and its hardlink share an inode. Whereas copied file has a new inode.

> [!TIP]
> `ls -i` lists all files in the current directory, along with their inode numbers.





