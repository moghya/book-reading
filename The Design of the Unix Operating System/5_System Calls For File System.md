# File system DS

* The User Descriptor Table

    - Every process has its own User Descriptor Table. This table is essentially an array of pointers to open file structures and other I/O object structures (such as pipes) currently in use by the process.

 

    - The array elements are numbered from 0 and files are referenced by using one of the array index numbers as a parameter to system calls such as read and write



* The System File Table
    
    - Each element in the User Descriptor Table is a pointer to a file structure in the System File Table.

    - The System File Table is a shared centralised data structure containing a file structure element for every file or I/O object that is currently in use in the system.

* Inode Table -

    - Each Entry in file table points to inode which stores that file


# 5.1 OPEN

* fd open(pathname, flags, inodes);

* inodes give the file permissions if the file is being created

* The
open system cal] returns an integer' called the user file

* The kernel searches the file system for the file name parameter using algorithm
namtei

* Allocates an entry in the file table for the open file.

*  The file table
entry contains a pointer to the inode of the open file and a field that indicates the
byte offset in the file where the kernel expects the next read or write to begin

* The kernel allocates an entry in a private table in the process u
area, called the user file descriptor table, and notes the index of this entry. 

* The
index is the file descriptor that is returned to the user. The entry in the user file
table points to the entry in the global file table.

     ![title](xyz1.png)

     -  file descriptors index into a per-process file descriptor table maintained by the kernel,
     
     - that in turn indexes into a system-wide table of files opened by all processes, called the file table


    > Each open returns a file descriptor to the process, and
the corresponding entry in the user file descriptor table points to a unique entry in  the kernel file table even though one file ("/etc/passwd") is opened twice. ??

* The process can read or write the file "/etc/passwd" but only through file
descriptors 3 and 5 in the figure. 

* The first three user file descriptors (0, 1, and 2) are called the standard input,
standard output, and standard error file descriptors

# 5.2 READ

* number=read(fd, buffer, count)

* buffer is the address of a data
structure in the user process that will contain the read data on successful
completion of the call,

* count is the number of bytes the user wants to read, and
number is the number of bytes actually read

* The kernel gets the file table entry that corresponds to user descriptor table
 
*   Kernel sets the following fields so that is doesnot have to pass them as function parameters  

    |Io Param | Description|
    |---------|-------------|
    |inode | indicates read or write|
    |Count |count of bytes to read or write
    |Offset|byte offset in file|
    |Address | target address to copy data, in user or kernel memory|
    |Flag| indicates if address is in user or kernel memory|




* When a process invokes the read system call, the kernel locks the inode for the
duration of the call.

* see read algorithm fig 5.5

* In Below example first read will read from  0 offest the next from 20 offset n next from 1044 as after each read as the offset field of file table will be changed
    ```
        #include <fcritl.h>
        main()
        {
            int fd;
            char libuf[1024],bigbuf[1024];
            fd=open(letc/passwd", O_RDONLY);
            read(fd, libbuf, 20);
            read(fd, bigbuf, 1024);
            read (fd, lilbuf, 20);
        }


    ```


    >  3rd para on page 99 how is it advantageous ??

    > First 2 para of page 100 ??

* It would be unfair for the system to keep an inode locked from the
time a process opened the file until it closed the file, because one process could
keep a file open and thus prevent other processes from ever accessing it.

* If the file
was "/etc/passwd", used by the login process to check a user's password, then one
malicious (or, perhaps, just errant) user could prevent all other users from logging
in.

* To avoid such problems, the kernel unlocks the inode at the end of each system
call that uses it and not from time when file is opened to when file is closed

# 5.3 WRITE

* number=write(fd, buffer, count);

* The algorithm for writing a regular file is similar
to that for reading a regular file.

*  However, if the file does not contain a block that
corresponds to the byte offset to be written, the kernel allocates a new block using
algorithm alloc and assigns the block number to the correct position in the inode's
table of contents

* If the byte offset is that of an indirect block, the kernel may have to allocate several blocks for use as indirect blocks and data blocks

* When the write is complete, the kernel updates the file size entry in
the inode if the file has grown larger.

* During each iteration, as in read it determines whether it
will write the entire block or only part of it.

* If it writes only part of a block, it
must first read the block from disk so as not to overwrite the parts that will remain
the same, but if it writes the whole block, it need not read the block, since it will
overwrite its previous contents anyway

# 5.4 FILE AND RECORD LOCKING

* The original UNIX system developed by Thompson and Ritchie did not have an
internal mechanism by which a process could insure exclusive access to a file.

* File locking is the capability to prevent other processes from reading
or writing any part of an entire file
* Record locking is the capability to prevent
other processes from reading or writing particular records (parts of a file between
particular byte offsets)

# 5.5 ADJUSTING THE POSITION OF FILE I/O LSEEK

* Processes can use the kernel system call to position the I/O and allow random
access to a file

* position=lseek(fd, offset, reference);

* The return value, position, is the byte offset where the next read or write will start.

* The lseek system call has nothing to do
with the seek operation that positions a disk arm over a particular disk sector.

* To implement lseek, the kernel simply adjusts the offset value in the file table;
subsequent read or write system calls use the file table offset as their starting byte
offset.

 # 5.6 CLOSE

 * close (fd) ;

 * The kernel does the close operation
by manipulating the file descriptor and the corresponding file table and inode table
entries.

* If the file
table reference count is 1, the kernel frees the entry and releases the in-core inode
originally allocated in the open system call (algorithm iput). 

* If other processes still
reference the inode, the kernel decrements the inode reference count but leaves it
allocated;


*   > On page 105 why even after closing the process B the inode for Private is not freed??

# FILE CREATION

* fd = create (pathname, inodes);

* If no such file previously existed, the kernel creates a new file
with the specified name and permission inodes; 

* if the file already existed, the kernel
truncates the file (releases all existing data blocks and sets the file size to 0) subject
to suitable file access permissions.

* If the kernel does not find the path name component in the
directory, it will eventually write the name into the empty slot just found.
* If the directory has no empty slots, the kernel remembers the offset of the end of the
directory and creates a new slot there.

* It also remembers the inode of the directory
being searched in its u area and keeps the inode locked; the directory will become
the parent directory of the new file.

* 1f the system
crashes between the write operations for the inode and the directory, there will be
an allocated inode that is not referenced by any path name in the system but the
system will function normally. 

* If, on the other hand, the directory were written
before the newly allocated inode and the system crashed in the middle, the file
system would contain a path name that referred to a bad inode.

* 1f the given file already existed before the create, the kernel finds its inode while
searching for the file name. The old file must allow write permission for a process
to create a "new" file by the same name, because the kernel changes the file
contents during the create call
*  It truncates the file, freeing all its data blocks using
algorithm free, so that the file looks like a newly created file.
*  However, the owner
and permission inodes of the file are the same as they were for the original file:
The kernel does not reassign ownership to the owner of the process, and it ignores
the permission inodes specified by the process.
*  Finally, the kernel does not check
that the parent directory of the existing file allows write permission, because it will
not change the directory contents.

# 5.8 CREATION OF SPECIAL FILES

* The system call mknod creates special files in the system, including named pipes,
device files, and directories.

* mknod(pathname, type and permissions, dev)

* dev specifies the major and minor device numbers for block and
character special files (Chapter 10).

* After assigning the inode it sets the file type field in
the inode to indicate that the file type is a pipe, directory or special file.

* > If the mknod call is creating a directory
node, the node will exist after the system call completes but its contents will be in
the wrong format (there are no directory entries for "." and ".."). ??

# 5.9 CHANGE DIRECTORY AND CHANGE ROOT

*  chdir (pathname);

* The kernel parses the name of the target directory using algorithm namei
and checks that the target file is a directory and that the process owner has access
permission to the directory

* Locks new inode & increases the reference count & releases the current inode lock

* The kernel contains a global variable that points to the inode of the
global root, allocated by (get when the system is booted. Processes can change their
notion of the file system root via the chroot system call

* This is useful if a user
wants to simulate the usual file system hierarchy and run processes there

* chroot (pathname) ;

* However, since the default root for the kernel is stored in a global variable, it does not release
the inode of the old root automatically, but only if it or an ancestor process had
executed the chroot system call. 

* The new inode is now the logical root of the file
system for the process (and all its children), meaning that all path name searches
in algorithm namei that start from root ("/") start from this inode, and that all
attempts to use ".." over the root will leave the working directory of the process in
the new root.

# 5.10 CHANGE OWNER AND CHANGE inode

* Changing the owner or inode (access permissions) of a file are operations on the
inode, not on the file per se. 


* chown(pathname, owner, group)

* chmod (path name, inode)

* The process owner must be superuser or match that of the file
owner . The
kernel then assigns the new owner and group to the file, clears the set user and set
group flags , and releases the inode via algorithm Iput.

# 5.11 STAT AND FSTAT

* The system calls stat and fstat allow processes to query the status of files, returning
information such as the file type, file owner, access permissions, file size, number of
links, m ode number, and file access times

* stat(pathname, statbuffer);

* fstat (fd, statbuffer);

* statbuffer is the address of a data structure in the user process that will
contain the status information of the file on completion of the call

* The system
calls simply write the fields of the inode into statbuffer


# 5.12 PIPES

* Pipes allow transfer of data between processes in a first-in-first-out manner (FIFO),
and they also allow synchronization of process execution.

* 2 types
    - named
    - unnamed

* Only related processes, descendants of a process that issued the pipe call,
can share access to unnamed pipes

* However, all processes can access a
named pipe regardless of their relationship, subject to the usual file permissions.

    # 15.12.1. Pipe System call
    * pipe (fdptr);

    * where fdptr is the pointer to an integer array that will contain the two file
    descriptors for reading and writing the pipe.

    * It also allocates a pair of user file descriptors
    and corresponding file table entries for the pipe: one file descriptor for reading
    from the pipe and the other for writing to the pipe.

    *  It uses the file table so that the
    interface for the read, write and other system calls is consistent with the interface
    for regular files

     # 15.12.2. Opening named pipe

     * named pipe is a file whose semantics are the same as those of an unnamed pipe,
except that it has a directory entry and is accessed by a path name.

    * Processes open
named pipes in the same way that they open regular files and, hence, processes that
are not closely related can communicate.

    * Named pipes r permanent but unnamed pipes are transient

    * process that opens the
named pipe for reading will sleep until another process opens the named pipe for
writing, and vice versa

    * Depending on
whether the process opens the named pipe for reading or writing, the kernel
awakens other processes that were asleep, waiting for a writer or reader process

    # 5.12.3 Reading and Writing Pipes

    * Processes access data from pipes in FIFO manner

    * if the number of readers
or writers is greater than 1, they must coordinate use of the pipe with other
mechanisms

    * The difference between storage allocation for a pipe and regular file is that a pipe uses only the direct blocks of the inode for greater
efficiency

    * But places no limit on how much data can pipe hold

    *  The number of bytes being written + the number of bytes already in the pipe <= to the pipe's
capacity.
        
        * Process increments files size in file table if it writes beyond EOF

        * Pipes increment pipe size after every write in inode itself

        * If the next byte offset in the pipe were to require
        use of an indirect block, the kernel adjusts the file offset value in the u area to
        point to the beginning of the pipe (byte offset 0). 
        
        *   >The kernel never overwrites data
        in the pipe; it can reset the byte offset to 0 because it has already determined that
        the data will not overflow the pipe's capacity ??

        * When the writer process bas written
        all its data into the pipe, the kernel updates the pipe's (inode) write pointer so that
        the next process to write the pipe will proceed from where the last write stopped.

        * The kernel then awakens all other processes that fell asleep waiting to read data
        from the pipe

    * Reading a pipe

        * if pipe has data read is same as that of regular file

        * initial offset of read pointer is stors in inode from where read takes place

        * After reading each block, the kernel decrements the size of the pipe according to the number of bytes it read 
        
        * And it adjusts the user area offset value to wrap around to the
        beginning of the pipe, if necessary

        * When the read system call completes, the
        kernel awakens all sleeping writer processes and saves the current read offset in the
        m ode (not in the file table entry).

    * If the pipe is empty, the process will typically sleep
until another process writes data into the pipe,

    * If a process writes a pipe and the pipe cannot hold all the data, the kernel
    marks the inode and goes to sleep waiting for data to drain from the pipe.
    
    *   >The exception to this statement is when a process writes
    an amount of data greater than the pipe capacity (that is, the amount of data that
can be stored in the inode direct blocks); ?? Isnt it same as above point ??
    * In above case, the kernel writes as much data as
        possible to the pipe and puts the process to sleep until more room becomes
        available. Thus, it is possible that written data will not be contiguous in the pipe

    * Implementation differs because the kernel stores the
    read and write offsets in the inode instead of in the file table.

    * This is because that processes can share their
    values: They cannot share values stored in file table entries because a process gets
    a new file table entry for each open call.

    # 5.12.4 Closing Pipes

    *  same as for regular file except that the kernel does special processing before releasing
the pipe's inode

    * The kernel decrements the number of pipe readers or writers,
    according to the type of the file descriptor

    * If the count of writer processes drops to
    0 and there are processes asleep waiting to read data from the pipe, the kernel
    awakens them, and they return from their read calls without reading any data

    * If the count of reader processes drops to 0 and there are processes asleep waiting to
write data to the pipe, the kernel awakens them and sends them a signal to indicate an error condition.

    * This is because if a process is waiting to read an unnamed pipe and
there are no more writer processes, there win never be a writer process.

    * If no reader or writer processes
    access the pipe, the kernel frees all its data blocks and adjusts the inode to indicate
    that the pipe is empty.

    * See examples on 116
        ```
        #include <fentl.h>
        char string[] = "hello";
        main(argc, argv)
        {
            int argc;
            char *are[];
            int fd;
            char buf[2561];
            /* create named pipe with read/write permission for all users 'V
            mknod("fifo", 010777, 0);
            if (argc == 2)
            fd open("fifo", O_WRONLY);
            else
            fd open("fifo", O_RDONLY);
            for (;;)
            if (argc 2)
            write(fd, string, 6);
            else
            read(fd, buf, 6);
        }
        ```
    *  If invoked with a second (dummy) argument, it continually writes the string "hello" into the pipe; 
    
    * if invoked without a second argument, it reads the
    named pipe. 
    
    * The two processes are invocations of the identical program and have
    secretly agreed to communicate through the named pipe "fifo", but they need not
    be related. Other users could execute the program and participate in (or interfere
    with) the conversation.
# Difference in named and unnamed pipes
> https://www.youtube.com/watch?v=dhFkwGRSVGk

1. As suggested by their names, a named type has a specific name which can be given to it by the user. Named pipe if referred through this name only by the reader and writer. All instances of a named pipe share the same pipe name.
On the other hand, unnamed pipes is not given a name. It is accessible through two file descriptors that are created through the function pipe(fd[2]), where fd[1] signifies the write file descriptor, and fd[0] describes the read file descriptor.

2. An unnamed pipe is only used for communication between a child and it’s parent process, while a named pipe can be used for communication between two unnamed process as well. Processes of different ancestry can share data through a named pipe.

3. A named pipe exists in the file system. After input-output has been performed by the sharing processes, the pipe still exists in the file system independently of the process, and can be used for communication between some other processes.
On the other hand, an unnamed pipe vanishes as soon as it is closed, or one of the process (parent or child) completes execution.

4. Named pipes can be used to provide communication between processes on the same computer or between processes on different computers across a network, as in case of a distributed system. Unnamed pipes are always local; they cannot be used for communication over a network.

5. A Named pipe can have multiple process communicating through it, like multiple clients connected to one server. On the other hand, an unnamed pipe is a one-way pipe that typically transfers data between a parent process and a child process.



# 5.13 DUP

* The dup system call copies a file descriptor into the first free slot of the user file
descriptor table, returning the new file descriptor to the user.

* newfd = dup(fd)

* Because dup duplicates the file descriptor, it increments the
count of the corresponding file table entry

* Its an inelegant system call,it assumes that the user knows
that the system will return the lowest-numbered free entry in the user file descriptor table

    # why is dup required
    * One example use would be I/O redirection. For this you fork a child process and close the stdin or stdout file descriptors (0 and 1) and then you do a dup() on another file descriptor of your choice which will now be mapped to the lowest available file descriptor, which is in this case 0 or 1.

    * Using this you can now exec any child process which is possibly unaware of your application and whenever the child writes on the stdout (or reads from stdin, whatever you configured) the data gets written on the provided filedescriptor instead.

# 5.14 MOUNTING AND UNMOUNTING FILE SYSTEMS

* A physical disk unit consists of several logical sections, partitioned by the disk
driver, and each section has a device file name.

* The mount system call connects the file system in a specified section of a disk to the
existing file system hierarchy, and the unmount system call disconnects a file system
from the hierarchy

* mount(special pathname, directory pathname, options);

* Special pathname is the name of the device special file of the disk section
containing the file system to be mounted,


* directory pathname is the directory in the
existing hierarchy where the file system will be mounted (called the mount point),
and options indicate whether the file system should be mounted "read-only"

* see fig 5.22 and read paragraph below it

# 5.14.1 Crossing Mount Points in File Path Names

* mount /dev/disk1 /usr</br>
    cd /usr/src/uts</br>
    cd ../../..

* 2 types of crossing mount points
    * crossing from the mounted-on file system to the mounted file system (in the direction from
    the global system root towards a leaf node)
    
    * crossing from the mounted file
    system to the mounted-on file system. 

* For the first case

    * we use diff iget where only change is that it checks if the inode is a mount point

    * If the inode is marked "mounted-on," the kernel knows that it is a mount point. It
    finds the mount table entry whose mounted-on inode is the one just accessed and
    notes the device number of the mounted file system.


    * Using the device number and
    the inode number for root, which is common to all file systems, it then accesses the root inode of the mounted device and returns that inode.

* for second case

    * In the example above (cd "../../.."), assume the starting current directory of the
process is "/usr/src/uts"

    * The kernel changes the working inode to
that of "/usr/src" as a result of parsing the first ".." in the path name. 
    * Then, it
parses the second ".." in the path name, finds the root inode of the (previously)
mounted file system, "usr", and makes it the working inode in namei.

    * Finally, it
parses the third ".." in the path name: It finds that the inode number for ".." is
the root inode number, its working inode is the root inode, and ".." is the current
path name component.

    *  The kernel finds the mount table entry for the "usr" mount
    point, releases the current working inode (the root of the "usr" file system), and
    allocates the mounted-on inode (the inode for directory "usr" in the root file
    system) as the new working inode. 

    * It then searches the directory structures in the
    mounted-on "/ust" for ".." and finds the inode number for the root of the file
    system ("1"). 

    * The chdir system call then completes as usual; the calling process is
    oblivious to the fact that it crossed a mount point.    


# 5.14 UNMOUNT

* When unmounting a file system the kernel accesses the inode of the device
to be unmounted, retrieves the device number for the special file, releases the inode, and finds the mount table entry whose device number equals that
of the special file.

* Before the kernel actually unmounts a file system, it makes sure
that no files on that file system are still in use by searching the inode table for all
files whose device number equals that of the file system being unmounted. 

* If any files from the file system are
active, the umount call fails: if it were to succeed, the active files would be
inaccessible.

* The buffer pool may still contain "delayed write" blocks that were not written
to disk, so the kernel flushes them from the buffer pool. 

* The kernel writes out all recently modified super blocks to disk, and updates the disk
copy of all inodes that need updating.

* kernel goes through the buffers in the buffer cache and
invalidates buffers for blocks on the now unmounted file system; there is no need to
cache data in those blocks any longer.

* It clears the "mounted-on" flag in the mounted-on inode set
during the mount call and releases the inode. 

* After marking the mount table entry
free for general use, the umount call completes.

# 5.15 LINK

* The link system call links a file to a new name in the file system directory
structure, creating a new directory entry for an existing inode.

* link(source file name, target file name);

* Oly for super user

* 1f arbitrary users could link directories, programs designed to traverse the file hierarchy would have to worry about getting
into an infinite loop if a user were to link a directory to a node name below it in
the hierarchy,

* The kernel first locates the inode for
the source file using algorithm namei, increments its link count, updates the disk
copy of the inode (for consistency, as will be seen), and unlocks the inode. 

* It then searches for the target file; if the file is present, the link call fails, and the kernel
decrements the link count incremented earlier

* It notes the location of
an empty slot in the parent directory of the target file, writes the target file name and the source file inode number into that slot, and releases the inode of the target
file parent directory via algorithm iput.

* The link count keeps count
of the directory entries that refer to the file and is thus distinct from the inode
reference count

* Two deadlock possibilities are worthy of note, both concerning the reason the
process unlocks the source file inode after incrementing its link count. If the kernel
did not unlock the inode, two processes could deadlock by executing the following
system calls simultaneously,</br><br>
process A: link("a/b/c/d", "c/f/g");</br>
process B: link("e/f", "a/b/c/d/ee");

* Suppose process A finds the inode for file "a/b/c/d" at the same time that process
B finds the inode for "e/f".

*  When process A now attempts to find the inode for
directory "elf", it would sleep awaiting the event that the inode for "1" becomes
free. But when process B attempts to find the inode for directory "a/b/c/d", it
would sleep awaiting the event that the inode for "d" becomes free. Process A
would be holding a locked inode that process B wants, and process B would be
holding a locked inode that process A wants.

* A single process could also deadlock itself. If it
executed</br></br>
link ("a/b/c", "a/b/c/d");

* If it would allocate the inode for file "c" in the first part of the algorithm; if the
kernel did not release the inode lock, it would deadlock when encountering the
m ode "c" in searching for the file "d".

* Since inodes are finitely allocatable resources, receipt of a signal cannot
awaken the process from its sleep (Chapter 7). Hence, the system could not break
the deadlock without rebooting.

* However, any processes that accessed those files (or attempted to access other files
via the locked directory) would deadlock. Thus, if the file were "Thin" or
"/usr/bin" (typical depositories for commands) or "/bin/sh" (the shell) the effect
on the system would be disastrous.


# 5.16 UNLINK

* The unlink system call removes a directory entry for a file.

* myfile. If the file being unlinked is the last link of the file, the kernel
eventually frees its data blocks. 

* However, if the file had several links, it is still
accessible by its other names.

* The kernel first uses a
variation of algorithm namei to find the file that it must unlink, but instead of
returning its inode, it returns the inode of the parent directory

* the kernel clears the file name from the parent directory: Writing a 0
for the value of the inode number suffices to clear the slot in the directory


* The
kernel then does a synchronous write of the directory to disk to ensure that the file
is inaccessible by its old name, 

* It decrements the link count, and releases the in-core
m odes of the parent directory and the unlinked file via algorithm iput.

* if the reference
count drops to 0, and if the link count is 0, the kernel reclaims the disk blocks
occupied by the file.

    # 5.16.1 File Systenrb Consistency
    
    * The kernel orders its writes to disk to minimize file system corruption in event of
system failure
     
     * when it removes a file name from its parent directory,
it writes the directory synchronously to the disk — before it destroys the contents of
the file and frees the mode

    * If the system were to crash before the file contents
were removed, damage to the file system would be minimal: There would be an
inode that would have a link count 1 greater than the number of directory entries
that access it, but all other paths to the file would still be legal.
    
    * If the directory
write were not synchronous, it would be possible for the directory entry on disk to
point to a free (or reallocated!) inode after a system crash.

    * In particular, if the file name was that of the last link to
the file, it would refer to an unallocated mode.

    * System damage is clearly less
severe and easier to correct in the first case

    * see example on page 137

    * The kernel also frees inodes and disk blocks in a specific order to minimize
corruption in event of system failure.

    * When removing the contents of a file and
clearing its mode, it is possible to free the blocks containing the file data first, or it
is possible to free and write out the mode first

    * result for both is same except for the case when system crashes

    * Suppose the kernel
first frees the disk blocks of a file and crashes.

        * When the system is rebooted, the
Mode still contains references to the old disk blocks, which may no longer contain
data relevant to the file

        * It is also possible that other files were
assigned those disk blocks. 
        
        * The effort to clean the file system with the fsek
program would be great.

    * If the system first writes the mode to disk and
the system crashes

        * The data blocks that previously belonged to the file
would be inaccessible to the system, but users would notice no apparent corruption

        * The fsck program also finds the task of reclaiming unlinked disk blocks easier than
the clean-up it would have to do for the first sequence of events.

    # 5.16.2 Race Conditions

    * The system call, particularly when unlinking rmdir command removes a directory after verifying that the directory contains no files

    * rmdir runs at user level thus its not atomic

    * Hence, another process could crew a file in the directory after rmdir
determined that the directory was empty

    * only solution is file and record locking

    * Recall the algorithm for the link system call and how the kernel unlocks the
 inode before completion of the call. 
    *  If another process should unlink the file while
the inode lock is free, it would only-decrement the link count; since the link count
had been incremented before unlinking the inode, the count would still be greater
than 0. 

    *   Hence, the file cannot be removed, and the system is safe. 
    
    * The condition is
equivalent to the case where the unlink happens immediately after the link call
completes.

    * Another race condition exists in the case where one process is converting a file
path name to an inode using algorithm namei and another process is removing a
directory in that path

    * See example on page 135

    * A process can unlink a file while another process has the file open

    * Also unlinking can happen after open sys call
    
    * Since the kernel
unlocks the mode at the end of the open call, the unlink call will succeed.

    * No other processes will be able to access
the now unlinked file.

    * since the open system call had incremented the
Mode reference count, the kernel does not clear the file contents when executing the
(put algorithm at the conclusion of the unlink call

    * So the opening process can do
all the normal file operations. But when it closes the file, the mode reference count drops to 0 in iput,
and the kernel clears the contents of the file

    * In short, the process that had open cd
the file proceeds as if the unlink did not occur, and the unlink happens as if the file
were not open.

    * consider following example

        ```
            #include < sys/types.h >
            #include < sys/stat.h>
            #include < fcntl.h>
            main (argc, argv)
            int argc;
            char *argv[]; ·
            int fd;
            char buf[ I 0241;
            struct stat statbuf;
            UNLINK
            if (argc != 2) /* need a parameter */
            exit O ;
            fd = open (argv[ I ], O_RDONLY) ;
            if (fd === - I ) /* open fails * /
            exit O ;
            i f (unlink (argv[ I ]) -- - I ) /* unlink file just opened */
            exit O ;
            i f (stat (argv[ I ], &statbuf) ...... - I ) /* stat the file by name*/
            printf("stat %s fails as it should\n ", argv[ I ]) ·
            else
            · printf("stat %s succeeded!!!!\n", argv[ I ]) ;
            i f (fstat(fd, &statbuO ....... - I ) /* stat the file by fd */
            printf("fstat %s fails!!!\n", argv[ I ]) ;
            else
            printf("fstat %s succeeds as it should\n", argv[ I ]) ;
            while (read (fd, buf, sizeof(buf) ) > 0) /* read open/unlinked file */
            printf("%I 024s", buf) ; /* prints IK byte field */     

        ```

        * a process opens a file supplied as a parameter and
then unlinks the file it just open cd. 
        
        * The stat call fails because the original path name no longer refers to a file after the unlink

        * but the fstat call succeeds because it gets to
the inode via the file descriptor.

        * After the close in exit , the file no. longer exists


# 5.17 FILE SYSTEM ABSTRACTIONS

* Weinberger introduced file system types to support his network file system

* File system types allow the kernel to
support multiple file systems simultaneously, such as network file systems or even file systems of other operating systems

* same sys call but the kernel maps a generic set of file operations Mto
operations specific to each file system type.

* The inode is the interface between the abstract file system and the specific file
system.

* Generic in-core mode contains data that is independent of particular file
systems, and points to a file-system-specific mode that contains file-system-specific
data.

* The file-system-specific mode contains information such as access permissions
and block layout, but the generic mode contains the device number, Mode number,
file type, size, owner, and reference count. Other data that is file-system-specific
includes the super block and directory structures

* The latter inode
presumably contains enough information to identify a file on a remote system. 

* A file system may not have an inode-like structure; but the file-system-specific code
manufactures an object that satisfies UNIX file system semantics and allocates its
"mode" when the kernel allocates a generic mode.

* When the kernel wants to access a file, it makes
an indirect function call, based on the file system type and the operation (see
Figure 5.34).

# 5.18 FILE SYSTEM MAINTENANCE

* kernel maintains consistency in file sys but incase of power loss file sys may be left inconsistent

* The command fsck
checks for such inconsistencies and repairs the file system if necessary

* It accesses
the file system by its block or raw interface (Chapter 10) and bypasses the regular
file access methods.

* When a file system is originally set up, all disk blocks are on the free list.

* When a disk block is assigned for use, the kernel removes it from the free list and
assigns it to an mode.

*  The kernel may not reassign the disk block to another Mode
until the disk block has been returned to the free list

* Consider the possibilities if the
kernel freed a disk block in a file, returning the block number to the in-core copy of
the super block, and allocated the disk block to a new file. 

* If the kernel wrote the
inode and blocks of the new file to disk but crashed before updating the inode of
the old file to disk, the two modes would address the same disk block number

* Similarly, if the kernel wrote the super block and its free list to disk and crashed
before writing

* _**If a block number is not on the free list of blocks nor contained in a file, the file
system is inconsistent because, as mentioned above, all blocks must appear
somewhere.**_

* This situation could happen if a block was removed from a file and
placed on the super block free list. If the old file was written to disk and the
system crashed before the super block was written to disk, the block would not
appear on any lists stored on disk.

* An inode may have a non-0 link count, but its mode number may not exist in
any directories in the file system.

    * If the system crashes after creating a pipe or after creating a
file but before creating its directory entry, the mode will have its link field set even
though it does not appear to be in the file system. 

    * The problem could also arise if a
directory were unlinked before making sure that all files contained in the directory
were unlinked.


* If the format of an Mode is incorrect (for instance, if the file type field has an
undefined value), something is wrong. 

    * This could happen if an administrator
    mounted an improperly formatted file system. 

    * The kernel accesses disk blocks that
    it thinks contain Modes but in reality contain data.

* If an Mode number appears in a directory entry but the mode is free, the file
system is inconsistent because an Mode number that appears in a directory entry
should be that of an allocated Mode.

    * This could happen if the kernel was creating
a new file and wrote the directory entry to disk but did not write the Mode to disk
before the crash.

    *  It could also occur if a process unlinked a file and wrote the
freed mode to disk, but did not write the directory element to disk before it
crashed. 

    * These situations are avoided by ordering the write operations properly.


* If the number of free blocks or free modes recorded in the super block does not conform to the number that exist on disk, the file system is inconsistent.

* The
summary information in the super block must always be consistent with the state of
the file system.
