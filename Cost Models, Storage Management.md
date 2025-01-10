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
   get_page(db, pid, buffer);
   for each tuple in buffer {
      get tuple data and extract name
      add (name) to result tuples
   }
}
```









































































