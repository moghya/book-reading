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


    > Each open returns a file descriptor to the process, and
the corresponding entry in the user file descriptor table points to a unique entry in  the kernel file table even though one file ("/etc/passwd") is opened twice. ??

* The process can read or write the file "/etc/passwd" but only through file
descriptors 3 and 5 in the figure. 

* The first three user file descriptors (0, 1, and 2) are called the standard input,
standard output, and standard error file descriptors

