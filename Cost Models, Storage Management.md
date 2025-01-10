# Cost Models

Cost can be measured in terms of
 - **Time Cost**: total time taken to execute method, or
 - **Page Cost**: number of pages read and/or written (page = fixed-size block of data = block; size determined by storage medium, typical PAGESIZE values: 1024, 2048, 4096, 8192)

Terminologies and variables:

In developing cost models, we make assumptions on how DBMSs store data:
 - a relation is a set of *r* tuples, with average size *R* bytes
 - the tuples are stored in b data pages on a storage device
 - each page has size *B* bytes and contains up to *c* tuples
 - the tuples which answer query *q* are contained in *b<sub>q</sub>* pages
 - data is transferred bulk-storage↔memory in whole pages
 - cost of bulk-storage↔memory transfer *T<sub>r/w</sub>* is high

Addressing in DBMSs:
 - *PageID = FileID+Offset* ... identifies a block of data
    - where *Offset* gives location of block within file
 - *TupleID = PageID+Index* ... identifies a single tuple
    - where *Index* gives "location" of tuple within page
  
## File Management

Typical file operations provided by the operating system:

```
fd = open(fileName,mode)
  // open a named file for reading/writing/appending, fd is an integer representing the file descriptor
close(fd)
  // close an open file, via its descriptor 
nread = read(fd, buf, nbytes)
  // attempt to read data from file into buffer 
nwritten = write(fd, buf, nbytes)
  // attempt to write data from buffer to file
lseek(fd, offset, seek_type)
  // move file pointer to relative/absolute file offset
  // seek_type:
  // SEEK_SET: set file pointer to absolute position; SEEK_CUR: relative to current position; SEEK_END: relative to the end of the file
fsync(fd)
  // flush contents of file buffers to disk
```

### Single File DBMS

Objects are allocated to regions (segments) of the file (e.g. SQLite).

```
|parameters|catalogue|update logs|table1|table2|catalogue(cont.)|index for table 1|table1(cont.)|...
```

Note:
 - If an object grows too large for allocated segment, allocate an extension (cont.).
 - What happens to allocated space when objects are removed?
    - **Space fragmentation**: the space they occupied before becomes fragmented, remains allocated but unused. DBMS will not automatically shrink the file or reclaim this space until a manual **VACUUM** process is run.

**Single File Storage Manager**

Consider the following simple single-file DBMS layout:

```
    | SpaceMap | TableMap | Employee Data Pages | Project Data Pages | ...
   [0]        [10]       [20]                  [620]                [720]
```

There are two key components:
 - **SpaceMap**: It tracks how space in the file is allocated and used.
    - SpaceMap = [(offset, #pages, status)] = [ (0,10,U), (10,10,U), (20,600,U), (620,100,U), (720,20,F) ]. U = Used; F = Free
 - **TableMap**: It maps logical tables to their physical locations within the file.
    - TableMap = [name, offset, #pages] = [ ("employee",20,500), ("project",620,40) ]

With the above disk manager, our example:

```
select name from Employee;
```

might be implemented as something like

```
DB db = openDatabase("myDB");
Rel r = openRelation(db,"Employee");
Page buffer = malloc(PAGESIZE*sizeof(char));
for (int i = 0; i < r->npages; i++) {
   PageId pid = r->start+i;
   get_page(db, pid, buffer); // read the page into the buffer
   for each tuple in buffer {
      get tuple data and extract name
      add (name) to result tuples
   }
}
```

Two important functions *get_page* and *put_page*:

```
// assume that Page = byte[PageSize]
// assume that PageId = block number in file

// read page from file into memory buffer
void get_page(DB db, PageId p, Page buf) {
   lseek(db->fd, p*PAGESIZE, SEEK_SET); // moves the fd to the beginning of that page
   read(db->fd, buf, PAGESIZE);
}

// write page from memory buffer to file
void put_page(DB db, PageId p, Page buf) {
   lseek(db->fd, p*PAGESIZE, SEEK_SET);
   write(db->fd, buf, PAGESIZE);
}
```

### Multiple File DBMS

Most DBMSs don't use a single large file for all data. The structure of multi-file disk manager looks like this:

```
| NameMap |
| table 1 pages | (add more)...
| table 2 pages | (add more)...
| table 3 pages | (add more)...
...
```

Structure of PageId for data pages in such systems ...

If system uses one file per table, PageId contains:
 - relation indentifier (which can be mapped to filename)
 - page number (to identify page within the file)

If system uses several files per table, PageId contains:
 - relation identifier
 - file identifier (combined with relid, gives filename) (because there could be multiple files per table, we need to specify which file the page belongs to)
 - page number (to identify page within the file)

**PostgreSQL Storage Manager**

PostgreSQL uses the following file organisation:

![](https://github.com/Magistus-Ninaruru/PostgreSQL/blob/main/images/file_organisation.png)

PGDATA is the top-level directory (root directory) of a PostgreSQL database cluster. Contains all the necessary files and subdirectories for managing the database cluster. Within this directory:
 - **base**: Stores the data files for all the databases in the cluster.
 - **global**: Contains cluster-wide tables and metadata shared across all databases in the cluster.
 - **pg_wal (Write-Ahead Logs)**: Stores WAL (Write-Ahead Log) files, which are critical for database durability and recovery. Log files are named sequentially, e.g., 000000010000000000000001.
 - **pg_tblspc**: Stores symbolic links (symlink1, symlink2) to tablespaces.

PostgreSQL has two basic kinds of files (PostgreSQL identifies relation files via their OIDs):
 - **heap files** containing data (tuples)
 - **index files** containing index entries

**File Descriptor Pool**

Unix has limits on the number of concurrently open files. When a file is needed, check the pool first before attempting to open a new file. Open files are referenced via:

```
typedef int File;
```

A *File* is an index into a table of "virtual file descriptors".

**Virtual File Descriptors (Vfds)**

Vfd typically contains:
 - fd: the actual file descriptor
 - pos: the current position in the file (offset)
 - and many more...

![](https://github.com/Magistus-Ninaruru/PostgreSQL/blob/main/images/vfd.png)

PostgreSQL stores each table
 - in the directory *PGDATA/pg_database.oid*
 - often in multiple data files (aka forks)

```
Oid      | table data pages |
Oid.1    | more table data pages |
Oid_fsm  | free space map |
Oid_vm   | visibility map |
```

Free space map   (Oid_fsm):
 - indicates where free space is in data pages
 - "free" space is only free after **VACUUM** (DELETE simply marks tuples as no longer in use xmax)

Visibility map   (Oid_vm):
 - indicates pages where all tuples are "visible" (visible = accessible to all currently active transactions)
 - such pages can be ignored by VACUUM


































