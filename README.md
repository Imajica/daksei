# PROCESS PLACEMENT
## Documentation on plugins and functions required for process placement
_see man pages for more information_  
https://slurm.schedmd.com/selectplugins.html
### cons_res plugin:
plugin needs to be modified according to Topology-aware job mapping
Most of the logic in the select plugin is dedicated to identifying resources to
be allocated to a new job. Input to that function includes: a pointer to the new
job, a bitmap identifying nodes which could be used, node counts (minimum,
maximum, and desired), a count of how many jobs of that partition the job can
share resources with, and a list of jobs which can be preempted to initiate the
new job. The first phase is to determine of all usable nodes, which nodes would
best satisfy the resource requirement. This consists of a best-fit algorithm
that groups nodes based upon network topology (if the topology/tree plugin is
configured) or based upon consecutive nodes (by default). Once the best nodes
are identified, resources are accumulated for the new job until its resource
requirements are satisfied.

### FUNCTIONS :
The following functions are plugins and are defined in src/plugins/cons_res
do not forget they are actually called with 'g' instead of 'p'.

#### extern int select_p_node_init :  
runs in read_config by slurmctld only for initialazing the node structures


#### IMPORTANT-> extern int select_p_job_test :  
the main function of selection, this function in called by slurmctld in
node_scheduler.c by the function **_pick_best_nodes** and job_scheduler.c by
the function **job_start_data**  
 
    /* now that we know that this job can run with the given resources,
	 * let's factor in the existing allocations and seek the optimal set
	 * of resources for this job. Here is the procedure:
	 *
	 * Step 1: Seek idle CPUs across all partitions. If successful then
	 *         place job and exit. If not successful, then continue. Two
	 *         related items to note:
	 *          1. Jobs that don't share CPUs finish with step 1.
	 *          2. The remaining steps assume sharing or preemption.
	 *
	 * Step 2: Remove resources that are in use by higher-priority
	 *         partitions, and test that job can still succeed. If not
	 *         then exit.
	 *
	 * Step 3: Seek idle nodes among the partitions with the same
	 *         priority as the job's partition. If successful then
	 *         goto Step 6. If not then continue:
	 *
	 * Step 4: Seek placement within the job's partition. Search
	 *         row-by-row. If no placement if found, then exit. If a row
	 *         is found, then continue:
	 *
	 * Step 5: Place job and exit. FIXME! Here is where we need a
	 *         placement algorithm that recognizes existing job
	 *         boundaries and tries to "overlap jobs" as efficiently
	 *         as possible.
	 *
	 * Step 6: Place job and exit. FIXME! here is we use a placement
	 *         algorithm similar to Step 5 on jobs from lower-priority
	 *         partitions.
	 */
#### IMPORTANT-> extern int cr_job_test_ :
does most of the real work for select_p_job_test, is called by slurmctld in
src/select/cons_res/job_test.c. This is the function with the 4 steps described
in select_p_job_test.
- #### The steps of cr_job_test :
1. _select_nodes_ function to be investigated - what exactly is a partition?
when is it determined?


#### extern bitstr_t * select_p_resv_test :
Identify the nodes which best satisfy a reservation request taking system
topology into consideration if applicable.

#### extern bitstr_t * select_p_step_pick_nodes :  
Select the "best" nodes for given job step from those available in
a job allocation.

For development functions **init** and **fini** must also be checked.

#### Select_nodes function called in cr_job_test:
Functions called by _select_nodes_ function:
- _get_res_usage
- _can_job_run_on_node 

