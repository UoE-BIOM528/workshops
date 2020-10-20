---
layout: page
title: Setup
order: 1
---
# Workshop setup

There are two ways of completing these wokshops:

1. Use a virtual machine on OpenStack
  * You will need to create an instance on OpenStack
  * All software and data needed will be pre-installed
  * An internet connection will be required (+ a VPN for off-campus access)
2. Use your own machine
  * A mac or linux machine will be best
  * Software will need to be installed on your local machine
  * Data will need to be downloaded

## 1. Openstack

### Creating an instance on Openstack

A video tutorial to connect to OpenStack and launching an instance is [here](https://youtu.be/JIoFJBWTtlg)

To successfully create an instance you will need to select the following options:

#### Log on details
* Domain:
* User name:
* Password:


#### Openstack instance
Please name your instance something that you will remember - including your name is a good idea. Then follow the instructions using the following options:

* Boot source: _Instance Snapshot_
* Name:
* Flavour: _u1.xxlarge_

Once the instance has started remember to take note of the IP address (use Edit-Copy).

### Log onto your instance

A video tutorial to connect to your virtual machine using Terminal (mac) or MobaXterm (windows) is [here](https://youtu.be/qPPBjTSppvE)

The log in details are:
* User name:
* Password:

Once you have successfully logged on, its time to begin the workshop!

## 2. Using your own machine

It will also be possible to use your own machine to complete the workshop. A Linux machine or Mac would be best suited for these workshops as most of the available software is only available on these systems and access to the command-line is essential.

To successfully use your own machine you will need to:
* Install all required software
* Download the workshop data

### Required software

In these workshops we will be using:
* SOAPdeNOVO (genome assembly)
* Augustus (gene prediction)
* OrthoMCL (gene family identification)
* RAxML (gene tree inference)
* PAML (Estimation of accelerated evolution)

More detailed information on these software, including installation instructions can be found below.

### Downloading workshop data

#### Augustus Gene Predictor

In order to predict genes from a genome sequence we will be using [Augustus](https://academic.oup.com/bioinformatics/article/24/5/637/202844)

The Augustus program, complete with tutorials, examples and documentation can be found on [GitHub](https://github.com/Gaius-Augustus/Augustus).

Augustus is primarily available for linux but there is an older version (3.2.1) that has been cloned for Mac OS X. See this [GitHub](https://github.com/nextgenusfs/augustus) page for code and instructions to download and install Augustus 3.2.1 for Mac OS X. Even with this version there can be issues with the auxiliary programs when installing the software. As these extra programs are not required for these workshops you can simply not compile them when installing Augustus.

To not install the auxiliary programs you will need to slightly alter the `MakeFile` in the Augustus directory. You will have to change the line:

```
cd auxprogs && ${MAKE} CC=$(BAMTOOLS_CC) CXX=$(BAMTOOLS_CXX) BAMTOOLS=$(BAMTOOLS)
```

to

```
#cd auxprogs && ${MAKE} CC=$(BAMTOOLS_CC) CXX=$(BAMTOOLS_CXX) BAMTOOLS=$(BAMTOOLS)
```

Adding the `#` at the start of the line makes your compiler interpret this line as a comment so it is never executed and the compiler doesn't try to install the unnecessary auxiliary programs.
