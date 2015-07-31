# miq-ci

This project provides a CI lifecycle process for ManageIQ using Jenkins.

# ManageIQ Host Setup

 1. Install `git`:


    ```
    yum install -y git
    ```

 2. Install <https://github.com/rhtconsulting/cfme-rhconsulting-scripts>

 3. Add and setup a build user for working with Git and the MIQ datastore. In
    the simplest case, this can just be root, but this is obviously not the
    most secure approach.

 4. Generate a new SSH key pair for the build user and copy the public key to
    your Git server:


    ```
    su - <build-user>
    mkdir -p ${HOME}/.ssh
    chmod 700 ${HOME}/.ssh
    ssh-keygen -N '' -f ${HOME}/.ssh/id_rsa
    ssh-copy-id <build-user>@<git-server>
    ```

# Jenkins Host Setup

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

 3. Use `ssh-copy-id` to send the Jenkins user key to all ManageIQ hosts which
    Jenkins will be managing:

    ```
    runuser -u jenkins ssh-copy-id <user>@<manageiq-host>
    ```

 4. Install the job configurations from this project's `jenkins/` directory.


    ```
    rsync -r jenkins/jobs jenkins@<jenkins-host>:/var/lib/jenkins/
    ```
