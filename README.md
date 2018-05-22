blockbackup suite
=================

Set of tools for incremental backup of block devices

Features
--------

- incremental backup of block device
- simple deduplication using hard links
- compressing backup on the fly using gzip
- parallel compression
- making md5 sums of chunks before compression
- full restore at decent speed
- partial restore (slower but any chunk can be restored separately)
- only bash and basic system tools used
- free and open source


-----------

How it works
============

__Blockbackup__ divide whole block device into chunks. Each chunk is saved, 
compressed (optional) and md5 sum is calculated (optional). __Blockbackup__ also 
compares every chunk with the same chunk from earlier backup and makes 
hard link if they are the same, so no additional disk space is used.  
Because of hard links, it looks like every backup is full backup, any backup in the 
middle could be safely removed, restore from any previous point in time is easy.

For every backup __blockbackup__ creates new directory named from current date/time.  
After every successful backup link _lastsync_ is created/updated, so it points 
to last backup. This link also points to reference directory for comparision.

Destination directory structure:
```
    dest-directory/
    ├── 2016-06-26_211549
    │   ├── 00000
    │   │   ├── 00000000.gz
    │   │   ├── 00000000.md5
    │   │   ├── 00000001.gz
    │   │   ├── 00000001.md5
    │   │   ├── ........
    │   │   ├── 00000999.gz
    │   │   └── 00000999.md5
    │   ├── 00001
    │   │   ├── 00001000.gz
    │   │   ├── 00001000.md5
    │   │   ├── 00001001.gz
    │   │   ├── 00001001.md5
    │   ................
    └── lastsync -> 2016-06-26_211549
```

When enabled, chunks are also compressed. There are 2 modes of operation:
- compression on the fly (-z). In this mode chunks are compressed while blocks
are being read. Number of compression threads could be specified by -n option.
Using number of threads two or even three times greater than number of cores
could give almost disk speed processing.
- compression after whole disk is imaged (-za). This option is to decrease access
disk duration to as short as possible. Whole compression is done after reading all
data from disk.

Note on chunk size: default is 1MB which is reasonable good value.
For better performance try using larger chunks, but it may impact
deduplication efficiency.

__Blockrestore__:
- has chunk size autodetect
- zip/nozip autodetect
- two modes of operation:
 - normal (slow) mode where disk offset is calculated for every chunk. By using this
   mode it's possible to restore only part of backup (depending on directory contents).
 - fast mode, where all chunks are sorted and then piped to one _dd_ process, so
   it's fast, but can't handle incomplete backups with some files missing.


Command line options
====================

blockbackup
-----------

    blockbackup -s device -d directory [-z|-za] [-n num_zips] [-c chunk] [-mn]

-s|--source - source device

-d|--dest - destination folder. For each run __blockbackup__ creates in this  
            directory another one named from date and time. After every  
            successful run it also creates link *lastsync* pointing to last  
            created directory. This link is used as reference for deduplication  
            on next run.

-z|--zip - compress during reading (if chunk is new)

-za|--zip-after - read and compare whole device, run compression after this

-n|--num-zips - run max num_zips in parallel

-c|--chunk - divide whole block device into chunks of size _chunk_

-nm|--no-md5 - don't generate md5 sum for every chunk
 

blockrestore
------------

    blockrestore -s directory -d device [-c chunk] [-f]

-s|--source - source directory. It should be one level deeper than directory
              passed to _blockbackup_.

-d|--dest - destination block device

-c|--chunk - force using chunks size. Normally chunk size is autodetected
             from first chunk found in source directory

-f|--fast - use fast restore method. Source directory MUST be complete
            with all chunks or result will be corrupted. But it's fast :-)


Legal
=====

    Copyright (C) 2016-2018 Marek Wodzinski

    This program is free software: you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with this program.  If not, see <http://www.gnu.org/licenses/>.
