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

# Job Descriptions

## Export from DEV CFME with no pipeline integration

## Import into user-specified CFME

## Export from DEV CFME

## Import HEAD into CFME

## Import into TEST CFME

## Import into QA CFME

## Import into PROD CFME
