# miq-ci

This project provides a continuous integration (CI) pipeline for ManageIQ region data using Jenkins. It provides a pipeline
view that allows you to visualize which version of region data (automate domains, dialogs, service catalogs, etc.)
is in a region at a given time.

## Contents

  * [Continuous Integration Workflow Overview](#continuous-integration-workflow-overview)

  * [Prerequisites](#prerequisites)

  * [ManageIQ Database Appliance Setup](#manageiq-database-appliance-setup)

  * [Jenkins Setup](#jenkins-setup)

  * [Job Information](#job-information)

    - [Export from DEV MIQ with no pipeline integration](#export-from-dev-miq-with-no-pipeline-integration)

    - [Import into user-specified MIQ](#import-into-user-specified-miq)

    - [Import HEAD to DEV](#import-head-to-dev)

    - [Export from DEV MIQ](#export-from-dev-miq)

    - [Import into TEST MIQ](#import-into-test-miq)

    - [Import into QA MIQ](#import-into-qa-miq)

    - [Import into PROD MIQ](#import-into-prod-miq)

## Continuous Integration Workflow Overview

  ![Workflow diagram](docs/miq_ci_workflow.png)

## Prerequisites

  * A Git server to act as a VCS for your ManageIQ region data

  * A ManageIQ appliance with the "Database Operations" role turned on in each region you wish to add to the CI pipeline


## ManageIQ Database Appliance Setup

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

## Jenkins Setup

 1. [Install Jenkins](http://jenkins-ci.org/) if you do not already have an instance. This project was tested
    using version 1.622

 2. [Install](https://wiki.jenkins-ci.org/display/JENKINS/Plugins#Plugins-Howtoinstallplugins) the following plugins
    and their dependencies:

    * [Build Pipeline Plugin v1.4.7](https://wiki.jenkins-ci.org/display/JENKINS/Build+Pipeline+Plugin)

    * [Git Plugin v2.4.0](https://wiki.jenkins-ci.org/display/JENKINS/Git+Plugin)

    * [SSH plugin v2.4](https://wiki.jenkins-ci.org/display/JENKINS/SSH+plugin)

    * [Validating String Parameter v2.3](https://wiki.jenkins-ci.org/display/JENKINS/Validating+String+Parameter+Plugin)

 3. Generate an SSH key pair for the Jenkins user:

    ```
    mkdir -p /var/lib/jenkins/.ssh
    chmod 700 /var/lib/jenkins/.ssh
    ssh-keygen -N '' -f /var/lib/jenkins/.ssh/id_rsa
    chown -R jenkins: /var/lib/jenkins/.ssh
    ```

 4. Send the Jenkins user key to all ManageIQ database appliances which will be managed by Jenkins in the CI Pipeline

    ```
    runuser -u jenkins ssh-copy-id root@<manageiq-host>
    ```

 5. Install the job configurations from this project's `jenkins/` directory


    ```
    rsync -r jenkins/jobs jenkins@<jenkins-host>:/var/lib/jenkins/
    ```


 6. Add an SSH site entry for each ManageIQ database appliance which will be managed by Jenkins in the CI
    pipeline

    * Manage Jenkins -> Configure System -> SSH remote hosts -> Add

    * For example, you would add an SSH site for the region 10 database appliance, and the region 20 database appliance,
      and the region 30 database appliance, etc.
 
 7. In order for the pipeline to be created, you must set the Downstream Project on a job. In our case,
    we want the Export from DEV MIQ job to be the first one in the pipeline so we will set the Downstream Project on it first. 
    It will not have an Upstream Project.  
    
    1. Make sure that you've imported the job you wish to add to the pipeline and then go to it in Jenkins

    2. Click "Configure" and then scroll down to the "Post-build Actions" section 

    3. Click the "Add post-build action" dropdown and select "Build other projects (manual step)" 

    4. For the "Downstream Project Names" field we want to set it to the next job in the pipeline, or Import into TEST MIQ

    5. Click the "Add Parameters" dropdown and select "Current build parameters". This will ensure that the current Git server, 
       tag, etc. all get passed through the pipeline

    6. Repeat steps 1-5 for each job in the pipeline to set a Downstream Project, other than for the last job in the pipeline.
       In our case, we would do steps 1-5 for Import into DEV MIQ, Import into TEST MIQ, and Import into QA MIQ. We would omit 
       Import into PROD MIQ as it is the last step in pipeline and does not have any jobs after it


## Job Information

  * Each region has a corresponding tag e.g. region10, region20, etc.

  * Each region is a different environment to promote to within the pipeline. For example, region 10 could be your DEV
    region, region 20 your TEST region, etc. We will use DEV, TEST, QA, and PROD for simplicity's sake. This pipeline 
    can be expanded to any arbitrary number of regions and environments. 

  * The region tag is used to denote which commit is currently in the corresponding region. For example, if the region
    data currently in region 10 is commit 8eb2096 then that commit will be tagged with the region10 tag.

  * For an instance to be added to the pipeline view, the first job that must be run is the Export from \<DEV-REGION\> job.

  * All fields are required unless otherwise specified.

### Export from DEV MIQ with no pipeline integration

  * Overview

    - Export Automate domains, buttons,  customization templates, roles, service catalogs, and tags from the DEV
      ManageIQ region

    - Commits the exported data to the user-specified Git repository using the user-specified commit message

    - Tags the commit with the DEV tag to denote that this commit is where the DEV region is currently

  * Available parameters

    - ```git_repo_location``` - The location of the Git repository including hostname and .git file location

    - ```commit_message``` - The message to be used in a commit - "git commit -m $commit_message" 

### Import into user-specified MIQ

  * Overview

    - Generic job that allows the user to import a user-specified tag into a MIQ appliance that is specified by user

    - This can be used to roll a feature-set into a specific appliance. It can also be used to roll-back to a
      specific tag.

  * Available parameters

    - ```git_repo_location``` - The location of the Git repository including hostname and .git file location

    - ```connection_string``` - The username@hostname/IP of the MIQ appliance to import into. This should be the location of
      the database MIQ appliance

    - ```keyfile_location``` - The full path of the keyfile used to authenticate via SSH

    - ```tag_name``` - The tag and associated commit that will be imported into the region. This allows you to import a
      specific tag into an appliance. It also gives you the ability to roll-back a region to a specific tag

    - ```region_name``` - The name of the region that the $tag_name commit will be imported into. This tag should match up
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

    - ```git_repo_location``` - The location of the Git repository including hostname and .git file location

    - ```tag_name``` - The tag name to be given to this commit. This allows you to tag a feature/milestone.
      This tag name is later used to import a specific commit into a region, giving you the ability to roll
      forward or backwards.

    - ```commit_message``` - The message to be used in a commit - "git commit -m $commit_message" 

### Import into TEST MIQ

  * Overview

    - Import the commit on the master branch in $git_repo corresponding with $tag_name into the TEST MIQ appliance

    - This job is part of the MIQ build pipeline and must be triggered manually from the pipeline view

    - The upstream job, Export from DEV MIQ, must have completed successfully to run this job

  * Available parameters

    - ```git_repo_location``` - The location of the Git repository including hostname and .git file location

    - ```tag_name``` - The tag and associated commit that will be imported into the region. That value is passed into this
      job through the build pipeline from the Export from DEV MIQ job

### Import into QA MIQ

  * Overview

    - Import the commit on the master branch in $git_repo corresponding with $tag_name into the QA MIQ appliance

    - This job is part of the MIQ build pipeline and must be triggered manually from the pipeline view

    - The upstream job, Import into TEST MIQ, must have completed successfully to run this job

  * Available parameters

    - ```git_repo_location``` - The location of the Git repository including hostname and .git file location

    - ```tag_name``` - The tag and associated commit that will be imported into the region. That value is passed into this
      job through the build pipeline from the Import into TEST MIQ job

### Import into PROD MIQ

  * Overview

    - Import the commit on the master branch in $git_repo corresponding with $tag_name into the PROD MIQ appliance

    - This job is part of the MIQ build pipeline and must be triggered manually from the pipeline view

    - The upstream job, Import into QA MIQ, must have completed successfully to run this job

  * Available parameters

    - ```git_repo_location``` - The location of the Git repository including hostname and .git file location

    - ```tag_name``` - The tag and associated commit that will be imported into the region. That value is passed into this
      job through the build pipeline from the Import into QA MIQ job
