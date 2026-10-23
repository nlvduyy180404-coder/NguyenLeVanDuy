---
title: "Week 1 Worklog"
date: 2026-09-14
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Week 1 Objectives:

* Get to know the team members and the internship rules at First Cloud AI Journey.
* Learn about the new AWS Free Tier program (2025) and create an AWS account.
* Set up basic account security (MFA, IAM user).
* Get first hands-on experience with Amazon EC2, AWS Lambda and Amazon Bedrock.
* Complete Free Tier activities to earn bonus credits.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | ---- | ---------- | --------------- | ------------------ |
| 2   | - Get acquainted with FCAJ members <br> - Read and take note of the internship rules and regulations | 14/09/2026 | 14/09/2026 | |
| 3   | - Read **"AWS Free Tier 2025: Revolution!"** to learn about the new Free Tier: <br>&emsp; + $100 sign-up credit, up to $100 more by completing activities <br>&emsp; + Two account plans: Free plan and Paid plan <br>&emsp; + Always Free services <br>&emsp; + Credit-earning activities: EC2, RDS, Lambda, Bedrock, AWS Budgets | 15/09/2026 | 15/09/2026 | <https://aws.amazon.com/free/> |
| 4   | - Create a new AWS account <br> - Select a working region in the AWS Console <br> - **Account security:** <br>&emsp; + Install an authenticator app on the phone <br>&emsp; + Enable MFA on the root user <br>&emsp; + Create an IAM user for daily work instead of using root | 16/09/2026 | 16/09/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5   | - **Amazon EC2 practice:** <br>&emsp; + Choose an AMI (operating system for the instance) <br>&emsp; + Choose a Free Tier eligible instance type <br>&emsp; + Configure a Security Group: open port 22 (SSH) and port 80 (HTTP) <br>&emsp; + Launch the instance and connect to it <br>&emsp; + Run basic commands on the instance <br>&emsp; + Stop and terminate the instance after testing to avoid charges | 17/09/2026 | 17/09/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6   | - **AWS Lambda practice:** <br>&emsp; + Create a Lambda function using the Node.js runtime <br>&emsp; + Modify the code to return a custom response <br>&emsp; + Enable a Function URL and invoke the function from the browser <br> - **Amazon Bedrock practice:** <br>&emsp; + Request model access <br>&emsp; + Send prompts and compare responses from several different models in the playground <br> - Check the bonus credits received after completing the Free Tier activities | 18/09/2026 | 18/09/2026 | <https://cloudjourney.awsstudygroup.com/> |


### Week 1 Achievements:

* Understood how the new AWS Free Tier works: sign-up credit, bonus credit for completing activities, and the difference between the Free plan and the Paid plan.

* Successfully created an AWS account and applied basic security practices:
  * Enabled MFA on the root user with an authenticator app
  * Created an IAM user for daily work to avoid using root
  * Understood why the root user should not be used for everyday tasks

* Learned the basic steps to create an EC2 instance:
  * AMI: choosing the operating system for the virtual machine
  * Instance type: CPU/RAM configuration, choosing a Free Tier eligible type
  * Security Group: acts as a firewall, only opening required ports (22 for SSH, 80 for HTTP)
  * Connecting to the instance and working from the command line
  * Terminating resources after use to avoid unexpected costs

* Created a Lambda function in Node.js, customized its code and invoked it via a Function URL; understood the serverless concept – running code without managing servers.

* Learned how to use Amazon Bedrock:
  * Model access must be requested before a model can be used
  * Tried several models and observed that each one answers the same prompt differently

* Completed the Free Tier activities (EC2, Lambda, Bedrock) and received bonus credits in the account.

* Became familiar with the AWS Management Console: searching for and accessing services, selecting regions.
