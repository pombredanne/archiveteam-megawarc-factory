Archive Team megawarc factory
=============================
Some scripts to bundle Archive Team uploads and upload them to Archive.org. Use at your own risk; the scripts will need per-project adjustment.

These scripts make batches of uploaded warc.gz files, combine them into megawarcs and upload them to their permanent home on Archive.org.

Three processes work together to make this happen:

1. The chunker
--------------
The chunker moves uploaded warc.gz files from the upload directory to a batch directory. When this directory has grown to 50GB, the chunker begins a new directory and moves the completed directory to the packing queue.

There can only be one chunker per upload directory. Chunking doesn't take long, if the files are not moving to a different filesystem.

2. The packer
-------------
The packer monitors the packing queue. When the chunker brings a new directory, the packer removes the directory from the queue and starts converting it into a megawarc (using the megawarc utility). When that is done, the packer moves the megawarc to the upload queue and removes the original warc files.

If necessary, multiple packers can work the same queue. Packing involves lots of gzipping and takes some time.

3. The uploader
---------------
The uploader monitors the upload queue. When the packer brings a new megawarc, the uploader removes the megawarc from the queue and uploads it to Archive.org. If the upload is successful, the uploader removes the megawarc.

If necessary, multiple uploaders can work the same queue.


Filesystems
-----------
From the chunker to the uploader, the chunks move through the system as timestamped directories, e.g., 20130401213900.) This timestamp will also be used in the name of the uploaded item on Archive.org. The queues are directories. Processes 'claim' a chunk by moving it from the queue directory to their working directory. This assumes that `mv` is an atomic operation.

For efficiency and to maintain the atomicity of `mv`, the filesystem of the directories is very important:

1. The Rsync upload directory, the chunker working directory, the packing queue and that side of the packer's working directory should all be on the same filesystem. This ensures that the uploaded warc.gz files never move to a different file system.
2. The megawarc side of the packer's working directory, the upload queue and the uploader's working directory should also share a filesystem.

Filesystems 1 and 2 do not have to be the same.


Configuration
-------------
Create a configuration file called `config.sh` and place it in the directory where you start the scripts. See the `config.example.sh` for more details.


Running
-------
Run the scripts in `screen`, `tmux` or something similar. `touch RUN` before you start the scripts. Use `rm RUN` to stop gracefully.

* `./chunk-multiple` (run exactly one)
* `./pack-multiple` (you may run more than one)
* `./upload-multiple` (you may run more than one)

Utility scripts:

* `./du-all` will run `du -hs` in all queues


Scheduling priorities
---------------------
The packing script will use all your I/O capacity. Consider using `nice` and `ionice` to run in at a lower priority, so it doesn't hinder your incoming Rsync or outgoing curl uploads.

* `ionice -c 2 -n 6 nice -n 19 ./pack-multiple`


Recovering from errors
----------------------
The scripts are designed not to lose data. If a script dies, you can look in its working directory for in-progress items and move them back to the queue.


Requirements
------------
These scripts use Bash and Curl.

You should clone https://github.com/ArchiveTeam/megawarc to the `megawarc/` subdirectory of these scripts. The megawarc utility requires Python and Gzip.

