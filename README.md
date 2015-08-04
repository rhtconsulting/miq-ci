# miq-ci

This project provides a CI pipeline for ManageIQ region data using Jenkins.

# Prerequisites

  * A Git server to act as a VCS for your ManageIQ region data

  * A ManageIQ appliance with the "Database Operations" role turned on in each region you wish to add to the CI pipeline


# ManageIQ Database Appliance Setup

 This configuration should be done on the database appliance in each region that you wish to include in the CI pipeline

 1. Install Git. This project was tested with version 1.7.1


    ```
    yum install -y git
    ```

 2. Use the instructions found at  <https://github.com/rhtconsulting/cfme-rhconsulting-scripts>
    to install the rake import/export scripts

 3. Generate a new SSH key pair for root and copy the public key to
    your Git server


    ```
    mkdir -p ${HOME}/.ssh
    chmod 700 ${HOME}/.ssh
    ssh-keygen -N '' -f ${HOME}/.ssh/id_rsa
    ssh-copy-id <git-user>@<git-server>
    ```

# Jenkins Setup

 1. [Install Jenkins](http://jenkins-ci.org/) if you do not already have an instance. This project was tested
    using version 1.622

 2. [Install](https://wiki.jenkins-ci.org/display/JENKINS/Plugins#Plugins-Howtoinstallplugins) the following plugins
    and their dependencies:

    * [Build Pipeline Plugin v1.4.7](https://wiki.jenkins-ci.org/display/JENKINS/Build+Pipeline+Plugin)

    * [Git Plugin v2.4.0](https://wiki.jenkins-ci.org/display/JENKINS/Git+Plugin)

    * [SSH plugin v2.4](https://wiki.jenkins-ci.org/display/JENKINS/SSH+plugin)

    * [Validating String Parameter v2.3](https://wiki.jenkins-ci.org/display/JENKINS/Validating+String+Parameter+Plugin)

 2. Generate an SSH key pair for the Jenkins user:

    ```
    mkdir -p /var/lib/jenkins/.ssh
    chmod 700 /var/lib/jenkins/.ssh
    ssh-keygen -N '' -f /var/lib/jenkins/.ssh/id_rsa
    chown -R jenkins: /var/lib/jenkins/.ssh
    ```

 3. Send the Jenkins user key to all ManageIQ database appliances which will be managed by Jenkins in the CI Pipeline

    ```
    runuser -u jenkins ssh-copy-id root@<manageiq-host>
    ```

 4. Install the job configurations from this project's `jenkins/` directory


    ```
    rsync -r jenkins/jobs jenkins@<jenkins-host>:/var/lib/jenkins/
    ```


 5. Add an SSH site entry for each ManageIQ database appliance which will be managed by Jenkins in the CI
    pipeline

    * Manage Jenkins -> Configure System -> SSH remote hosts -> Add

    * For example, you would add an SSH site for the region 10 database appliance, and the region 20 database appliance,
      and the region 30 database appliance, etc.


# Job Information

  * Each region has a corresponding tag e.g. region10, region20, etc.

  * Each region is a different environment to promote to within the pipeline. For example, region 10 could be the DEV
    region, region 20 the TEST region, etc.

  * The region tag is used to denote which commit is currently in the corresponding region. For example, if the region
    data currently in region 10 is commit 8eb2096 then that commit will be tagged with the region10 tag.

  * To be added to the pipeline view, the first job that must be run is the Export from <DEV-REGION> job. In that job,
    you must specify a tag. That tag corresponds to a feature/milestone/bugfix that can later be promoted to other
    regions or rolled-back. For example, say that you just finished sprint #7 and are ready to promote to the other
    regions for testing and later production workloads. Run the Export from <DEV-REGION> job and specify $tag_name,
    something like "v7". That job will then be added to the pipeline view and v7 is what will be promoted from region
    to region. You can use the pipeline view to track exactly what tag (feature/milestone/bugfix) is in what region.

  * All fields are required unless otherwise specified

### Export from DEV MIQ with no pipeline integration

  * Overview

    - Export Automate domains, buttons,  customization templates, roles, service catalogs, and tags from the DEV
      ManageIQ region

    - Commits the exported data to the user-specified Git repository using the user-specified commit message

    - Tags the commit with the DEV tag to denote that this commit is where the DEV region is currently

  * Available parameters

    - git_repo_location - The location of the Git repository including hostname and .git file location

    - commit_message - The message to be used in a commit - "git commit -m $commit_message" 

### Import into user-specified MIQ

  * Overview

    - Generic job that allows the user to import a user-specified tag into a MIQ appliance that is specified by user

    - This can be used to roll a feature-set into a specific appliance. It can also be used to roll-back to a
      specific tag.

  * Available parameters

    - git_repo_location - The location of the Git repository including hostname and .git file location

    - connection_string - The username@hostname/IP of the MIQ appliance to import into. This should be the location of
      the database MIQ appliance

    - keyfile_location - The full path of the keyfile used to authenticate via SSH

    - tag_name - The tag and associated commit that will be imported into the region. This allows you to import a
      specific tag into an appliance. It also gives you the ability to roll-back a region to a specific tag

    - region_name - The name of the region that the $tag_name commit will be imported into. This tag should match up
      with the database appliance specified by $connection_string. For example, if the DEV database appliance is at
      10.15.75.242, the commit that is imported into that appliance will be tagged with DEV ($region_name)

### Import HEAD to DEV 

  * Overview

    - Polls the configured Git repository every 1 minute looking for a change to the master branch

    - If a change is detected on the master branch, the newest code is automatically imported in the DEV MIQ appliance

    - This allows you to have private DEV environments that all roll-up into the DEV appliance. 

    - Each contributor will check their changes into the master branch and resolve any merge conflicts and/or rebase.
      This job will then automatically detect the change to the master branch and keep the DEV appliance current with
      the master branch

### Export from DEV MIQ

  * Overview

    - Export Automate domains, buttons,  customization templates, roles, service catalogs, and tags from the DEV
      ManageIQ region

    - Commits the exported data to the user-specified Git repository using the user-specified commit message

    - Tags the commit with the DEV tag to denote that this commit is where the DEV region is currently

    - This job is the beginning job of the MIQ pipeline integration. Once this job is executed, a new pipeline ID is
      created and the user can then use the pipeline view to run the rest of the jobs in the pipeline

    -  The downstream project to be built after this job completes is Import into TEST MIQ

  * Available parameters

    - git_repo_location - The location of the Git repository including hostname and .git file location

    - tag_name - The tag name to be given to this commit. This allows you to tag a feature/milestone.
      This tag name is later used to import a specific commit into a region, giving you the ability to roll
      forward or backwards.

    - commit_message - The message to be used in a commit - "git commit -m $commit_message" 

### Import into TEST MIQ

  * Overview

    - Import the commit on the master branch in $git_repo corresponding with $tag_name into the TEST MIQ appliance

    - This job is part of the MIQ build pipeline and must be triggered manually from the pipeline view

    - The upstream job, Export from DEV MIQ, must have completed successfully to run this job

  * Available parameters

    - git_repo_location - The location of the Git repository including hostname and .git file location

    - tag_name - The tag and associated commit that will be imported into the region. That value is passed into this
      job through the build pipeline from the Export from DEV MIQ job

### Import into QA MIQ

  * Overview

    - Import the commit on the master branch in $git_repo corresponding with $tag_name into the QA MIQ appliance

    - This job is part of the MIQ build pipeline and must be triggered manually from the pipeline view

    - The upstream job, Import into TEST MIQ, must have completed successfully to run this job

  * Available parameters

    - git_repo_location - The location of the Git repository including hostname and .git file location

    - tag_name - The tag and associated commit that will be imported into the region. That value is passed into this
      job through the build pipeline from the Import into TEST MIQ job

### Import into PROD MIQ

  * Overview

    - Import the commit on the master branch in $git_repo corresponding with $tag_name into the PROD MIQ appliance

    - This job is part of the MIQ build pipeline and must be triggered manually from the pipeline view

    - The upstream job, Import into QA MIQ, must have completed successfully to run this job

  * Available parameters

    - git_repo_location - The location of the Git repository including hostname and .git file location

    - tag_name - The tag and associated commit that will be imported into the region. That value is passed into this
      job through the build pipeline from the Import into QA MIQ job
