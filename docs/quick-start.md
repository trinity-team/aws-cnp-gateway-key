# Getting Started

This repository contains a CloudFormation Template designed to help complete the AWS configuration process required to enable cross account replication for AWS workloads protected by Rubrik's Cloud Native Protection for AWS. 

The impetus for this requirement stems from the way KMS keys and EBS volume encryption work in AWS. In order to replicate EBS snapshots across regions or AWS accounts, the appropriate keys must be shared and availabile for encryption purposes. This is particularly important during restore in place operations when the original source CMK must be used to create a restored volume from a snapshot in another account and/or region. The issue is further complicated by the fact that the default KMS keys in each region cannot be shared across the account boundary. Luckily, Rubrik has abstracted away most of the complexity associated with these recovery worflows through automation. The only requirement is that the appropriate KMS keys are pre-shared with the appropriate accounts. 

The diagram below depicts these requirements visually for an example environment where we are replicating instance snapshots from us-west-2 in Account A to us-east-2 in Account B. With this configuration, the environment has the necessary permissions to protect instances and volumes within the us-west-2 region of account A while replicating those snapshots to the us-east-2 region of Account B. The keys and shares depicted allow for replication and in place restore regardless of whether the source volumes were encrypted with the default KMS key or a Customer Managed KMS key at the source. More detail on the purposes of each key is provided below.

![image](https://user-images.githubusercontent.com/16825470/107691473-78e44300-6c79-11eb-82bc-a2b86507c5dc.png)


This configuration allows for the restore of instances and volumes in the the source region of Account A using either the local images and snapshots in that region, or the replicas in the replica region of Account B. This topology is depicted in the diagram below.

![image](https://user-images.githubusercontent.com/16825470/107682396-48e37280-6c6e-11eb-8e82-b89ea7084a08.png)


## Gateway Keys
Gateway keys are used for two purposes. 1) To encrypt source snapshots of volumes encrypted with the default KMS key in their source region so that they can be replicated to the target account. 2) To encrypt snapshots stored in the replica region at rest.

When you create a stack using the [CloudFormation Template](/aws-cnp-gateway-key.template) in this repository, CloudFormation will create a rubrik gateway key in that region where you create the stack. This means that that you will need to create two stacks for most use cases, one in the source region of the source account trusting the replica account, and one in the replica region of the replica account targeting the source account. 

That said, in more complicated configurations, you may need to modify this template to accomodate all of the replication and export flows required. Additionally, if no volumes in the source region are encrypted with the default KMS key the gateway in the source region is not required. 

## Customer CMKs
Customer CMKs must be shared with the replica account in the region corresponding to the source instance and volumes. These keys are made available to the replica account so that source snapshots encrypted with the CMK can be replicated across accounts. This template does not handle the sharing of these CMKs for you, but there are example [Key Policies](/customer_cmk_key_policy_example.json) you can apply to your KMS keys in order to share them as required above.

# Creating a Stack
Using the [CloudFormation Template](/aws-cnp-gateway-key.template) in this repository to create a gateway key is a very simple process. Simply navigate to the cloudformation console in the desired account and region, then utilize the template to launch a stack.

![image](https://user-images.githubusercontent.com/16825470/107678280-3155bb00-6c69-11eb-9297-f10c7d4b0fc2.png)

When prompted, enter the account number of the corresponding account that this gateway key should trust then complete the wizard to create the stack.

![image](https://user-images.githubusercontent.com/16825470/107678478-737efc80-6c69-11eb-9acc-ec2f00b3b5a5.png)

Once stack creation is complete, the required gateway key will be present in the desired region.

![image](https://user-images.githubusercontent.com/16825470/107680970-80511f80-6c6c-11eb-9cb0-2e54f5b0aa7a.png)