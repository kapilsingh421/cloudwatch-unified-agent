Short Description
Regardless of your platform (Windows or Linux), the steps to install the unified Amazon CloudWatch agent are:

Create an AWS Identity and Access Management (IAM) role to run the CloudWatch agent. Then, attach that IAM role to the EC2 instance.
Download and install the CloudWatch agent.
Create an agent configuration file that specifies the metrics/logs that you plan to push to CloudWatch from your EC2 instance.
Start the CloudWatch agent using the configuration file that you created in step 3.
Resolution
Note: Before you begin, be sure that you've established internet connectivity in your EC2 instance. Internet connectivity is necessary to connect your EC2 instance to the required endpoints.

Installing the unified CloudWatch agent (Amazon Linux or Amazon Linux 2)

1.    Create an IAM role to run the CloudWatch agent on your EC2 instance:

       Open the IAM console.
       In the navigation pane, choose Roles.
       Choose Create role.
       For Choose the service that will use this role, choose EC2.
       Choose Next: Permissions.
       In the list of policies, select the CloudWatchAgentServerPolicy check box.
       Choose Next: Tags, and then choose Next: Review.
       For Role name, enter a name for the role, such as CloudWatchAgentServerRole.
       (Optional) Provide a role description.
       Confirm that CloudWatchAgentServerPolicy appears next to Policies.
       Choose Create role.
       Attach this newly created IAM role to the EC2 instance.

2.    Download and install the unified CloudWatch agent on your EC2 instance:

Download:
wget https://s3.amazonaws.com/amazoncloudwatch-agent/amazon_linux/amd64/latest/amazon-cloudwatch-agent.rpm


Install:

sudo rpm -U ./amazon-cloudwatch-agent.rpm

Change Region 
vim /etc/awslogs/awslogs.conf

3.    Create the agent configuration file.


Note: For simplicity, you can create the agent configuration file using the wizard. Later, you can manually edit the file to add or remove metrics or logs. See "Tips for completing the agent configuration file wizard" below.

Run the wizard:

sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
4.    Start the CloudWatch agent:
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a -a fetch-config -s -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json

Note: Replace <configuration-file-path> with the configuration file path that you received in step 3.



Creating the Agent Configuration File
The CloudWatch agent gets its log and metric collection configuration from a file in the CloudWatch agent directory.
I have found that the CloudWatch Agent configuration file wizard does a pretty good job of generating the initial configuration and I highly recommend you run it. However, if you are using the CloudWatch agent primarily for log forwarding, you'll need to edit the config file by hand.

1.vim /opt/aws/amazon-cloudwatch-agent/bin/config.json

{
     "agent": {
         "run_as_user": "root"
     },
     "logs": {
         "logs_collected": {
             "files": {
                 "collect_list": [
                     {
                         "file_path": "/var/www/html/app/.pm2/logs/app-dev-out.log",
                         "log_group_name": "app-dev-out-log-staging",
                         "log_stream_name": "{instance_id}",
			 "timestamp_format": "%d/%b/%Y:%H:%M:%S %z"
                     },
		     {
                        "file_path": "/var/www/html/app/.pm2/logs/app-dev-error.log",
                        "log_group_name": "app-dev-error-log-staging",
                        "timestamp_format": "%d/%b/%Y:%H:%M:%S %z"
                    }
                 ]
             }
         }
     }
 }


RUN command to STOP cloudwatch agent resync configuration of config.json
1.sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a stop
2.sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a fetch-config -s -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json



