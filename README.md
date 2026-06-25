## copy-gfs - maintain copies of file in GFS directory structure

### Introduction
Use this script to maintain copies of original file (usually backup) in GFS directory structure.

GFS here stands for Grandfather-Father-Son, this term is used in description of backup retention policy.
Usually it is used for this task: "Save N last daily copies of file (one for each day), N weekly, N monthly, N yearly".

The reason is to make limited amount of different copies of file while maintaining historical depth.

The main differences from other implementations:
- If you fail to run the script at the beginning or at the end of certain period (week/month/...),
you will not totally miss copy of file in this period - it will appear
when next job will run (of course if there will be job in this period).
I.e., if you run your backup job daily, and it fails to run at 1st of month,
the monthly copy will be made at 2nd.
- It saves first (oldest) copy in each period.
So, your 'yearly' copy will not run with your daily copy until year ends.

### Typical invocation
The idea at large is that you do your backup job producing single file, then pass this file as an argument to ``copy-gfs``.
The ``copy-gfs`` will make proper copies of file in given GFS directory, maintain directory structure under it, and remove old copies from it.

Say your backup file is ``/backup/my-system-2019-05-22.tar.gz``.
Before running ``copy-gfs``, create directory for GFS structure, say, ``/backup-gfs``.
Then just run:
```
copy-gfs -c -l -d /backup-gfs -m '' -s /backup/my-system-2019-05-22.tar.gz
```
If you have few different jobs that produce different file names (i.e. application backup and database backup),
and you want to save them into one GFS directory, then you need to define file mask for each job with ``-m`` parameter.
If you will not do this, ``copy-gfs`` will treat them as one job, deleting another job's files from GFS or preventing making a copy.
To do this, run something like this:
```
copy-gfs -c -l -d /backup-gfs -m 'my-app-*' -s /backup/my-app-2019-05-22.tar.gz
copy-gfs -c -l -d /backup-gfs -m 'my-db-*'  -s /backup/my-db-2019-05-22.tar.gz
```
And then, backups will not interfere with each other. ``copy-gfs`` will simply ignore files that do not fit the mask.

### How to maintain "latest" copies in addition to GFS historical copies
Due to historical compatibility reasons, this script saves latest copy only if ``-l`` (``--keep-latest``) option is given.

### Command-line arguments
```
$ copy-gfs -h
usage: copy-gfs [-h] [-s SOURCE] [-c] -d DESTDIR [-m MASK] [-k] [-l] [-a] [-f]
                [--keep-daily N] [--keep-weekly N] [--keep-monthly N]
                [--keep-yearly N] [--keep-minutely N] [--keep-hourly N] [-v]

Maintain GFS-style directory structure
It will:
 - Copy SOURCE file to GFS destination directories under DESTDIR:
   DESTDIR/daily, weekly, monthly, yearly,
   and, disabled by default: minutely, hourly
   Date and time of source file are preserved
 - (if -c given) Cleanup old files from GFS directories:
   Delete from these directories all files based on MASK except one copy
   Will keep only one OLDEST copy of each period
   (i.e., for monthly - 1st of January, 1st of February, etc, or for other
   dates if there was no files for 1st, anyway only one file per month)
It will not delete SOURCE file.

Simplest sample: say you have backup.tar and target directory backups-gfs:
  copy-gfs -a -f -c -l -s backup.tar -d backups-gfs

options:
  -h, --help            show this help message and exit
  -s SOURCE, --source SOURCE
                        Source file name
  -c, --cleanup         Perform cleanup of GFS directories
  -d DESTDIR, --destdir DESTDIR
                        Destination directory, under which GFS subdirs will be made
                        Subdirectories: minutely, hourly, daily, weekly, monthly, yearly
                        No changes will be made outside these subdirectories
  -m MASK, --mask MASK  File mask (as in shell)
                        All files that don't begin with mask will be ignored
                        Source file must match the mask, otherwise command will finish with an error
                        i.e. if your file is my-bck-2018-05-17.tar, then good mask is 'my-bck-*.tar'
                        other backup jobs MUST NOT match mask, or some useful data will be deleted from GFS
                        or other files may be kept instead of your files
                        If mask doesn't end with *, it will be appended to mask
                        Default is '*' (match all files - treat all files in GFS dirs as originating from single source)
                        And DON't FORGET to embrace mask in quotes or escape shell expansion symbols!
  -k, --skipexist       Skip copy to GFS directory if it already contains a copy for period that
                        corresponds to SOURCE date
                        It is more safe not to use -k, but use -c instead
                        With -c file will always be first (and always!) copied to GFS,
                          and only then deleted if it is not needed
                          or else it is possible to skip copying if there is an error in this program :-)
  -l, --keep-latest     Make a copy of file in 'latest' GFS subdirectory
                        and remove other files with given mask
  -a, --append-timestamp
                        Always append timestamp to the name of the file copy in GFS dir,
                        not only when a name conflict occurs.
                        By default only the date (YYYYMMDD) is appended,
                        HHMM[SS] appended only if file with same name already exists;
                        see -f for the full timestamp.
  -f, --always-full-timestamp
                        Always append the full timestamp (YYYYMMDD-HHMMSS) instead of just the date (YYYYMMDD).
                        Affects both -a and the conflict-resolution naming
  --keep-daily N        default N=8
  --keep-weekly N       default N=5
  --keep-monthly N      default N=13
  --keep-yearly N       default N=5
  --keep-minutely N     default N=0 (disabled)
  --keep-hourly N       default N=0 (disabled)
  -v, --verbose         Print verbose output, -vv for debugging output
```

### Testing
I didn't make unit tests, just made script named mktestdata which generates a number of files,
so I (and you) could see how script works :-)

### Author
Nikita Smirnov

### License
MIT
