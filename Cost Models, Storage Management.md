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

## Buffer Pool

Aim of buffer pool: hold pages read from database files, for possible re-use.

Buffer pool operations:   (both take single PageID argument)
 - *request_page(pid)*: Request a page based on its pid and return the bufID in the buffer pool. If the requested page is already in the buffer pool, then no need to read it from disk again.
 - *release_page(pid)*: Mark the page as not being used (i.e. pin count --).
 - *mark_page*: Set dirty bit on a specified page.
 - *flush_page*: Write the specified page to disk.

Buffer pool data structures:

```
Page frames[NBUFS]         // frames is an array of pages
FrameData directory[NBUFS] // directory stores the metadata of pages
Page is byte[BUFSIZE]      // a page is just an array of bytes
```

The buffer pool looks like this:

```
directory    | info about frame 0 | info about frame 1 | ...
                       [0]                  [1]
frames       |   data or empty    |   data or empty    | ...
                       [0]                  [1]
```

For each frame, we need to know:   (FrameData)
 - which Page it contains, or whether empty/free
 - whether it has been modified since loading (**dirty bit**)
 - how many transactions are currently using it (**pin count**)
 - time-stamp for most recent access (assists with replacement)

How scans are performed without Buffer Pool:

```
Buffer buf;
int N = numberOfBlocks(Rel);
for (i = 0; i < N; i++) {
   pageID = makePageID(db,Rel,i);
   getBlock(pageID, buf);
   for (j = 0; j < nTuples(buf); j++)
      process(buf, j)
}
```

Requires N page reads. If we read it again, N page reads.

How scans are performed with Buffer Pool:

```
Buffer buf;
int N = numberOfBlocks(Rel);
for (i = 0; i < N; i++) {
   pageID = makePageID(db,Rel,i);
   bufID = request_page(pageID);
   buf = frames[bufID]
   for (j = 0; j < nTuples(buf); j++)
      process(buf, j)
   release_page(pageID);
}
```

Requires N page reads on the first pass. If we read it again, 0 ≤ page reads ≤ N, because there may be some pages already loaded in the buffer pool.

Evicting a page ... 
 - find frame(s) preferably satisfying
    - pin count = 0   (i.e. nobody using it)
    - dirty bit = 0   (not modified)
 - if selected frame was modified, flush frame to disk
 - flag directory entry as "frame empty"
 - If multiple frames can potentially be released, we need a policy to decide which is best choice

**Page Replacement Policies**

Several schemes are commonly in use:
 - **Least Recently Used (LRU)** (items in cache are ordered by their recent usage)
 - **Most Recently Used (MRU)** (items in cache are ordered by their recent usage)
 - **First in First Out (FIFO)** (items in cache are ordered by their insertion time)
 - **Random**

Note: LRU / MRU require knowledge of when pages were last accessed. How to keep track of "last access" time?
 - Timestamp: maintain a timestamp for each page
 - Linked list: the most recently accessed page is moved to the *front* of the list

**Sequential flooding**: suppose the buffer pool has n frames, we perform a sequential scan, LRU, n < b, then all scans costs b reads.

**Effect of Buffer Management**

Consider a query to find customers who are also employees:

```
select c.name
from   Customer c, Employee e
where  c.ssn = e.ssn;
```

This might be implemented inside the DBMS via **nested loops**:

```
for each tuple t1 in Customer {
    for each tuple t2 in Employee {
        if (t1.ssn == t2.ssn)
            append (t1.name) to result set
    }
}
```

In terms of page-level operations, the algorithm looks like:

```
Rel rC = openRelation("Customer");
Rel rE = openRelation("Employee");
for (int i = 0; i < nPages(rC); i++) {
    PageID pid1 = makePageID(db,rC,i);
    Page p1 = request_page(pid1);
    for (int j = 0; j < nPages(rE); j++) {
        PageID pid2 = makePageID(db,rE,j);
        Page p2 = request_page(pid2);
        // compare all pairs of tuples from p1,p2
        // construct solution set from matching pairs
        release_page(pid2);
    }
    release_page(pid1);
}
```

## Page/Tuple Management

Important terminologies:
 - *Record* = sequence of bytes stored on disk (data for one tuple)
 - *Tuple* = "interpretable" version of a *Record* in memory
 - *TupleId* = index of record within page = *tid*
 - *RecordId* = *(PageId, TupleId)* = *rid*

### Page Format

For **fixed-length records**, use **record slots**.
 - insert: place new record in first available slot
 - delete: mark slot as free, or set *xmax*

Important: xmin/xmax are fields used to manage tuple/record visibility in MVCC
 - xmin: the transaction id of the transaction that *inserted* the tuple/record
 - xmax: the transaction id of the transaction that *deleted/update* the tuple/record

![](https://github.com/Magistus-Ninaruru/PostgreSQL/blob/main/images/record_slot.png)

For **variable-length records**, must use **record directory**, where directory[i] gives location within page of i th record.

An important aspect of using record directory:
 - location of tuple within page can change, tuple index does not change (we refer to tuple index within directory as   *TupleId tid*)

Issue with variable-length records
 - managing space withing the page (esp. after deletions)
 - recording used and unused regions of the page

Possibilities for handling free-space within block:
 - compacted (one region of free space)
 - fragmented (distributed free space)

In practice, a combination is useful:
 - normally fragmented (cheap to maintain)
 - compacted when needed (e.g. record won't fit)

**Storage Utilisation**

How many records can fit in a page? (denoted c = capacity)

Depends on:
 - page size ... typical values: 1KB, 2KB, 4KB, 8KB
 - record size ... typical values: 64B, 200B, app-dependent
 - page header data ... typically: 4B - 32B
 - slot directory ... depends on how many records

We typically consider average record size (R)
 - Given c, *HeaderSize + c*SlotSize + c*R  ≤  PageSize*

### Overflows

Sometimes, it may not be possible to insert a record into a page:
(1) no free-space fragment large enough
(2) overall free-space in page is not large enough
(3) the record is larger than the page
(4) no more free directory slots in page

Overflow pages for full buckets in a hashed file:

![](https://github.com/Magistus-Ninaruru/PostgreSQL/blob/main/images/overflow1.png)

Overflow file for very large records and BLOBs:

![](https://github.com/Magistus-Ninaruru/PostgreSQL/blob/main/images/ov2.png)

Note:
 - The **BLOB** (**Binary Large Object**) data type is used in databases to store large amounts of binary data, such as images, videos, audio, and other multimedia files, as well as binary executables or encrypted data. It is designed to handle unstructured data that does not fit into traditional text or numeric data types.
 - The record that is too large to fit in a data page stores metadata and a pointer to the overflow file, instead of actual data.
 - **OV**: overflow flag **offset**: the offset in the overflow file **length**: the length of data stored in the ov file

### PostgreSQL Page Representation

PostgreSQL page layout:

![](https://github.com/Magistus-Ninaruru/PostgreSQL/blob/main/images/page_layout.png)

PostgreSQL has two kinds of pages:
 - **heap pages** which contain tuples
 - **index pages** which contain index entries

Both kinds of pages have the same page layout. They both have similar page management mechanism. On important difference is that:
 - index entries tend be smaller than tuples
 - can typically fit more index entries per page

**TOAST Files**

Each data file has a corresponding **TOAST** (**The Oversized Attribute Storage Technique**) file (if needed):

![](https://github.com/Magistus-Ninaruru/PostgreSQL/blob/main/images/TOAST.png)

Key ideas:
 - Tuples in data pages contain rids for long values.
 - PostgreSQL has a size limit for rows (typically 8 KB, the page size), and TOAST is used to efficiently store and manage data values that *exceed this limit*, such as large strings, arrays, or binary objects.
 - PostgreSQL first tries to compress large data values using techniques like pglz. If the compressed value fits within the row size limit, it's stored inline. If the compressed data still doesn't fit, PostgreSQL moves the oversized value to a separate TOAST table. The original row stores only a pointer to the external TOAST data.

**Differences Between TOAST and BLOB**

![](https://github.com/Magistus-Ninaruru/PostgreSQL/blob/main/images/toast_and_blob.png)



















