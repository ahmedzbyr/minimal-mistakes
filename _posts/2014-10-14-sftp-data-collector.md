---
title: SFTP Data Collector
category: ['Linux']
tags: ['sftp', 'ftp', 'data-collector', 'linux', 'python', 'script', 'github']
---

Easy way to collect files recursively over a `sftp` server is to connect to the server over scp and do `scp -r`. 
Problem was that the device we were connecting to did not support recursive :( over a regEx expression.

####  Example:

    scp -r ahmed@remote-host:/home/ahmed/*file_123*

Was not working. Here is a simple SFTP Data Collector script.
This script can be used, if the source device/server is unable to `get` file recursively.

###  Step in this script:
1. `get` listing of the files present.
2. select files required from the list using a reg_ex or a pattern.
3. Download select files. Below is the command usage.

###  Usage:

    usage: sftp_data_collector.py [-h] -sh SRC_HOST_NAME -su SRC_USERNAME
                                  (-sp SRC_PASSWORD | -es) -dh DEST_HOST_NAME -du
                                  DEST_USERNAME (-dp DEST_PASSWORD | -ed)
                                  [-c SRC_DIRECTORY] [-y DEST_DIRECTORY]
                                  (-t YYYYMMDD_HH | -p PATTERN_IN_FILE) [-d]
                                  [--version]
    
    SFTP Data Collector.
    ----------------------
    
    This script can be used, if the source device/server is unable to "get" file recursively.
    To use password from environment, set values for :
    
        PASS_SRC for source.
        PASS_DEST for destination.
    
    Steps in this script:
    
        1. "get" listing of the files present.
        2. select files required from the list using a reg_ex or a pattern. 
                    (currently this is yyyymmdd_hh)
        3. Download select files.
    
    ----------------------
    
    optional arguments:
      -h, --help            show this help message and exit
      -sh SRC_HOST_NAME, --src-host-name SRC_HOST_NAME
                            Source Host name to get Files from.
      -su SRC_USERNAME, --src-username SRC_USERNAME
                            Source Host - Username.
      -sp SRC_PASSWORD, --src-password SRC_PASSWORD
                            Source Host - Password.
      -es, --src-env-password
                            Source Host - Password. Pick From Environment Variable
                            PASS_SRC
      -dh DEST_HOST_NAME, --dest-host-name DEST_HOST_NAME
                            Destination Host name to send Files to.
      -du DEST_USERNAME, --dest-username DEST_USERNAME
                            Destination Host - Username.
      -dp DEST_PASSWORD, --dest-password DEST_PASSWORD
                            Destination Host - Password.
      -ed, --dest-env-password
                            Destination Host - Password. Pick From Environment
                            Variable PASS_DEST
      -c SRC_DIRECTORY, --cd-src-directory SRC_DIRECTORY
                            Source Directory, If not provided then "."
      -y DEST_DIRECTORY, --cd-dest-directory DEST_DIRECTORY
                            Destination Directory, If not provided then "."
      -t YYYYMMDD_HH, --date-hour YYYYMMDD_HH
                            Enter date_hour in yyyymmdd_hh format,File has
                            date_hour pattern in the filename
      -p PATTERN_IN_FILE, --pattern-in-file PATTERN_IN_FILE
                            Enter pattern in filename which needs to be collected
                            from sFTP server.
      -d, --debug           Running Debug mode - More Verbose
      --version             show program's version number and exit
      

###  Code Usage:

Call function with parameters. Might want to do a check for the pattern here.

    sftp_data_collector.get_file_from_src (src_host_name, dest_host_name, src_username, src_passwd,
                          dest_username, dest_passwd, pattern,
                          cd_src_directory_args=current_dir_src, 
                          cd_dest_directory_args=current_dir_dest)
                          
###  Code Location:
Code can be found on github : <https://github.com/zubayr/sftp_data_collector>