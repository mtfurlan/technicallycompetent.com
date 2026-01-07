---
title: Backup Solution Comparison
---
I would really like backups on so many things


Comparison Criteria:
* incremental backups
* encrypted
* show versions of a file
* get a version of a file
* prune backups reducing over time, not just older than




## Comparison
|                    | [Duplicity][duplicity] | [borg][borg]               | [Duplicacy][duplicacy]                          | [rdiff-backup][rdiff-backup] | [rclone][rclone] |
| ----------         | ---------              | ----                       | ---------                                       | ------------                 | ---------------- |
| started            | 2002                   | 2015                       | 2016                                            | 2001                         | 2014             |
| license            | gpl                    | bsd                        | free for persnal use                            | gpl                          | MIT              |
| encryption         | yes                    | yes                        | yes                                             | no                           | yes              |
| incremental        | yes                    | yes                        | yes                                             | yes                          | no?              |
| show file versions | reasonable commands    | fuse mount maybe sometimes | git like, but bad in other ways (see notes)     | reasonable commands          | n/a              |
| backend            | any                    | borg                       | any                                             | sftp/rsync                   | anything         |
| prune mode         | age, count             | [GFS][gfs]                 | GFS                                             | age, count                   | n/a              |
| website vibes      | mature software        | new but alright            | trying to sell you something                    | mature software              | mature software  |
| abandonment vibes  |                        |                            | one commit in 2025, many ignored PRs and issues |                              |                  |
| debian packaging   | 3.0.4 vs 3.0.6.3       | 1.4.0 vs 1.4.3             | github release binary                           | 2.2.6 vs 2.2.6               | 1.60.1 vs 1.72.1 |


|                    | [restic][restic]       | [bup][bup]          | [duplicati][duplicati] |
| ------------------ | ----------------       | ----------          | ---------------------- |
| started            | 2019                   | 2010                | 2008                   |
| license            | BSD-2                  | LGPL v2             | MIT ish paid           |
| encryption         |                        | no                  | yes                    |
| incremental        | yes                    | yes                 | yes                    |
| show file versions | only compare snapshots | maybe `bup ls`      | yes                    |
| backend            | sftp / rclone / cloud  | ssh + remote `bup`? | any                    |
| prune mode         | GFS, age               | age                 | GFS, age, count        |
| website vibes      | readthedocs            | mature              | sales pitch fuckery    |
| abandonment vibes  |                        |                     |                        |
| debian packaging   | 0.18.0 vs 0.18.1       | 0.33.7 vs 0.33.7    | download deb           |




Options:
* [Duplicity][Duplicity]
  * stores original tarball and then differential tarballs
  * `duplicity collection-status --file-changed 'filename' 'remote-url'`
  * reddit says it's slow, and based on the tar approach, that makes sense to me, and they talk about the downsides here: https://duplicity.us/new_format.html
* [borg][borg] née Attic
* [Duplicacy][Duplicacy]
  * research project
  * looks like it has good cli interface for showing file history (history, diff, cat, list, etc)
    * `duplicacy history $file`
  * Uses single dash with long arguments(`-storage <storage name>`)
  * `duplicacy prune -keep 7:30      # Keep 1 snapshot every 7 days for snapshots older than 30 days`
  * they have a nice if out of date [comparison of some backup solutions](https://github.com/gilbertchen/duplicacy#comparison-with-other-backup-tools)
  * Has really good revision & diff tools
  * returns 0 even on failure, author suggests grepping the logs, but the error/warning logs are not consistant
  * uses single dash for long options
* [rdiff-backup][rdiff-backup]
  * mirrors the directory to $remoteBackupDir, and stores tracking data in $remoteBackupDir/rdiff-backup-data (`rdiff-backup-data/increments/file.2003-03-05T12:21:41-07:00.diff.gz`, `rdiff-backup-data/session_statistics*`)
  * in the middle of a cli syntax migration, old cli has been removed but is still in docs as of 2025-12-19
  * showing file history
    * files changed in last 5 days: `rdiff-backup list files --changed-since 5D $backupSpecifier/subdir`
    * available versions of file: `rdiff-backup list increments $backupSpecifier/file`
  * restore file file backed up from `$whatever/file` to `/tmp/file`
    `rdiff-backup restore --at 10D $backupSpecifier/file /tmp/file`
* [restic][restic]
  * have to enter password every command?
* [bup][bup]
  * must have `bup` on remote server
* [duplicati][duplicati]
  * CLI exclusively refers to backups by remote URL
    * Including credentials??
  * Secrets handling is somewhere between "obtuse" and "actual garbage"
  * GUI and CLI only kinda link maybe? or not at all?
    * CLI can't refer to backups with no encryption password???
  * Exploring backups in the GUI is just not a thing

## Other
* [backupninja](https://0xacab.org/liberate/backupninja) is a tool to manage running backups aroiund borg or duplicity or whatever on a schedule including databases and stuff

[duplicity]: https://duplicity.gitlab.io/docs.html
[borg]: https://borgbackup.readthedocs.io/en/stable/
[duplicacy]: https://github.com/gilbertchen/duplicacy
[rdiff-backup]: https://rdiff-backup.net/
[rclone]: https://github.com/rclone/rclone
[restic]: https://restic.net/
[bup]: https://github.com/bup/bup
[duplicati]: https://docs.duplicati.com/

[gfs]: https://en.wikipedia.org/wiki/Backup_rotation_scheme#Grandfather-father-son
