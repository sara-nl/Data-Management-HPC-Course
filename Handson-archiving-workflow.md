# Hands-on: Archiving Data

## Data Management Course
#### Trainers: Narges Zarrabi (SURFsara), Arthur Newton (SURFsara)

## Introduction

Welcome to the hands-on part of archiving data from Lisa  to the Data Archive, using the dmftar tool. After finishing the hands-on, you will learn how to optimally archive your data from the Compute environment to the Data Archive service in SURFsara.

The module consists of two parts:

- Exploring the command line environment of Lisa, Data Archive and the DMF commands

- Archiving data using dmftar: learn to create and extract dmftar archives,  and do a remote archiving workflow from Lisa to the Data Archive service.

Each part has several excercises. For those who are faster, several extra bonus exercises are provided which will educate more advanced usage of the tools.

**Requirements:**

- Commandline tool (Terminal in Mac and linux or MobaXterm in Windows)
- logins to lisa and archive (`sdemo<XXX>` accounts)

<!--
**General syntax:**

- dmget: ```dmget –a [file]```

- dmput: ```dmput [-r] [file]```


- tar: ```tar [OPTIONS] <tarball_name>.tar <input-files..>```

- dmftar: ```dmftar [OPTIONS] <dmftar_name>.dmftar <input-files..>```

**General tips:**

- Using the tar command: combine multiple options into a single argument, e.g. `-a -b -c` becomes `-abc`
-->

## Part1: Explore the Environment

### Access the archive from HPC
To explore the environment, open a command line and try to login to the lisa service with the following command:

```
ssh <username>@lisa.surfsara.nl  
```
Welcome to Lisa system! Lets create a directory with some data to work with:

```
mkdir workdir
```
 
Change your working directory to the newly created directory and make some dummy data (each 2MB size):

```
cd workdir/
truncate -s 2M data1 data2 data3
```


Now, let's explore the connection to the Archive. The Archive is mounted to Lisa (or Cartesius) via NFS mounts. From Lisa (or Cartesius), you can see your archive directory here:


```
ls /archive/<username>/
```

Lets make an archiving folder in your archive directory:

```
mkdir /archive/narges/archivedir
```

Copy some data to your archive directory:

```
cp /home/narges/workdir/* /archive/narges/archivedir
```

Now do a dmls to 


### DMF commands

DMF (Data Migration Facility) is a hierarchical storage management system for Silicon Graphics environments. Its primary purpose is to augment the economic value of storage media and stored data. DMF commands ate Tape aware and facilitate staging data from tape and putting data on tape.

If you are handling really large amounts of data, the DMF utilities come in useful. On Lisa and Cartesius the DMF utilities are system-wide installed.

The most important DMF commands are:

- `dmls` (Lists contents of directories)
- `dmput` (Migrates files from disk to tape media)
- `dmget` (retrieve files from tape to disk) 

To get more information about each command you can run the manual by:
`man dmls`

In the workdir directory do a dmls to list the files you made in Lisa:
 
```
dmls -l

total 0
-rw-------  1 narges    narges    2097152 2020-01-21 22:06 (N/A) data1
-rw-------  1 narges    narges    2097152 2020-01-21 22:06 (N/A) data2
-rw-------  1 narges    narges    2097152 2020-01-21 22:06 (N/A) data3

```

Now list the contents of the archivedir in our archive directory, using `dmls` command:

```
dmls -l /archive/narges/archivedir

total 0
-rw-------  1 narges    narges    2097152 2020-01-21 22:15 (REG) data1
-rw-------  1 narges    narges    2097152 2020-01-21 22:15 (REG) data2
-rw-------  1 narges    narges    2097152 2020-01-21 22:15 (REG) data3
```

What is the difference between the output of running `dmls` on your workdir on Lisa and on archivedir in your archive directory?

Note: `dmls` command gives information about the status of a file and wether it is migrated from disk to tape. Files status can have several values in DMF:

- REG: Regular files are user files residing only on disk
- DUL: Dual-state files whose data resides both online and offline
- OFL: Offline files whose data is no longer on disk
- MIG: Migrating files are files which are being copied from disk to tape
- UNM: Unmigrating files are files which are being copied from tape to disk


Now, change your curreent directory to the archivedir in your archive directory. 

```
cd /archive/narges/archivedir
```

Put data1 file on tape using `dmput` command:

```
dmput -r data1
```

Now do a `dmls` in your archivedir to see the status of the files again. Are there any changes? Why?

## Part 2: Archive using dmftar

Learn how to pack directory contents using the dmftar tool. dmftar is available on the Data Archive, Cartesius and Lisa services. Recently the tool is made open source (https://gitlab.com/surfsara/dmftar)

The dmftar tool is invoked by entering `dmftar` as a command. It is available to all users and in all directories. On the Lisa cluster, you need to load modules below:

```
module load pre2019
module load dmftar
```
### Step 1: Create some more data

In your home directory, create a new `file` directory. Change your working directory to `file` and create some data (data4, data5, data6) using the `truncate` command. 

```
mkdir file
cd file
truncate -s 14M file1 file2 file3
```



### Step 2: Make dmftar archive files

First explore the options of dmftar command by

```
dmftar --help
```

Questions:

- Which options are available to get additional help?
- Are there any man pages available?
- What is the difference in command structure between tar and dmftar?
- Can you combine the options to a single option? E.g. `-a -b` becomes `-ab`?
- Which option is always required?

**Exercise: Pack the files `data1`, `data2` , `data3` into a dmftar archive called `data.dmftar`**

Hint: look for the create option.

**Solution:** Use the `-c` option:

```
dmftar -c -f data.dmftar data1 data2 data3

```

**Exercise: List the contents of the newly created dmftar folder using standard Linux tools.**

To understand the actual contents of a dmftar archive it is useful to have a look at which files are actually stored in the folders.

**Solution**

```
ls -l data.dmftar/0000

total 6160
-rw------- 1 narges narges 6297600 Jan 22 00:02 data.dmftar.tar
-rw------- 1 narges narges      53 Jan 22 00:02 data.dmftar.tar.chksum
-rw------- 1 narges narges     168 Jan 22 00:02 data.dmftar.tar.idx
```

**Exercise (BONUS): Make a dmftar archive file which consists of tow other directories 
`data` and `file`.**

dmftar can easily process the contents of entire directories compared to the more file-oriented nature of tar. Therefore the tool is mostly used to pack individual directories.

**Solution:** In your work directory, first make a  directory called `files` including `file1`, `file2`, and `file3`. Then make a second directory called `data`, including `data1`, `data2`, and `data3`. Then You can pack the directories `data` and `files` into one dmftar archive file called `data-all.dmftar`, by passing the directory names as input arguments of the `dmftar` command.

```
mkdir data
cp data1 data2 data3 data/

mkdir files
cp file1 file2 file3 files/

dmftar -c -f data-all.dmftar data/ files/
```

Now that you have made archive dmftar files of the data, lets remove the data in your workdirectory

```
rm data1 data2 data3
rm file1 file2 file3
```

### Step 3: Verify your archive

An important aspect of data archiving is data integrity. The archiving facility of SURFsara automatically checks the data stored on tape by comparing files every once in a while.

Archived folders created using dmftar can be verified based on a checksum that is stored within the dmftar file.

**Exercise:  Verify the contents of the archive folder `data.dmftar` on the bit-level**

Hint: look for the verify option.

**Solution:** Add the `-V` option:

```
dmftar -V -f data.dmftar

verifying checksum for 1 volumes
verifying checksum for volume #1
OK
```

### Step 4: Extracting dmftar archives

Learn how to inspect and unpack an existing archive folder using dmftar. This can be done without unpakking the folder.

**Exercise: List the contents of the archive folder `data.dmftar`**

Hint: look for the list option.

**Solution:** Use the `-t` option:

```
dmftar -t -f data.dmftar
```
**Questions**

- What type of contents can be seen in the dmftar archive?
- What is the alternative for option `-t`?

**Exercise: Unpack the contents of the archive folder `data.dmftar`**

Hint: look for the extract option.

**Solution:** Use the `-x` option:

```
dmftar -x -f file/data.dmftar/
```

**Questions**

- Which files and folder are created?
- Where is the dmftar file extracted?

Remove the unpacked data in your home folder:

```
rm data1 data2 data3
```


**Eercise (BONUS): Unpack only a single file from a dmftar archive folder. Unpack the file `data3` from the archive file `data.dmftar`**

Sometimes it is useful to only unpack specific files or folders from an archive file. Using dmftar this is easily accomplished by adding the file name or a name pattern as an argument to the command.

Hints:

- Use the `--help` option to learn how to add a file extraction pattern.
- Investigate in what subdirectory the file is located.

**Solution:** Add the file name and folder as the last argument in the command:

```
dmftar -x -f data.dmftar data3
```

**Questions**

- Can you extract the file by only adding the file name?
- Where is the extracted file stored?


**Exercise (BONUS): Unpack a specific directory from a dmftar archive folder. Repeat the previous bonus excercise to extract all files in the subdirectory `data` in the archive folder `data-all.dmftar`**

**Solution:** Solely add the subdirectory name:

```
dmftar -x -f data-all.dmftar data/
```

### Step 5: Direct archive usage with dmftar 

You can use the `dmftar` command from Lisa or Cartesius to directly create dmftar archive files on the archive, or extract dmftar files from the archive.

**Exercise: Create a dmftar archive directory from the data directly on your Lisa account and directly put it on the archive**

**Solution:** 
To create dmftar archive file from Lisa on the archive, you should use the same dmftar command, but give a full path to the archive for the the dmftar file to be created 

```
dmftar -c -f /archive/narges/archivdir/ files/
```

**BOPNUS Exercise: Unpack directly from the archive the contents of the archived folder**

Lets unpack the dmftar file `data-files-extract.dmftar` directly from the archive.

**Solution:** 

```
dmftar -x -f /archive/<username>/archivedir/files.dmftar
```


