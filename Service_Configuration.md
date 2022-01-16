## Configure SSH servers and clients

* `/etc/ssh/sshd_config` ssh server configuration file
  * `PermitRootLogin no` Disable `root` login with ssh client
  * `PasswordAuthenticaion no` Disable login with password. This means that only login with public and private keys is allowed
* `/etc/ssh/ssh_config` ssh client configuration file
  * `ForwardX11 yes` allows use of X11 Server with ssh



Server management

* `systemctl status sshd` to control ssh server status
* `systemctl stop sshd` stop ssh server
* `systemct start sshd` start ssh server
* `systemctl restart sshd` restart ssh server
  * It must be executed each time configuration file will be changed
* `systemctl disable sshd` disable the ssh server start at boot
* `systemctl enable sshd` enable the ssh server start at boot



Client commands

* `ssh 129.123.123.123 ` it try to connect current user to an ssh server located on 192.123.123.123
* `ssh root@129.123.123.123 ` it try to connect root user to an ssh server located on 192.123.123.123
* `ssh -X root@129.123.123.123 ` 
  * `-X` enable X11 forwarding. This means that graphical application can be started
  * NOTE: It must be allowed on client configuration file as well.

* First time that an ssh connection is established with a server, the server will send a public key that it is used to verify its identity.
* The server public key is stored in the user's home inside file`.ssh/know_hosts`
  * E.g. `/home/user/.ssh/know_hosts`



Authentication with public/private keys

* On the ssh client machine a couple of ssh public/private keys can be generated using `ssh-keygen`
* The keys will be stored in the user's home inside directory `.ssh`
  * `id_rsa` private key
  * `id_rsa.pub` public key
* `ssh-copy-id 123.123.123.123` it is used to copy current user public key to home directory of same user on ssh server. The key will be stored in the user's home inside file `.ssh/authorized_keys`

* After that public key is copied on the server, user can use ssh client to login into the server without providing password



scp

* Secure copy. It use ssh to copy file on a server
* `scp /test/source 123.123.123.123:/dest` It will copy local file /test/source in /dest directory on the server 123.123.123.123
* `scp 123.123.123.123:/source /dest` It will copy source file from server to local directory dest



## Restrict access to the HTTP proxy server

* To enable the use of a proxy server environment variable `http_proxy` must be configured
  * `export http_proxy=http://127.0.0.1:3128/` use a local proxy listening on port 3128
  *  `export http_proxy=http://username:password@192.168.0.1:8080/` use a remote proxy on server 192.168.0.1, listening on port 8080 that require user and password
* `unset http_proxy` Disable use of proxy

* The keep configuration permanent for all user insert variable configuration in `/etc/environment` 

## Configure an IMAP and IMAPS service

* Server used to manage IMAP protocol is dovecot

  * `apt install dovecot`

* Basic configuration

  * `/etc/dovecot/dovecot.conf`

    * `protocols = imap pop3`

      This will enable imap and pop3 protocol

  * `/etc/dovecot/conf.d/10-mail.conf`

    * `mail_location = maildir:~/Maildir`

      This indicate to server where is located mail file

  * `/etc/dovecot/conf.d/10-ssl.conf`

    * Nothing to change, default configuration will enable ssl version of protocols that are enable in `dovecot.conf`

## Query and modify the behavior of system services at various operating modes

* `/usr/lib/systemd/system` contain unit file *.service* used by systemctl to start various service
* `/etc/systemd/system` can contain unit file that "override" the files contained in /usr/lib/systemd/system. If a unit file for a service is present in this directory, it will be used in substitution of file present in /usr.
* The correct way to permanently alter a start property of a service is to copy original file from `/usr/lib/systemd/system` to `/etc/systemd/system` and modify copy
* From the output of `system status service` it is possible to find from which file service was start`ed`
  * `Loaded` show the name of .service file used
* Under `[install]` session, voice `WantedBy` indicates for which target service is required
* When a service is enabled, a symbolic link to file `.service` of service will be created in `/etc/systemd/system/targetname.target.wants` where *targetname*  is the name of target for which service is required



* Some service properties can be changed at runtime

  * `systemctl set-property httpd.service MemoryLimit=500M`

    Command will change property and will create a file in `/etc/systemd/system` for future boot

  * `system status service` will show

    * `Loaded` will show the name of .service file used

    * `Drop-in` will show the change in `/etc/systemd`


* `systemctl list-dependencies service` It will show service dependencies

## Configure an HTTP server

* Used server: Apache HTTP Server
* `apt install apache2` will install server
* `systemctl start apache2` will start server
* `/etc/apache2/conf/apache2.conf` is the principal configuration file
  * `ServerName localhost` contains the local server name. 
    * **NOTE**: it must correspond to an IP. Simple solution is to modify /etc/hosts to insert a name-IP mapping
* Virtual host can be created inserting a file *.conf* in `/etc/apache2/conf.d/`
  * E.g. `/etc/apache/conf.d/file.conf`
* The file structure can be copied from  `/usr/share/doc/httpd-2.4.6/httpd-vhosts.conf`
  * **NOTE**: The version depends by server version installed
* Normally as *DocumentRoot*, directory that will contain site's files, it will be used a directory in `/var/www`
* `sudo mkdir -p /var/www/music.com && sudo nano /var/www/music.com/index.html` - this will create the folder and file for the music.com web pag. Do not forgot to give the directory a 755 permisson /var/ww.
* to create the virtual host for new sites . Copy the default.conf file from the sites-available folder and create a new one for the new host.
* run ` sudo apache2ctl configtest ` to test for config errors 
* run ` sudo a2ensite your_domain.conf ` to enable the conf file in the sites-available folder.
* run ` sudo a2dissite 000-default.conf ` to disable the default sited defined.

## Configure HTTP server log files

* E.g.

  ```bash
  ErrorLog /var/log/httpd/example.com_error_log
  LogFormat %s %v combined
  CustomLog /var/log/httpd/example.com_access_log combined
  ```

  * This will generate store Error log in /var/log/httpd/example.com_error_log

  * Plus will generate a log with a custom format in /var/log/httpd/example.com_access_log

* Normally log are stored in /var/log/httpd



* `yum -y install httpd-manual` will install httpd manuals
* Manuals are in http format
* In `/usr/share/httpd/manual/vhosts` are stored manual for vhost

## Configure a database server

* Used database: MariaDB
* `apt install mariadb-client mariadb-server` will install database
* `systemctl start mariadb` will start database
* `mysql -u root -p` will connect to database as root database user
  * Default password is blank
* `mysql_secure_installation` improves MariaDB security
  * It will permit to configure root password

## Restrict access to a web page

* Edit `/etc/httpd/conf/apache2.conf` and change

  ```bash
  <Directory "/var/www">
  	AllowOverride All
  ```

* In subdirectory of `/var/www` where site pages are contained create a file `.htaccess` whit follow content:

  ```bash
  Order Deny, Allow
  Deny from 192.168.3.1
  ```

  This will deny accesso to pages from IP 192.168.3.1 and allow access from all other IPs

* Alternatively:

  ```
  Order Allow, Deny
  Allow from 192.168.3.1
  ```

   This will allow access to pages from IP 192.168.3.1 and deny access from all other IPs

## Manage and configure containers

* Concepts:
  * *Images*: Read only template used to create container.
  * *Container*: Isolated application platform, it contains all the need to execute application



* `apt install docker` It will install docker
* `systemctl start docker`It start docker
* `docker version` to test if docker is working properly
* `usermod -aG dockerroot user`
  * This will enable *user* to use docker
* `docker search java`
  * Search java image in docker hub
* `docker images`
  * List local images
* Run container, examples:
  * `docker run busybox ls`
  * `docker run busybox echo "hello"`
  * `docker run centos:7 ping 127.0.0.1`
* `docker run -i -t centos:7 bash`
  * Run container with terminal
  * `-i` connects standard input to container
  * `-t` get pseudo terminal
  *  **NOTA**: `ctrl+p+q` exit form terminal without terminate container execution
* `docker run -d centos:7 ping 127.0.0.1`
  * Container will be executed in detached mode. This means that is in execution in background and not attached to Bash shell
* `docker ps -a`
  * List all container
  * `-a` show container stopped as well
* `docker attach containername`
  * Attach to container in detached mode
* `docker logs containername`
  * Show logs of a container
* `docker run -d  -P nginx`
  * Map container ports to host ports
  * **NOTE**: *firewalld* must be enable and running
* `docker run -d -P --restart always nginx`
  * This container will be restarted at bootstrap if the guest host will be restarted
* `docker update --restart=no containername`
  * Disable auto restart at bootstrap
* Stop container:
  * `docker stop containername`
  * `docker kill containername` forced stop
* `docker start name`
  * Restart a stopped container
* `docker rm containername`
  * Remove a container
  * **NOTE**: It must be stopped
* `docker rmi imageid`
  * Remove local image
* `docker diff containername`
  * List differences between container and original images. E.g. Some software can be installed in running container
* `docker commit containername`
  * Create a new image using based on the content of current running container. E.g It will contain software that was installed in container

