## General Info
This pipeline is used to deploy IAM roles via a CICD pipeline.  

## USed Skills
* Ansible
* CICD 
* CF Templates
* GitHub
* Jenkins

## Environments
* STAGING
* NONPROD

## Setup
Jenkinsfile is called in jenkins console which holds the references to all the underlying code files.
On Jenkins server make sure ansible, boto3, awscli, botocore is prsent. Also make sure the SG on the Jenkins server has all the necessary ports enable like  port 22,443,80. Make sure the IAM roles has sufficient permission to deploy the cloudformation template and S3. 

Once the Jenkins server has all preconfiguration setup, provide the bucket name in group_vars all.yml file and build the pipeline from jenkins console. 

## Working of the pipeline
Jenkinsfile  (to be specific different stages define within Jenkinsfile) work as the base or reference file to refer to whole structure of the CICD pipeline.

Accoding to the setup here, once we build the do a Build on Jenkins console, it would go and fetch the latest code from Github. And pickups the variables at he runtime define under properties folder. 
(The reference is happening because of the function define in the Jenkinsfile- line 27-34).

The required credetials to login to the instance at the run time is kept under vault credentials. On jenkins console, provide the vault credentials(username/password) and in groovy file refer those with withCredentials parameter of Jenkinsfile. The vault credential is the credential for a file created with command ansible-vault create test.yml file. Inside this file provide all the required parametes needed in Key-value format.

Further the stages start getting executing step by step, Starting from Stage Build Environment. This stage refers to the file site.yml and refer to the task with tag "prepare". On site.yml file it refers to the role prepare-cf-template and execute the tasks provided in the main.yml file. 
Once all the action under this role has been executed, then it goes back to Jenkinsfile second stage ie Validate CF Template, followed by Third stage and proceed with similar approach, going from site.yml file wrt to tag and execute the action define under that role.

If we wish to delete the template later, then the DeleteStack can be set "Yes" in .properties file and accordingly the stage four ie Delete CF template would get deleted.

## Different file use case:
* Jenkinsfile( written in Groovy format):This file contains different stages like build environment, Validating CF template, Deploying CF template and deleting the CF template.
* Site.yml: Jenkinsfile calls site.yml file which corresponds to individual roles like prepare, validate,deploy and delete the CF template.
* Inventory: Defines the inventory for the script. Here we are using locally wrt branches.
* ansible.cfg: It holds the ansible configuration. Inventory will be checked locally here.
* .gitignore: use to exclude files with particular extension while running the code.
* groupvars: holds the common variable required in script.
* roles: site.yml file calls indiviual roles(main.yml file corresponding to each role) with respect to its defined tags.
* cf_templates: Hold the code written in Cloud formation format.

## Usage
To deploy IAM roles with CICD pipeline, thereby reducing the manual efforts and  have a better tracking.
