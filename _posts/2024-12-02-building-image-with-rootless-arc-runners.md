---
layout: post
title: Building Images using Rootless ARC Runners and Kaniko
description: A journey into rootless runners on OpenShift to build container images.
comments: false
tags: actions actions-runner-controller arc openshift container rootless
minute: 20
---

Recently, while I was placing my pods in a Kubernetes cluster like a monk arranging stones in a zen garden, a customer asked if it was possible to deploy the **GitHub Actions Runner Controller (ARC)** on **OpenShift**.  

This challenge sparked a journey of adaptation and discovery, where the goal was to deploy **rootless runners** capable of **building container images** using **Kaniko**—while honoring **OpenShift's security principles**.  

In this post, I’ll share the steps I took to make this happen, from creating a **custom runner image** to configuring the **ARC scale set** for OpenShift. Let’s dive in!  

## 1 - Why Kaniko and Rootless Runners?  

**OpenShift**, with its strict security policies, emphasizes running containers without elevated privileges or a root UID. While the default **GitHub Actions Runner Controller (ARC)** works well for most **Kubernetes** setups, it can’t be deployed out of the box on OpenShift.  

The main reason lies in the **runner image**, which [uses a **hardcoded UID**](https://github.com/actions/runner/blob/078eb3b381939ee6665f545234e1dca5ed07da84/images/Dockerfile#L53) for the `runner` user, alongside other issues (e.g., a permissive **sudoers file** and Docker-in-Docker mode requiring a privileged security context).  

To build container images on OpenShift, we need a solution that respects its security constraints.  

This is where **Kaniko** and **rootless unprivileged runners** come into play:  
- **Kaniko** doesn’t require a **Docker daemon**, which aligns with OpenShift’s removal of Docker.  
- Kaniko executes each command within a Dockerfile [entirely in userspace](https://github.com/GoogleContainerTools/kaniko/blob/e328007bc1fa0d8c2eacf1918bebbabc923abafa/README.md), eliminating the need for elevated privileges.  
- **Rootless unprivileged containers** run with arbitrary UIDs, making them a perfect fit for OpenShift’s security model.  

Pairing **Kaniko** with a **rootless runner** lets us safely build and push container images without compromising the platform’s security principles.  

Note: we don't cover here using ARC Kubernetes mode due to [its various limitations](https://github.com/actions/runner-container-hooks/tree/main/packages/k8s#limitations).