Backup system.

Contains 5 components all of which may optionally reside on separate machines:

1. "file server/daemon": an app that reads files from data source and transfers to a storage server.
2. "storage server", storage: reads data from a file server and writes to the backup storage *(disk, tape, etc)*.
3. "database server", catalog: a catalogue of backups/files on the storage server.
4. "backup server", director: an app controlling services operation and running schedules.
5. Tray monitor and console: administration clients.

# Configuration

## Syntax

It is awkward.

Configuration consists mainly of paragraphs written as `ParName {…}`. E.g. `FileSet {…}`. Paragraph content is either another paragraph or a `key = val` assignment. Whitespace is mainly insignificant, except for the `key = val` oddity.

`key = val` is where the fun begins. Both key and val allow inserting spaces inside, Bacula just strips them. So e.g. `Name = XY` and `N a m e = X Y` end up being same thing. Then, whenever you want to avoid space being stripped from `val`, you can enclose them to double quotes, e.g. `Name = "X Y"`. But for some reason, double quotes are not supported for "key" part, Bacula would just bail out if you tried writing `"Name" = XY`.

## `bacula-dir.conf`

* `Director`
* `Job`: a backup, restore or other operation for Bacula to perform. May refer to `FileSet`, `JobDefs`, `Pool`. It may not write to a volume but to a pool.
* `JobDefs`: "job defaults", a template for `Job`. May refer to `Schedule`.
* `FileSet`: files to back up. Referred to by `Job`
* `Schedule`: time when job gets run.
* `Client`: connection config for a "file service".
* `Autochanger`: Idk exactly, but it's something about grouping multiple storage devices into a single unit.
* `Catalog`: connection config for a "database service".
* `Messages`: logging: things to go to email and to log, separately.
* `Console`: console configs, may be restricted in different ways for e.g. just status monitoring.
* `Pool`: a group of "volumes".
  * A volume is a storage device like a disk or a tape and it may not be referred directly by a job.

# Misc

* "Restoring": done with `restore` command which uses "jobid" of the jobs log to restore from. But for some reason it also requires a `Job` with `Type = Restore` to be present, otherwise restore won't work. In particular, the `FileSet` directive is completely ignored *(but required to be present)*, and instead you'd have to manually chose which files you want to restore.

# Gotchas

* Don't confuse `jobs` and `jobs`! Already confused, are you? In Bacula the top-level thing represented by `Job {` config paragraph is called a job. And then if you ran a job multiple times, its history entries are called jobs as well! So when you see word "job" it may mean any of those concepts.
* Increasing a pool size won't propagate to volumes till you exec `update volume` in console.
* If you don't start postgresql before installing Bacula, then initial installation will get completely screwed up on the database side. Bacula says that to fix this you'd have to run some `dbconfig-common` after starting postgresql service, but Idk where this script *(or even if it's a single script or a group)*.

  So happened that I didn't know any of that, so below is my unfinished research on how to fix this. The errors below are fixed in `sudo -u postgres psql`. You'll be getting:

  1. `Unable to connect to PostgreSQL server`. This is because it didn't create a user that's written in the config. Fix with `create user bacula password 'insert_pswd';`
  2. `Unable to connect to PostgreSQL server`. This time it's because it didn't create a database, fix with `create DATABASE bacula;`

  There will be more errors, but at that point I decided to completely remove the Podman/Docker image with bacula to get it reinstalled. Worth noting that simply `apt purge bacula` and reinstalling it won't be enough, it leaves some configs at random places that you'd have to search for. Yeah, that sucks.
* For most usecases Bacula will be non-functional till you assign a non-localhost IP address to SD in both `bacula-sd.conf`'s `SDAddress` and in `bacula-dir.conf`'s `Storage { … Address = …}`. This is because SD don't typically run on the same host as the client. "Why does that matter?" you might wonder. Well, it turns out during backup or restore a client will be communicated a SD's address, so they can exchange data. This alone makes sense, so that director isn't a bottleneck; what doesn't though is that director can't establish connection channel between the two, instead it simply sends whatever's written in the config and gives 0 useful information if that's not gonna work.
* `Options` inside `FileSet` are semicolon-separated. The gotcha here is the separator is completely undocumented, and if you try to separate them by a space *(because why not, docs just say it's a list but not X-separated)*, Bacula will simply ignore text past the first option. IOW, it would look like everything works, but it actually doesn't.

    Another valid separator is a newline *(semicolon in this case is not necessary)*.
