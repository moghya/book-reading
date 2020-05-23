# 1. General Overview of the System

- UNIX system has two parts
    * programs and services e.g shell, mail, text processing ...
    * operating system that supports above programs and services
- book focuses on operating system

## 1.1 History  

- multics project
    * simultaneous computer access
- reasons for popularity and success  
    * written in high level language
    * simple user interface
    * provides primitives to be able to build complex programs
    * uses hierarchical file system
    * consistent format for files and byte streams
    * simple and consistent interface to peripheral devices
    * hides machine architechture

## 1.2 System Structure

- set of layers
    - hardware -> UNIX OS (Kernel) -> system programs -> user / application programs
    - system call : methods used by user programs to interact with kernel
    - encourage combnination of existing programs to accomplish a task

## 1.3 User Perspective  

### 1.3.1 The File System  

- hirerarchical structure
- consistent treament of file data
- ability to create, delete files
- dynamic growth of files
- protection of files data
- peripheral devices as files
- root (/) ==> regual files, directory files, special device files
- name is path name to locate in file hierarchy
- programs in UNIX system have no knowlege of internal format of storage of files, it is just unformatted stream of bytes
- sytax is common all types of files but symantics are to be implemented by programs
- data in directory files is list of files it has
- access permission for managing access of files
- same as above goes for device files
- open, creat, read, write syscall for file manipulaions
- _int open(path, mode)_ -> creates regular file and returns fd of file or -1
- _int creat(patm mode)_ -> returns fd of new file or -1
- _int read(fd, buf, bufsize)_ -> returns bytes read or 0, error_code
- _int write(fd, bud, bufsize)_ -> TODO(moghya): check what does it return
- _mknod_ is syscall for creating directory file



### 1.3.2 Processing Environment

- program is an executable, process is an instance of the program
- multiprogramming/multitasking is supported, multiple programs simultaneoulsy or one prorgam multiple times simultaneoulsy
- syscalls for process creation, termination or sychronization
- fork - for creating a new process
- execl - for excetuing a program copy
- shell user program helps ins executing commands, essentialy executable programs by using fork and exec sys calls.

### 1.3.3 Building block primitives

- supported io redirection of standard input, output and error using >, <, and 2>
- pipe a mechanism to pass a stream of data between processes


## 1.4 Operating System Services

- control execution of processes
- scheduling processes
- allocating main memory
    * swapping if necessary
    * paging'
- allocating secondary memory for efficient storage
- issuing contolled access of peripheral devices to processes

## 1.5 Assumptions about Hardware

- two modes of execution, user and kernel
- safety of hardware is considered
- kernel is NOT a separate set of processes, instead kernel code execution happens as part of user processes executing in kernel mode
- i.e every user process executing in kernel mode is kernel process

### 1.5.1 Interrupts and Exceptions

- allows IO devices or system clock to interrup the CPU async way
- interrupts are serviced by kernel
- exception is unexpected condition, happens between execution of an instruction
- interrupt happens between two instructions ( makes sense )

### 1.5.2 Processor Execution Levels

- to make sure interrupts don't crash system or result in inconsistent state, execution level is maintained
- execution level decides what kind interrupts can get generated

### 1.5.3 Memory Management

- kernel maintains virtual to physical address translation
- depends on capabilities of hardware

## 1.6 Summary

- system has user and kernel mode
- user processes invoke sys calls to get into kernel mode
- encourages to write small programs, do few operations but do them well
- use io redirect and pipe to combine small programs
- kernel taks
    * service sys call
    * bookkeeping of multiple users
    * contolling process scheduling
    * maganing storage 
    * protection of process in main memory
    * fielding interuppts
    * managing files, devices

