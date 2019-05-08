---
title: how to get a sql console on an android sqlite database
excerpt: Options include 1) `adb exec-out run-as` on an USB device, 2) aSQLiteManager and 3) Android Debug database.
toc: true
author: Eimert Vink
tags:
  - tech
---
This is a short post on all the options I've been through to get a (simple) sql console for Android development.
And it saves time when you can pick the right tool straight away :). In descending order of goodness:

## 1) adb -s 52006e5660588515 exec-out run-as nl.eimertvink.smv cat databases/smv_db > smv_db_copy
This is the best and fastest way from [stackoverflow](https://stackoverflow.com/questions/18471780/android-adb-retrieve-database-using-run-as){:target="_blank"}.
You'll need your phone connected and debugging enabled.<br>
Copy db to machine:
```bash
adb -s 52006e5660588515 exec-out run-as nl.eimertvink.smv cat databases/smv_db > smv_db_copy
```
Happy querying:
```bash
eimert@EIM SMV $ sqlite3 smv_db_copy
-- Loading resources from /home/eimert/.sqliterc

SQLite version 3.11.0 2016-02-15 17:29:24
Enter ".help" for usage hints.
sqlite> .tables
Question           ScoreDetail        android_metadata
Rating             TestSession        room_master_table
sqlite> .quit
```
Note: doing this on an emulated device (AVD) doesn't work. It gives back a seemingly _"empty"_ db file:
```bash
eimert@EIM SMV $ adb exec-out run-as nl.eimertvink.smv cat databases/smv_db > smv_db_copy2
eimert@EIM SMV $ sqlite3 smv_db_copy2
-- Loading resources from /home/eimert/.sqliterc

SQLite version 3.11.0 2016-02-15 17:29:24
Enter ".help" for usage hints.
sqlite> .tables
sqlite>
```
The database size is a bit odd on the device. But the app runs fine in the emulator.
```bash
eimert@EIM SMV $ adb shell
127|generic_x86:/ $ run-as nl.eimertvink.smv
cd databases/
generic_x86:/data/data/nl.eimertvink.smv/databases $ du -sh smv_db
4.0K    smv_db
```
Comparing the size of the working db file with the faulty one:
```bash
eimert@EIM SMV $ du -sh smv_db_copy smv_db_copy2
48K     smv_db_copy
4,0K    smv_db_copy2
```
Let me know if there is something to get `adb exec-out` to work on an AVD ;).

## 2) [aSQLiteManager](https://play.google.com/store/apps/details?id=dk.andsen.asqlitemanager){:target="_blank"}
This app gives a SQL console on your phone. You can access the database of other apps when your phone is rooted.
<img src="https://lh3.ggpht.com/zIm_Ai93gGfiKgGDHKb9LddN-elxqJ4IylTzYqtLoGdw2lU_ieqjvDEIT0d5uxxzZd0=w1920-h1008-rw"><br>
[source](https://play.google.com/store/apps/details?id=dk.andsen.asqlitemanager){:target="_blank"}

## 3) [Android Debug Database](https://github.com/amitshekhariitbhu/Android-Debug-Database){:target="_blank"}
You'll get a nice web gui on [localhost:8080](http://localhost:8080). But in my case I had quite some trouble with the tool (URL not loading).
Running `adb forward tcp:8080 tcp:8080` solved that issue only a few times.

## Conclusion
I'd go for `adb exec-out run-as` because it is simple and just works.


