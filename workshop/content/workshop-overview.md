## Introduction

# Best Practices for Securing Containerized Applications in Kubernetes Engine

This guide demonstrates a series of best practices that will allow the user to improve the security of their containerized applications deployed to Kubernetes Engine.

The [principle of least privilege]((https://en.wikipedia.org/wiki/Principle_of_least_privilege))
is widely recognized as an important design consideration in enhancing the protection of critical systems from faults and malicious behavior. It suggests that every component must be able to access **only** the information and resources that are necessary for its legitimate purpose. This guide will go about showing the user how to improve a container's security by providing a systematic approach to effectively remove unnecessary privileges.

## Architecture

### Containers

At their core, containers help make implementing security best practices easier by providing the user with an easy interface to run processes in a chroot environment as an unprivileged user and removing all but the kernel [capabilities](http://man7.org/linux/man-pages/man7/capabilities.7.html) needed to run the application. By default, all containers are run in the root user namespace so running containers as a non-root user is important.

