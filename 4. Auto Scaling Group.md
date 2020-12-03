Auto-scaling groups

Auto scaling spins up new instances which might have new IP addresses. ELB needs to sit on top of auto scaling groups to ensure fault tolerance. Auto scaling works in different AZs with ELB but not in different regions. To cover different regions use route53. 

You can launch and automatically scale a fleet of On-Demand Instances and Spot Instances within a single Auto Scaling group. In addition to receiving discounts for using Spot Instances, you can use Reserved Instances or a Savings Plan to receive discounted rates of the regular On-Demand Instance pricing. Auto scaling groups can be created using launch templates/ configurations. Scheduled scaling may be used for predictable loads.

Auto scaling may be triggered by: capacity settings, Health check replacements, and scaling policies.
 

You may also specify maximum instance lifetime to replace instances. Min 7 days. In capacity setting if a fixed number of instances are needed, this can be achieved by using the same value for desired, min, max. 



Scanning policies: They never exceed capacity settings
Target tracking scaling—Increase or decrease the current capacity of the group based on a target value for a specific metric. 
Step scaling—Increase or decrease the current capacity of the group based on a set of scaling adjustments, known as step adjustments, that vary based on the size of the alarm breach. No cooldown period.
Simple scaling—Increase or decrease the current capacity of the group based on a single scaling adjustment. Default cooldown period 300 seconds (can be modified)

Cooldown period: Amount of wait time after one scaling  adjustment. (Not applicable when instance is unhealthy obviously)

Basically step scaling allows multiple conditions/step wise scaling and simple scaling allows only one.
To illustrate how multiple policies work together, consider an application that uses an Auto Scaling group and an Amazon SQS queue to send requests to a single EC2 instance. To help ensure that the application performs at optimum levels, there are two policies that control when the Auto Scaling group should scale out. One is a target tracking policy that uses a custom metric to add and remove capacity based on the number of SQS messages in the queue. The other is a step scaling policy that uses the Amazon CloudWatch CPUUtilization metric to add capacity when the instance exceeds 90 percent utilization for a specified length of time.
When there are multiple policies in force at the same time, there's a chance that each policy could instruct the Auto Scaling group to scale out (or in) at the same time. When these situations occur, Amazon EC2 Auto Scaling chooses the policy that provides the largest capacity for both scale out and scale in.
Monitoring: Configure Auto Scaling groups to determine the health status of an instance using Amazon EC2 status checks (default), Elastic Load Balancing health checks, or custom health checks.
Horizontal scaling is adding redundancy. Vertical scaling in increasing resources which results in downtime. E.g. changing instance type of EC2 or increasing memory size in EBS in vertical scaling. Adding a new EBS volume would be horizontal scaling.

Lifecycle hooks: Auto scaling groups can pause the instance during scale-in/out to perform certain actions. let's say that your newly launched instance completes its startup sequence and a lifecycle hook pauses the instance. While the instance is in a wait state, you can install or configure software on it, making sure that your instance is fully ready before it starts receiving traffic. For another example of the use of lifecycle hooks, let's say that when a scale-in event occurs, the terminating instance is first deregistered from the load balancer (if the Auto Scaling group is being used with Elastic Load Balancing). Then, a lifecycle hook pauses the instance before it is terminated. While the instance is in the wait state, you can, for example, connect to the instance and download logs or other data before the instance is fully terminated. As the instance is first deregistered form load balancer sticky sessions may be lost.

Hands-on for ASG/EFS

Step 1: Create a launch template with Amazon linux AMI and a security group that allows SSH. Attach the instance profile/role with SSM policy.
Step 2:  Create Auto Scaling Group with the launch template above. Use default VPC. keep a fixed scaling setting to begin with. 
Step 3: When instances are created, they should also be visible in Session manager. 
Step 4: Delete one/both and see fixed scaling.
Step 5: Create EFS. login with the session manager and mount EFS. security groups chosen for EFS in each subnet must allow NFS from instances for this to work.

An Auto Scaling instance that has just come into service needs to warm up before it can pass the health check. Amazon EC2 Auto Scaling waits until the health check grace period ends before checking the health status of the instance. Amazon EC2 status checks and Elastic Load Balancing health checks can complete before the health check grace period expires. However, Amazon EC2 Auto Scaling does not act on them until the health check grace period expires. To provide ample warm-up time for your instances, ensure that the health check grace period covers the expected startup time for your application. If you add a lifecycle hook, the grace period does not start until the lifecycle hook actions are completed and the instance enters the InService state.
Instance termination policy for Scaling in:
The default termination policy behavior is as follows:
Determine which Availability Zones have the most instances, and at least one instance that is not protected from scale in.
Determine which instances to terminate so as to align the remaining instances to the allocation strategy for the On-Demand or Spot Instance that is terminating. This only applies to an Auto Scaling group that specifies allocation strategies.
For example, after your instances launch, you change the priority order of your preferred instance types. When a scale-in event occurs, Amazon EC2 Auto Scaling tries to gradually shift the On-Demand Instances away from instance types that are lower priority.
Determine whether any of the instances use the oldest launch template or configuration:
[For Auto Scaling groups that use a launch template]
Determine whether any of the instances use the oldest launch template unless there are instances that use a launch configuration. Amazon EC2 Auto Scaling terminates instances that use a launch configuration before instances that use a launch template.
[For Auto Scaling groups that use a launch configuration]
Determine whether any of the instances use the oldest launch configuration.
After applying all of the above criteria, if there are multiple unprotected instances to terminate, determine which instances are closest to the next billing hour. If there are multiple unprotected instances closest to the next billing hour, terminate one of these instances at random.
