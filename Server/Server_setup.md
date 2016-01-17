# Ubuntu Server setup

This is all the work that I will do for a fresh ubuntu server setup. If you find anything wrong or strange, feel free to send me pull requests or submit issues. I will try my best to merge and discuss with you. 

## [User accounts setup and securing them](https://www.youtube.com/watch?v=EuIYabZS3ow)

1. Setup a not-root user (with superuser privileges)
    * login using `ssh root@[server IP]`
    * add new user using `adduser [username]`, and use `su root` to switch to `root` and do superuser jobs
    * Give [username] privileges to upload to `/var/www` by running
    ```
    sudo adduser <username> www-data
    sudo chown -R www-data:www-data /var/www
    sudo chmod -R g+rwX /var/www
    ```
    * *(optional)* give [username] sudo privileges using `gpasswd -a [username] sudo`
        * remove user from user group `sudo gpasswd -d [username] [group]`
        * look up user group `groups [username]`

2. Disable remote root login
    * go to `vim /etc/ssh/sshd_config`, and search for `/PermitRootLogin`. Change `PermitRootLogin` to `no` and save it.
    * run `service ssh restart`
    * Logout of `root`
3. Public key authentication
    * Login to `[username]`
    * Generate SSH key on **local machine** using `ssh-keygen -t rsa -b 4096 -C "your_email@example.com"`
    * `cat .ssh/id_rsa.pub` or `more .ssh/id_rsa.pub`
    * Run `mkdir .ssh` on [username], and restrict its permission by running `chmod 700 .ssh`.
    * Add local machine's public key to [username]'s `vim .ssh/authorized_keys`
4. Disable password authentication
    * Switch to `root` user
    * run `vim /etc/ssh/sshd_config`
    * search for `PasswordAuth`
    * change `#PasswordAuthentication yes` to `PasswordAuthentication no`
    * run `service ssh restart`

## Connection protection service

1. Cloudflare
    * Manually setup A to point to the IP address(@), and CNAME www(@)
    * Run throught all security service setup
2. [Let's encrypt](https://letsencrypt.readthedocs.org/en/latest/using.html#installation)
    * If you are using Cloudflare, please read [this article](https://bepsvpt.wordpress.com)
    * Otherwise, do
    ```
    git clone https://github.com/letsencrypt/letsencrypt

    cd letsencrypt

    ./letsencrypt-auto
    ```

## Apache

### Installation

* `sudo apt-get update && sudo apt-get install apache2`
    * Use `ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'` to check server IP address

###  Important configuration
  * apache configuration directory listing off
    * The best way to do this is disable it with webserver apache2. In my Ubuntu 14.X - open `/etc/apache2/apache2.conf`, change from
    ```
    <Directory /var/www/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>
    ```
    to
    ```
    <Directory /var/www/>
        Options FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>
    ```
    then restart apache by `sudo service apache2 restart`
    * [Hide Apache Version and OS Identity from Errors](http://www.tecmint.com/apache-security-tips/)
      * run
        ```
        # vim /etc/httpd/conf/httpd.conf (RHEL/CentOS/Fedora)
        # vim /etc/apache/apache2.conf (Debian/Ubuntu)
        ```
      * Add
        ```
        ServerSignature Off
        ServerTokens ProductOnly
        ```
      * restart apache by `sudo service apache2 restart`

### Possible issues

* [Could not reliably determine the servers fully qualified domain name](http://askubuntu.com/questions/256013/could-not-reliably-determine-the-servers-fully-qualified-domain-name)
  * add `ServerName localhost` to `apache2.conf` in `/etc/apache2`then restart apache by `sudo service apache2 restart`

### Notes on [Local testing](http://askubuntu.com/questions/92069/how-to-add-custom-directory-e-g-phpmyadmin)
  * Simply grant current user the root permission to edit `/var/www/` folder

## PHP5

### Installation

* `sudo apt-get install php5 libapache2-mod-php5 php5-mcrypt`
* `sudo php5enmod mcrypt`
* Go to `sudo vim /etc/apache2/mods-enabled/dir.conf`, and move `index.php` to the front of `index.html`. Then, `sudo service apache2 restart`

### Possible problem

* [Headers and client library minor version mismatch](http://stackoverflow.com/questions/10759334/headers-and-client-library-minor-version-mismatch)
  * install `sudo apt-get install php5-mysqlnd` and restart apache2

### Error log location

* `/var/log/apache2/error.log` or `/var/log/httpd/error_log`

## [Composer](https://getcomposer.org/doc/00-intro.md)

You can start with this [good video tutorial](https://www.youtube.com/watch?v=FKUAAZSJiGY).

### Installation

* Php must be installed first.
* Run these commands to globally install composer on Linux/Unix/OSX
```
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
```

# [SWAP](https://www.digitalocean.com/community/tutorials/additional-recommended-steps-for-new-ubuntu-14-04-servers)

## One-liner

`sudo fallocate -l 1G /swapfile && sudo chmod 600 /swapfile && sudo mkswap /swapfile && sudo swapon /swapfile && sudo sh -c 'echo "/swapfile none swap sw 0 0" >> /etc/fstab'`

## Create a Swap File Explained

Adding "swap" to a Linux server allows the system to move the less frequently accessed information of a running program from RAM to a location on disk. Accessing data stored on disk is much slower than accessing RAM, but having swap available can often be the difference between your application staying alive and crashing. This is especially useful if you plan to host any databases on your system.

Advice about the best size for a swap space varies significantly depending on the source consulted. Generally, an amount equal to or double the amount of RAM on your system is a good starting point.

Allocate the space you want to use for your swap file using the `fallocate` utility. For example, if we need a 4 Gigabyte file, we can create a swap file located at `/swapfile` by typing: `sudo fallocate -l 1G /swapfile`.

After creating the file, we need to restrict access to the file so that other users or processes cannot see what is written there: `sudo chmod 600 /swapfile`.

We now have a file with the correct permissions. To tell our system to format the file for swap, we can type: `sudo mkswap /swapfile`

Now, tell the system it can use the swap file by typing: `sudo swapon /swapfile`

Our system is using the swap file for this session, but we need to modify a system file so that our server will do this automatically at boot. You can do this by typing: `sudo sh -c 'echo "/swapfile none swap sw 0 0" >> /etc/fstab'`. With this addition, your system should use your swap file automatically at each boot.

# [ufw](https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server)

## Installation

* `sudo apt-get install ufw`
* `sudo ufw status`
* If your VPS is configured for IPv6, ensure that UFW is configured to support IPv6 so that will configure both your IPv4 and IPv6 firewall rules. To do this, open the UFW configuration with this command: `sudo vi /etc/default/ufw`. Then make sure "IPV6" is set to "yes", like so:`IPV6=yes` Save and quit. Then restart your firewall with the following commands: `sudo ufw disable`
`sudo ufw enable`

# ntp

* `sudo dpkg-reconfigure tzdata`
*  select time zone
* `sudo apt-get update`
* `sudo apt-get install ntp`

## Setup

* `sudo ufw default deny incoming`
* `sudo ufw default allow outgoing`
* `sudo ufw allow ssh` (22/tcp)
* `sudo ufw allow www` (80/tcp)
* `sudo ufw allow ntp` (123/tcp)
* `sudo ufw allow sftp`
* `sudo ufw allow 8834/tcp` (nessus)
* if needed
  * `sudo ufw allow 443/tcp` (SSL/TLS)

# Notes

## [Linux file system permission](http://superuser.com/questions/295591/what-is-the-meaning-of-chmod-666)

```
permission to:  owner      group      other     
                /¯¯¯\      /¯¯¯\      /¯¯¯\
octal:            6          6          6
binary:         1 1 0      1 1 0      1 1 0
what to permit: r w x      r w x      r w x

binary         - 1: enabled, 0: disabled

what to permit - r: read, w: write, x: execute

permission to  - owner: the user that create the file/folder
                 group: the users from group that owner is member
                 other: all other users
```

## [DNS](https://support.dnsimple.com/articles/differences-between-a-cname-alias-url/)

```
Here’s the main differences:

The A record maps a name to one or more IP addresses, when the IP are known and stable.

The CNAME record maps a name to another name. It should only be used when there are no other records on that name.

The ALIAS record maps a name to another name, but in turns it can coexist with other records on that name.

The URL record redirects the name to the target name using the HTTP 301 status code.


Some important rules to keep in mind:

The A, CNAME, ALIAS records causes a name to resolve to an IP. Vice-versa, the URL record redirects the name to a destination. The URL record is simple and effective way to apply a redirect for a name to another name, for example to redirect www.example.com to example.com. The A name must resolve to an IP, the CNAME and ALIAS record must point to a name.
```

# Old server setup notes

## MariaDB

### [Installation](https://downloads.mariadb.org/mariadb/repositories/#mirror=opencas&distro=Ubuntu&distro_release=trusty--ubuntu_trusty&version=10.1)

* `sudo apt-get install software-properties-common`
* `sudo apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xcbcb082a1bb943db`
* `sudo add-apt-repository 'deb [arch=amd64,i386] http://mirrors.opencas.cn/mariadb/repo/10.1/ubuntu trusty main'`
* `sudo apt-get update`
* `sudo apt-get install mariadb-server`
* `sudo service mysql stop`
* `sudo mysql_install_db`

### [Secure MariaDB](https://mariadb.com/kb/en/mariadb/mysql_secure_installation/)

* `sudo service mysql start`
* run `mysql_secure_installation`
* rename `root` user
  * Method 1:
    * login to mysql using `mysql -u [rootusername] -p`
    * run `rename user 'root'@'localhost' to 'newAdminUser'@'localhost';`
    * check by running `select user,host,password from mysql.user;`
    * flush privileges for these changes to happen: `FLUSH PRIVILEGES;`
  * Method 2 (preferred):
    * login to mysql using `mysql -u [rootusername] -p`
    * run `use mysql; update user set user='admin' where user='root'; flush privileges;`
    * If you also want to change password, run `update user set password=PASSWORD('new password') where user='admin';` before `flush privileges;`

## phpmyadmin

### Installation

* `sudo apt-get install phpmyadmin apache2-utils`

### [Important config](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-phpmyadmin-on-ubuntu-12-04)
  * [Change access url](http://www.thetechrepo.com/main-articles/488) from `phpmyadmin` to something else
    * `vim /etc/phpmyadmin/apache.conf`
    * `Alias /[whatever the name is] /usr/share/phpmyadmin`
    * `sudo service apache2 restart`
  * change to utf8_general

### Possible problem

* [Can't open phpmyadmin](http://stackoverflow.com/questions/10111455/i-cant-access-http-localhost-phpmyadmin)
  * `vim /etc/apache2/apache2.conf`
  * Add the phpmyadmin config to the file: `Include /etc/phpmyadmin/apache.conf`
  * `sudo service apache2 restart`