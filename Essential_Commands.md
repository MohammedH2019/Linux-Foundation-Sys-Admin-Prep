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
