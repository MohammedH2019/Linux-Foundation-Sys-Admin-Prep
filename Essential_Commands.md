# Essential Commands

## Log into local & remote graphical and text mode consoles

Basic concept to know:

* **Text Terminal**: text input/output environment.
  * Originally, they meant a piece of equipment through which you could interact with a computer: in the early days of Unix, that meant a teleprinter-style device resembling a typewriter, sometimes called a teletypewriter, or “tty” in shorthand
  * Tty were used to establish a connection to a mainframe computer and share operating system provided by it
  * A typical text terminal produces input and displays output and errors
* **Console**: terminal in modern computers that don't use mainframe but have an own operating system. It is generally a terminal in the physical sense that is, by some definition, the primary terminal directly connected to a machine. 
  * The console appears to the operating system "like" a remote terminal
  * In Linux and FreeBSD, the console, in realty, appears as several terminals (*ttys*) called *Virtual Consoles*
* **Virtual Consoles**: to provide several text terminals on a single computer
  * Multiple virtual consoles can be accessed simultaneously
* **Shell**: command line interface or CLI
  * It is the primary interface that users see when they log in,  whose primary purpose is to start other programs
  * It is presented inside console
  * There are many different Linux shells
  * Command-line shells include flow control constructs to combine commands. In addition to typing commands at an interactive prompt, users can write shell scripts 

To summarize: A virtual console is a shell prompted in a non-graphical environment, accessed from the physical machine, not remotely. 

* **Pseudo-terminal**: Terminal provided by programs called terminal emulators e.g. `ssh`, `tmux`

* **X Windows System**: is a windowing system for bitmap displays
  * X provides the basic framework for a graphical user interface (GUI) environment: drawing and moving windows on the display device and interacting with a mouse and keyboard
  * X does not mandate the user interface – this is handled by individual programs, like KDE or GNOME
  * It is considered "*graphical terminal*"
  * When is executed it will substitute one of the text terminal provided by virtual console. In CentOS the terminal will be 1, in other system could be 7.
  * Some applications running inside X Windows System provide pseudo-terminal e.g. Konsole, Gnome Terminal
  * If graphical environment is not started, you can run command `startx` to execute it



Log in:

* To log into local environment you must provide, when prompted, *userID* and *password* for both graphical and text mode
* To login into a remote text environment you can use command `ssh`
* To login into a remote graphical environment you can use command `ssh -X`

Once logged command `w` can be used to show who is logged and what they are doing:

~~~bash
[root@localhost ~]# w
23:41:16 up 2 min,  2 users,  load average: 0.02, 0.02, 0.01
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     tty1                      23:40   60.00s  0.01s  0.01s -bash
root     pts/0    192.168.0.34     23:41    1.00s  0.02s  0.00s w
~~~

First column shows  which user is logged into system and the second one to which terminal.

* For Virtual Console in terminal is showed tty1, tty2 etc.

* For ssh remote sessions (pseudo-terminal) in terminal is showed pts/0, pts/1 etc.
* :0 is for X11server namely used for graphical login

## Search for files

* `find` is recursive without parameters

* Base syntax: find PATH PARAMETERS

* `find /etc -name "\*host*"`

  Search in /etc all file/directories with host in their name. \* is a wildcard

* `find . -perm 777 -exec rm -f '{}' \;`

  Search from current position all files/directories with permissions 777 and after remove them

  `-exec` uses the result of find to do something

  `{}` will be substitute with result of find

  The exec's command must be contained between `-exec` and `\;`. 

  ; is treated as end of command character in bash shell. For this I must escape it with \\. If escaped it will be interpreted by find and not by bash shell.

* Some parameter accepts value n with + or - in front. The meaning is:

  * +n - for greater than n
  * -n - for less than n
  * n - for exactly n

* `find /etc -size -100k`

  Search in /etc all files/directories with size less of 100 kilobytes

* `find . -maxdepth 3 -type f -size +2M`

  Search starting from current position, descending maximum three directories levels, files with size major of 2 megabyte

* `find . \( -name name1 -o -name name2 \)`

  * `-o` or, it is used to combine two conditions. \ is escape to avoid that ( or ) will be interpreted by bash shell

* f`ind . -samefile file`

  * Find all files that have same i-node of file

* f`ind . \! -user owner` 

  * It will show all files that aren't owned by user owner. `!` means negation, but must be escaped by \ to  not be interpreted by bash shell

* `find . -iname name`

  * Search name ignoring case

* `find . -perm 222`

  * Find all files with permissions equal to 222. E.g. only file with permissions 222 will be showed

* `find . -perm -222`

  * Find all files with at least permissions 222. E.g. 777 match as valid.

* `find . -perm /222`

  * Find all files with write for owner or write for group or write for others (at least one)

* `find . -perm -g=w`

  * Find all files with at least  permission write for group

* `find . -atime +1` 

  * Show all files accessed at least two days ago (more than 24 hours)

* `find . -group developer` 

  * Show all files that are owned by the group named developer

* `find . -nouser -nogroup` 

  * Show all files/folder that have no user or group.

* `find . -empty ` 

  * Show all files/folder that are empty

* `find . -ctime -60` 

  * Show all files that have been changed in the last 60 min

## Grep command + regex

 Search pattern inside the strings of the files in path/*. Show file name and row matching pattern

  It is no recursive and key sensitive. To have recursion -r must be added

  Pattern can be a regular expression. The regular expression must be surrounded by '  ' otherwise content could match bash globing. 

  * `grep -l patter path/*`

    Search pattern inside file in path/*. Show only file name

  * `grep -lr patter path/*`

    Search pattern inside file in path/* and path subdirectories. Show only file name

  * `grep -ilr patter path/*`

    Search pattern ignoring case inside file in path/* and path subdirectories. Show only file name



Regular Expressions

| Character |                Definition                |  Example   |        Result         |
| :-------: | :--------------------------------------: | :--------: | :-------------------: |
|     ^     |            Start of a string             |    ^abc    |    abc, abcd, abc1    |
|     $     |             End of a string              |    abc$    |  abc, rasabc, 2aabc   |
|     .     |       Any character except newline       |    a.c     |     abc, acc, a1c     |
|           |                                          | Alteration |           a           |
|   {...}   | Explicit quantity of preceding character |   ab{2}c   |         abbc          |
|   [...]   |   Explicit set of characters to match    |   a[bB]c   |        abc,aBc        |
| [a-z0-9]  |   One lower case characters or number    | a[a-z0-9]c |        aac,a1c        |
|   (...)   |           Group of characters            |  (abc){2}  |        abcabc         |
|     *     | Null or more of the preceding characters |    a*bc    | bc, abc, aabc, aaaabc |
|     +     |  One or more of the preceding character  |    a+bc    |       abc, aabc       |
|     ?     |  Null or one of the preceding character  |    a?bc    |        bc, abc        |
|    ^$     |               Empty string               |            |                       |

* Not all regular expressions are supported by `grep`. As alternative can be used `egrep`

example:
* ` grep '^.[pr]' file1.txt
   looks for lines where the second character is either p or in the file1.txt

## Sed command
SED command in UNIX stands for stream editor and it can perform lots of functions on file like searching, find and replace, insertion or deletion. Though most common use of SED command in UNIX is for substitution or for find and replace.

* `sed OPTIONS... [SCRIPT] [INPUTFILE...]*`

* `sed 's/source/target/' file`

  In any row of file, it will change first occurrence of source to target. Print all rows e.g 
  * `sed 's/james/davids/' file1.txt` - will replace the first occurence in the row of james to davids in the file1.txt

* `sed 's/source/target/g' file`

  In any row of file, it will change all occurrences of source to target. Print all rows

* `sed 's/source/target/gI'`

  In any row of file, it will change all occurrences of source to target. Ignore case = case insensitive. Print all rows
    
  
* `sed -n '/source/p' file` 

  It will print only rows that contain source

  It is equal to grep source file
  * `sed -n 2,4p file` 

    It prints rows from 2 to 4

* `sed '/source/d' file`

   Delete rows with source
   
   * 'sed '2,$d' a.txt = will delete from the second line till the end of the file in a.txt

* `sed -n 12d file`

   Delete row 12
   
* `sed '3 i newline' file`

    It will insert newline as line 3
    * `sed '3 i hello' file.txt` = will insert hello in line 3 in file.txt

* ` sed ' /pattern/ i newline' file.txt`
   it will insert a new line before every line in which the pattern matches
   * ` sed '/8/ i hello' file.txt = will insert hello before wherever it see the number 8 in the file.txt 

* ' sed ' 3 a newline' file.txt `
   it will append a new line after the 3rd line
   * `sed '3 a hello' file.txt` = will append hello after line 3 in file.txt

## AWK command

awk works on programs that contain rules comprised of patterns and actions. The action is executed on the text that matches the pattern. Patterns are enclosed in curly braces ({}). Together, a pattern and an action form a rule. The entire awk program is enclosed in single quotes (').


* `who | awk '{print $1,$4}' `
   This will print out the first 4 fields from the who command 
  
speical field identifiers:
* ` $0 ` represents the entire line of text
* ` $1 ` represents the first line of text
* ` $34 ` represents the 34th line of text
* ` $NF ` represents the last field
 

Input Field separators :

If you want awk to work with text that doesn’t use whitespace to separate fields, you have to tell it which character the text uses as the field separator. For example, the /etc/passwd file uses a colon (:) to separate fields.

We would need to use that file and -F(separator string) option to tell awk to use the colon as the separator.

Example:
* `awk -F: '{print $1,$6}' /etc/passwd` - this output contains the name of the user account and the home folder 

Awk pre-processing:

If you need to create a title or a header for your result or so. You can use the BEGIN keyword to achieve this. It runs before processing the data.

Example:
* `awk 'BEGIN {print "Dennis Ritchie"} {print $0}' dennis_ritchie.txt `
 
## Archive,compress and uncompress files
 
* `tar` Save many files into a single file

  File permissions are maintained by default only for file users. For other user I must explicit say to maintain permission during decompression using `-p` parameter

  * `tar jcfv file.tar.bz2 *`

    Save all files of current directory in new bzip2 compressed file called file.tar.bz2

  * `tar jxfv file.tar.bz2`

    Extract content of file.tar.bz2
 
 * `tar rvf file.tar xyz.txt *`
    Add xyz.txt to the existing tar file. 
    You cannot add file to a bzip using the tar command
 
 * `tar xvf file.tar --wildcards '*.php'*`
    extract a php files from tar
 
 * ` tar zxvf file.tar.gz --wildcards '*.php'`
    extract a php files from gzip
 
 * ` tar jxvf file.tar.gz --wildcards '*.php'`
    extract a php files from bzip
   
 * ` tar xvf file.tar -C /home/user `
    extract the tar file into the home user folder
