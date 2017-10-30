---
title: "Is docker web-scale?"
teaching: 10
exercises: 0
questions:
- "How much of what I have told you is wrong?"
objectives:
- "Start considering the changes required to horizontally scale docker applications in the cloud"
keypoints:
---

All of the examples in this workshop have been designed for a single Docker host where one person is in charge. This is fine for a developer's system, or on a very limited production system, but this doesn't scale up.

A few examples:
- volumes: So far volumes have been on a single host. When running applications on a Docker cluster a volume on one host is accessible on another.
- ports: Manually defining port-forwarding is problematic when multiple users are running applications, especially when running independent copies of the same application.

Many of these issues will become apparent when you start doing advanced work with [devspace](https://github.com/openmicroscopy/devspace/).

{% include links.md %}
