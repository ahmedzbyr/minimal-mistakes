---
title: Zabbix History Table Clean Up
category: ['Linux', 'Monitoring', 'Zabbix', 'Nagios', 'Nagiosxi']
tags: ['linux', 'monitoring', 'zabbix', 'nagios', 'nagiosxi']
---
Zabbix history table gets really big, and if you are in a situation where you want to clean it up.
Then we can do so, using the below steps. 

1. Stop zabbix server.
2. Take table backup - just in case.
3. Create a temporary table.
4. Update the temporary table with data required, upto a specific date using `epoch`.
5. Move old table to a different table name.
6. Move updated (new temporary) table to original table which needs to be cleaned-up.
7. Drop the old table. (Optional)
8. Restart Zabbix

Since this is not offical procedure, but it has worked for me so use it at your own risk.
Here is another post which will help is reducing the size of `history` tables - [http://zabbixzone.com/zabbix/history-and-trends/](http://zabbixzone.com/zabbix/history-and-trends/)

Zabbix Version : Zabbix v2.4
Make sure MySql 5.1 is set with InnoDB as `innodb_file_per_table=ON`

### Step 1 Stop the Zabbix server

``` ruby
sudo service zabbix-server stop
```

Script.

``` sh
echo "------------------------------------------"
echo "	1. Stopping Zabbix Server			"
echo "------------------------------------------"
sudo service zabbix-server stop; 
```

### Step 2 Table Table Backup.

``` bash
mysqldump -uzabbix -pzabbix zabbix history_uint > /tmp/history_uint.dql
```

Script.

``` sh
echo "------------------------------------------"
echo "	2. Backing up ${ZABBIX_TABLE_NAME} Table.	"
echo "	Location : ${BACKUP_FILE_PATH}		"
echo "------------------------------------------"
mkdir -p ${BACKUP_DIR_PATH}
mysqldump -u$ZABBIX_USER -p$ZABBIX_PASSWD $ZABBIX_DATABASE ${ZABBIX_TABLE_NAME} > ${BACKUP_FILE_PATH}
```

### Step 3 Open your favourite MySQL client and create a new table

``` sql
CREATE TABLE history_uint_new_20161007 LIKE history_uint;
```

Script.

``` sh
echo "------------------------------------------------------------------"
echo "	3. Create Temp (${ZABBIX_TABLE_NAME}_${EPOCH_NOW}) Table"
echo "------------------------------------------------------------------"
echo "CREATE TABLE ${ZABBIX_TABLE_NAME}_${EPOCH_NOW} LIKE ${ZABBIX_TABLE_NAME}; " | mysql -u$ZABBIX_USER -p$ZABBIX_PASSWD $ZABBIX_DATABASE;
```


### Step 4 Insert the latest records from the history_uint table to the history_uint_new table

Getting `epoch` time in `bash` is simple. 

Current Date.

``` bash
date --date "20160707" +%s
```


Date 3 Months Ago.


``` bash
date --date "20161007" +%s
```

Here is the output.

```
[ahmed@localhost ~]$ date --date "20160707" +%s
1467829800
[ahmed@localhost ~]$ date --date "20161007" +%s
1475778600
```

Now insert data for 3 months.  

``` sql
INSERT INTO history_uint_new SELECT * FROM history_uint WHERE clock > '1413763200';
```

Script. 

``` sh
echo "------------------------------------------------------------------"
echo "	4. Inserting from ${ZABBIX_TABLE_NAME} Table to Temp (${ZABBIX_TABLE_NAME}_${EPOCH_NOW}) Table"
echo "------------------------------------------------------------------"
echo "INSERT INTO ${ZABBIX_TABLE_NAME}_${EPOCH_NOW} SELECT * FROM ${ZABBIX_TABLE_NAME} WHERE clock > '${EPOCH_3MONTHS_BACK}'; " | mysql -u$ZABBIX_USER -p$ZABBIX_PASSWD $ZABBIX_DATABASE;
```


### Step 5 – Move `history_uint` to `history_uint_old` table

``` sql
ALTER TABLE history_uint RENAME history_uint_old;
```

Script.

``` sh
echo "------------------------------------------------------------------"
echo "	5. Rename Table ${ZABBIX_TABLE_NAME} to ${ZABBIX_TABLE_NAME}_${EPOCH_NOW}_old"
echo "------------------------------------------------------------------"
echo "ALTER TABLE ${ZABBIX_TABLE_NAME} RENAME ${ZABBIX_TABLE_NAME}_${EPOCH_NOW}_old;" | mysql -u$ZABBIX_USER -p$ZABBIX_PASSWD $ZABBIX_DATABASE;
```

### Step 6. Move newly created `history_uint_new` to `history_uint` 

``` sql
ALTER TABLE history_uint_new_20161007 RENAME history_uint;
```

Script.

``` sh
echo "------------------------------------------"
echo "	6. Rename Temp Table (${ZABBIX_TABLE_NAME}_${EPOCH_NOW}) to Original Table (${ZABBIX_TABLE_NAME})"
echo "------------------------------------------"
echo "ALTER TABLE ${ZABBIX_TABLE_NAME}_${EPOCH_NOW} RENAME ${ZABBIX_TABLE_NAME}; " | mysql -u$ZABBIX_USER -p$ZABBIX_PASSWD $ZABBIX_DATABASE;
```

### Step 7. [OPTIONAL] Remove Old Table. 

As we have backed-up the table we no long need it. So we can drop the old table.

``` sql
DROP TABLE hostory_uint_old;
```

Script.

``` sh
echo "------------------------------------------"
echo "	7. Dropping Old Table (${ZABBIX_TABLE_NAME}_${EPOCH_NOW}_old), As we have already Backed it up. "
echo "------------------------------------------"
echo "DROP TABLE ${ZABBIX_TABLE_NAME}_${EPOCH_NOW}_old; " | mysql -u$ZABBIX_USER -p$ZABBIX_PASSWD $ZABBIX_DATABASE;
```


### Step 8 – Start the Zabbix server

``` bash
sudo service zabbix-server start
```

Script.

``` sh
echo "------------------------------------------"
echo "	8. Starting Zabbix Server		"
echo "------------------------------------------"
sudo service zabbix-server start;
```



### Step 9. Optional to reduce the history table. 

Additionally you can update the items table and set the item history table record to a fewer days.

``` sql
UPDATE items SET history = '15' WHERE history > '30';
```    
   
   
## Complete Script.

Location in [Github](https://raw.githubusercontent.com/zubayr/zbx_snmptrap_templates_creation/master/zabbix_installer_script/backup_scripts/history_table_clean_up_db_script.sh)
   
``` sh

#!/bin/bash

THREE_MONTH_BACK_DATE=`date -d "now -3months" +%Y-%m-%d`
CURRENT_DATE=`date -d "now" +%Y-%m-%d`

EPOCH_3MONTHS_BACK=`date -d "$THREE_MONTH_BACK_DATE" +%s`
EPOCH_NOW=`date -d "$CURRENT_DATE" +%s`

ZABBIX_DATABASE="zabbix"
ZABBIX_USER="zabbix"
ZABBIX_PASSWD="zabbix"

ZABBIX_TABLE_NAME="history_uint"

BACKUP_DIR_PATH=/tmp/zabbix/zabbix_table_backup_${ZABBIX_TABLE_NAME}
BACKUP_FILE_PATH=${BACKUP_DIR_PATH}/${ZABBIX_TABLE_NAME}_${CURRENT_DATE}_${EPOCH_NOW}.sql

echo "------------------------------------------"
echo "Date to Keep Backup : $THREE_MONTH_BACK_DATE"
echo "Epoch to keep Backup : $EPOCH_3MONTHS_BACK"
echo "Today's Date : $CURRENT_DATE"
echo "Epoch For Today's Date : $EPOCH_NOW"
echo "------------------------------------------"

echo "##########################################"

echo "------------------------------------------"
echo "	1. Stopping Zabbix Server			"
echo "------------------------------------------"
sudo service zabbix-server stop; 
sleep 1

echo "------------------------------------------"
echo "	Display Tables				"
echo "------------------------------------------"
echo "show tables;" | mysql -u$ZABBIX_USER -p$ZABBIX_PASSWD $ZABBIX_DATABASE;
sleep 1

echo "------------------------------------------"
echo "	2. Backing up ${ZABBIX_TABLE_NAME} Table.	"
echo "	Location : ${BACKUP_FILE_PATH}		"
echo "------------------------------------------"
mkdir -p ${BACKUP_DIR_PATH}
mysqldump -u$ZABBIX_USER -p$ZABBIX_PASSWD $ZABBIX_DATABASE ${ZABBIX_TABLE_NAME} > ${BACKUP_FILE_PATH}
sleep 1

echo "------------------------------------------------------------------"
echo "	3. Create Temp (${ZABBIX_TABLE_NAME}_${EPOCH_NOW}) Table"
echo "------------------------------------------------------------------"
echo "CREATE TABLE ${ZABBIX_TABLE_NAME}_${EPOCH_NOW} LIKE ${ZABBIX_TABLE_NAME}; " | mysql -u$ZABBIX_USER -p$ZABBIX_PASSWD $ZABBIX_DATABASE;
sleep 1

echo "------------------------------------------------------------------"
echo "	4. Inserting from ${ZABBIX_TABLE_NAME} Table to Temp (${ZABBIX_TABLE_NAME}_${EPOCH_NOW}) Table"
echo "------------------------------------------------------------------"
echo "INSERT INTO ${ZABBIX_TABLE_NAME}_${EPOCH_NOW} SELECT * FROM ${ZABBIX_TABLE_NAME} WHERE clock > '${EPOCH_3MONTHS_BACK}'; " | mysql -u$ZABBIX_USER -p$ZABBIX_PASSWD $ZABBIX_DATABASE;
sleep 1

echo "------------------------------------------------------------------"
echo "	5. Rename Table ${ZABBIX_TABLE_NAME} to ${ZABBIX_TABLE_NAME}_${EPOCH_NOW}_old"
echo "------------------------------------------------------------------"
echo "ALTER TABLE ${ZABBIX_TABLE_NAME} RENAME ${ZABBIX_TABLE_NAME}_${EPOCH_NOW}_old;" | mysql -u$ZABBIX_USER -p$ZABBIX_PASSWD $ZABBIX_DATABASE;
sleep 1

echo "------------------------------------------"
echo "	6. Rename Temp Table (${ZABBIX_TABLE_NAME}_${EPOCH_NOW}) to Original Table (${ZABBIX_TABLE_NAME})"
echo "------------------------------------------"
echo "ALTER TABLE ${ZABBIX_TABLE_NAME}_${EPOCH_NOW} RENAME ${ZABBIX_TABLE_NAME}; " | mysql -u$ZABBIX_USER -p$ZABBIX_PASSWD $ZABBIX_DATABASE;
sleep 1

echo "------------------------------------------"
echo "	7. Dropping Old Table (${ZABBIX_TABLE_NAME}_${EPOCH_NOW}_old), As we have already Backed it up. "
echo "------------------------------------------"
echo "DROP TABLE ${ZABBIX_TABLE_NAME}_${EPOCH_NOW}_old; " | mysql -u$ZABBIX_USER -p$ZABBIX_PASSWD $ZABBIX_DATABASE;
sleep 1

echo "------------------------------------------"
echo "	8. Starting Zabbix Server		"
echo "------------------------------------------"
sudo service zabbix-server start;

echo "##########################################"
```