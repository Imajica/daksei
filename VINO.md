# VINO-Slurm
## Documentation and logging of VINO-Slurm
### Progress
  VINO-Slurm can successfully launch jobs inside a Virtual Node
### Problems implementing VINO-Slurm
  The main problem is terminating the VM gracefully
- Using ssh from slurmd to the Virtual Machine alters the normal
  launching of a task in slurm. Now we cannot know when the task
  is finished.
- The normal flow forked to slurmstepd which was again forked to
  run a new task. Find the messages to communicate.
### Tasks
1. Find the messages between srun slurmd slurmctld slurmd slurmstepd and srun
   blocks until the task is   
   done while slurmd and slurmctld have to
   continue receiving other tasks.
  - Srun seems to wait until the task completes so I make this hypothesis:
      - srun sends a message to slurmctld in order to terminate the job
      - srun.c file `_launch_app` function `need_mpir` variable is set true
      - `_launch_one_app` function seems to wait for the task to finish  
      however it is not a thread as it seems no thread is created.
      - `fini_srun` function sends a message to slurmctld
