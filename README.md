## General Info
The **IAM Roles CICD Deployment Pipeline** automates the provisioning and management of AWS IAM roles using Jenkins, Ansible, and CloudFormation. This pipeline enables a secure, repeatable, and auditable way to deploy IAM infrastructure, minimizing manual effort and improving consistency.

## Technologies Used
- Jenkins (Pipeline as Code with Groovy)  
- Ansible (Playbooks and Roles)  
- AWS CloudFormation  
- GitHub (Version Control)  
- AWS CLI, boto3  

## Environments
* STAGING
* NONPROD

## Features
- Automated deployment and validation of CloudFormation templates  
- Environment-specific configuration via properties files  
- Secure credential management with Jenkins Vault integration  
- Conditional stack deletion capability  
- Modular design using Ansible roles and tags  
- Full integration with Jenkins CI/CD workflows  

## Prerequisites
- Jenkins server with:  
  - Ansible installed  
  - Python libraries: boto3, botocore  
  - AWS CLI configured  
- Jenkins IAM role with permissions to deploy CloudFormation and access S3  
- Security group allowing ports 22, 80, 443 for Jenkins server  
- Vault credentials stored in Jenkins Credentials Plugin  

## Setup Instructions
1. **Configure Variables**  
   - Set S3 bucket name in `group_vars/all.yml`  
   - Define environment variables in `parameters/<branch_name>.properties`  

2. **Prepare Jenkins**  
   - Ensure Jenkins server has required software and network access  
   - Create Jenkins credentials for Vault (username/password)  

3. **Run Pipeline**  
   - Trigger the pipeline from Jenkins console  
   - The pipeline will checkout code, load environment variables, and execute Ansible playbooks to prepare, validate, deploy, and delete IAM roles  


## Pipeline Workflow
1. **Checkout Code**  
   Fetch latest from GitHub repository.  

2. **Load Environment Variables**  
   Load branch-specific variables from properties files.  

3. **Build Environment**  
   Run Ansible playbook with `prepare` tag to setup prerequisites.  

4. **Validate Templates**  
   Run Ansible playbook with `validate` tag to verify CloudFormation syntax and parameters.  

5. **Deploy IAM Roles**  
   Run Ansible playbook with `deploy` tag to create or update AWS resources.  

6. **Optional Delete**  
   If `DELETE_STACK` is `YES`, run playbook with `delete` tag to remove the deployed stack.  

## File Structure
```bash
├── Jenkinsfile                 # Jenkins pipeline definition (Groovy)
├── site.yml                    # Main Ansible playbook referencing role tags
├── inventory                   # Ansible inventory configuration
├── ansible.cfg                 # Ansible configuration file
├── group_vars/                 # Common variables for all environments
│   └── all.yml
├── parameters/                 # Environment-specific property files
│   └── <branch_name>.properties
├── roles/                      # Ansible roles for each stage
│   ├── prepare-cf-template/
│   │   └── tasks/main.yml
│   ├── validate-cf-template/
│   │   └── tasks/main.yml
│   ├── deploy-cf-template/
│   │   └── tasks/main.yml
│   └── delete-cf-template/
│       └── tasks/main.yml
├── cf_templates/               # AWS CloudFormation templates
├── .gitignore                  # Git ignore rules
```

## Working of the pipeline
Jenkinsfile
   - Written in Groovy.
   - Defines stages:
        - Build Environment (prepare)
        - Validate CF Templates (validate)
        - Deploy CF Templates (deploy)
        - Delete CF Templates (delete)
    - Loads environment variables dynamically from parameters/${BRANCH_NAME}.properties.
    - Uses withCredentials to inject Vault credentials securely into Ansible runs.

Ansible
  - site.yml is the main playbook.
  - Each tag corresponds to a role:
        - prepare → prepare-cf-template
        - validate → validate-cf-template
        - deploy → deploy-cf-template
        - delete → delete-cf-template
  - Roles execute CloudFormation tasks via the Ansible cloudformation module.

Vault Credentials
  - Stored in Jenkins credentials store.
  - Referenced in the Jenkinsfile with:
    ```
      withCredentials([usernamePassword(credentialsId: 'vault', passwordVariable: 'VAULT_PASSWORD', usernameVariable: 'VAULT_USER')]) { ... }
    ```
  - Created using: ansible-vault create test.yml. Inside this file provide all the required parametes needed in Key-value format.

Running the Pipeline
  - Commit your changes to the branch.
  - In Jenkins, select the job and click Build.
  - The pipeline will:
        - Prepare templates
        - Upload to S3
        - Validate templates
        - Deploy IAM roles
  - (Optional) To delete a stack, ensure the delete stage is triggered via .properties file or by running the pipeline with:
     ```
     DELETE_STACK=YES
    ```
## Notes
  - CloudFormation stacks are named using the pattern:
    ```
    {STACK_PREFIX}-{ENV}
    ```
  - Deleting stacks is destructive — use with caution.
  - This setup can be extended for other AWS resources beyond IAM roles.
  - Jenkinsfile( written in Groovy format):This file contains different stages like building, Validating, Deploying and Deleting CF template
  - Site.yml: Jenkinsfile calls site.yml file which corresponds to individual roles like prepare, validate,deploy and delete the CF template.
  - Inventory: Defines the inventory for the script. Here we are using locally wrt branches.
  - ansible.cfg: It holds the ansible configuration. Inventory will be checked locally here.
  - .gitignore: use to exclude files with particular extension while running the code.
  - groupvars: holds the common variable required in script.
  - roles: site.yml file calls indiviual roles(main.yml file corresponding to each role) with respect to its defined tags.
  - cf_templates: Hold the code written in Cloud formation format.

## Usage
This pipeline enables infrastructure-as-code deployment of AWS IAM roles through an automated CI/CD process, improving security, consistency, and traceability.

## Troubleshooting & Tips
- Ensure Vault credentials are up-to-date and accessible by Jenkins  
- Verify network access from Jenkins to AWS services  
- Check Ansible logs for detailed error messages  
- Validate CloudFormation templates independently using AWS CLI  

