---
title: how to get a sql console on an android sqlite database
excerpt: Options include 1) `adb exec-out run-as` on an USB device, 2) aSQLiteManager and 3) Android Debug database.
toc: true
author: Eimert Vink
tags:
  - tech
---
This is a short post on all the options I've been through to get a (simple) sql console for Android development.
And it saves time when you can pick the right tool straight away :).<br>**Update 11-05-2019**: database export is possible on AVD devices too.
Added a fix for when the exported database file is empty (call close() to let SQLite write its journal to disk).<br>

In descending order of goodness:
## 1) adb exec-out run-as nl.eimertvink.smv cat databases/smv_db > smv_db_copy
This is the best and fastest way from [stackoverflow](https://stackoverflow.com/questions/18471780/android-adb-retrieve-database-using-run-as){:target="_blank"}.
You'll need ~~your phone connected and~~ a physical phone <sup>(with debugging enabled)</sup> *or* an AVD device 
<sup>([AVD: debugging is enabled by default](https://developer.android.com/studio/debug){:target="_blank"})</sup>.<br>

A script that simplifies the copy action is [humpty-dumpty-android](https://github.com/Pixplicity/humpty-dumpty-android){:target="_blank"}. I'm not going to describe that tool here.<br>
This is the basic way to get the database on your local machine:
1. Copy db to machine:
```bash
adb exec-out run-as nl.eimertvink.smv cat databases/smv_db > smv_db_copy
```
2. View tables:
```bash
eimert@EIM SMV $ sqlite3 smv_db_copy 
(... omitted ...)
sqlite> .tables
sqlite> 
```

No tables found; empty database file. Read on how to resolve this.

### What to do when the database file is empty
You'll need to call close() on the database instance, to let SQLite write its journal to the database file. For that we'll create a button that calls close();.
### Create a (hidden) button for db.close()
Some Java code that handles the button click:
```java
public class MainActivity extends AppCompatActivity {
        // ... omitted ...
        
        // Called when the user selects a contextual menu item
        @Override
        public boolean onActionItemClicked(ActionMode mode, MenuItem item) {
            switch (item.getItemId()) {
                case R.id.action_close_db:
                    Toast.makeText(MainActivity.this, getString(R.string.action_close_db), Toast.LENGTH_SHORT).show();
                    Log.d(TAG, getString(R.string.action_close_db));
                    AsyncTask task0 = new AsyncTask() {
                        @Override
                        protected Object doInBackground(Object[] params) {
                            if (db != null) {
                                Log.d(TAG, "Closing database instance " + db);
                                AppDatabase.close(db);
                            }
                            else {
                                db = AppDatabase.getInstance(MainActivity.this);
                                Log.d(TAG, "Opened database instance " + db);
                            }
                            return null;
                        }
                    };
                    task0.execute();
                    mode.finish(); // Action picked, so close the contextual menu
                    return true;
                default:
                    return false;
            }
        }
        // ... omitted ...
}                   
```
The close(db) call refers to this static method in the AppDatabase class:
```java
public abstract class AppDatabase extends RoomDatabase {
    // ... omitted ...
    public static void close(AppDatabase db) {
        db.close(); // calls close() on RoomDatabase instance.
    }
}
```
See [this blogpost](https://medium.com/@ajaysaini.official/building-database-with-room-persistence-library-ecf7d0b8f3e9){:target="_blank"}
 for more details on how I've setup Android Room.
### Test the db.close() button
Launch the emulator and start an adb shell:
```bash
adb shell
```
Execute "run-as your.app.realm":
```bash
generic_x86:/ $ run-as nl.eimertvink.smv
generic_x86:/data/data/nl.eimertvink.smv $
```
Check the database (called smv_db) file size. In this example the actual db has no tables (Size: 4096).
```bash
generic_x86:/data/data/nl.eimertvink.smv $ stat databases/smv_db                                                          <
  File: `databases/smv_db'
  Size: 4096	 Blocks: 8	 IO Blocks: 512	regular file
Device: fc00h/64512d	 Inode: 123232	 Links: 1
Access: (660/-rw-rw----)	Uid: (10088/  u0_a88)	Gid: (10088/  u0_a88)
Access: 2019-05-11 12:37:50.711995122
Modify: 2019-05-11 12:37:50.711995122
Change: 2019-05-11 12:56:09.251990576
```

Click on the button that is calling AppDatabase.close() in the app:
<iframe src="https://giphy.com/embed/KZYnGwVh8ICN73jXod" width="276" height="480" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/KZYnGwVh8ICN73jXod">via GIPHY</a></p>

Again check the database file size. Notice the file has increased in size (from 4096 to to 40960).
```bash
generic_x86:/data/data/nl.eimertvink.smv $ stat databases/smv_db                                                          <
  File: `databases/smv_db'
  Size: 40960	 Blocks: 80	 IO Blocks: 512	regular file
Device: fc00h/64512d	 Inode: 123232	 Links: 1
Access: (660/-rw-rw----)	Uid: (10088/  u0_a88)	Gid: (10088/  u0_a88)
Access: 2019-05-11 12:37:50.711995122
Modify: 2019-05-11 12:57:09.121990328
Change: 2019-05-11 12:57:09.121990328
``` 
Let's copy the filled database file to our local machine.
1. Copy db to machine:
```bash
adb exec-out run-as nl.eimertvink.smv cat databases/smv_db > smv_db_copy
```
2. Happy querying:
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

## 2) [aSQLiteManager](https://play.google.com/store/apps/details?id=dk.andsen.asqlitemanager){:target="_blank"}
This app gives a SQL console on your phone. You can access the database of other apps when your phone is rooted.<br>
<img src="https://lh3.ggpht.com/zIm_Ai93gGfiKgGDHKb9LddN-elxqJ4IylTzYqtLoGdw2lU_ieqjvDEIT0d5uxxzZd0=w1920-h1008-rw"><br>
[source](https://play.google.com/store/apps/details?id=dk.andsen.asqlitemanager){:target="_blank"}

## 3) [Android Debug Database](https://github.com/amitshekhariitbhu/Android-Debug-Database){:target="_blank"}
You'll get a nice web gui on [localhost:8080](http://localhost:8080). But in my case I had quite some trouble with the tool (URL not loading).
Running `adb forward tcp:8080 tcp:8080` solved that issue only a few times.

## Conclusion
I'd go for `adb exec-out run-as` because it is simple and just works.


