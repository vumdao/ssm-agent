<p align="center">
  <a href="https://dev.to/vumdao">
    <img alt="Amazon SSM Agent - Risk Of Security" src="https://github.com/vumdao/ssm-agent/blob/master/ssm_iam_policy/img/cover.jpg?raw=true" width="700" />
  </a>
</p>
<h1 align="center">
  <div><b>Amazon SSM Agent - Risk Of Security</b></div>
</h1>

## Table Of Contents
 * [What is AWS SSM Agent?](#What-is-AWS-SSM-Agent?)
 * [Understand Amazon SSM Agent In 2 Minutes](#Understand-Amazon-SSM-Agent-In-2-Minutes)
 * [How is the security risk?](#How-is-the-security-risk?)
 * [Solution](#Solution)
 * [Conclusion](#-Conclusion)

---

## 🚀 **What is AWS SSM Agent?** <a name="What-is-AWS-SSM-Agent?"></a>
- [AWS Systems Manager Agent](https://docs.aws.amazon.com/systems-manager/latest/userguide/ssm-agent.html) (SSM Agent) is Amazon software that can be installed and configured on an EC2 instance, an on-premises server, or a virtual machine (VM). SSM Agent makes it possible for Systems Manager to update, manage, and configure these resources. The agent processes requests from the Systems Manager service in the AWS Cloud, and then runs them as specified in the request. SSM Agent then sends status and execution information back to the Systems Manager service by using the Amazon Message Delivery Service (service prefix: ec2messages).

## 🚀 **Understand Amazon SSM Agent In 2 Minutes**
- In order to provide access to an EC2 through SSM (from console or AWS CLI), we need to install SSM agent on it (as default) and then provide IAM policy to the EC2 so that the SSM Agent service inside the EC2 has permission to get the EC2 information, SSM documents
- Reference to [Understand Amazon SSM Agent In 2 Minutes](https://dev.to/vumdao/understand-amazon-ssm-agent-in-2-minutes-1363) for more detail

## 🚀 **How is the security risk?** <a name="How-is-the-security-risk?"></a>
- We often attach following IAM policy to the EC2 to enable SSH access from Session Manager
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

- With this policy, we are providing the SSM agent within the EC2 ability to access any EC2 instances that have SSM agent enabled. Eg.

```
# Access the EC2
$ aws ssm start-session --target i-011ce869cbf141225 --region ap-northeast-1

Starting session with SessionId: jack.dao-01fe8e68e8f5c70f5
$ sudo su
root@ec2-instance:/var/snap/amazon-ssm-agent/3553# 

# From this one we can install session manager plugin
root@ec2-instance:/var/snap/amazon-ssm-agent/3553# curl "https://s3.amazonaws.com/session-manager-downloads/plugin/latest/ubuntu_arm64/session-manager-plugin.deb" -o "session-manager-plugin.deb"
root@ec2-instance:/var/snap/amazon-ssm-agent/3553# dpkg -i session-manager-plugin.deb 

# And then access to anywhere
$ aws ssm start-session --target i-0df199f1ba0b1fc11 --region ap-southeast-1

Starting session with SessionId: i-011ce869cbf141225-0b8c635c96e4aa038
```

## 🚀 Solution <a name="Solution"></a>
- Best practice of provide IAM policy is avoiding wildcard as much as possible

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "ssm:ListDocuments",
                "ssm:ListCommands",
                "ssm:ListCommandInvocations",
                "ssm:ListDocumentVersions",
                "ssm:DescribeDocument",
                "ssm:GetDocument",
                "ssm:DescribeInstanceInformation",
                "ssm:DescribeDocumentParameters",
                "ssm:DescribeInstanceProperties",
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
            "Resource": "*",
            "Effect": "Allow"
        },
        {
            "Sid": "AmazonSSMtoEC2",
            "Effect": "Allow",
            "Action": [
                "ssm:*"
            ],
            "Resource": "arn:aws:ec2:ap-northeast-1:123456789012:instance/i-011ce869cbf141225"
        }
    ]
}
```

- With the above policy we now restrict the EC2 to provide SSM itself and not able to acess others through SSM
```
$ aws ssm start-session --target i-0df199f1ba0b1fc11 --region ap-southeast-1

An error occurred (AccessDeniedException) when calling the StartSession operation: User: arn:aws:sts::123456789012:assumed-role/role-ssm/i-011ce869cbf141225 is not authorized to perform: ssm:StartSession on resource: arn:aws:ec2:ap-southeast-1:123456789012:instance/i-0df199f1ba0b1fc11
```

- View the log here: /var/log/amazon/ssm/amazon-ssm-agent.log

## 🚀 Conclusion <a name="Conclusion"></a>
- We use SSM to provide access EC2 instnance or send commands without key pem, so please be careful with setup IAM permission to ensure security.
- More about SSM agent, [AWS SSM Agent - Connection Error](https://dev.to/awscommunity-asean/aws-ssm-agent-connection-error-3kn9)