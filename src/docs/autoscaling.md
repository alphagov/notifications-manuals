# GOV.UK Notify AutoScaler
Notify has bursty, periodic loads and traffic. During the night, the traffic stays low and most of the time, is consist of background scheduled testing jobs; and twe get more traffic during office hours. Once in a while, we would receive a task that is consisted of >20k notifications. This has motivated us to develop an autoscaler to response to these tasks and scale up and down Notify apps instances running on PaaS. 

These terms used in this article refer to:

* a **task** is used to described when a user upload a csv file consist of a batch of notifications to be sent 
* a **job** is a unit of notification, database query or a process to be run on the servers at a time
* a **request** describes an 'http request' to retrieve or process information.  
 
## Metrics ##
Notify's auto scaling uses these metrics:

* **RequestCounts (for Inbound-apps)** <br>
A metric returned by AWS Cloudwatch for the Elastic Load Balancers (ELB) to scale each of the ELB apps running on PaaS

* **ApproximateNumberOfMessages (for Delivery-apps)** <br>
A metric returned by AWS Cloudwatch for the SQS queues to scale each notification delivery and database apps on PaaS.

* **Number of scheduled notifications (for scheduled-jobs)**<br>
A metric obtained from the database for csv files to be sent in the next 1 minutes, and is updated everytime when the autoscaler is scheduled run.

The autoscaler is scheduled to run every 20 seconds using the python ```sched``` module. Autoscaling applies to each individual apps. 


## Scaling instances ##
A factor `desired_instances_count` is used to determine the desired number of instances to run for an App at an evaluated period. This factor is calculated differently for our three types of apps and is bounded by:
 
`min_instance_count<=desired_instances_count<=max_instance_count`

The definition of `min_instance_count` and `max_instance_count` can be found in the latter section. 

### Delivery workers ###
The autoscaler checks the total number of messages in the queues (ApproximateNumberOfMessages) of the app and scaled accordingly. One app can use multiple queues. This scaling is applicable to database tasks, CSV job processor, notifications senders and various research and priority jobs. 

For example, 
```notify-delivery-worker-sender``` is an app for delivering the notifications to the service providers, i.e. AWS SES and text messaging providers. The celery worker consumes job from two queues: ```send-sms-tasks``` and ```send-sms-tasks``` are being monitored. The number of instances is then calculated as:-

```
desired_instances_count = total_message_count / request_per_instance
```
where `total_message_count` is the sum of all the queues of this app and `request_per_instance` is a predefined estimated number of jobs an app instance can handle at one time.

### Request handlers ##

The inbound apps are scaled based on last 5 `request_counts`, where `request_counts` is the total number of incoming requests in one minute.  

```
desired_instance_count = max(request_counts)/request_per_instance
```

`request_per_instance` is a pre-defined scaler that indicates the number of requests that can be handled by one instance of app. 


### Scheduled jobs ##
Notify uses a proactive schedule for scheduled tasks. Scheduled tasks are CSV files the users uploaded to be run at a given time. We already have the information of job size (number of notificaitons) and the scheduled time. The proactive scheduler scales up the number of instances before the jobs start. This scaling applies to the same task as the Delivery workers above. 

The desired number of instances for the particular app is calculated as follow:

```
desired_instance_count=num_of_scheduled_jobs/request_per_instance/scheduled_job_scaling_factor
and
min_instance_count<=desired_instances_count<=max_instance_count
```

The `num_of_scheduled_jobs` is the number of notifications to be sent in that task. `Request_per_instance` gives a first indication on the number of jobs that can be handled by one app instance. We can use `scheduled_job_scaling_factor ` to control how aggressively we want the scaling to be. Currently, this factor is set to 2 in Notify. Moreover, as the jobs are first delivered to the queues by delivery worker before these queues are consumed by senders and db workers. Hence, they are scaled up less agressively. 

## The Scaling function ##
The function `scale_paas_apps` uses the `desired_instance_count` factor calculated to scale up or down the number of instances for the app. Below is the Psuedocode for the autoscaler. 

```
if desired_instance_count > current_instance_count
	scale up to desired_instance_count
else if time since last scale up > 5 mins
	current_instance_count = current_instance_count - 1
```

The apps are instantly scaled up to the `desired_instance_count` if the current number of running instances is lower. However, on scaling down, No apps are scaled down if there has been a scaling up event in any app in the last 5 mins.  

In a cooling down period, we only specify how many instances an app has. We do not specify which instance to kill. Any instance can be killed at scaling down. 

For Example,

* Initial instances: A, B, C
* Instances after scale up: A, B, C, D, E
* Possible state after scale down: B, C, E

## Min and max instance counts ##
The `min_instance_count` and `max_instance_count` used to bound the `desired_instance_count` is defined differently in Notify's environment as below:

| App | Queues | req per instance  (Delivery & DB apps) | msg per instance (inbound apps) | default in prod. | Preview | Staging/ Production |
| --- | ------ | ---------------- | --- | --------- | ------- | ------- | ----------|
| notify-delivery-worker-sender | send-sms, send-email, send-tasks |  250 |  | 4 | 1-2 | 4-20 |
| notify-delivery-worker | notify, retry, process-job, notify-internal-tasks, retry-tasks, job-tasks, periodic-tasks | 250 |  | 2 | 1-1 | 2-5 |
| notify-delivery-worker-database | db-sms, db-email, db-letter, database-tasks | 250 | | 2 | 1-2 | 2-20 |
| notify-delivery-worker-periodic | periodic, statistics, periodic-tasks, statistics-tasks | 250 | | 2 | 1-1 | 2-5 |
| notify-delivery-worker-research | research-mode, research-mode-tasks | 250 | |2 | 1-1 | 2-5 |
| notify-delivery-worker-priority | priority, priority-tasks | 250 | | 2 | 1-1 | 2-5 |
| notify-api | notify-paas-proxy | | 1500 | 4 | 1-2 | 2-20 |

## Non-scaled apps ##

The following apps are not scaled:

* notify-delivery-celery-beat (default: 2)
* notify-admin-failwhale (default: 1)
* notify-template-preview (default: 1)
* notify-admin (default: 2)


## Source Code

The source code of our autoscaler can be found [here](https://github.com/alphagov/notifications-paas-autoscaler/blob/master/main.py) in the Notify's [notificaitos-paas-autoscaler](https://github.com/alphagov/notifications-paas-autoscaler) repository.