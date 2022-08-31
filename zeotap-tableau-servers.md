Tableau-Cluster incident

Date: 2022-08-24

Authors: Shubham Awasthi, Arpit shridhar, Mahesh Saidu

Status: Complete

Summary: Tableau application down and Infra team is unable to SSH in all 3 tableau VMs -


VM Names:
tableau-xxx-xxx-eu-1
tableau--xxx-xxx-eu-2
tableau--xxx-xxx-eu-3

Impact: Unknown

Root Causes: Some permission changes should be performed on a perticular log directory inside the /Var folder in order to run a harness shell script, however we made the permission changes for /var directory recursively, which affects the file permssions for tableu services and ssh service inside all 3 nodes of Tableau servers.


Trigger: Manual execution

Resolution: Replicated all the file permssions from a demo server where Tableau service was running, we checked all the files/directories permissions present under the /var in demo server and set the same permission on Node1, Node2 and Node3.



Detection: Rackspace Infra team detected that tableau servers are un-responsive and team is unable to SSH Tableau nodes.


Action Items: 

Day1-

Raised a support case with MGCP team to stating the SSH issue - Aravind Informed the engagement manager over the phone and explained that MGCP is not considering this case as P1.

Investigated the serial console logs with zeotap team and found that the permission/ownership for /var/empty/sshd is not correct.

Created a bash startup script to add a temp user and change the permission for /var/empty/sshd.Stopped all the VMs and executed the startup script on all 3 nodes through Metadata scripts option present in GCP VMs.

Rebooted the machine and from now, we were able to SSH Tableau nodes.

Checked the application status - tsm status -v and found that Application is not running on VM.



Day2-

Removed all ACL's from /var/opt/xxx/xxx/xxx/backgrounder (setfacl -b -R var).

Created a clone server from an older snapshot (Jan 2022).Checked permssion for /var files and folder on clone server and replicated the same on Node1, Node2 and Node3.

Checked permission for /var files and folder on clone server and replicated the same on Node1, Node2 and Node3.

Checked the application status again and found that it is not running.



Day3-

Had a call with tableau support where support person checked the application controller status and logs, and found that, admin controller is not working over the node1 so we have been adviced by support to re-install the tab admin controller on node1 and check the admin controller logs after completion of installation to identify if there are any permission exceptions in the admin controller logs under /var/opt/tableau/xxx/xxx/xxx/logs/tabadmincontroller.

Changed the permissions for particular directories which were listed as exceptions in the admin controller logs. At this stage there were no more permission exceptions however a new error occured-
Caused by: java.lang.RuntimeException: org.apache.commons.exec.ExecuteException: Error while running openssl. Error Code 1. See output above for details. (Exit value: 1)

This error was troubleshooted by client over the weekend and they made few more changes to start the tableau services on node1 and tableau services were showing up and running 89/90 in tableau GUI console.



Day4 and Day5-

In further troubleshooting we found tableau Gateway service was in error state.

Checked the apache startup and error logs and found proxy mutex error and also some permission related exceptions for apache_access_logs as it was owned by "root"  instead of "tableau" user.

Changed the ownership for apache_access_logs and restarted all three nodes.
After making these changes, checked the application status on node1, node2 and node3 which was running so we requested BI team to verify the application data.

Received the formal confirmation from BI team on 30 August at 4PM IST that all services are running without any issues.




Lessons Learned-

Backup policy should be in place, automated daily or weekly snapshots should be always scheduled for critical applications.

Although permssion had changed on a day before (23th Aug'22) but we observed the issue on 24th Aug when we tried to SSH one of the cluster's node.
Which makes clear that Monitoring policies are not configured. There should be monitoring alerts/Uptime checks to monitor the application status.

Never perform changes in Production environment as "production changes" should be always performed in well planned "change-management" process with a roll-back plan if anything goes wrong in production.

Some linux commands are too small but very powerful, so please always execute them in a test environment unless you dont know the impact/output of the command.

Never change the file/folder permissions which has application files, as It can intrupt the running services.

While performing the production changes, always go in a sequence, for example -
If same changes need to performed in a cluster of 3 nodes - 

1. Take the latest snapshot of 1st node.
2. Peform the neccessary changes.
3. Verify the application and services status.
4. If everything is working, only then move to the 2nd node.
5. Take the latest snapshot of 2nd node.
6. Peform the neccessary changes.
7. Verify the application and services status on 2nd node and continue in the same sequence ........!








