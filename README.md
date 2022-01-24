# aws-arch

In general the DR strategies needs to be tested every 6 months.

https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-options-in-the-cloud.html

AWS provides 4 types of DR strategies 

1)Backup and restore : 
This approach can  be used to mitigate against a regional disaster by replicating data to other AWS Regions
you must redeploy the infrastructure, configuration, and application code in the recovery Region. 
To enable infrastructure to be redeployed quickly without errors, you should always deploy using infrastructure as code (IaC) using services such as AWS CloudFormation
Need to create snapshots for all data devices such as EBS, RDS and put them in S3 bucket which has CRR enabled and versioning needs to be enabled. Can also use RDS replicas in other region.
Need to build IaC to spin up environment in other region with the backups from original infrastructure.
You can back up Amazon EC2 instances used by your workload as Amazon Machine Images (AMIs). The AMI is created from snapshots of your instance's root volume and any other EBS volumes attached to your instance. You can use this AMI to launch a restored version of the EC2 instance. This AMI can be copied accross regions.
You can implement automatic restore to the DR region using the AWS SDK to call APIs for AWS Backup using AWS lambda

Figure 7 - Backup and restore architecture
Figure 8 - Restoring and testing backups


Advantages: cost efficient
Cons: latency is more

Less strigent RPO and RTO

2) Pilotlight: 

With the pilot light approach, you replicate your data from one Region to another and provision a copy of your core workload infrastructure.
application servers, are loaded with application code and configurations, but are switched off and are only used during testing or when disaster recovery failover is invoked
This recovery option requires you to change your deployment approach.You need to make core infrastructure changes to each Region and deploy workload (configuration, code) changes simultaneously to each Region. 
Can use read replicas for supported databases.

Figure 9 - Pilot light architecture

Pros: little more cost compared to #1 approach
cons: Latency is still there because multiple steps needed.
Less strigent RPO and RTO

3) Warn Standby:

The warm standby approach involves ensuring that there is a scaled down, but fully functional, copy of your production environment in another Region.
This approach also allows you to more easily perform testing or implement continuous testing to increase confidence in your ability to recover from a disaster.
The pilot light approach requires that you to “turn on” servers, possibly deploy additional (non-core) infrastructure, and scale up, whereas Warm Standby only requires you to scale up (everything is already deployed and running).
All of the AWS services covered under backup and restore and pilot light are also used in warm standby for data backup, data replication, active/standby traffic routing, and deployment of infrastructure including EC2 instances.
AWS auto scaling can be used to slaceup the infrastructure as needed in DR region.

Figure 11 - Warm standby architecture

Pros: Infra is always available with less efficiency.
Cons: Costly
More strigent RTO and RPO

4) Multistite active/active
You can run your workload simultaneously in multiple Regions as part of a multi-site active/active or hot standby active/passive strategy. 
Multi-site active/active serves traffic from all regions to which it is deployed, whereas hot standby serves traffic only from a single region, and the other Region(s) are only used for disaster recovery. 
With a multi-site active/active approach, users are able to access your workload in any of the Regions in which it is deployed. 
This approach is the most complex and costly approach to disaster recovery, but it can reduce your recovery time to near zero for most disasters.
With multi-site active/active, because the workload is running in more than one Region, there is no such thing as failover in this scenario.

Maynot be possible because multi-region write DBs/EBS are not available.

Figure 12 - Multi-site active/active architecture 

Pros: Zero downtime
Cons: two time costly to the current setup. Not possible with current postgresql DB.
