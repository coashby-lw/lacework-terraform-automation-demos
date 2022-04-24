# Lacework Terraform Automation Demos
This repository contains demos and information to enable management of the Lacework Security Platform using Terraform infrastructure as code (IaC). The first part of this journey entails installation and authentication of the Lacework Terraform provider to connect the IaC resources with the Lacework account (or accounts) you wish to manage. From there, remaining demos highlight key aspects of usage and provide guidance for best practices when using the Lacework Terraform provider.

## Contents
| Demo | Description | Prerequisite Demos | Documentation |
|------|-------------|--------------------|---------------|
| **Lacework Terraform Provider Authentication** | *Learn how to authenticate the Lacework Terraform provider, the first step in managing your Lacework account with IaC* | None | [README.md](./lacework-terraform-provider-authentication/README.md) |
| **Cloud Account Integrations** | *Learn how to perform integrations of your cloud accounts into Lacework* | Lacework Terraform Provider Authentication | [README.md](./cloud-account-integrations/README.md)
| **Container and Artifact Registry Integrations** | *Learn how to perform integrations of your container and artifact registries into Lacework.* | Lacework Terraform Provider Authentication | [README.md](./container-and-artifact-registry-integrations/README.md) |
| **Resource Groups** | *This demo illustrates how to manage resource groups within the Lacework Platform* | Lacework Terraform Provider Authentication | [README.md](./resource-groups/README.md) |
| **Alert Rules** | *This demo illustrates how to manage alert rules for events generated by the Lacework Platform* | Lacework Terraform Provider Authentication | [README.md](./alert-rules/README.md) |
| **Alert Channels** | *Learn how to create custom alert channels for notification of events generated by the Lacework Platform* | Lacework Terraform Provider Authentication, Alert Rules, Resource Groups | [README.md](./alert-channels/README.md)
| **Agent Deployments** | *In this demo you will learn about the steps required to enable and deploy agents within your workload environments* | Lacework Terraform Provider Authentication | [README.md](./agent-deployments/README.md) |


## Resources
| Resource | Format | Notes |
|----------|--------|-------|
| [Lacework Terraform Registry Documentation](https://registry.terraform.io/providers/lacework/lacework/latest/docs) | HTML | Covers authentication, organization and sub-account switching. |
| [Lacework CLI Documentation](https://docs.lacework.com/cli) | HTML | Covers installation, API key generation, and configuration of the Lacework CLI |

## Requirements
| Item                                    | Description | Sections Required | Notes |
|-----------------------------------------|-------------|-------------------|-------|
| **Lacework Account**                    | Multi-cloud security platform | *ALL* | If you do not have access to Lacework account you can visit [Lacework: Schedule a Demo](https://www.lacework.com/schedule-demo/) for ways to get started. |
| **Account Level Lacework API Key**      | An API key that has been created by a user with admin level privileges for the account or managing account. | *ALL* | See [CLI Documentation: Create API Key](https://docs.lacework.com/cli#create-api-key) for details on how to create an API key for use. |
| **Organization Level Lacework API Key** | An API key that has been created by a user with admin level privileges for the account organization. | [Organization Admin Configurations](./lacework-terraform-provider-authentication/README.md#organization-admin-configurations) | See [ Lacework CLI Documentation: Create API Key](https://docs.lacework.com/cli#create-api-key) for details on how to create an API key for use. |
| **Lacework CLI +v0.32.0**               | Open source tool for managing the Lacework multi-cloud security platform | *ALL* | See [Lacework CLI Documentation](https://docs.lacework.com/cli) for details on how to install the latest version of the tool.      |
| **Terraform**                           |  The Hashicorp infrastructure as code management tool | *ALL* | See [Install Terraform](https://learn.hashicorp.com/tutorials/terraform/install-cli) for details on how to install the Terraform binary |


# Preface: Providing Context
Beyond the illustrative purpose of this demo, it is also important to know some of the inner workings an rationale behind the enabling resources and tools. Our discussion will focus on these items:
- *The difference between a Lacework account, organization, and its sub-accounts*
- *How best to source values for the Lacework provider configuration*

## The Lacework Account Structure
For the purposes of this demo it is important to be somewhat familiar with how the Lacework Platform is organized. After all, the goal of using the Lacework Terraform provider is to help automate the management of the Lacework Platform. So before we discuss *how* we will manage the Lacework Platform, we'll first see how accounts are structured within. For more detailed information on Lacework account structure go to [Lacework Academy](https://academy.lacework.com) and checkout this great video on [Initial Account Setup](https://academy.lacework.com/learn/video/initial-account-setup).

### The Lacework Organization
First in the hierarchy is the *organization level*. The organization is an elevated account that has an overall view of all the accounts contained within the Lacework Platform. This is why in the Lacework Platform console, the organization and first listed account share the same name. In the console, the organization will denoted by "(Organization)" appearing after the name, and the primary account that was elevated to create the organization will be denoted by "(Account)".

Users within the Platform that have organization admin privileges are able to manage the organization itself, along with all accounts under the organization. These privileges also extend to API keys created by organization admin users. Using API keys created by an organization admin is required when managing the organization through IaC. It is also a very convenient way to manage multiple accounts using a single API key. Configuring your IaC environment with a organization admin API key will allow the user to mange all of the organizations accounts as well.

### The Lacework Account and Sub-accounts
The Lacework account refers to the primary account used to access the Lacework Platform. It has the property that its name is used in the URL created to access the Lacework Platform as shown below.

`<your-account-name>.lacework.net`

Other accounts may also be associated with your primary Lacework account and will have a different name. These accounts are sub-accounts. The primary Lacework account and sub-accounts both have admin and user profiles. However, unlike with an organization profile, users and admins of accounts or sub-accounts can only access and manage features **within** that account or sub-account. This is also true of API keys created to manage the accounts through IaC.

## Managing Provider Configuration with the Lacework CLI
Although there are several ways to configure the Lacework provider, using the Lacework CLI to manage Lacework providers is the safest and easiest method. Documentation of how to install and use the CLI can be found on the [CLI Get Started](https://docs.lacework.com/cli) page.

### How it Works
Once an account has been setup with the Lacework CLI, a Lacework configuration TOML is generated and saved on the HOST machine as `.lacework.toml`, within a default directory<sup>1</sup>. An example TOML file is given below to illustrate the fields present. Within the Lacework TOML, **profiles** are specified in *section name* denoted by square brackets `[]`. Therefore, in this example, there are four profiles configured:
1. `primary`
1. `secondary-a`
1. `secondary-b`
1. `default`

![Example TOML File](./collateral/images/example-toml.png)

<sup>1</sup>*Please reference the [CLI Get Started](https://docs.lacework.com/cli) documentation for more details on Lacework account configurations.*

The Lacework provider automatically searches for a Lacework TOML and uses it to populate the `account`, `api_key`, `api_secret`, and `subaccount` provider arguments if they have not been defined in the Terraform configuration file. Hence, all the fields are *optional*, because the Lacework Terraform provider will first search for the existence of the configuration TOML `.lacework.toml` file created by the Lacework CLI. More details about the provider arguments is given in the [Lacework Provider Arguments](#lacework-provider-arguments) section were you will see all the values that can be configured.

To override any parameter sourced from the Lacework configuration *toml*, simply add the argument name and value within the provider block. Several examples are provided to illustrate this point in the section [Lacework Provider Configurations](#lacework-provider-configurations). Nevertheless, the recommended option is to use the *Default Provider* or specify the account to manage using its **profile** name. Both of these options are covered in future sections.


# Getting Started
There are two basic steps to getting started with the Lacework Terraform provider:
1. Install the Lacework CLI
1. Install the Lacework Terraform provider

Additional resources and up-to-date references for the Lacework Terraform provider can be found on its registry page: [Lacework Terraform Registry](https://registry.terraform.io/providers/lacework/lacework/latest).

## Recommended Installation
This section provides additional context and examples for using the provider for the first time.

### Step 1: Install the Lacework CLI
Detailed instruction on how to install the Lacework CLI for your machine can be found on the [Lacework CLI Docs: Get Started](https://docs.lacework.com/cli) page. There you will be provided with instructions on how to:
1. Download and install the Lacework CLI
1. Generate the required API secret and API key to enable use of the Lacework CLI.

### Step 2: Integrate the Lacework Provider
To begin with the Lacework Terraform provider you must first make sure the provider gets installed on your host machine. This can be achieved by using this block of code:
```hcl
terraform {
  required_providers {
    lacework = {
      source = "lacework/lacework"
      version = "0.17.0"
    }
  }
}

provider "lacework" {
  # Configuration options
}
```
**Note**: *The `source` argument **MUST** be specified in order for the host to correctly point to the Lacework provider registry.*

If you are familiar with Terraform and have used other providers in the past, one important thing to note is the contents of the `required_providers` block. In this block we **MUST** give the provider `source` which will point to the Lacework Terraform Registry.

Other providers may not need this explicit declaration of the provider's `source`. However, in those instances Hashicorp is the owner of the provider, where as for the Lacework provider, Lacework is the owner and maintainer. Therefore, the `source` **MUST** be specified as `lacework/lacework`.

Once this code is added to your Terraform configuration you can run the command below install the provider locally.
```sh
> terraform init
```

If you do not receive any errors when running the `terraform init` command, then you should be able to now see `lacework/lacework` listed as an available provider when running the command below:
```sh
> terraform providers
```
An example output from this command looks like this:
```sh
Providers required by configuration:
.
└── provider[registry.terraform.io/lacework/lacework] 0.17.0
```

**Now we are ready to begin. So let's get started!**