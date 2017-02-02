# Keep Your Code Up to Date

Galaxy development occurs in GitHub. Changes are stabilized in the release_YY.MM branches and then merged to master for each ```YY.MM.point``` release.

To be made aware of new Galaxy releases, please join the Galaxy Developers mailing list. Each release is accompanied by a news brief.

At any time, you can check to see if a new stable release is available by using the ```git log``` command:

```
 $ git log ..origin/master
 commit 3f314974c9c3742b118518881a6d392123ccc05d
 Merge: d8eeaae c78b7b6
 Author: Nate Coraor <nate@bx.psu.edu>
 Date:   Mon Mar 9 22:26:54 2015 -0400

     Merge branch 'release_15.03' to master for v15.03

  ...
  ```

If you see no output, you are up to date. If you see a list of commits, a new version is available. We suggest checking the accompanying news brief first (if the release is to a newer major version of Galaxy), but you can also immediately pull the commits to your local Galaxy clone with:

```
 $ git pull
   ...
```

Note: After pulling changes, you will need to stop your Galaxy server and restart with the updated code. This will interrupt any running jobs, unless you are using a cluster configuration. For more information on how to make Galaxy restartable without interrupting users, see the production server documentation.

Note: Occasionally, updated code includes structural changes to the Galaxy database tables. The news brief will alert you if a release contains a database change. After updating Galaxy, if you attempt to restart, Galaxy will refuse to load, and will output an error message indicating that your database is the wrong version. The error message indicates that you should run backup your database and then run ```sh manage_db.sh upgrade``` - follow those instructions carefully - especially the part about backing up your database safely!

Database updates are carefully tested before release, but it is always wise to be able to back out if something goes wrong during an update.

In the unlikely event that something goes wrong with updated code, you can return to an older release by guessing the release tag name from the news brief page and using the git checkout command. For example, to return to the latest version of the January 2015 release, use:

```
 $ git checkout release_15.01
```

You can also use tags to check out specific releases:

```
 $ git tag
  v13.01
  v13.01.1
  v13.02
   ...
  v14.10.1
  v15.01
  v15.01.1
  v15.01.2
  v15.03
```

Restore the fresh backup if a database update was required, and then restart Galaxy to get back to where you started.
