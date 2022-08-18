# Operation of Running Systems

## Boot, reboot, and shut down a system safely

* `shutdown -h now` shutdown
* `shutdown -r now` reboot

## Boot or change system into different operating modes

Boot sequence:

* POST (PowerOn Self Test) -> Find disk -> Inside disk there's bootloader -> bootloader load kernel -> kernel load init process

* Systemd is the default init process in CentOS
* Systemd starts services. Last service started will be a shell



Systemd

* Previous versions of Red Hat Enterprise Linux, which were distributed with SysV init or Upstart, implemented a predefined set of runlevels that represented specific modes of operation. These runlevels were numbered from 0 to 6 and were defined by a selection of system services to be run when a particular runlevel was enabled by the system administrator. In CentOS and Red Hat Enterprise Linux 7, the concept of runlevels has been replaced with systemd targets.

* Systemd targets are represented by target units. Target units end with the .target file extension and their only purpose is to group together other systemd units through a chain of dependencies. 

* Systemd units are the objects that systemd knows how to manage. These are basically a standardized representation of system resources that can be managed by the suite of daemons and manipulated by the provided utilities.

* Systemd units in some ways can be said to similar to services or jobs in other init systems. However, a unit has a much broader definition, as these can be used to abstract services, network resources, devices, filesystem mounts, and isolated resource pools.

* Systemd was designed to allow for better handling of dependencies and have the ability to handle more work in parallel at system startup.



Systemd commands:

* `systemctl get-default`

  It shows default target

* `systemctl list-units --type target --all`

  It shows all available targets

* `systemctl set-default multi-user.target`

  Set multi-user target as default



Change target at boot time

* If during boot ESC is pressed the grub2 prompt will be showed

* Highlight a voice and press 'e'

* Now is it possible to modify the boot parameter used to load the kernel. 

  **NOTE**: the changes are not persistent

  E.g `systemd.unit=emergency.target` can be added to boot system in emergency mode. NOTE: in this modality disk is mounted read only, to mount it read/write, after boot execute `mount`
  `-o remount,rw /`

* When the parameter change is end, press 'Ctrl + x' to boot system



References:

* [https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/sect-managing_services_with_systemd-targets](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/sect-managing_services_with_systemd-targets)
* [https://en.wikipedia.org/wiki/Power-on_self-test](https://en.wikipedia.org/wiki/Power-on_self-test)
* [https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files](https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files))

## Install, configure and troubleshoot bootloaders

* The default bootloader is Grub2.

* The to change bootloader configuration edit /etc/default/grub

  `vi /etc/default/grub`

* The configuration information can be found with:

  * `info -f grub -n 'Simple configuration'`

  * `man 7 bootparam` 

    It shows the kernel boot parameter

* check the firmware before compilation

  `ls -larth /sys/firmware`

* if its efi then

  `grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg`
        
   else

  `grub2-mkconfig -o /boot/grub2/grub.cfg`       

* if no errors during compilation then reboot otherwise kernel might enter panic state and wont reboot 

  `reboot now`


## Diagnose and manage processes

mpstat

* `yum -y install sysstat`

* `mpstat -P ALL -u 2 3`

  CPU usage statistics. 

  `-P` Indicate the processor number for which statistics are to be reported, ALL for all cpu

  `-u` Report CPU utilization

  `2 3` Display three reports at two second intervals.



ps

* `ps` Processes of which I'm owner

* `ps aux` All processes.

  It will print:

  * user - user owning the process

  * pid - process ID of the process
    * It is set when process start, this means that provide info on starting order of processes

  * %cpu - It is the CPU time used divided by the time the process has been running.

  * %mem - ratio of the process’s resident set size to the physical memory on the machine

  * VSZ (virtual memory) - virtual memory usage of entire process (in KiB)

  * RSS (resident memory) - resident set size, the non-swapped physical memory that a task has used (in KiB)

  * tty - On which process is running. 
    * **NOTE**: *?* means that isn't connect to a tty

  * stat - process state

  * start- starting time or date of the process

  * time - cumulative CPU time

  * command - command with all its arguments

    * Those within *[ ]* are system processes or kernel thread

* `ps -eo pid,ppid,cmd,%cpu,%mem --sort=-%cpu`

   `-e` show same result of `aux`

   `-o` chose columns to show

  `--sort` sort by provided parameter

  `ppid` parent process id

* `ps -e -o pid,args --forest`

  `--forest` show a graphical view of processes tree



* In /proc/[pid]

  There is a numerical subdirectory for each running process; the subdirectory is named by the process ID.

* /proc/[pid]/fd

  This is a subdirectory containing one entry for each file which the process has open, named by its file descriptor, and which is a symbolic link to the actual file.  Thus, 0 is standard input, 1 standard output, 2 standard error, and so on.



* `lsof -p pid`

  Lists open files associated with process id of pid



Background processes

* End a command with `&` execute a process in background

  `sleep 600 &`

* `jobs`

  List processes in background

* `fg pid` 

  To return a process in foreground 



Process priority

* `ps -e -o pid,nice,command`

  nice (NI) is the process priority

* More priorities and more CPU time will be assigned to process

* nice value can be between -20 and 90

* -20 is highest and 90 is lowest

* **NOTE**: only root can assign negative values

* `nice -n value command &`

  It will execute command in background with nice equal to value

* `renice` ri-assign priority to a process

  `renice -n value pid`



Signals

* `kill pid`

  Send a SIGTERM to process with pid equal to pid

* `kill -9 pid`

  Send a SIGKILL to process with pid equal to pid

* `kill -number pid`

  Send a signal that correspond to number to process with pid equal to pid

* `kill -l`

  List all available signal and corresponding number




References:

* [https://superuser.com/questions/117913/ps-aux-output-meaning](https://superuser.com/questions/117913/ps-aux-output-meaning)
* [http://man7.org/linux/man-pages/man5/proc.5.html](http://man7.org/linux/man-pages/man5/proc.5.html)


## Locate and analyze system log files

There are many different log files that all serve different purposes. When trying to find a log about something, you should start by identifying the most relevant file. Below is a list of common log file locations.

System Logs:
System logs deal with exactly that - the Ubuntu system - as opposed to extra applications added by the user. These logs may contain information about authorizations, system daemons and system messages.

# Authorization logs
Location : /var/log/auth.log
This keeps track of authorization systems , such as password prompt, the sudo command and the remote logins

# Daemon logs 
Location: /var/log/daemon.log
Daemons are programs that run in the background, usually without user interaction. For example , display server , SSH sessions, bluetooth and more.

#Debug log
Location: /var/log/debug
Provides debugging information from the ubuntu system and application

#kernal log 
Location: /var/log/kern.log
Logs from the linux kernal

#System log
Location: /var/log/syslog
Contains more information about the system. Great place to check for general system errors and messages

Some application also create logs in /var/log. 

#Apache logs
Location: /var/logs/apache2(subdirectory)
Apache creates several log files in this subdirectory. The access.log file records all request made to the server to access files. error.log reports all erros thrown 
by the server.




## Schedule tasks to run at a set date and time

* Daemon that schedule tasks, called jobs, to run at a set date and time is cron
* The schedule of various tasks depend by configuration contained in below files/directories:
  * /etc/crontab
    * Normally isn't edited
      * **NOTE**: It's content can be used as remainder of cron files syntax
    * Each row is a task that must be executed in a scheduled way
    * A special syntax indicates the schedule of each commands
  * /etc/cron.d
    * It contains files with same syntax of /etc/crontab
    * Normally it used by software packages installed in system
  * /var/spool/cron
    * It contains tasks for users
    * Contents can be edited using `crontab` command
  * /etc/cron.hourly
    * Each script in this directory will be executed every hour
    * Exact time isn't specified but execution is granted, with a combination of deamon cron and anacron
  * /etc/cron.daily
    * Each script in this directory will be executed every day
    * Exact time isn't specified but execution is granted, with a combination of deamon cron and anacron
  * /ect/cron.weekly
    * Each script in this directory will be executed every week
    * Exact time isn't specified but execution is granted, with a combination of deamon cron and anacron
  * /etc/cron.monthly
    * Each script in this directory will be executed every month
    * Exact time isn't specified but execution is granted, with a combination of deamon cron and anacron



To modify cron jobs:

* `crontab -e` It is used by user to modify his jobs
* `crontab -e -u user` It is used by root to modify user's jobs
system-wide cron file: /etc/crontab
Cron syntax:

```bash
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of the month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday;
# │ │ │ │ │                                   7 is also Sunday on some systems)
# │ │ │ │ │
# │ │ │ │ │
# * * * * * command to execute
```

* `#` this line is a comment
* `*` always
* `1 0 * * * /command` command will be executed one minute past midnight (00:01) every day
*  `1-30 * * * * /command` command will be executed every day, every hour at minutes 1 to 30
*  `*/10 * * * * /command` command will be executed every 10 minutes, or rather when minutes are 00, 10, 20, 30, 40 and 50.
*  `00 */2 15 * * /command` command will be executed the fifteenth day of every month, every two hours
*  `00 1-9/2 1 5 * /command` command will be executed on 1st May at 1,00 - 3,00 - 5,00 - 7,00 - 9,00, or rather every two hours from 1,00 to 9,00
*  `00 13 2,8,14 * * /command` command will be executed second, eighth and fourteenth day of each month at 13.00

#Verify completion of scheduled jobs
Check cron logs:
* `grep cron /var/log/syslogs`
check cron service:
* `service cron status`
* `sudo less /var/log/syslogs | grep -i cron`
