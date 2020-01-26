# File Systems: Appendix

### Creating Files

This can be accomplished with the open system call; by calling open\(\) and passing it the O CREAT ﬂag, a program can create a new ﬁle.

The example below creates a ﬁle called “foo” in the current working directory

```text
int fd = open("foo", O_CREAT|O_WRONLY|O_TRUNC, S_IRUSR|S_IWUSR);
```

The routine open\(\) takes a number of different ﬂags:

* the second parameter creates the ﬁle \(O CREAT\) if it does not exist, ensures that the ﬁle can only be written to \(O WRONLY\), and, if the ﬁle already exists, truncates it to a size of zero bytes thus removing any existing content \(O TRUNC\)
* the third parameter speciﬁes permissions, in this case making the ﬁle readable and writable by the owner

open\(\) returns a **ﬁle descriptor**, an integer, private per process, and is used in UNIX systems to access ﬁles; thus, once a ﬁle is opened, you use the ﬁle descriptor to read or write the ﬁle, assuming you have permission to do so.

Another way to think of a ﬁle descriptor is as a **pointer to an object of type ﬁle**; once you have such an object, you can call other “methods” to access the ﬁle, like read\(\) and write\(\)

### Reading and Writing Files

![](.gitbook/assets/image%20%2823%29.png)

```text
$ echo hello > foo
$ cat foo
$ strace cat foo 
... 
open("foo", O_RDONLY|O_LARGEFILE) = 3 
read(3, "hello\n", 4096) = 6 
write(1, "hello\n", 6) = 6 
hello read(3, "", 4096) = 0 
close(3) = 0 
... 
```

Why does the first call to open\(\) returns 3?

Each running process already has three ﬁles open, standard input \(which the process can read to receive input\), standard output \(which the process can write to in order to dump information to the screen\), and standard error \(which the process can write error messages to\). These are represented by ﬁle descriptors 0, 1, and 2, respectively. Thus, when you ﬁrst open another ﬁle \(as cat does above\), it will almost certainly be ﬁle descriptor 3.

### Making Directories

Beyond ﬁles, a set of directory-related system calls enable you to make, read, and delete directories. Note you can never write to a directory directly. Because the format of the directory is considered ﬁle system metadata, the ﬁle system considers itself responsible for the integrity of directory data; thus, you can only update a directory indirectly by, for example, creating ﬁles, directories, or other object types within it. In this way, the ﬁle system makes sure that directory contents are as expected.

To create a directory, a single system call, mkdir\(\), is available. When such a directory is created, it is considered “empty”, although it does have a bare minimum of contents. Speciﬁcally, an empty directory has two entries: one entry that refers to itself, and one entry that refers to its parent. The former is referred to as the “.” \(dot\) directory, and the latter as “..” \(dot-dot\)

### Reading Directories

Below is an example program that prints the contents of a directory. The program uses three calls, opendir\(\), readdir\(\), and closedir\(\), to get the job done.

```text
int main(int argc, char *argv[]) { 
    DIR *dp = opendir("."); 
    assert(dp != NULL); 
    struct dirent *d; 
    while ((d = readdir(dp)) != NULL) { 
        printf("%lu %s\n", (unsigned long)d->d_ino, d->d_name); 
    } 
    closedir(dp); 
    return 0; 
}

struct dirent { 
    char d_name[256]; // filename 
    ino_t d_ino; // inode number 
    off_t d_off; // offset to the next dirent 
    unsigned short d_reclen; // length of this record 
    unsigned char d_type; // type of file 
};
```

### Deleting Directories

You can delete a directory with a call to rmdir\(\) \(which is used by the program of the same name, rmdir\). Unlike ﬁle deletion, however, removing directories is more dangerous, as you could potentially delete a large amount of data with a single command. Thus, rmdir\(\) has the requirement that the directory be empty \(i.e., only has “.” and “..” entries\) before it is deleted. If you try to delete a non-empty directory, the call to rmdir\(\) simply will fail.

### Hard Links

A new way to make an entry in the ﬁle system tree is through a system call known as link\(\). The link\(\) system call takes two arguments, an old pathname and a new one; when you “link” a new ﬁle name to an old one, you essentially create another way to refer to the same ﬁle. The command-line program ln is used to do this, as we see in the example below. 

The way link\(\) works is that it simply creates another name in the directory you are creating the link to, and refers it to the same inode number \(i.e., low-level name\) of the original ﬁle. The ﬁle is not copied in any way; rather, you now just have two human-readable names \(file and file2\) that both refer to the same ﬁle.

```text
$ echo hello > file 
$ cat file 
hello 
$ ln file file2 
$ cat file2 
hello
$  ls -i file file2 
67158084 file 
67158084 file2 
```

To remove a ﬁle from the ﬁle system, we call unlink\(\). In the example above, we could for example remove the ﬁle named file, and still access the ﬁle without difﬁculty.

```text
$ rm file 
removed ‘file’
$ cat file2 
hello
```

The reason this works is because when the ﬁle system unlinks ﬁle, it checks a reference count within the inode number. This reference count \(sometimes called the link count\) allows the ﬁle system to track how many different ﬁle names have been linked to this particular inode. When unlink\(\) is called, it removes the “link” between the human-readable name \(the ﬁle that is being deleted\) to the given inode number, and decrements the reference count; only when the reference count reaches zero does the ﬁle system also free the inode and related data blocks, and thus truly “delete” the ﬁle.

Limitations:

* you can’t create one to a directory \(for fear that you will create a cycle in the directory tree\)
* you can’t hard link to ﬁles in other disk partitions \(because inode numbers are only unique within a particular ﬁle system, not across ﬁle systems\)

### Symbolic \(or Soft\) Links

To create a symbolic link, you can use the same program ln, but with the -s ﬂag

```text
$ echo hello > file 
$ ln -s file file2 
$ cat file2 
hello
```

Beyond the surface similarity, symbolic links are actually quite different from hard links. The ﬁrst difference is that a symbolic link is actually a ﬁle itself, of a different type. Besides regular ﬁles and directories; symbolic links are a third type the ﬁle system knows about. A stat on the symlink reveals this.

And because of the way symbolic links are created, they leave the possibility for what is known as a **dangling reference**:

```text
$ echo hello > file 
$ ln -s file file2 
$ cat file2 
hello 
$ rm file 
$ cat file2 
cat: file2: No such file or directory
```

