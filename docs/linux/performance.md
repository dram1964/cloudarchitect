## Monitoring System Performance with SAR

sar (System Activity Reporting) is part of the sysstat package. Configuration 
for sysstat is held in /etc/sysstat/sysstat. To enable the service run `systemctl enable sysstat`
and to start the service run `systemctl start sysstat`.


The sar command can be used to report on activity counters collected in the system activity data
files. These are usually stored in `/var/log/sa` or `/var/log/sysstat` directories. Files will 
be named `saDD` or `sarDD`, where DD represents the day of the month. 

### Basic Commands

To show current CPU activity at 5 second intervals, 10 times, run: `sar 5 10`

To show all cpu data from yesterday, try: `sar -1`

For memory usage: `sar -r -1`

Various flags exist to report different classes of activity:

1. `-B`: displays paging statistics. 
    - the %vmeff shows the efficiency of VM paging: values approaching 100% are good; values less than 30% indicate problems with paging, although a value of 0 indicates that no paging is occurring.
2. `-b`: I/O and transfer statistics (or disk activity).
3. `-d`: activity for each block device. Use with `-j PATH` to show device path names.
4. `-n DEV`: activity for network devices 
5. `-q`: show statistics for queue length and load
6. `-r`: show memory statistics. `%memused` is the total of active memory and buffered/cached memory. `%memcommit` is the amount of memory required by current workload and includes pagefile allocation. `%memcommit` gives a better idea of how much physical memory is in use, and therefore how much is free for new processes, when the figure is below 100%. If the figure is higher than 100% then this suggests that physical memory plus pagefile memory is being used for the current workload. 
7. `-u`: show CPU statistics
8. `-A`: display data for all classes of activity collected

## Free

The `free` command shows memory usage on the system, specifically Total, Used, Free, Cached and 
Available memory. The `available` column indicates how much memory is availabe to new processes
without having to perform paging: that is, it does not count memory that is buffered or cached. 
