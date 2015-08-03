# miq-ci

This project provides a CI lifecycle process for ManageIQ using Jenkins.

# ManageIQ Host Setup

 1. Install Git:


    ```
    yum install -y git
    ```

 2. Use the instructions found at  <https://github.com/rhtconsulting/cfme-rhconsulting-scripts> to install the rake import/export scripts

 3. Generate a new SSH key pair for root and copy the public key to
    your Git server:


    ```
    mkdir -p ${HOME}/.ssh
    chmod 700 ${HOME}/.ssh
    ssh-keygen -N '' -f ${HOME}/.ssh/id_rsa
    ssh-copy-id <git-user>@<git-server>
    ```

# Jenkins Setup

 1. Install Jenkins <http://jenkins-ci.org/>

 2. Install the following plugins:

    - Build Pipeline Plugin v1.4.7

    - Credentials Plugin v1.22

    - Git Client Plugin v1.18.0

    - Git Parameter Plugin v0.4.0

    - Git Plugin v2.4.0

    - Parameterized Trigger v2.27

    - SSH Credentials v1.11

    - SSH v2.4

    - Validating String Parameter v2.3

 2. Generate an SSH key pair for the Jenkins user:

    ```
    mkdir -p /var/lib/jenkins/.ssh
    chmod 700 /var/lib/jenkins/.ssh
    ssh-keygen -N '' -f /var/lib/jenkins/.ssh/id_rsa
    chown -R jenkins: /var/lib/jenkins/.ssh
    ```

 3. Send the Jenkins user key to all ManageIQ hosts which Jenkins will be
    managing:

    ```
    runuser -u jenkins ssh-copy-id root@<manageiq-host>
    ```

 4. Install the job configurations from this project's `jenkins/` directory.


    ```
    rsync -r jenkins/jobs jenkins@<jenkins-host>:/var/lib/jenkins/
    ```

 5. In Jenkins, configure each lifecycle job with the following:

    - The SSH site for the ManageIQ host for that specific lifecycle.

    - The Git repo location.

# Job Information

  * Each region has a tag e.g. DEV, TEST, etc.

  * The region tag is used to denote which commit is currently in the corresponding region

  * The user can create arbitrary tags, specified in the Export jobs

  * When importing into a region, you can specify a tag in order to control which version is imported. This gives you the ability to precisely control which commit or set of commits is imported into an environment. This also gives you the ability to rollback to an older tag is you need to. 

  * A \* denotes a required field

### Export from DEV CFME with no pipeline integration

  * Overview

    - Export Automate domains, buttons,  customization templates, roles, service catalogs, and tags from the DEV CloudForms region

    - Commits the exported data to the user-specified git repository using the user-specified commit message

    - Tags the commit with the DEV tag to denote that this commit is where the DEV region is currently

  * Available parameters

    - git_repo_location\* - The location of the git repo including hostname and .git file location

    - commit_message\* - The message to be saved with the commit 

### Import into user-specified CFME

  * Overview

    - Generic job that allows the user to import a user-specified tag into a CFME appliance that is specified by user 

    - This can be used to roll a feature-set into a specific appliance. It can also be used to roll-back to a specific tag. 

  * Available parameters

    - git_repo_location\* - The location of the git repo including hostname and .git file location

    - connection_string\* - The username@hostname/IP of the CFME appliance to import into. This should be the location of the database CFME appliance  

    - keyfile_location\* - The full path of the keyfile used to authenticate via SSH

    - tag_name\* - The tag and associated commit that will be imported into the region. This allows you to import a specific tag into an appliance. It also gives you the ability to roll-back a region to a specific tag

    - region_name\* - The name of the region that the $tag_name commit will be imported into. This tag should match up with the database appliance specified by $connection_string. For example, if the DEV database appliance is at 10.15.75.242, the commit that is imported into that appliance will be tagged with DEV ($region_name) 

### Export from DEV CFME

### Import HEAD into CFME

### Import into TEST CFME

### Import into QA CFME

### Import into PROD CFME
