<p align="center">
  <a href="https://dev.to/vumdao">
    <img alt="Understand Amazon SSM Agent In 2 Minutes" src="https://dev-to-uploads.s3.amazonaws.com/i/q3gmk4640lm6dupjz2zj.png" width="500" />
  </a>
</p>
<h2 align="center">
  <b>Understand Amazon SSM Agent In 2 Minutes.</b>
</h2>


### 🚀 **[Install SSM Agent on Ubuntu Server instances](#-Install-SSM-Agent-on-Ubuntu-Server-instances)**
[To install SSM Agent on Ubuntu Server 20.10 STR & 20.04, 18.04, and 16.04 LTS 64-bit instances (with Snap package)](https://docs.aws.amazon.com/systems-manager/latest/userguide/agent-install-ubuntu.html#agent-install-ubuntu-tabs)
```
~ $:/home/ubuntu# sudo snap install amazon-ssm-agent --classic
```

### 🚀 **[Check SSM Agent log](#-Check-SSM-Agent-log)**
```
~ $:/home/ubuntu# systemctl restart snap.amazon-ssm-agent.amazon-ssm-agent.service
~ $:/home/ubuntu# tail -f /var/log/amazon/ssm/amazon-ssm-agent.log
        status code: 400, request id: ea74ed4f-70d4-4610-8221-ce7868c3c9fb
2021-01-08 08:40:20 INFO [amazon-ssm-agent] [SelfUpdate] Initializing self update ...
2021-01-08 08:40:20 INFO [amazon-ssm-agent] Starting Core Agent: amazon-ssm-agent - v3.0.161.0
2021-01-08 08:40:20 INFO [amazon-ssm-agent] OS: linux, Arch: amd64
2021-01-08 08:40:22 INFO [amazon-ssm-agent] [LongRunningWorkerContainer] [WorkerProvider] Worker ssm-agent-worker is not running, starting worker process
2021-01-08 08:40:22 INFO [amazon-ssm-agent] [LongRunningWorkerContainer] [WorkerProvider] Worker ssm-agent-worker (pid:11067) started
2021-01-08 08:40:22 ERROR Error adding the directory to watcher: no such file or directory
2021-01-08 08:40:22 INFO [amazon-ssm-agent] [LongRunningWorkerContainer] Monitor long running worker health every 60 seconds
2021-01-08 08:40:22 INFO [ssm-agent-worker] Dial to Core Agent broadcast channel
2021-01-08 08:40:22 INFO [ssm-agent-worker] Dial to Core Agent broadcast channel
2021-01-08 08:40:22 INFO [ssm-agent-worker] Create new startup processor
2021-01-08 08:40:22 INFO [ssm-agent-worker] Start to listen to Core Agent health channel
2021-01-08 08:40:22 INFO [ssm-agent-worker] Start to listen to Core Agent termination channel
2021-01-08 08:40:22 INFO [ssm-agent-worker] [StartupProcessor] Executing startup processor tasks
2021-01-08 08:40:22 INFO [ssm-agent-worker] [StartupProcessor] Write to serial port: Amazon SSM Agent v3.0.161.0 is running
2021-01-08 08:40:22 INFO [ssm-agent-worker] [StartupProcessor] Write to serial port: OsProductName: Ubuntu
2021-01-08 08:40:22 INFO [ssm-agent-worker] [StartupProcessor] Write to serial port: OsVersion: 18.04
2021-01-08 08:40:23 INFO [ssm-agent-worker] Entering SSM Agent hibernate - AccessDeniedException: User: arn:aws:sts::111111111111:assumed-role/SSMAutomation/i-06f3424a03d04c66d is not authorized to perform: ssm:UpdateInstanceInformation on resource: arn:aws:ec2:eu-central-1:111111111111:instance/i-06f3424a03d04c66d
        status code: 400, request id: f0580f42-a0c8-4038-8242-46e4818a391b
```

- It shows that the SSM agent does not have permission on SSM action such as `UpdateInstanceInformation` for the instance that it relies on

> 2021-01-08 08:40:23 INFO [ssm-agent-worker] Entering SSM Agent hibernate - AccessDeniedException: User: arn:aws:sts::111111111111:assumed-role/SSMAutomation/i-06f3424a03d04c66d is not authorized to perform: ssm:UpdateInstanceInformation on resource: arn:aws:ec2:eu-central-1:111111111111:instance/i-06f3424a03d04c66d


### 🚀 **[Attach instance profile for ssm agent permission](#-Attach-instance-profile-for-ssm-agent-permission)**
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AmazonSSMtoEC2",
            "Effect": "Allow",
            "Action": [
                "ssm:*",
                "ssmmessages:CreateControlChannel",
                "ssmmessages:CreateDataChannel",
                "ssmmessages:OpenControlChannel",
                "ssmmessages:OpenDataChannel",
                "ec2messages:AcknowledgeMessage",
                "ec2messages:DeleteMessage",
                "ec2messages:FailMessage",
                "ec2messages:GetEndpoint",
                "ec2messages:GetMessages",
                "ec2messages:SendReply"
            ],
            "Resource": "*"
        }
    ]
}
```

- Restart the ssm agent for the role affects
```
2021-01-08 08:44:27 INFO [ssm-agent-worker] Starting SSM Agent Worker: amazon-ssm-agent - v3.0.161.0
2021-01-08 08:44:27 INFO [ssm-agent-worker] OS: linux, Arch: amd64
2021-01-08 08:44:27 INFO [ssm-agent-worker] [MessagingDeliveryService] Starting document processing engine...
2021-01-08 08:44:27 INFO [ssm-agent-worker] [OfflineService] Starting document processing engine...
2021-01-08 08:44:27 INFO [ssm-agent-worker] [OfflineService] [EngineProcessor] Starting
2021-01-08 08:44:27 INFO [ssm-agent-worker] [OfflineService] [EngineProcessor] Initial processing
2021-01-08 08:44:27 INFO [ssm-agent-worker] [HealthCheck] HealthCheck reporting agent health.
2021-01-08 08:44:27 INFO [ssm-agent-worker] [OfflineService] Starting message polling
2021-01-08 08:44:27 INFO [ssm-agent-worker] [OfflineService] Starting send replies to MDS
2021-01-08 08:44:27 INFO [ssm-agent-worker] [LongRunningPluginsManager] starting long running plugin manager
2021-01-08 08:44:27 INFO [ssm-agent-worker] [LongRunningPluginsManager] there aren't any long running plugin to execute
2021-01-08 08:44:27 INFO [ssm-agent-worker] [MessagingDeliveryService] [EngineProcessor] Starting
2021-01-08 08:44:27 INFO [ssm-agent-worker] [MessagingDeliveryService] [EngineProcessor] Initial processing
2021-01-08 08:44:27 INFO [ssm-agent-worker] [MessagingDeliveryService] Starting message polling
2021-01-08 08:44:27 INFO [ssm-agent-worker] [MessagingDeliveryService] Starting send replies to MDS
2021-01-08 08:44:27 INFO [ssm-agent-worker] [instanceID=i-06f3424a03d04c66d] Starting association polling
2021-01-08 08:44:27 INFO [ssm-agent-worker] [MessagingDeliveryService] [Association] [EngineProcessor] Starting
2021-01-08 08:44:27 INFO [ssm-agent-worker] [MessagingDeliveryService] [Association] Launching response handler
2021-01-08 08:44:27 INFO [ssm-agent-worker] [MessagingDeliveryService] [Association] [EngineProcessor] Initial processing
2021-01-08 08:44:27 INFO [ssm-agent-worker] [MessagingDeliveryService] [Association] Initializing association scheduling service
2021-01-08 08:44:27 INFO [ssm-agent-worker] [MessagingDeliveryService] [Association] Association scheduling service initialized
2021-01-08 08:44:27 INFO [ssm-agent-worker] [MessageGatewayService] Starting session document processing engine...
2021-01-08 08:44:27 INFO [ssm-agent-worker] [MessageGatewayService] [EngineProcessor] Starting
2021-01-08 08:44:27 INFO [ssm-agent-worker] [MessageGatewayService] SSM Agent is trying to setup control channel for Session Manager module.
2021-01-08 08:44:27 INFO [ssm-agent-worker] [MessageGatewayService] agent telemetry cloudwatch metrics disabled
2021-01-08 08:44:27 INFO [ssm-agent-worker] [MessageGatewayService] Setting up websocket for controlchannel for instance: i-06f3424a03d04c66d, requestId: 8ad8e181-fff5-4aee-81ca-4bbaf6542c3e
2021-01-08 08:44:27 INFO [ssm-agent-worker] [MessageGatewayService] listening reply.
```

### 🚀 **[Conclusion](#-Conclusion)**
- For using AWS Systems Manager Run Command, not only set IAM policy for the instance sending SSM command but also the target need IAM policy so that its SSM agent can assume EC2 resource

**Read More**
- [Pelican-resume with docker-compose and AWS + CDK](https://dev.to/vumdao/pelican-resume-with-docker-compose-and-aws-cdk-33e5)
- [Using Helm Install Botkube Integrate With Slack On EKS](https://dev.to/vumdao/using-helm-install-botkube-integrate-with-slack-on-eks-gmn)
- [Ansible AWS EC2 Dynamic Inventory Plugin](https://dev.to/vumdao/ansible-aws-ec2-dynamic-inventory-plugin-3bme)
- [How To List All Enabled Regions Within An AWS account](https://dev.to/vumdao/list-all-enabled-regions-within-an-aws-account-4oo7)
- [Using AWS KMS In AWS Lambda](https://dev.to/vumdao/using-aws-kms-in-aws-lambda-2jm2)
- [Create AWS Backup Plan](https://dev.to/vumdao/create-aws-backup-plan-a0f)
- [Techniques For Writing Least Privilege IAM Policies](https://dev.to/vumdao/techniques-for-writing-least-privilege-iam-policies-4fc7)
- [EKS Persistent Storage With EFS Amazon Service](https://dev.to/vumdao/eks-persistent-storage-with-efs-amazon-service-14ei)
- [Create k8s Cronjob To Schedule Delete Expired Files](https://dev.to/vumdao/create-k8s-cronjob-to-schedule-delete-expired-files-1i41)
- [Amazon ECR - Lifecycle Policy Rules](https://dev.to/vumdao/amazon-ecr-lifecycle-policy-rules-1l59)
- [Connect Postgres Database Using Lambda Function](https://dev.to/vumdao/connect-postgres-database-using-lambda-function-1mca)
- [Using SourceIp in ALB Listener Rule](https://dev.to/vumdao/using-sourceip-in-alb-listener-rule-377b)
- [Amazon Simple Systems Manager (SSM)](https://dev.to/vumdao/amazon-simple-systems-manager-ssm-2pb0)
- [Invalidation AWS CDN Using Boto3](https://dev.to/vumdao/invalidation-aws-cdn-using-boto3-2k9g)
- [Create AWS Lambda Function Triggered By S3 Notification Event](https://dev.to/vumdao/create-aws-lambda-function-triggered-by-s3-notification-event-9p0)
- [CI/CD Of Invalidation AWS CDN Using Gitlab Pipeline](https://dev.to/vumdao/ci-cd-of-invalidation-aws-cdn-using-gitlab-pipeline-34op)
- [Create CodeDeploy](https://dev.to/vumdao/create-codedeploy-4425)
- [Gitlab Pipeline With AWS Codedeploy](https://dev.to/vumdao/gitlab-pipeline-with-aws-codedeploy-30cl)
- [Create AWS-CDK image container](https://dev.to/vumdao/create-aws-cdk-image-container-43ei)
- [Deploy Python Lambda Functions With Container Image](https://dev.to/vumdao/deploy-python-lambda-functions-with-container-image-5hgj)
- [Custom CloudWatch Events](https://dev.to/vumdao/custom-cloudwatch-events-4j3j)
- [How To Get Lastest Image Version in AWS ECR](https://dev.to/vumdao/how-to-get-lastest-image-version-in-aws-ecr-4op2)
- [Multi Threading In Lambda Function](https://dev.to/vumdao/multi-threading-in-lambda-function-3e5h)

<h3 align="center">
  <a href="https://dev.to/vumdao">:stars: Blog</a>
  <span> · </span>
  <a href="https://vumdao.hashnode.dev/">Web</a>
  <span> · </span>
  <a href="https://www.linkedin.com/in/vu-dao-9280ab43/">Linkedin</a>
  <span> · </span>
  <a href="https://www.linkedin.com/groups/12488649/">Group</a>
  <span> · </span>
  <a href="https://www.facebook.com/CloudOpz-104917804863956">Page</a>
  <span> · </span>
  <a href="https://twitter.com/VuDao81124667">Twitter :stars:</a>
</h3>
