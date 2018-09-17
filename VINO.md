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
/* VINO-Slurm support create the configuration file launching the Virtual Machines */
static uint16_t** _create_vm_conf_file(struct step_record *step_ptr, char *nodelist,char *buffer){
	char *name;
	FILE *fp;
	char ip_address[32];
	char filename[128]="/home/vardas/vm_configs/vm_map";
	char job_buffer[8];
	int i,id,index;
	uint16_t **list;
	uint16_t j,pos;
	uint32_t offset = 0;
	uint32_t k;
	slurm_step_layout_t *layout = step_ptr->step_layout;
	sprintf(job_buffer,"%d",step_ptr->job_ptr->job_id);
	strcat(filename,job_buffer);
	strcat(filename,".conf");
	fp=fopen(filename,"w");
	if(fp==NULL){
		error("VINO-SLURM ERROR: Wrong path for VM File");
		exit(0);
	}
	fprintf(fp,"<VM_LIST>\n\t\t<VM>\n");
	list = xmalloc((layout->node_cnt) * sizeof(uint16_t*));
	name = strtok(nodelist,",");
	for (i=0; i<layout->node_cnt; i++){
		if(layout->tasks[i] > 0){
			_hostname_to_ip(name, ip_address);
			list[i]=xmalloc(sizeof(uint16_t)* (layout->tasks[i]+1));
			for(k=0; k < node_record_count; k++){
				if(xstrcmp(node_record_table_ptr[k].name,name)==0)
					break;
			}
			for(j=0; j<layout->tasks[i]; j++)
				list[i][j] = 0;
			index = 1;
			for(j=0; j<layout->tasks[i]; j++){
				for(pos=0;  pos<node_record_table_ptr[k].cpus; pos++){
					if(vm_images[k][pos]){
						vm_images[k][pos] = step_ptr->job_ptr->job_id;
                        /* allocate the VM image, exit
						 * the loop and write it to file */
						list[i][index] = pos;
						index++;
						break;
					}
				}
				fprintf(fp,"\t\t\t<item id=\"%u\" hm=\"%s\" mem_mb=\"%u\" cores=\"%u\" "
						"arch=\"%s\"></item>\n",
						pos,ip_address,step_ptr->pn_min_memory,
						step_ptr->cpus_per_task,VM_ARCH);
			}
			list[i][0]=index-1;
		}
		name = strtok(NULL,",");
	}
	/*for (i = first_bit; i <= last_bit; i++){
		if (bit_test(ptr->step_node_bitmap, i)) {
			pos = bit_get_pos_num(ptr->job_ptr->job_resrcs->node_bitmap,i);
			if(ptr->job_ptr->job_resrcs->cpus[pos] > 0){
				_hostname_to_ip(name, ip_address);
				inx = 0;
				list[pos]=xmalloc(sizeof(uint16_t)* node_record_table_ptr[pos].cpus);
				for(j=0; j<ptr->job_ptr->job_resrcs->cpus[pos]; j=j+ptr->cpus_per_task){
					for(k=0; k < node_record_table_ptr[pos].cpus; k++){
						if(vm_images[pos][k]){
							vm_images[pos][k] = 0;
							printf("POS: %d\tINDEX: %d\n",pos,inx);
							list[pos][inx] = k;
							inx++;
							break;
						}
					}
					fprintf(fp,"\t\t\t<item id=\"%u\" hm=\"%s\" mem_mb=\"%u\" cores=\"%u\" "
							"arch=\"%s\"></item>\n",
							k,ip_address,ptr->pn_min_memory,
							ptr->cpus_per_task,VM_ARCH);
				}
			}
			name = strtok(NULL,",");
		}

	}*/
	fprintf(fp,"\t\t</VM>\n</VM_LIST>\n");
	fclose(fp);
	strcpy(buffer,filename);
	return list;
}
