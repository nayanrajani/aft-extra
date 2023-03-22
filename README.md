# M&M AFT

    - new account is there
    - Management account is set
    - create a structure and architecture
        - azure AD is the SSO
        - direct connect is not set-up
    - share the document of pre-requisite (Done)

    - mpass
        - it is a UI in front of terraform
        - to create resource they are using mpass for different resources and modules.

    - aft
        - sign-up
        - the create a separate account for the aft account

# Start

    - Configure and launch your AWS Control Tower Account Factory for Terraform
    Five steps are required to configure and launch your AFT environment.

        - Step 1: Launch your AWS Control Tower landing zone

            - Before launching AFT, you must have a working AWS Control Tower landing zone in your AWS account. You will configure and launch AFT from the AWS Control Tower management account.

        - Step 2: Create a new organizational unit for AFT (recommended)

            - We recommend that you create a separate OU in your AWS organization, where you will deploy the AFT management account. Create an OU through your AWS Control Tower management account. For instructions on how to create an OU, refer to Create an organization in the AWS Organizations User Guide.

        - Step 3: Provision the AFT management account

            - AFT requires a separate AWS account to manage and orchestrate its own requests. From the AWS Control Tower management account that's associated with your AWS Control Tower landing zone, you'll provision this account for AFT.

            - To provision the AFT management account, see Provision Account Factory accounts with AWS Service Catalog. When specifying an OU, be sure to select the OU you created in Step 2. When specifying a name, use AFT-Management.

            - Note
                - It can take up to 30 minutes for the account to be fully provisioned. Validate that you have access to the AFT management account.

        - Step 4: Ensure that the Terraform environment is available for deployment

            - This step assumes that you are experienced with Terraform, and that you have procedures in place for executing Terraform. AFT supports Terraform Version 0.15.x or later.

        - Step 5: Call the Account Factory for Terraform module to deploy AFT

            - The Account Factory for Terraform module must be called while you are authenticated with AdministratorAccess credentials in your AWS Control Tower management account.

            - AWS Control Tower, through the AWS Control Tower management account, vends a Terraform module that establishes all infrastructure necessary to orchestrate your AWS Control Tower account factory requests. You can view that module in the AFT repository. Refer to the module’s README file for information about the input required to run the module and deploy AFT.

            - If you have established pipelines for managing Terraform in your environment, you can integrate this module into your existing workflow. Otherwise, run the module from any environment that is authenticated with the required credentials.

            - Note
                - The AFT Terraform module does not manage a backend Terraform state. Be sure to preserve the Terraform state file that’s generated, after applying the module, or set up a Terraform backend using Amazon S3 and DynamoDB.

            - Certain input variables may contain sensitive values, such as a private ssh key or Terraform token. These values may be viewable as plain text in Terraform state file, depending on your deployment method. It is your responsibility to protect the Terraform state file, which may contain sensitive data. See the Terraform documentation for more information.

            - Note
                - Deploying AFT through the Terraform module requires several minutes. Initial deployment may require up to 30 minutes. As a best practice, use AWS Security Token Service (STS) credentials and ensure that the credentials have a timeout sufficient for a full deployment, because a timeout causes the deployment to fail. The minimum timeout for AWS STS credentials is 60 minutes or more. Alternatively, you can leverage any IAM user that has AdministratorAccess permissions in the AWS Control Tower management account.

---

## CT Setup (M&M) (11-11-22)

    - Create control tower LZ from console
        - log and audit email required
    - Create a new OU for AFT console
        - go to organizations
        - create a organization under root name- AFT
        - go to control tower -> account factory
        - change network settings
            - edit
                - max number of private subnet to 0
                - add a range of CIDR
            - save
        - create a temp IAM user
        - then go to control tower -> organization
            - select AFT and register it.
        - then go to landing zone settings
            - modify settings
                - next
                - next
                - update landing zone
        - go to products list
            - see control tower is visible or not
        - control tower
            - organization refresh
        <!-- - go to account factory
            - create account
                -
        - go to service catalog
        - go to products list
            - see control tower is visible or not -->
    - Create an AFT management account under the AFT OU console
    ---------------------------------
    - Requirements (11-11-22) (Done)
        - Log configuration
        - KMS key setting
        - DEFINE the repo controller (github or code commit, etc)
            - creating repos
        - Who will be doing AFT part, mahindra/blazeclan
            - if blazeclan then we need access for the mahindra account
            - as well as repo account access

        - SECONDARY REGION
        - ct_management_account_id
        - log_archive_account_id
        - audit_account_id
        - aft_management_account_id

---

# 14-11-22

    - Creating a Project in bitbucket "M&M-AWS-LZ" & repo with name "M&M-AWS-LZ-Terraform".
    - Choosing a Terraform provider region to "Mumbai".
        - choose a profile as well.
    - need to create a S3 bucket to store tfstate file. (Creating in Mumbai region)
        - need to create a dynamodb table for the state locking functionality.
        - By default, S3 does not support State Locking functionality.
        - You need to make use of the DynamoDB table to achieve state locking functionality.

---

# 17-11-22

    - prerequisite for Account creation
        - AccountEmail- network Email
        - AccountName- name appear and which Uo it will fall under
        - SSOUserEmail, SSOUserFirstName,SSOUserLastName
        - account_tags(Optional)
        - account_customizations_name- referenced in the other file
        - custom_fields(optional) - if any kind of resource is required to be used in any other resource for automation purpose.

        - vpc is created to be in AFT-management account can we use this CIDR?
        - this mandatory
        - this will not be created but in future it may require if aws change anything.
        # AFT VPC Specs
        aft_vpc_cidr                   = "10.0.0.0/22"
        aft_vpc_private_subnet_01_cidr = "10.0.0.0/24"
        aft_vpc_private_subnet_02_cidr = "10.0.1.0/24"
        aft_vpc_public_subnet_01_cidr  = "10.0.2.0/25"
        aft_vpc_public_subnet_02_cidr  = "10.0.2.128/25"

---

# 18-11-22

    - Initialization of control tower via terraform
        - create a folder aft-aws-control-tower/terraform
        - create main.tf and add below code to initialize pipeline via bitbucket

            module "aft_pipeline" {
                source = "github.com/aws-ia/terraform-aws-control_tower_account_factory"

                # Required Variables
                ct_management_account_id    = var.ct_management_account_id
                log_archive_account_id      = var.log_archive_account_id
                audit_account_id            = var.audit_account_id
                aft_management_account_id   = var.aft_management_account_id
                ct_home_region              = "ap-south-1"
                tf_backend_secondary_region = "ap-southeast-1"

                # Terraform Variables
                terraform_version      = "1.1.9"
                terraform_distribution = "oss"

                # VCS Variables
                vcs_provider                                  = "bitbucket"
                account_customizations_repo_name              = "{bucketname}/aft-account-customizations"
                account_provisioning_customizations_repo_name = "{bucketname}/aft-account-provisioning-customizations"
                account_request_repo_name                     = "{bucketname}/aft-account-request"
                global_customizations_repo_name               = "{bucketname}/aft-global-customizations"

                account_customizations_repo_branch              = "master"
                account_provisioning_customizations_repo_branch = "master"
                account_request_repo_branch                     = "master"
                global_customizations_repo_branch               = "master"

                # AFT Feature Flags
                aft_vpc_endpoints                       = false
                aft_feature_delete_default_vpcs_enabled = true


                # AFT VPC Specs
                aft_vpc_cidr                   = "10.0.0.0/22"
                aft_vpc_private_subnet_01_cidr = "10.0.0.0/24"
                aft_vpc_private_subnet_02_cidr = "10.0.1.0/24"
                aft_vpc_public_subnet_01_cidr  = "10.0.2.0/25"
                aft_vpc_public_subnet_02_cidr  = "10.0.2.128/25"
                }

        - then create tf-config.tf to add below code for provide configuration and backend initialization.

            terraform {
                required_version = ">= 1.1.9"

                required_providers {
                    aws = { version = ">= 3.70" }
                }

                backend "s3" {
                    bucket         = "{bucketname}"
                    key            = "terraform.tfstate"
                    region         = "{region}"
                    dynamodb_table = "{tablename}"
                    profile        = "aft-account"
                }
            }

            provider "aws" {
                region  = "ap-south-1"
                profile = "master-account"
            }

        - create variables and terraform.tfvars file for variable.
        - add the below given folder in the common folder with other file or clone as a repo.
            - aft-account-customization
            - aft-account-provisioning-customizations
            - aft-account-request
            - aft-aws-control-tower
            - aft-global-customization

        - then push this repo to the github, bitbucket or anything.
        - terraform init
        - terraform plan
        - terraform apply -auto-approve

### How to Create Accounts?

        - First create a file in aft-account-request/terraform name with account name.

            module "network_account" {
                source = "./modules/aft-account-request"

                control_tower_parameters = {
                    AccountEmail              = "{AccountMail}"
                    AccountName               = "{name}"
                    ManagedOrganizationalUnit = "{manager-name}"
                    SSOUserEmail              = "{sso-existing-useremail}"
                    SSOUserFirstName          = "{first name}"
                    SSOUserLastName           = "{last name}"
                }

                account_tags = {
                    "account_name" = "tags"
                }

                change_management_parameters = {
                    change_requested_by = "{name of developer}"
                    change_reason       = "Provision example account as part of landing zone setup"
                }

                account_customizations_name = "{Network}" //this will be used in aft-account-customizations to create same name folder.
                }

        - Now add some required folder in aft-account-customizations with "account_customizations_name"
        - push the code into the repo, and check if the pipeline is working fine or not?

        - Do above steps for multiple accounts

# 22/11/22

### How to create OU via terraform[Separately]?

- Got to "aws-tf-ou/terraform" folder, and create a new file main.tf for provider configuration.
- then create new files according to your nested-OU name under "Root".
- add below code to create a new OU under root

  - Create "AD-OU" from "Root"
    resource "aws_organizations_organizational_unit" "ad-ou" {
    name = "AD-OU"
    parent_id = aws_organizations_organization.root.roots.0.id
    }

- to add more OU under "AD-OU", see below code as an example

  - change the parent_id format and then done.
  - Create "AD-OU" from "Root"
    resource "aws_organizations_organizational_unit" "dms-ou" {
    name = "DMS2.0"
    parent_id = aws_organizations_organizational_unit.ad-ou.id
    }

- create a variables & terraform.tfvars file
- terraform init
- terraform plan
- terraform apply -auto-approve

## what if you are not able to create an account?

- please go to service catalogue in master
- go to provisioned products
- change filter for accounts, and see what is the error or look for step function and lambda.

## what if you get error like OU is not registered under organization but in real it is registered when creating an account?

- you need to change the ManagedOrganizationalUnit = "level-1/level-2/level-3/level-4"

# 24/11/22

- Two accounts created, Network(14/11/22) and DMS-DEV(18/11/22) Account.
- clean-up strategy?
- DMS Account should not get direct access.

# 25/11/22

- creating two accounts under DMS
  - UAT
  - PRD
