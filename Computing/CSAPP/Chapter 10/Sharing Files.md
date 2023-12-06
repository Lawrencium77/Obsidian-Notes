This section is *super* related to a [part of the Linux Documentation Project](4%20-%20Processes#files) book. I actually prefer how it is described in that book over CSAPP. I therefore use these notes to make some clarifying points.

To summarise TLDP, each `task_struct` points to a `files_struct`, which contains all information on the files that a process is using. The `files_struct` contains a **descriptor table**, whose entries are indexed by file descriptors. Each entry points to a `file` struct.

Each `file` struct can be shared between processes. 

Finally, each `file` struct contains an entry that points to a file's `inode`. The overall picture is:
![](_attachments/Screenshot%202022-12-12%20at%2017.46.05.png)

To reiterate: `task_struct` and `files_struct` are process specific, whereas the `file` and `inode` structs are shared between processes.

The details provided by CSAPP are a little different, but the picture is still the same. It separates out the concept of a **descriptor table**, and adds the extra layer of a **v-node table**, which contains some metadata and a pointer to a process inode: ^c6d1a0

![](_attachments/Screenshot%202023-06-04%20at%2013.37.59.png)

It also raises an interesting point: multiple descriptors can point to the same file through different `file` structs. This might happen, for example, if you open the same file twice. The key idea is that each descriptor has its own distinct file position (given by `f_pos`), so different reads on different descriptors can fetch data from different locations in the file.

