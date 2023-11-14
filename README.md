# VPC-Troubleshooting
Demonstrating how to troubleshoot a Virtual Private Cloud (VPC) configuration and analyze VPC  Flow Logs

<div align="center">
  <img src="https://github.com/cooksies/VPC-Troubleshooting/assets/75002188/10a159f2-5272-40f7-83c3-15118b0d7d71">
  
  <sup>This is an Amazon Lab. The environment has been set up automatically for me.</sup>
</div>

# Connect to CLI Host
<div align="center">
  <img src="https://github.com/cooksies/VPC-Troubleshooting/assets/75002188/dc5f236f-384a-48b7-9406-795ca1b7ced4">
</div>

1. On the EC2 Management console, select **CLI Host** and click on connect.
2. Choose the **"EC2 Instance Connect"** tab and choose Connect.
3. You have successfully connected to the CLI Host

**Configuring the AWS CLI on the CLI Host Instance**\
<div align="center">
  <img src="https://github.com/cooksies/VPC-Troubleshooting/assets/75002188/217bfc34-9fb2-4d60-91e0-296a4b05fc1f">
</div>

1. Configure the AWS CLI profile in the EC2 Instance Connect terminal by running
```
aws configure
```
2. At the prompts, input the values obtained from the lab (These were provided to me by the AWS Lab environment, which is unique for users)
<div align="center">
  
|Prompts|Input|
|---|---|
|AWS Access Key ID|AKIAW5CV5VIXPE723DJ5|
|AWS Secret Access Key|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|
|Default Region Name|us-west-2|
|Default output format|json|

</div>

# Creating VPC Flow Logs

1. Create an S3 bucket where the flow logs will be published.
   - Replace the ###### with 6 random numbers
```
aws s3api create-bucket --bucket flowlog###### --region 'us-west-2' --create-bucket-configuration LocationConstraint='us-west-2'
```
  - If a _Bucket name already exists_ error occurs, use a different set of 6 digits and run the command again.
  - The output should come out as `"Location": "http://flowlog171171.s3.amazonaws.com/"`
2. Get the VPC ID for VPC1 to create Flow Logs

```
aws ec2 describe-vpcs --query 'Vpcs[*].[VpcId,Tags[?Key==`Name`].Value,CidrBlock]' --filters "Name=tag:Name,Values='VPC1'"
```

 - Output:
   ```
    [
        "vpc-0e42f6c6ff0f02c94", 
        [
            "VPC1"
        ], 
        "10.0.0.0/16"
    ]

3. Create a VPC Flow Logs on VPC1
 - Replace <flowlog######> with the bucket name you choose
 - replace <vpc-id> with the VPC ID for VPC1 obtained from the previous step
```
aws ec2 create-flow-logs --resource-type VPC --resource-ids <vpc-id> --traffic-type ALL --log-destination-type s3 --log-destination arn:aws:s3:::<flowlog######>
```
  - Output returns **FlowLogIds** and **ClientToken**
    ```
    "Unsuccessful": [], 
    "FlowLogIds": [
        "fl-01136438091ca0c7d"
    ], 
    "ClientToken": "nNMreEGopaAhuZlvjkdvKyOdybdAsZe9izWs8+R0HiM="
    ```
  - Ignore the "Unsuccessful" message

4. Confirm that the flow log was created
```
aws ec2 describe-flow-logs
```
# Troubleshooting VPC config issues

1. To find details about a web server instance. Take its IP address and plug it in the command by replacing <WebServerIP>
```
aws ec2 describe-instances --filter "Name=ip-address,Values='<WebServerIP>'"
```
  - The output should be a large JSON document

2. Let's try filtering the results
```
aws ec2 describe-instances --filter "Name=ip-address,Values='<WebServerIP>'" --query 'Reservations[*].Instances[*].[State,PrivateIpAddress,InstanceId,SecurityGroups,SubnetId,KeyName]'
```
  - Again, make sure to replace the <WebServerIP> with the web server's IP address
<div align="center">
  <img src="https://github.com/cooksies/VPC-Troubleshooting/assets/75002188/102bb369-16a2-4549-acb9-17af8d566127">
  
  <sup>The Results indicate the instance is running</sup>
</div>

# Troubleshooting

# Flow Log Analysis
