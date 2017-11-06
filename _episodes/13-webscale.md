---
title: "Is docker web-scale?"
teaching: 10
exercises: 0
questions:
- "How much of what I have told you is wrong?"
objectives:
- "Start considering the changes required to horizontally scale docker applications in the cloud"
keypoints:
- "Scaling Docker in production is complicated"
---

All of the examples in this workshop have been designed for a single Docker host where one person is in charge. This is fine for a developer's system, or on a very limited production system, but this doesn't scale up.

A few examples:
- volumes: So far volumes have been on a single host. When running applications on a Docker cluster a volume on one host is inaccessible on another, some form of networked storage is needed.
- ports: Manually defining port-forwarding is problematic when multiple users are running applications, especially when running independent copies of the same application. Automatic port forwarding requires some sort of discovery method so external clients can connect.
- Managing a large number of containers or deployments over multiple hosts is complicated.

Many of these issues will become apparent when you start doing advanced work with [devspace](https://github.com/openmicroscopy/devspace/).


## Advanced orchestration in production:

- [Kubernetes](https://kubernetes.io/) is an orchestration system created and open-sourced by Google. Try the [interactive online tutorials](https://kubernetes.io/docs/tutorials/kubernetes-basics/)
- [Docker Swarm](https://docs.docker.com/engine/swarm/swarm-tutorial/)
- [Apache Mesos](https://mesos.apache.org/) and the [Datacenter Operating system](https://dcos.io/)
- [Rancher](https://rancher.com/)


## The dark side...

- [Why would anyone choose Docker over fat binaries?](http://www.smashcompany.com/technology/why-would-anyone-choose-docker-over-fat-binaries)
- [Docker in Production: A History of Failure](https://thehftguy.com/2016/11/01/docker-in-production-an-history-of-failure/)
- [Docker in Production: An Update](https://thehftguy.com/2017/02/23/docker-in-production-an-update/)


{% include links.md %}
