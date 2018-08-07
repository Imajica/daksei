 # PROCESS PLACEMENT
## Documentation on plugins and functions
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

#### extern int cr_job_test :
does most of the real work for select_p_job_test

#### extern bitstr_t * select_p_resv_test :
Identify the nodes which best satisfy a reservation request taking system
topology into consideration if applicable.

#### extern bitstr_t * select_p_step_pick_nodes :  
Select the "best" nodes for given job step from those available in
a job allocation.

For development functions **init** and **fini** must also be checked.
