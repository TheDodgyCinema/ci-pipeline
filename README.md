# CI Pipeline
## Overview
This repository contains the resources to set up a local continuous integration pipeline for use in Java application development.
It consists of three CentOS machines:
- a `control` machine that runs [Ansible](https://www.ansible.com/) for provisioning
- a `build` server with Java 8, [Maven](https://maven.apache.org/), [Git](https://git-scm.com/) and [Jenkins](https://jenkins.io/) (with [GitHub](https://wiki.jenkins.io/display/JENKINS/GitHub+Plugin) and [publish-over-ssh](https://wiki.jenkins.io/display/JENKINS/Publish+Over+SSH+Plugin) plugins)
- a `live` server running [WildFly](http://wildfly.org/)

When set up properly, this set-up can be used to automatically build and deploy an application from any commit to GitHub.

## Set up

### Install prerequisites
- [Git](https://git-scm.com/downloads)
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- [Vagrant](https://www.vagrantup.com/)

### Spin up and provision machines
- Clone this repository and change into this directory
- Generate a pair of ssh keys in the project root (`ssh-keygen -t rsa -f id_rsa` with an empty passphrase (use [Git Bash](https://gitforwindows.org/) on Windows))
- Modify the hostname and address of each machine in `inventory.yml` as appropriate
- Run `vagrant up`
- If it fails on installation of Jenkins plugins, run `vagrant provision` (this is a known bug)

### Set up automatic building deployment
- SSH into the build machine (`vagrant ssh YOUR_BUILD_MACHINE_NAME`)
- Provide Jenkins with the ssh private key:
```
sudo mkdir /var/lib/jenkins/.ssh
sudo cp .ssh/id_rsa /var/lib/jenkins/.ssh/
sudo chown jenkins:jenkins /var/lib/jenkins/.ssh/id_rsa
```
- Restart Jenkins to load plugins (`sudo service jenkins restart`)
- Login to the Jenkins console (`http://YOUR_BUILD_MACHINE_ADDRESS:8080`) using `admin` as username and password
- Navigate to the Publish over SSH configuration (_Manage Jenkins_ > _Configure System_ > _Publish over SSH_) and enter:
  - _Path to key_: `.ssh/id_rsa`
  - _SSH servers_ > _Add_:
    - _Name_: `YOUR_LIVE_MACHINE_NAME`
    - _Hostname_: `YOUR_LIVE_MACHINE_ADDRESS`
    - _Username_: `vagrant`
- _Test Configuration_  and _Save_
- Set up a new _Freestyle project_ with [automatic build triggers](https://wiki.jenkins.io/display/JENKINS/Building+a+software+project) (turn on _GitHub hook trigger for GITScm polling_)
- _Add build step_ > _Invoke top-level Maven targets_ with Goals `clean install`
- Add a new _Post-build Action_ > _Send build artifacts over SSH_:
  - Source files: `**/*.war`
  - Exec command: `sudo cp target/*.war /opt/wildfly/standalone/deployments/`
- Press _Save_ then _Build Now_ to check that the app deploys correctly
