# Introduction
* Section 4.1 examines
the mode and how the kernel manipulates it,
* Section
4.3 investigates the structure of directories, the files that allow the kernel to
organize the file system as a hierarchy of files
* Section 4.4 presents the
algorithm for converting user file names to modes. Section 4.5 gives the structure
of the super block
* Sections 4.6 and 4.7 present the algorithms for assignment
of disk modes and disk blocks to files. 
* Section 4.8 talks about other file
types in the system, namely, pipes and device files.


# 4.1 Inodes
* Basic Contents of Disk Inode
     - Ownership
     - File type
    - Access Permissions
    - Last modifies
    - Last Accessed
    - Table of content
    - File size
    - Number of links to the file, representing the number of names the file has in the
    directory hierarchy

* The contents of a file change only when
writing it.
*  The contents of an inode change when changing the contents of a file or
when changing its owner, permission, or link settings.

>  in - core  copy of inode ?? pg 63 
http://prashantagmnnit.blogspot.com/2014/03/os-incore-inode-vs-disk-inode.html

> The in-core representation of the Mode differs from the disk copy as a result
of a change to the data in the mode, -->


 > The in-core representation of the file differs from the disk copy as a result of
a change to the file data,  -- >pg63

> Pointers to other in-core inodes. The kernel links inodes on hash queues and on
a free list in the same way that it links buffers on buffer hash queues and on the
buffer free list --> pg63


* Incore copy of Inode Content list
    -  is Locked
    -  Whether some process is waiting for inode to be unlocked.
    -  Inode Changed 
    -  File changed
    - Device no. of File System from which this inode is copied into Main Memory.
    - Inode Number
    - Refernce Count - Active instances of the file
    - Pointer to other incore inodes
* Inodes are very similar to buffer caches
    - lf a process attempts to access a file whose
Mode is not currently in the in-core inode pool, the kernel reallocates an in-core
inode from the free list for its use

* The kernel identifies particular inodes by their file system and mode number

* iget is similar to getblk of previous chapter

* Similar to getblk ,The kernel maps the
device number and mode number into a hash queue and searches the queue for the
inode 


* The iget algorithm returns a locked mode structure
with reference count 1 greater than it had previously been
