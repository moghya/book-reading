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

* fd open(pathname, flags, modes);

* modes give the file permissions if the file is being created

* The
open system cal] returns an integer' called the user file

* The kernel searches the file system for the file name parameter using algorithm
namtei

* Allocates an entry in the file table for the open file.

*  The file table
entry contains a pointer to the mode of the open file and a field that indicates the
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
    |Mode | indicates read or write|
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

* It would be unfair for the system to keep an mode locked from the
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
by manipulating the file descriptor and the corresponding file table and mode table
entries.

* If the file
table reference count is 1, the kernel frees the entry and releases the in-core mode
originally allocated in the open system call (algorithm iput). 

* If other processes still
reference the mode, the kernel decrements the inode reference count but leaves it
allocated;


*   > On page 105 why even after closing the process B the inode for Private is not freed??

# FILE CREATION

* fd = create (pathname, modes);

* If no such file previously existed, the kernel creates a new file
with the specified name and permission modes; 

* if the file already existed, the kernel
truncates the file (releases all existing data blocks and sets the file size to 0) subject
to suitable file access permissions.

* If the kernel does not find the path name component in the
directory, it will eventually write the name into the empty slot just found.
* If the directory has no empty slots, the kernel remembers the offset of the end of the
directory and creates a new slot there.

* It also remembers the mode of the directory
being searched in its u area and keeps the inode locked; the directory will become
the parent directory of the new file.

* 1f the system
crashes between the write operations for the mode and the directory, there will be
an allocated inode that is not referenced by any path name in the system but the
system will function normally. 

* If, on the other hand, the directory were written
before the newly allocated Mode and the system crashed in the middle, the file
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
the permission modes specified by the process.
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
the mode to indicate that the file type is a pipe, directory or special file.

* > If the mknod call is creating a directory
node, the node will exist after the system call completes but its contents will be in
the wrong format (there are no directory entries for "." and ".."). ??

# 5.9 CHANGE DIRECTORY AND CHANGE ROOT

*  chdir (pathname);

* The kernel parses the name of the target directory using algorithm namei
and checks that the target file is a directory and that the process owner has access
permission to the directory

* Locks new inode & increases the reference count & releases the current inode lock

* The kernel contains a global variable that points to the mode of the
global root, allocated by (get when the system is booted. Processes can change their
notion of the file system root via the chroot system call

* This is useful if a user
wants to simulate the usual file system hierarchy and run processes there

* chroot (pathname) ;

* However, since the default root for the kernel is stored in a global variable, it does not release
the mode of the old root automatically, but only if it or an ancestor process had
executed the chroot system call. 

* The new inode is now the logical root of the file
system for the process (and all its children), meaning that all path name searches
in algorithm namei that start from root ("/") start from this inode, and that all
attempts to use ".." over the root will leave the working directory of the process in
the new root.

# 5.10 CHANGE OWNER AND CHANGE MODE

* Changing the owner or mode (access permissions) of a file are operations on the
Mode, not on the file per se. 


* chown(pathname, owner, group)

* chmod (path name, inode)

* The process owner must be superuser or match that of the file
owner . The
kernel then assigns the new owner and group to the file, clears the set user and set
group flags , and releases the mode via algorithm Iput.

# 5.11 STAT AND FSTAT

* The system calls stat and fstat allow processes to query the status of files, returning
information such as the file type, file owner, access permissions, file size, number of
links, m ode number, and file access times

* stat(pathname, statbuffer);

* fstat (fd, statbuffer);

* statbuffer is the address of a data structure in the user process that will
contain the status information of the file on completion of the call

* The system
calls simply write the fields of the mode into statbuffer


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

    * The difference between storage allocation for a pipe and regular file is that a pipe uses only the direct blocks of the mode for greater
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
        all its data into the pipe, the kernel updates the pipe's (mode) write pointer so that
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
can be stored in the Mode direct blocks); ?? Isnt it same as above point ??
    * In above case, the kernel writes as much data as
        possible to the pipe and puts the process to sleep until more room becomes
        available. Thus, it is possible that written data will not be contiguous in the pipe

    * Implementation differs because the kernel stores the
    read and write offsets in the mode instead of in the file table.

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
    access the pipe, the kernel frees all its data blocks and adjusts the mode to indicate
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

