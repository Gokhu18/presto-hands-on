# presto-hands-on

Welcome! This repo contains instructions for setting up Presto on an Amazon EMR cluster, loading some data and playing around with some queries. Feel free to open issues of pull requests if something with the instructions is off.

# Pre-requisites

We assume that you have provisioned a 3 node EMR cluster that you have access to via SSH and that all nodes in the cluster have access to the Internet.

For the Boston Presto hands on meetup, check out this [gist](https://gist.github.com/petroav/1ef757d1673bc6a7436e) for getting your credentials to an EMR cluster.

# Setting up the environment

1. Log into your cluster as the `ec2-user` user. That is the user we'll be using throughout the tutorial because passwordless `ssh` for root is not setup by default.
2. We'll use Teradata's `presto-admin` CLI tool for the administrative tasks needed to setup our environment. You should download and set it up with the following instructions on the master node:
```
wget https://s3-us-west-2.amazonaws.com/presto-admin/meetup-2015-11/prestoadmin-1.2-offline.tar.bz2
tar -jxf prestoadmin-1.2-offline.tar.bz2
```
3. Download JRE8u66 from Oracle on your master node
```
wget --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u65-b17/jre-8u65-linux-x64.rpm
```
4. 