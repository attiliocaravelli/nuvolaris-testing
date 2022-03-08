<!--
  ~ Licensed to the Apache Software Foundation (ASF) under one
  ~ or more contributor license agreements.  See the NOTICE file
  ~ distributed with this work for additional information
  ~ regarding copyright ownership.  The ASF licenses this file
  ~ to you under the Apache License, Version 2.0 (the
  ~ "License"); you may not use this file except in compliance
  ~ with the License.  You may obtain a copy of the License at
  ~
  ~   http://www.apache.org/licenses/LICENSE-2.0
  ~
  ~ Unless required by applicable law or agreed to in writing,
  ~ software distributed under the License is distributed on an
  ~ "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
  ~ KIND, either express or implied.  See the License for the
  ~ specific language governing permissions and limitations
  ~ under the License.
  ~
-->
# nuvolaris-testing

This folder includes a Openshift Installer for AWS


# Prerequisites

## AWS account and a Red Hat account. 

The OpenShift cluster uses bigger compute nodes than are available in the AWS "Free Tier", 
so you will definitely end up with charges on your account! 

If you do not have a Red Hat account to pull-secret then you can use the fake version:
`{"auths":{"fake":{"auth": "bar"}}}`


## AWS Network
So that requests can be routed from the internet to your applications in OpenShift 
you will need to set up a couple of things:
Register a domain name, such as "yourcompany.com", in Amazon's Route 53 Domain Name System (DNS). 
Details of how to register a new domain can be found in the AWS documentation: 
`https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/registrar.html`
`https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/AboutHZWorkingWith.html`


## AWS User

The OpenShift installer application uses the AWS API to create all 
the underlying infrastructure components for the cluster, 
such as machines, networking, etc. 
In order to do this you will need a user with "Programmatic access" 
and the "AdministratorAccess" policy. 
You can create a user in AWS here: `https://console.aws.amazon.com/iam/home#/users$new` 

## Create a public / private key pair to enable SSH access to the machines

```
e.g.
ssh-keygen -t ed25519 -N '' -f ~/.ssh/openshift_rsa
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/openshift_rsa
./openshift-install create cluster --dir=aws-install/
```

# Installation/Uninstallation

## Core
```
mkdir openshift-install
cd openshift-install/
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.7.2/openshift-install-linux.tar.gz
tar xvf openshift-install-linux.tar.gz
```

## Installation
`./openshift-install create cluster --dir=aws-install/`

## Uninstallation
`./openshift-install destroy cluster --dir=aws-install`

# Interactive Mode
e.g.
```
? SSH Public Key /home/ian/.ssh/openshift_rsa.pub
? Platform aws
? AWS Access Key ID AKIAV72XFGYFFUXWAE5Y
? AWS Secret Access Key [? for help] ****************************************
? Region eu-west-2
? Base Domain tier2openshift.co.uk
? Cluster Name demo
? Pull Secret [? for help] **************************************************
```

# Automatic Mode

Run `init.sh`
This script will generate an install-config.yaml
```
apiVersion: v1
  baseDomain: nuvolaris.com
  compute:
  - architecture: amd64
    hyperthreading: Enabled
    name: worker
    platform: {}
    replicas: 2
  controlPlane:
    architecture: amd64
    hyperthreading: Enabled
    name: master
    platform: {}
    replicas: 3
  metadata:
    creationTimestamp: null
    name: demo
  networking:
    clusterNetwork:
    - cidr: 10.128.0.0/14
      hostPrefix: 23
    machineNetwork:
    - cidr: 10.0.0.0/16
    networkType: OpenShiftSDN
    serviceNetwork:
    - 172.30.0.0/16
  platform:
    aws:
      region: eu-west-4 
  publish: External
  pullSecret: '{"auths":{"fake":{"auth": "bar"}}}'
  sshKey: |
    ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIOIX1Grh2jabz2QX+DiZISc+Cm7m8ZmrBPkhVJnyKIf robot@robot.nuvolaris.com
```

Run `./openshift-install create install-config --dir=aws-install`





