Zeotap-Tableau-Cluster incident

Date: 2022-08-24

Authors: Shubham Awasthi, Arpit shridhar, Mahesh Saidu

Status: In-progress

Summary: Tableau application down and Infra team is unable to SSH in all 3 tableau VMs -

Project:
zeotap-prod-corpwebsite

VM Names:
tableau-server-prod-eu-1
tableau-server-prod-eu-2
tableau-server-prod-eu-3

Impact: Unknown

Root Causes: Some permission changes should be performed on a perticular log directory inside the /Var folder in order to run a harness shell script, however we made the permission changes for /var directory recursively, which affects the file permssions for tableu services and ssh service inside all 3 nodes of Tableau servers.


Trigger: Manual execution

Resolution: Replicated all the file permssions from a demo server where Tableau service was running, we checked all the files/directories permissions present under the /Var and set the same permission on Node1, Node2 and Node3.



Detection: Rackspace Infra team detected that tableau servers are un-responsive and team is unable to SSH Tableau nodes.

Action Items:^165^

Action Item	Type	Owner

Day1
Raised a support case with MGCP team to stating the SSH issue - Aravind (Zeotap)
Informed the engagement manager over the phone and expalined that MGCP is not cosidering this case as P1 - Shubham (Rackspace)
Investigated the serial console logs with zeotap team and found that the permssion/ownership for /var/empty/sshd is not correct (Zeotap/Rackapce) 

<img width="1422" alt="image" src="https://user-images.githubusercontent.com/106727340/187621975-bda5c3e4-9fc9-4028-a53f-77cfa1c1e8db.png">

Created a bash startup script to add a temp user and change the permssion for /var/empty/sshd (Zeotap/Rackspace)
Stopped all the VMs and executed the startup script on all 3 nodes through Metadata scripts option present in GCP VMs. (Zeotap/Rackspace)
Rebooted the machine and from now we were able to SSH Tableau nodes. (Zeotap/Rackspace)
Checked the application status - tsm status -v and found that Application is not running on VM. (Zeotap)

Day2
We created a clone server from an older snapshot (Jan 2022)
Checked permssion for /var files and folder on clone server and replicated the same on Node1 Node2 and Node3



Lessons Learned
What went well

Monitoring quickly alerted us to high rate (reaching ~100%) of HTTP 500s
Rapidly distributed updated Shakespeare corpus to all clusters
What went wrong

We're out of practice in responding to cascading failure
We exceeded our availability error budget (by several orders of magnitude) due to the exceptional surge of traffic that essentially all resulted in failures
Where we got lucky [166]

Mailing list of Shakespeare aficionados had a copy of new sonnet available
Server logs had stack traces pointing to file descriptor exhaustion as cause for crash
Query-of-death was resolved by pushing new index containing popular search term
Timeline [167]
2015-10-21 (all times UTC)

14:51 News reports that a new Shakespearean sonnet has been discovered in a Delorean's glove compartment
14:53 Traffic to Shakespeare search increases by 88x after post to /r/shakespeare points to Shakespeare search engine as place to find new sonnet (except we don't have the sonnet yet)
14:54 OUTAGE BEGINS --- Search backends start melting down under load
14:55 docbrown receives pager storm, ManyHttp500s from all clusters
14:57 All traffic to Shakespeare search is failing: see https://monitor
14:58 docbrown starts investigating, finds backend crash rate very high
15:01 INCIDENT BEGINS docbrown declares incident #465 due to cascading failure, coordination on #shakespeare, names jennifer incident commander
15:02 someone coincidentally sends email to shakespeare-discuss@ re sonnet discovery, which happens to be at top of martym's inbox
15:03 jennifer notifies shakespeare-incidents@ list of the incident
15:04 martym tracks down text of new sonnet and looks for documentation on corpus update
15:06 docbrown finds that crash symptoms identical across all tasks in all clusters, investigating cause based on application logs
15:07 martym finds documentation, starts prep work for corpus update
15:10 martym adds sonnet to Shakespeare's known works, starts indexing job
15:12 docbrown contacts clarac & agoogler (from Shakespeare dev team) to help with examining codebase for possible causes
15:18 clarac finds smoking gun in logs pointing to file descriptor exhaustion, confirms against code that leak exists if term not in corpus is searched for
15:20 martym's index MapReduce job completes
15:21 jennifer and docbrown decide to increase instance count enough to drop load on instances that they're able to do appreciable work before dying and being restarted
15:23 docbrown load balances all traffic to USA-2 cluster, permitting instance count increase in other clusters without servers failing immediately
15:25 martym starts replicating new index to all clusters
15:28 docbrown starts 2x instance count increase
15:32 jennifer changes load balancing to increase traffic to nonsacrificial clusters
15:33 tasks in nonsacrificial clusters start failing, same symptoms as before
15:34 found order-of-magnitude error in whiteboard calculations for instance count increase
15:36 jennifer reverts load balancing to resacrifice USA-2 cluster in preparation for additional global 5x instance count increase (to a total of 10x initial capacity)
15:36 OUTAGE MITIGATED, updated index replicated to all clusters
15:39 docbrown starts second wave of instance count increase to 10x initial capacity
15:41 jennifer reinstates load balancing across all clusters for 1% of traffic
15:43 nonsacrificial clusters' HTTP 500 rates at nominal rates, task failures intermittent at low levels
15:45 jennifer balances 10% of traffic across nonsacrificial clusters
15:47 nonsacrificial clusters' HTTP 500 rates remain within SLO, no task failures observed
15:50 30% of traffic balanced across nonsacrificial clusters
15:55 50% of traffic balanced across nonsacrificial clusters
16:00 OUTAGE ENDS, all traffic balanced across all clusters
16:30 INCIDENT ENDS, reached exit criterion of 30 minutes' nominal performance
Supporting information: [168]
[163]Impact is the effect on users, revenue, etc.

[164] An explanation of the circumstances in which this incident happened. It's often helpful to use a technique such as the 5 Whys [Ohn88] to understand the contributing factors.

[165] "Knee-jerk" AIs often turn out to be too extreme or costly to implement, and judgment may be needed to re-scope them in a larger context. There's a risk of over-optimizing for a particular issue, such as adding specific monitoring/alerting when reliable mechanisms like unit tests can catch problems much earlier in the development process.

[166] This section is really for near misses, e.g., "The goat teleporter was available for emergency use with other animals despite lack of certification."

[167] A "screenplay" of the incident; use the incident timeline from the Incident Management document to start filling in the postmortem's timeline, then supplement with other relevant entries.

[168] Useful information, links, logs, screenshots, graphs, IRC logs, IM logs, etc.
