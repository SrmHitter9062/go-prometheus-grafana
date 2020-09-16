# go-prometheus-grafana
This is about data or information is collected by prometheus and sent to grafana


1. https://povilasv.me/prometheus-go-metrics/


On initialisation client_golang registers 2 Prometheus collectors:

## Process Collector –
 which collects basic Linux process information like CPU, memory, file descriptor usage and start time.
## Go Collector – 
which collects information about Go’s runtime like details about GC, number of gouroutines and OS threads.
Process Collector
What this collector does is reads proc file system. proc file system exposes internal kernel data structures, which is used to obtain information about the system.1

So Prometheus client reads /proc/PID/stat file, which looks like this:

1 (sh) S 0 1 1 34816 8 4194560 674 43 9 1 5 0 0 0 20 0 1 0 89724 1581056 209 18446744073709551615 94672542621696 94672543427732 140730737801568 0 0 0 0 2637828 65538 1 0 0 17 3 0 0 0 0 0 94672545527192 94672545542787 94672557428736 140730737807231 140730737807234 140730737807234 140730737807344 0
You can get human readable variant of this information using cat /proc/PID/status.

process_cpu_seconds_total – it uses utime – number of ticks executing code in user mode, measured in jiffies, with stime – jiffies spent in the system mode, executing code on behalf of the process (like doing system calls). A jiffy is the time between two ticks of the system timer interrupt. 2

process_cpu_seconds_total equals to sum of utime and stime and divide by USER_HZ. This makes sense, as dividing number of scheduler ticks by Hz(ticks per second) produces total time in seconds operating system has been running the process. 3

process_virtual_memory_bytes – uses vsize – virtual memory size is the amount of address space that a process is managing. This includes all types of memory, both in RAM and swapped out.

process_resident_memory_bytes – multiplies rss – resident set memory size is number of memory pages the process has in real memory, with pagesize 4. This results in the amount of memory that belongs specifically to that process in bytes. This excludes swapped out memory pages.

process_start_time_seconds – uses start_time – time the process started after system boot, which is expressed in jiffies and btime from /proc/stat which shows the time at which the system booted in seconds since the Unix epoch. start_time is divided by USER_HZ in order to get the value in seconds.

process_open_fds – counts the number of files in /proc/PID/fd directory. This shows currently open regular files, sockets, pseudo terminals, etc.

process_max_fds – reads /proc/{PID}/limits and uses Soft Limit from “Max Open Files” row. The interesting bit here is that /limits lists Soft and Hard limits.
As it turns out, the soft limit is the value that the kernel enforces for the corresponding resource and the Hard limit acts as a ceiling for the soft limit.
An unprivileged process may only set its soft limit to a value up to the hard limit and (irreversibly) lower its Hard limit. 5

In Go you can use err = syscall.Setrlimit(syscall.RLIMIT_NOFILE, &syscall.Rlimit{Cur: 9, Max: 10}) to set your limit.
