# AmazonS3Compromise
# PROJECT NAME

AWS Cloud Forensics- Investigated Compromise of Amazon S3

## Objective

Today I am going to complete a lab provided by CyberDefenders. 

Scenario:

Your organization uses AWS for hosting critical data and applications. An incident has been reported involving unauthorized data access and potential exfiltration. The organization's security team has detected unusual activities and needs to investigate the incident to understand the scope, identify the attacker, and prevent further data breaches.


### Tactics

 - Persistence
 - Privilege Escalation
 - Credential Access


### Tools Used

-Splunk

## Steps

Question: Knowing which user account was compromised is essential for understanding the attacker's initial entry point into the environment. What is the username of the compromised user? 

First thing I am going to do is look for log on events. eventType=AwsConsoleSignIn

![image](https://github.com/user-attachments/assets/328c3761-43c1-4177-94d6-117d2f4d86da)

35 Events. Under the usernames I seen an unusual amount of logins from helpdesk.luke. The answer is helpdesk.luke. 

![image](https://github.com/user-attachments/assets/14e88391-7167-48d2-aeda-14f763bd558b)

Question: We must investigate the events following the initial compromise to understand the attacker's motives. What's the UTC timestamp of the attacker's first access to an S3 object? 

I know what user was compromised. According to AWS Documentation, S3 is a storage service that stores data as objects within a bucket. I need to find events related to S3. Under eventSource I see events related to s3

![image](https://github.com/user-attachments/assets/f1a1bdea-e541-4f86-864b-e4681edb6a98)

I run the query index="aws_cloudtrail" "userIdentity.userName"="helpdesk.luke" eventSource="s3.amazonaws.com"

![image](https://github.com/user-attachments/assets/6f3364ae-701f-4e26-9cf4-cf09a3fadf96)

Under eventName I see a field for GetObject. Run the following query


![image](https://github.com/user-attachments/assets/d7bc5713-0131-4cef-b301-654e09f5c642)

Looks like the first access to an S3 Object was 2023-11-02 09:55:53

Question: Among the S3 buckets accessed by the attacker, one contains a DWG file. What is the name of this bucket?

I look up with a DWG file is. It is a binary file format that stores 2D and 3D design data and meta data.

I run a simple search for a DWG file and there is only 1 event.

![image](https://github.com/user-attachments/assets/d93571c5-8551-4256-94a5-11bd4e84c19f)

In the event we get our answer. The file name is Product2_CAD_Designs.dwg and the bucket name is  product-designs-repository31183937 

![image](https://github.com/user-attachments/assets/b59bfb2b-c528-43e7-bd2d-324115d18d85)

Question: We've identified changes to a bucket's configuration that allowed public access, a significant security concern. What is the name of this particular S3 bucket?

Under eventName there are 23 events related to GetBucketPublicAccessBlock

![image](https://github.com/user-attachments/assets/f48cb7cf-f64f-4379-926e-2940ea687b4d)

Out of the 23 events, 9 were under the bucket backup and restore, suggesting data exfiltration. 
The answer is backup-and-restore98825501

![image](https://github.com/user-attachments/assets/5f6bf711-4166-4e31-863c-0d0bfc1c8754)

Question: Creating a new user account is a common tactic attackers use to establish persistence in a compromised environment. What is the username of the account created by the attacker?

I run a simple search for eventName=createuser. Only 1 event shows up. 

![image](https://github.com/user-attachments/assets/d7758dae-6806-40e1-a008-0036dae9aa49)

In the event we get the answer. Marketing.mark

![image](https://github.com/user-attachments/assets/04b0348f-3ef0-44f1-9f63-09463f5a289f)

Question: Following account creation, the attacker added the account to a specific group. What is the name of the group to which the account was added? 

I search for eventName=AddUserToGroup

![image](https://github.com/user-attachments/assets/35cfd36e-c6a7-43eb-8172-69ffbe775283)

No surprise here. marketing.mark was added to Admins.

![image](https://github.com/user-attachments/assets/2262790b-4e68-4cb6-b2ce-261f36ab40a1)

End of Lab

