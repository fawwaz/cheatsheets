---
title: "Terraform quickstart"
category: Terraform
layout: 2017/sheet
description: |
  A quick guide to getting started with Terraform. The cheatsheet is based on [Scraly's cheatsheet](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet-fr.adoc)
---

### About Terraform CLI

Terraform, a tool created by https://www.hashicorp.com/[Hashicorp] in 2014, written in Go, aims to build, change and version control your infrastructure. This tool have a powerfull and very intuitive Command Line Interface.

### Install Terraform 

```bash
$ curl -O https://releases.hashicorp.com/terraform/0.11.10/terraform_0.11.10_linux_amd64.zip
$ sudo unzip terraform_0.11.10_linux_amd64.zip -d /usr/local/bin/
$ rm terraform_0.11.10_linux_amd64.zip
```

See: [Installation](https://learn.hashicorp.com/terraform/getting-started/install)

## Usage

### Show Version

```bash
~$ terraform --version
Terraform v0.11.10
```

### Init Terraform

```bash
~$ terraform init
```

It’s the first command you need to execute. Unless, terraform plan, apply, destroy and import will not work.
The command terraform init will install :
 - terraform modules
 - eventually a backend
 - and provider(s) plugins

### Get

This command is useful when you have defined some modules. Modules are vendored
so when you edit them, you need to get again modules content.

```bash
~$ terraform get -update=true
```

When you use modules, the first thing you’ll have to do is to do a terraform get. This pulls modules into the .terraform directory. Once you do that, unless you do another `terraform get -update=true`, you’ve essentially vendored those modules.

### Plan

The plan step check configuration to execute and write a plan to apply to target infrastructure provider.

```bash
~$ terraform plan -out plan.out
```

It’s an important feature of Terraform that allows a user to see which actions Terraform will perform prior to making any changes, increasing confidence that a change will have the desired effect once applied.

When you execute terraform plan command, terraform will scan all *.tf files in your directory and create the plan.


### Apply

Now you have the desired state so you can execute the plan.

```bash
~$ terraform apply plan.out
```

*Good to know:* : Since terraform v0.11+, in an interactive mode (non CI/CD/autonomous pipeline), you can just execute `terraform apply` command which will print out which actions TF will perform.

By generating the plan and applying it in the same command, Terraform can guarantee that the execution plan won’t change, without needing to write it to disk. This reduces the risk of potentially-sensitive data being left behind, or accidentally checked into version control.

```bash
~$ terraform apply
```
#### Apply and auto approve

```bash
~$ terraform apply -auto-approve
```

#### Apply and define new variables value

```
~$ terraform apply -auto-approve  -var tags-repository_url=${GIT_URL}
```

#### Apply only one module

```
~$ terraform apply -target=module.s3
```

### Destroy

```bash
~$ terraform destroy
```

Delete all the resources!

A deletion plan can be created before:

```bash
~$ terraform plan –destroy
```

`-target` option allow to destroy only one resource, for example a S3 bucket :

```bash
~$ terraform destroy -target aws_s3_bucket.my_bucket
```

### Debug

The Terraform console command is useful for testing interpolations before using them in configurations. Terraform console will read configured state even if it is remote.

```
~$ echo "aws_iam_user.notif.arn" | terraform console arn:aws:iam::123456789:user/notif
```

### Graph

```bash
~$ terraform graph | dot –Tpng > graph.png
```

Visual dependency graph of terraform resources.

### Validate

Validate command is used to validate/check the syntax of the Terraform files. A syntax check is done on all the terraform files in the directory, and will display an error if any of the files doesn't validate. The syntax check does not cover every syntax common issues.

```bash
~$ terraform validate
```

### Providers

You can use a lot of providers/plugins in your terraform definition resources, so it can be useful to have a tree of providers used by modules in your project.

```bash
~$ terraform providers
.
├── provider.aws ~> 1.24.0
├── module.my_module
│   ├── provider.aws (inherited)
│   ├── provider.null
│   └── provider.template
└── module.elastic
    └── provider.aws (inherited)
```

## State

### Pull remote state in a local copy

```bash
~$ terraform state pull > terraform.tfstate
```

### Push state in remote backend storage

```bash
~$ terraform state push
```

This command is usefull if for example you riginally use a local tf state and then you define a backend storage, in S3 or Consul...

### How to tell to Terraform you moved a ressource in a module?

If you moved an existing resource in a module, you need to update the state:

```bash
~$ terraform state mv aws_iam_role.role1 module.mymodule
```

### How to import existing resource in Terraform?

If you have an existing resource in your infrastructure provider, you can import it in your Terraform state:

```bash
~$ terraform import aws_iam_policy.elastic_post  arn:aws:iam::123456789:policy/elastic_post
```

## Workspaces

To manage multiple distinct sets of infrastructure resources/environments.

Instead of create a directory for each environment to manage, we need to just create needed workspace and use them:

### Create workspace

This command create a new workspace and then select it

```bash
~$ terraform workspace new dev
```

### Select a workspace

```bash
~$ terraform workspace select dev
```

### List workspaces

```bash
~$ terraform workspace list
  default
* dev
  prelive
```

### Show current workspace

```bash
~$ terraform workspace show
dev
```

## Tools

### jq

jq is a lightweight command-line JSON processor. Combined with terraform output it can be powerful.

#### Installation

For Linux:

```bash
~$ sudo apt-get install jq
```

For OS X:

```bash
~$ brew install jq
```

#### Usage

For example, we defind outputs in a module and when we execute _terraform apply_ outputs are displayed:

```bash
~$ terraform apply
...
Apply complete! Resources: 0 added, 0 changed,
 0 destroyed.

Outputs:

elastic_endpoint = vpc-toto-12fgfd4d5f4ds5fngetwe4.
eu-central-1.es.amazonaws.com
```

We can extract the value that we want in order to use it in a script for example. With jq it's easy:

```bash
~$ terraform output -json
{
    "elastic_endpoint": {
        "sensitive": false,
        "type": "string",
        "value": "vpc-toto-12fgfd4d5f4ds5fngetwe4.
        eu-central-1.es.amazonaws.com"
    }
}

~$ terraform output -json | jq '.elastic_endpoint.value'
"vpc-toto-12fgfd4d5f4ds5fngetwe4.eu-central-1.
es.amazonaws.com"
```

### Terraforming

If you have an existing AWS account for examples with existing components like S3 buckets, SNS, VPC … You can use terraforming tool, a tool written in Ruby, which extract existing AWS resources and convert it to Terraform files!

#### Installation

```bash
~$ sudo apt install ruby
~$ gem install terraforming
```

#### Usage

Pre-requisites :

Like for Terraform, you need to set AWS credentials

```bash
~$ export AWS_ACCESS_KEY_ID="an_aws_access_key"
~$ export AWS_SECRET_ACCESS_KEY="a_aws_secret_key"
~$ export AWS_DEFAULT_REGION="eu-central-1"
```

You can also specify credential profile in _~/.aws/credentials_s and with _–profile_ option.

```bash
~$ cat ~/.aws/credentials
[aurelie]
aws_access_key_id = xxx
aws_secret_access_key = xxx
aws_default_region = eu-central-1
```

```bash
~$ terraforming s3 --profile aurelie
```

After that, you can use terraforming

```bash
~$ terraforming --help
Commands:
terraforming alb # ALB
...
terraforming vgw # VPN Gateway
terraforming vpc # VPC
```

Example:

```bash
$ terraforming s3 > aws_s3.tf
```

Remarks: As you can see, terraforming can’t extract for the moment API gateway resources so you need to write it manually.


## Read more

* [Getting started with Terraform](https://learn.hashicorp.com/terraform/getting-started/build.html/) _(learn.hashicorp.com.com)_
* [Using Terraform for Cloud Deployments](https://dev.to/koenighotze/using-terraform-for-cloud-deployments---part-1) _(dev.to)_
* [A Comprehensive Guide to Terraform](https://blog.gruntwork.io/a-comprehensive-guide-to-terraform-b3d32832baca#.w9x897ywp) _(blog.gruntwork.io)_