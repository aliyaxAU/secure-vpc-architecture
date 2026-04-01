# Monitoring and Flowlogs

This section documents the monitoring, observability, and threat‑detection setup for the environment.

The monitoring implementation focuses on:

- CloudWatch Metrics
- CloudWatch Alarms
- VPC Flow Logs
- CloudWatch Logs Insights
- CloudTrail auditing

## VPC Flow Logs (REJECT‑Only Mode)

VPC Flow Logs capture network traffic metadata. To keep costs low, I configure logs in REJECT‑only mode (in my VPC -> Flow logs Tab -> Create flow log ), which will capture blocked traffic only.

![alt text](<img/vpc flow logs-1.png>)

## CloudWatch Logs Insights Queries

![alt text](<img/cloud watch group-1.png>)

### Query 1 - REJECTs to app servers

This helps debug ALB → EC2 issues, SG mistakes, and health check failures.

`fields @timestamp, srcAddr, dstAddr, dstPort, action
| filter dstPort = 80 and action = "REJECT"
| sort @timestamp desc
| limit 50`

![alt text](<img/CloudWatch log REJECTS.png>)

### Query 2 - ALB health check traffic

This helps to see if the ALB is reaching your instances.

`fields @timestamp, srcAddr, dstAddr, dstPort, bytes, action
| filter dstPort = 80
| sort @timestamp desc
| limit 100`

![alt text](<img/Cloudwawtch log ALB health check traffic.png>)

## CloudWatch Alarms

In this project the following cloudwatch alarms have been implemented for now to keep costs low:

| Resource | Alarm | Threshold | 
-|-|-|
| ALB | HealthyHostCount | < 1 | 
| EC2 | StatusCheckFailed | > 0 | 
| RDS | FreeStorageSpace | < 20% | 


These alarms notify via SNS and provide early warning of availability issues.

### Alarm 1 - ALB HealthyHostCount

**Metric: HealthyHostCount**

- If one instance is unhealthy then ALB still works -> no alarm
- If both instances are unhealthy then ALB cannot serve traffic -> alarm

### Alarm 2 EC2 StatusCheckFailed

**Metric: StatusCheckFailed**

Detects instance hardware/network failure

Metric EC2 status check failed (created 2 for app-server-1 and app-server-2 ec2 instances)

### Alarm 3 RDS Free Storage

Detects application‑level failure

Application‑level failure detected by RDS FreeStorageSpace alarm.

Triggers when RDS free storage drops below 5 GB. Detects application‑level failures such as runaway temporary tables, unbounded inserts, or log accumulation. 
________________________________

## REPLICATION TESTING SCENARIOS

### Incidents EC2 + ALB + Health Checks

In the following scenario to replicate the fail in ALB Target group -> Health checks modify path (to break it) , e.g. `/does-not-exist` . It will make both instances appearing unhealhy in target group.

#### How to handle the incident.

As i configured CloudWatch Alarm for HealthyCostCount and SNS, I received an email regarding fail

![alt text](<img/Test 1 ALB fail email .png>)

Verify in CloudWatch: can see cloud watch alarm is firing

![alt text](<img/Test 1 ALB fail cloud watch alarm.png>)

However my EC2 instances are healthy and running

![alt text](<img/Test1 ALB fail EC2 instances are healthy.png>)

Next i'm verifying the ALB target group 

![alt text](<img/Test 1 ALB fail target group.png>)

I can see both instances are unhealthy.

![alt text](<img/Test1 ALB target group health check tab.png>)

Then i verify health checks tab and observe application logs is in a correct path

Health check path was misconfigured, causing all targets to fail health checks.

![alt text](<img/Test1 ALB target group health check tab-1.png>)

**Recovery**

Restore health path to `/`
Wait for ALB to re‑evaluate
Instances return to Healthy

![alt text](<img/Test 1 ALB back to healthy state.png>)

Alarm return to OK state

**Prevention Strategy (Future consideration)**

- Add automated smoke tests
- Add CI/CD validation for health check paths
- Add alarms for sudden spikes in 5xx errors

### Incident 2: EC2 Instance Status Check Failure

I should receive CloudWatch alarm if there any incidents thanked to EC2 Status Check Failed alarm.

This time I will try to break network by running in EC2 instance app-server-1:

`sudo ip link set enX0 down`

#### How to handle the incident

I received an email regarding ALARM: "EC2 Status check failed App 1"

![alt text](<img/Test 2 received an email.png>)

When i click cloudwatch alarm link from email 

![alt text](<img/Test 2 Cloudwatch alarm.png>)

And ALB the instace is unhealthy

![alt text](<img/Test 2 ALB ec2 failed.png>)


I'm checking affected EC2 instance and can see 1/checks passed, clicking on Status check and redireirng to cloudtrail -> event history. Filtering by Resource Name : `<EC2_INSTANCE_ID>`, filtered by today date. Checking whether AWS (or a human using AWS) did something that could break anythng.

Searching for eventss, e.g. 
- ModifyNetworkInterface
- DetachNetworkInterface
- AttachNetworkInterface
- AssignPrivateIpAddresses
- UnassignPrivateIpAddresses

If any of these happened -> AWS-side change caused the outage.

 Security group or NACL changes
- AuthorizeSecurityGroupIngress
- RevokeSecurityGroupIngress
- CreateNetworkAclEntry
- DeleteNetworkAclEntry
If these changed -> traffic could be blocked

 Instance lifecycle events
- RebootInstances
- StopInstances
- StartInstances
- TerminateInstances
If someone rebooted/stopped the instance -> explains the outage.

The only event recorded during the incident window was StartInstances, which occurred earlier in the day. No events were found related to ENI modifications, security group changes, route table updates, or instance lifecycle operations. This confirms that the failure originated inside the EC2 instance’s operating system rather than from AWS infrastructure or configuration changes.

![alt text](<img/test 2 cloud trail.png>)

**Recovery**

Rebooted the EC2 instance (app-server-1) from the AWS console to reinitialize the network interface. After 2 minutes:

Status check 2/2 passed:

![alt text](<img/test2 after reboot ec2 .png>)

ALB instances both are healthy

![alt text](<img/test 2 aftre reboot target group health.png>)

Cloud watch alarm OK

![alt text](<img/Test 2 after reboot cloud watch OK.png>)

The incident was caused by a network stack failure that made the EC2 instance unreachable despite (not possible to SSH etc) healthy AWS infrastructure, and normal operation was restored after rebooting the instance to reinitialize its networking.
This incident demonstrates how OS-level networking failures can render an EC2 instance unreachable even when AWS infrastructure is functioning normally, and highlights the importance of combining AWS diagnostics (CloudTrail, ALB health checks, CloudWatch alarms)

### Incident 3 RDS FreeStorageSpace Alarm

The CloudWatch FreeStorageSpace alarm for the RDS MySQL instance in the secure private VPC entered ALARM state

Symptoms
- CloudWatch alarm entered ALARM state
- FreeStorageSpace metric showed a steep downward trend
- Application latency increased slightly
- Write operations slowed due to increased I/O

In this TESTING scenario To validate the alarm, I temporarily adjusted the threshold to match the current free storage level. This allowed me to confirm that the alarm transitions correctly into ALARM state and notifies the on‑call channel

What will happen:

- My free space is ~19.5 GB
- Threshold changed to 20 GB
- 19.5 < 20 → ALARM

#### How to handle the incident

I'm receiving an alarm ALARM: "RDS storage space" 

![alt text](<img/Test 3 RDS alarm email.png>)

Checking CloudWatch RDS storage space:

![alt text](<img/Test 3 RDS alarm cloudwatch graph-1.png>)

Verified the metric showed FreeStorage storage space breaching the threshold.

### Metrics analytics

![alt text](<img/Test 3 RDS metrics in cloudwatch.png>)

**Recovery**

- Restore the real threshold:
- 5 GB → 5,368,709,120 bytes

![alt text](<img/Test 3 RDS alarm back ok.png>)
