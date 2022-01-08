# User and Group Management

## Create, delete, and modify local user accounts

useradd

* Add users

*  `useradd -D` print the default configuration used by useradd command

  ```bash
  GROUP=100
  HOME=/home
  INACTIVE=-1
  EXPIRE=
  SHELL=/bin/bash
  SKEL=/etc/skel
  CREATE_MAIL_SPOOL=yes
  ```

  *GROUP=100* -> default group

  *HOME=/home* -> base for home directory

  *INACTIVE=-1* -> user password won't expire

  *EXPIRE=* -> user account won't expire

  *SHELL=/bin/bash* -> default shell

  *SKEL=/etc/skel* -> skeleton directory. <u>It's content will be copied in new user home directory</u>

  *CREATE_MAIL_SPOOL=yes* -> User will have a mail spool to receive email

* This configuration is saved in  `/etc/default/useradd`

* Also  `/etc/login.defs` parameter are evaluated during user add

* Some parameter of `/etc/login.defs` will overwrite `/etc/default/useradd` parameters

* `/etc/login.defs` contains:

  * Location of mail spool
  * Settings about password
  * *CREATE_HOME   yes* -> create home directory
  * *USERGROUPS_ENAB  yes* -> means that a group with same name of user must be created. This group will become default user group. This means that value of GROUP in `/etc/default/useradd` is overwritten

* `useradd` parameters:

  * `-c` Any text string. It is generally a short description of the login, and is currently used as the field for the user's full name.
  * `-e` date after which the/ user will be disabled
  * `-g` primary group. NOTE: if not specified it will be created a new group with same name of user that will be become user's primary group
  * `-G` secondary groups
  * `-m` create home directory. Useless because CREATE_HOME is yes
  * `-p` configure password. **NOTE**: value must be provided encrypted
    * Normally password is not provided during user add
  * `-s` shell to use
  
* examples useradd command :

*` useradd tecmint` This will add a new user called 'tecmint'
* `passwd tecmint` this will unlock the user tecmint and will set a password
* `useradd -m  tecmin` this will create a  home directory for tecmint and user account -m is set to /home in the /etc/defaults/useradd config
* `userradd -m -d /var/www tecmint` this will create a new home directory specified
* `useradd -M tecmint` this will create no home directory for tecmint
* `useradd -e 2022-06-27 tecmint` this will create a expiry date on the user tecmint account **NOTE** you can verify the change by running `chage -l tecmint`
* `useradd -e 2022-06-27 -f 45 tecmint` this will create a expiry date on the user + a password expiry for 45 days
* `useradd -s /bin/zsh tecmint` this will create a user login shell zsh for tecmint
* `useradd -s /bin/false -c "no shell" tecmint` this will create a no shell for tecmint and a comment is added
  

* examples usermod command:

* ` usermod -aG sudo tecmint` this will add tecmint to the sudo group.
* ` usermod -d /var/www/ tecmint` this will change tecmint home directory
* ` usermod -g test tecmint` this will set test as the primary directory for tecmint

* When a user is created two file will be changed:

  * `/etc/passwd` It contains users information, no passwords
    * Syntax:  
      * user name
      * x: means that password isn't stored here
      * userid: user id (UID) 
      * groupid: primary group id (GID)
      * User Info: The comment field
      * home: home directory
      * shell: shell
    * To edit file: `vipw`
  * `/etc/shadow` It contains passwords plus passwords properties
    * To edit file: `vipw -s`



usermod

* used to modify a user
* `usermod` parameters:
  * `-L`lock user password
  * `-U` unlock user password
* `usermod -e 1 user` disable user
* `usermod -e "" user` enable user



userdel

* remove user
* `userdel -r user`
  * `-r` remove home and email spool. **NOTE**: if it won't be used, if it will be tried to insert same user, there will be a conflict
  * `-f` force. Delete user though he is logged



passwd

* Change password of current user
* `passwd user`
  * Used by root
  * Change password of user
* `passwd -l user`
  * Used by root
  * Lock password of user
* `echo newpass | passwd --stdin brenda`
  * It will change password of brenda
  * Can be used in a script
  * **NOTE**: Dangerous, password is in clear text



chage

* Change user password expiry information
* If used without parameters will prompt for information
* It will permit to change date when the password was last changed
* `chage -E 2014-09-11 user`
  * Set a date after which user will be locked
