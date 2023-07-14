# Perf stat

`perf stat` is an easy way to see how many event triggered in a certain period.

For example to see the count of all scheduler events over 1 second:
```
$ sudo perf stat -e 'sched:*' -a -- sleep 1

 Performance counter stats for 'system wide':

                 0      sched:sched_kthread_stop
                 0      sched:sched_kthread_stop_ret
               644      sched:sched_waking
               644      sched:sched_wakeup
                 0      sched:sched_wakeup_new
             1,257      sched:sched_switch
                91      sched:sched_migrate_task
                 0      sched:sched_process_free
                 1      sched:sched_process_exit
                 0      sched:sched_wait_task
                 1      sched:sched_process_wait
                 0      sched:sched_process_fork
                 0      sched:sched_process_exec
                 0      sched:sched_stat_wait
                 0      sched:sched_stat_sleep
                 0      sched:sched_stat_iowait
                 0      sched:sched_stat_blocked
       202,981,694      sched:sched_stat_runtime
                 0      sched:sched_pi_setprio
                 0      sched:sched_process_hang
                 0      sched:sched_move_numa
                 0      sched:sched_stick_numa
                 0      sched:sched_swap_numa
               200      sched:sched_wake_idle_without_ipi

       1.006735679 seconds time elapsed
```

This gives you some insight into how much overhead profiling on one of these
events will cause. If you trigger on `sched_switch` (context switch) you can
expect ~1200 events per second.

To see all possible events you can trigger on run `sudo perf list`.
