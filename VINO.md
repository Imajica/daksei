# VINO-Slurm
## Documentation and logging of VINO-Slurm
### Problems so far
- Using ssh from slurmd to the Virtual Machine alters the normal
  launching of a task in slurm. Now we cannot know when the task
  is finished.
- The normal flow forked to slurmstepd which was again forked to
  run a new task.
