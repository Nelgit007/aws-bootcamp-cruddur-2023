# Week 0 — Billing and Architecture

## Required Homework

**Task 1**

I created a napkin design for the cruddur application

Here is my [napkin](https://lucid.app/lucidchart/1055c035-8413-46f8-a799-02ce5e3b60dd/edit?invitationId=inv_1969876e-88fd-4891-a5ac-b65a96f008e2)

![Conceptual Diagram](assets/Cruddur%20App%20Napkin%20Design%20Week%20-%200.png)

**Task 2**

I created my Logical Architectural Diagram for the cruddur app using Lucid

Here is my [Logical Diagram with Lucid](https://lucid.app/lucidchart/bd9e0c8b-aee8-4dd3-9d57-3a134904b558/edit?invitationId=inv_a642497a-eaec-42b7-ae4e-911f3abadf28)

![Logical Architectural Diagram](assets/Logical%20Design.png)

**Task 3**

I created a new user with username - Nelson_O

I created an Admin Group and assigned an AdministrativeAcces policy

I added user Nelson_O to Admin Group as shown [here](assets/New%20Admin%20User%2C%20Week%20-%200.png)

**Task 4**

I created a zero-spend-budget in my AWS Account

Here is a link to

**Task 5**
### Installing AWS CLI

I Installed the AWS CLI following the ![AWS CLI Installation documentation](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

I configured the environmantl variable on the CLI for user Nelson_O using the step outlined ![here](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html)


set env variables
```sh
export AWS_ACCESS_KEY_ID=""
export AWS_SECRET_ACCESS_KEY=""
export AWS_DEFAULT_REGION=""
```


I set gitpod to remember env variables
```sh
gp env AWS_ACCESS_KEY_ID=""
gp env AWS_SECRET_ACCESS_KEY=""
gp env AWS_DEFAULT_REGION=""
```


I modified the .gitpod configuration file to include the following task

```sh
tasks:
  - name: aws-cli
    env:
      AWS_CLI_AUTO_PROMPT: on-partial
    init: |
      cd /workspace
      curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
      unzip awscliv2.zip
      sudo ./aws/install
      cd $THEIA_WORKSPACE_ROOT

```

**Task 6**
### Created an AWS Budget via AWS CLI

I followed the instructions in the [AWS CLI Reference Documentation](https://docs.aws.amazon.com/cli/latest/reference/budgets/create-budget.html#examples) to create a Cost and Usage Budget

I created an as folder where i created a json folder

I added templates for my budget ans SNS service inside of the folder

The budget templates are in json format


**Task 7**
### Enable Billing

I enabled Billing on my AWS Account

Here is a ![link]() that shows billing in my root account


### Created A Billing Alarm

I enabled Alarm using CloudWatch service in AWS

I set the metrics for the alarm to Billing

I set the amount to $10

I created a new SNS Topic **Cloud-Project-Billing_Alarm** and attached an email

Gave my Alarm a name and created the alarm

I was prompted to go to my email and activate the SNS notification

Here is my billing alarm set 

![billing alarm](assets/AWS-Billing-Alarm.png)










