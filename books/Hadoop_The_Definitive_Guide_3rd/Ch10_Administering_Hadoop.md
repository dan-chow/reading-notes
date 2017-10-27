## Chapter 10: Administering Hadoop

### HDFS

A newly formatted namenode creates the following directory structure:
```bash
${dfs.name.dir}/
└── current/
   ├── VERSION
   ├── edits
   ├── fsimage
   └── fstime
```

When a filesystem client performs a write operation (such as creating or moving a file),
it is first recorded in the edit log.
The fsimage file is a persistent checkpoint of the filesystem metadata.
If the namenode fails, then the latest state of its metadata
can be reconstructed by loading the fsimage from disk into memory, and then applying
each of the operations in the edit log.
The fsimage file contains a serialized form of all the directory and file
inodes in the filesystem.

The solution is to run the secondary namenode, whose purpose is to produce checkpoints
of the primary’s in-memory filesystem metadata.1 The checkpointing process
proceeds as follows:
1. The secondary asks the primary to roll its edits file, so new edits go to a new file.
2. The secondary retrieves fsimage and edits from the primary (using HTTP GET).
3. The secondary loads fsimage into memory, applies each operation from edits, then
creates a new consolidated fsimage file.
4. The secondary sends the new fsimage back to the primary (using HTTP POST).
5. The primary replaces the old fsimage with the new one from the secondary and the
old edits file with the new one it started in step 1. It also updates the fstime file to
record the time that the checkpoint was taken.

The checkpointing process
**TODO**


A useful side effect of the checkpointing process is that the secondary has a checkpoint
at the end of the process, which can be found in a subdirectory called previous.checkpoint.
This can be used as a source for making (stale) backups of the namenode’s
metadata:
```bash
${fs.checkpoint.dir}/
├── current/
│  ├── VERSION
│  ├── edits
│  ├── fsimage
│  └── fstime
└── previous.checkpoint/
   ├── VERSION
   ├── edits
   ├── fsimage
   └── fstime
```

When the namenode starts, the first thing it does is load its image file (fsimage) into
memory and apply the edits from the edit log (edits). Once it has reconstructed a consistent
in-memory image of the filesystem metadata, it creates a new fsimage file
(effectively doing the checkpoint itself, without recourse to the secondary namenode)
and an empty edit log. Only at this point does the namenode start listening for RPC
and HTTP requests. However, the namenode is running in safe mode, which means
that it offers only a read-only view of the filesystem to clients.

Recall that the locations of blocks in the system are not persisted by the namenode;
this information resides with the datanodes, in the form of a list of the blocks it is
storing. During normal operation of the system, the namenode has a map of block
locations stored in memory. Safe mode is needed to give the datanodes time to check
in to the namenode with their block lists, so the namenode can be informed of enough
block locations to run the filesystem effectively.

Safe mode is exited when the minimal replication condition is reached, plus an extension
time of 30 seconds. The minimal replication condition is when 99.9% of the blocks in
the whole filesystem meet their minimum replication level.

An administrator has the ability to make the namenode enter or leave safe mode at any
time. It is sometimes necessary to do this when carrying out maintenance on the cluster
or after upgrading a cluster to confirm that data is still readable. To enter safe mode,
use the following command:
```bash
% hadoop dfsadmin -safemode enter
Safe mode is ON
```
You can make the namenode leave safe mode by using:
```bash
% hadoop dfsadmin -safemode leave
Safe mode is OFF
```


The dfsadmin tool is a multipurpose tool for finding information about the state of
HDFS, as well as for performing administration operations on HDFS.

Hadoop provides an fsck utility for checking the health of files in HDFS. The tool looks
for blocks that are missing from all datanodes, as well as under- or over-replicated
blocks.

Every datanode runs a block scanner, which periodically verifies all the blocks stored
on the datanode. This allows bad blocks to be detected and fixed before they are read
by clients.

The balancer program is a Hadoop daemon that redistributes blocks by moving them
from overutilized datanodes to underutilized datanodes, while adhering to the block
replica placement policy that makes data loss unlikely by placing block replicas on
different racks.

### Monitoring

