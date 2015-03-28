# aws-helpers

The following small tasks for AWS has to be improved

1. Wrapper for CF scripts (comments, join lines in user data)
2. DNS update (instance report public and/or local IP to dns to Route53) (instead of elastic IP)
3. Bastion implementation using only userdata with public AMI (create users and assign public keys for them) 
4. Emulation MaxInstancesInService during AutoScalingRollingUpdate.


Limitations of AutoScalingRollingUpdate for AWS AutoScaling Group:
  - PauseTime max is 1 hour (for CreationPolicy max PauseTime is 12 hours but it doesn't help during update)
  - health check of ASG via ELB doesn't work properly (or I wasn't able to setup it, instance becomes ASG-healthy immediatelly even if it's out of service in ELB)
  - cannot provide zero downtime if we need to change ASG itself (because it re-creates ASG and terminate all instance inside)
  - complains about SpotInstances if MinInstancesInService > 0
  - instance with different versions(old and new one) might co-exists in ELB (i.e. there is no MaxInstanceInService option)
  - no mechanism for excluding old instances from ELB(for quick rollback), only terminate during update

Custom RollingUpdate tool should
  input: 
  - Updated launch config  (params)
  - ELB name
  - ASG name

  Steps:
  0. (optional) freeze ASG for scaling
  1. Create new LaunchConfig (with user data)
  2. Assign new LC for ASG
  3. Increase number of instances in ASG
  4. Wait until new instance will be in ELB in InService state
  5. Exclude old one from ELB (don't terminate it)
  ... wait for approval (optional)
  6. Decrease size of ASG (Termination policy should be OldestInstance)

  (#4 together with #5 can be a separate tool)
