---
toc: true 
toc_label: 'Contents' 
toc_icon: 'cog'
title: Mysql Database Moving Data Directory to New Location.
category: ['Linux', 'Mysql', 'Centos']
tags: ['mysql', 'centos', 'linux']
---

How to move an existing `data` directory in mysql to a new location. We were running out of space and had to move the existing `data` directory to a new drive. Below are the steps to move the `data` directory to new location.

Stop `mysql` using the following command:

    sudo service mysqld stop

Copy the existing data directory (default located in `/var/lib/mysql`) using the following command:

	sudo mkdir /db 
    sudo cp -R -p /var/lib/mysql /db

Update the `mysql` configuration file with new path `/db/mysql` the following command:

    sudo vim /etc/my.cnf
    
Here is how the configuration looks like.
    
    [ahmed@localhost mysql]$ cat /etc/my.cnf 
    [mysqld]
    datadir=/db/mysql
    socket=/db/mysql/mysql.sock
    user=mysql
    #  Disabling symbolic-links is recommended to prevent assorted security risks
    symbolic-links=0

    [mysqld_safe]
    log-error=/var/log/mysqld.log
    pid-file=/var/run/mysqld/mysqld.pid
    [ahmed@localhost mysql]$ 


Look for the entry for datadir, and change the path (which should be `/var/lib/mysql`) to the new data `/db/mysql` directory.

In the terminal, enter the command:

    sudo vim /etc/rc.d/init.d/mysqld

Look for lines beginning with `/var/lib/mysql`. Change `/var/lib/mysql` in the lines with the new path `/db/mysql`. Save and close the file.

    51 get_mysql_option mysqld datadir "/db/mysql"
    52 datadir="$result"
    53 echo $datadir
    54 get_mysql_option mysqld socket "$datadir/mysql.sock"
    55 socketfile="$result"
    56 get_mysql_option mysqld_safe log-error "/var/log/mysqld.log"
    57 errlogfile="$result"
    58 get_mysql_option mysqld_safe pid-file "/var/run/mysqld/mysqld.pid"
    59 mypidfile="$result"

Move the existing `mysql` directory 

	sudo mv /var/lib/mysql /var/lib/mysql_old
	
Create a symbolic link to the new path.

	cd /var/lib
	sudo ln -s /db/mysql mysql
   
Restart `mysql` with the command:

    sudo service mysqld restart

Now login to `mysql` and you can access the same databases you had before.