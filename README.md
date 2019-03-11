MySQL Partitions
=======================


Installation
------------

Run the attached SQL file into the database of your choice. Please make sure that event scheduling is enabled on the database server.

When you run this file: 

* Code to automatically manage partitions will be deployed.
* Settings table will be created to populate how you want to partition tables. 
* This table is empty for the first run.
* An event will be created with a default run time frequency of 24 hours.

This code has been successfully tested on Percona’s flavor of MySQL 5.5 and MySQL 5.6. It should work on previous versions too. We are currently testing it for MySQL 5.7 as well.

Usage
------------------

Choose a table that you’d like to partition.

The pre-requisite to partition a table is:

* The table should have a numeric time column storing either epoch unix format or a factored version of it e.g. hour
* The chosen column should be part of the primary key
* Insert a row into the partition manager settings table. 
* Table name is `partition_manager_settings`

The following code shows the settings table:

```sql
CREATE TABLE `partition_manager_settings` (
  `table` varchar(64) NOT NULL COMMENT 'table name',
  `column` varchar(64) NOT NULL COMMENT 'numeric column with time info',
  `granularity` int(10) unsigned NOT NULL COMMENT 'granularity of column, i.e. 1=seconds, 60=minutes...',
  `increment` int(10) unsigned NOT NULL COMMENT 'seconds per individual partition',
  `retain` int(10) unsigned NULL COMMENT 'seconds of data to retain, null for infinite',
  `buffer` int(10) unsigned NULL COMMENT 'seconds of empty future partitions to create',
  PRIMARY KEY (`table`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1 ROW_FORMAT=Dynamic;
```

Column Name | Definition
----------- | ----------
table       | table_name
column      | column that you’d like to partition
granularity | factoring granularity in seconds (1 denotes seconds, 60 denotes minutes, 3600 denotes hours)
increment   | Number of seconds per individual partition (86400 denotes 1 day)
retain      | Seconds worth of data to retain or null for infinite
buffer      | Seconds worth of empty feature partitions to maintain

Optionally, call `schedule_partition_manager` to update the partitioning schedule based on the new table. This is taken care of during the next run automatically, hence its optional.

Enable the event scheduler 
``SET GLOBAL event_scheduler = ON;``
And enable it in the my.cnf 
``event_scheduler=ENABLED``

To initiate
``call schedule_partition_manager``

Check partitions
``SELECT * FROM information_schema.partitions WHERE TABLE_SCHEMA='database' AND TABLE_NAME = 'tablename' AND PARTITION_NAME IS NOT NULL ;``

