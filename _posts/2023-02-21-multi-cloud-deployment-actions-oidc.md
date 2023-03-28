---
layout: post
#title: Multi-cloud deployment with GitHub Actions and OIDC
title: Versioned and passwordless multi-cloud deployment
description: A story about versioned & passwordless multi-cloud deployment 
comments: false
tags: actions oidc container azure aws gcp release reusable-workflows
minute: 15
---

That's it ninjas, spring has arrived, birds are singing and beautiful clouds adorn the sky. It make me remember about these old stories, where some mountain monks could target multiple of them at once.

Today, we're going to do the same thing, but with different clouds. Let's use our GitHub Actions katas to deploy an application to Azure, AWS and GCP ☁️☁️☁️

## 0 - Introduction

We'll deploy the [Spring Petclinic sample application](https://github.com/spring-projects/spring-petclinic) as a container application, and this is the [forked repository](https://github.com/ghsioux-octodemo/spring-petclinic) where we'll work on. The ["Multi-Cloud Deployment"](https://github.com/ghsioux-octodemo/spring-petclinic/blob/main/.github/workflows/maven-build.yml) workflow is where the magic happens. The overall process is as follows:

1. Build the application using Maven to generate a JAR file;
2. Build a container image embedding this JAR file;
3. Push the container image to the GitHub Container Registry;
4. Deploy the container image to each of the cloud providers.

Overall, the deployment will have the following properties:

| Properties | How |
|----------|------------|
| __Automated__ | The deployment will be performed with GitHub Actions |
| __Versioned__ | The deployment will be triggered when a release with a specific tag is published, or manually on a specific tag |
| __Protected__ | The deployment can only be triggered by a repository administrator, and only tags starting with `v*` are allowed to be deployed |
| __Passwordless__ | The deployment will use the OpenID Connect (OIDC) standard to authenticate to the cloud providers, hence no need to manage any password |
| __Geo-distributed__ | The application will be deployed on three different geographical locations: `us-east-1` (AWS), `westeurope` (Azure) and `europe-west1` (GCP) |  


## 1 - Controlling the version to deploy

Let's now have a look at the first section of the workflow:

```yaml
name: "🚀 Multi-cloud PetClinic deployment"

on:
  workflow_dispatch:
  release:
    types: [published]
```

The workflow will be triggered manually or when a release is published. I'm working alone on this project, but let's imagine more complex scenario where multiple people collaborate. As a repository administrator, I'd likely need to restrict __i)__ who can trigger/re-run the workflow (e.g. only admin), and __ii__) which tags (and thus associated releases) are allowed to be deployed.

That's why I've added a few checks in the first job of the workflow to ensure that:

```yaml
  prereq_checks:
    name: Ensure this workflow has been trigger on a protected release tag (v*)
    runs-on: ubuntu-latest
    env:
      TRIGGERING_ACTOR: {% raw %}${{ github.triggering_actor }}{% endraw %}
    steps:
      - name: Fail if the workflow has been triggered manually with a non-release tag (i.e. not using the 'v*' regex)
        run: |
          if [[ "$GITHUB_REF" != refs/tags/v* ]]; then
            echo "Only 'v*' tags and associated releases can be deployed, exiting."
            exit 1
          fi
      - name: Ensure the triggering actor has admin permissions on the repository
        id: check_admin
        uses: actions/github-script@v6
        with:
          script: |
            const TRIGGERING_ACTOR = process.env.TRIGGERING_ACTOR
            github.rest.repos.getCollaboratorPermissionLevel({
              owner: context.repo.owner,
              repo: context.repo.repo,
              username: TRIGGERING_ACTOR
              }).then(response => {
                if (response.data.permission == 'admin') {
                  core.info('The github actor has admin permission on this repository')
                } else {
                  core.info('The github actor does not have admin permission on this repository')
                  process.exit(1)
                }
              })
```

As you can see, only member with the `admin` permission can trigger the workflow, and only tags starting with `v` are allowed to be deployed. In parallel, I've set up a [tag protection rules](https://docs.github.com/en/enterprise-cloud@latest/repositories/managing-your-repositorys-settings-and-features/managing-repository-settings/configuring-tag-protection-rules) to restrict `v*` tags creation to privileged members only. This tag protection rule will also apply when creating a release (i.e. a non-privileged member won't be authorized to create a release with a tag matching `v*`).

To summarize so far:
* the deployment workflow can only be triggered by a repository administrator, 
* it can be triggered when **publishing a release** associated with a tag matching `v*`;
* or it can be triggered **manually** on commits associated with tags matching `v*`;
* tags matching `v*` can only be created by a privileged member thanks to the tag protection rules. 


## 2 - OIDC authentication in Actions

Allright, we have implemented some mecanisms to make the deployment **protected** and **versionable**. Now, let's make it **passwordless**. 

But first, why are we talking about passwords? Because we need to authenticate to the cloud providers of course! The good news is that passwords are not the only mean, and we can use the OpenID Connect (OIDC) standard to perform the authentication. In addition, we'll use service accounts to authenticate to the cloud providers. This is a good practice to avoid using personal accounts, and to have a clearly defined permissions and an audit trail of who did what.

The first step is to create a service account for each cloud provider, as well as associated RBAC:
* for Azure, we need to create and setup a service principal, and configure federated authentication;
* for AWS, we need to create an IAM user, and configure trust policy;
* for GCP, we need to create a service account, and configure workload identity federation.

I won't go into the details of the OIDC authentication setup, but you can find all the steps I've done in the [`docs/oidc-setup` folder]().

You might have noticed that I'm using [reusable workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows) - one for each cloud provider - to deploy the container app. The underlying idea is to be able to use them in other repositories in the future.

## 3 - The Clouds setup

Each of the cloud providers have their own way to deploy container applications on their infrastructure, and provide the associated GitHub Actions to do so. Here are the ones I've used:

| Cloud provider | Container Service | GitHub Actions |
|----------|-------|-------------|
| Microsoft Azure | [Azure Container Instances](https://azure.microsoft.com/en-us/services/container-instances/) | [Azure/aci-deploy](https://github.com/Azure/aci-deploy) |
| Amazon Web Services | [Elastic Container Service](https://aws.amazon.com/ecs/) | {::nomarkdown}<ul class="table-list"><li>{:/}[aws-actions/amazon-ecs-render-task-definition](https://github.com/aws-actions/amazon-ecs-render-task-definition) {::nomarkdown}</li><li>{:/}[aws-actions/amazon-ecs-deploy-task-definition](https://github.com/aws-actions/amazon-ecs-deploy-task-definition) {::nomarkdown}</li></ul>{:/} |
| Google Cloud Platform | [Cloud Run](https://cloud.google.com/run) | [google-github-actions/deploy-cloudrun](https://github.com/google-github-actions/deploy-cloudrun) |

These services requires some prerequisite setup:
* for Azure, we need to create a resource group;
* for AWS, we need to create an ECS cluster and a ECS service from a task definition;
* for GCP, we need to enable some APIs and set up an Artifcat registry (Cloud Run does not allow to deploy from a public container registry)

I have detailed all the steps I've done in the [`doc/cloud-setup` foder](https://github.com/ghsioux-octodemo/spring-petclinic/tree/main/docs). After this, the three cloud environments are ready for our workflow to deploy the application.

Also, in the next section we're going to set up stuff on the cloud providers, and I've created a codespace.


Cool cool cool, we know how we're going to deploy and everything is ready. But we need to authenticate first, right?



## 4 - The pieces together a.k.a. time to trigger

Since it is likely that I'll deploy other applications like this in the future, I've created reusable workflows for each cloud providers that I can use in other repositories. These workflows are located under the [`.github/workflows/reusable_workflows`](https://github.com/ghsioux-octodemo/spring-petclinic/tree/main/.github/workflows/reusable_workflows/) folder, and are called by the main are called by the ["🚀 Multi-cloud deployment"]() workflow which  

So to recap, so far we have:
* set up services accounts and associated RBAC on the cloud providers
* have created reusable workflows that we can use to deploy (with OIDC authentication)
* defined the triggers to implement versioned deployment.

: Azure, AWS and GCP. In addition to the multi-cloud aspect, this deployment will have the following properties:
* __automated__: we'll use Github Actions to perform the deployment;
* __passwordless__: we'll use OIDC to authenticate to the cloud providers; 
* __versioned__: we'll use GitHub releases and protected tags to choose which version of the application to deploy.
* __geo-distributed__: (in this is only one of the key benefit of multi-cloup).

Let's now dive into the details.


Now, we can put all these pieces together in a single workflow, which is available [here]().

Let's trigger it by creating a release and publishing it. I am allowed to create a `v.*` tag.

Now let's trigger manually on some tags.

Show the Deployment Active and Waiting.

## 5 - Next steps

parler des prechecks, et mentionner branch model vs. fork
audit log for release creation? workflow execution?
gestion du package de l'application (docker image): public ou authent?
mentionner qu'on met le package container image en public car pas envie de se prendre la tête sur l'authent à ghcr côté cloud provider

mentionner https://docs.github.com/en/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility#disabling-automatic-inheritance-of-access-permissions-in-an-organization et comme repo public, on a l'image publique ce qui facilite grandement son accès depuis les différents cloud providers

* implement load-balancing mechanisms (e.g. [Traefik](https://traefik.io/), DNS round-robin, etc.);
* push further to implemebt A/B testing / canary release;
* release protection rules and tag rules + triggering actor rules: investigate the use of the fork model
