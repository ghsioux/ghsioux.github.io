---
layout: post
title: "EMU mappings manipulation"
description: A story about tweaking user/group mappings when provisioning Enterprise Managed Users.   
comments: false
tags: actions oidc container-app azure aws gcp release reusable-workflows rollback
minute: 30
---

Well, well, well... It's been a while since I last wrote a blog post. Since then, the sun has shone upon us, the leaves turned green before falling to the ground, and the chill has come and gone. This is maybe what Mufasa meant when he talked about "The Circle of Life."

All this time I have trained in the Cloud Dojo, practicing DevSecOps Kung Fu, and one of the new skills I learned was the dark art of manipulating EMU mappings :mage:.

No more talking in riddles, let's get our training sticks out and let's go!

## What is GitHub EMU?

Enterprise Magaged Users is a flavor of GitHub Enterprise Cloud that allows you to manage users and groups directly from your identity provider (IdP). This is designed for companies that want to manage their users in a centralized way and enforce some compliance rules (e.g. MFA, password policies, account naming, etc.). 

The official documentation summarizes it as follows:
> With Enterprise Managed Users, you manage the lifecycle and authentication of your users on GitHub.com from an external identity management system, or IdP. You can provide access to GitHub Enterprise Cloud to people who have existing identities and group membership on your IdP. Your IdP provisions new user accounts with access to your enterprise on GitHub.com. You control usernames, profile data, team membership, and repository access for the user accounts from your IdP.

GitHub has partnered with several IdPs for best integration experience, and one of them is Azure AD. This is the IdP I will be using in this story but the principles are the same for other IdPs.

> **Note:** Since a couple of week it is also now possible to [bring your own IdP to use EMU](https://github.blog/changelog/2024-03-13-public-beta-bring-your-own-identity-provider-to-enterprise-managed-users/). This is a great news for companies that have specific requirements or that want to use an IdP that is has not (yet?) partnered with GitHub.

## User management overview

In EMU, to provide a GitHub account to a developer, few steps needs to be done on the IdP side:
1. assign the user to the enterprise application;
2. wait for the next provisioning cycle or trigger a provisioning on-demand;

Once this is done, the developer will be able to log in to GitHub with his EMU account using his IdP credentials. 

The provisioning cycle consists in creating / updating / deleting GitHub EMU accounts based on the user and group assigned in the IdP. The IdP leverages [the GitHub SCIM API](https://docs.github.com/en/enterprise-cloud@latest/rest/enterprise-admin/scim?apiVersion=2022-11-28) to do so. For more information about SCIM, you can refer to the [SCIM RFC](https://datatracker.ietf.org/doc/html/rfc7644).

For instance, if in my Azure Entra ID, I have assigned for the first time the user `octo_siouxx` to the enterprise application, a corresponding account will be created in the GitHub EMU enterprise. Octo the developer will simply have to log in to GitHub with his Azure AD credentials and he will be able to access the GitHub EMU enterprise.

Not only users can be provisioned, but also groups. If I assign the group `octo_devs` to the enterprise application, all the members of the group will be added to the GitHub EMU enterprise. In addition, I'll be able to [synchronize one or more GitHub teams with the Azure AD group](https://docs.github.com/en/enterprise-cloud@latest/organizations/organizing-members-into-teams/synchronizing-a-team-with-an-identity-provider-group) if I need to. Consider having a group dedicated to Copilot users: in order to remove or add a copilot user, I just have to manage the group membership in Azure AD.

To remove user from the GitHub EMU enterprise, I have to remove it individually from the enterprise application, but also from the groups it belongs to.

## Provisioning mappings

By default, the Azure enterprise application has some predefined mappings for users and groups. 

Here are the default user mappings:

| Source attribute | Target attribute |
|------------------|------------------|
| userPrincipalName | userName |
| displayName | displayName |
| mail | emails[type eq "work"].value |
| givenName | name.givenName |
| surname | name.familyName |

And the default group mappings:

| Source attribute | Target attribute |
|------------------|------------------|
| userPrincipalName | userName |
| displayName | displayName |
| mail | emails[type eq "work"].value |
| givenName | name.givenName |
| surname | name.familyName |

Going back to the octo_siouxx user, after the provisioning is done we have the following attributes in GitHub:


That's neat. But what if I want to customize the mappings? For instance, perhaps the users have multiple emails addresses and one of them is the primary one? Or perhaps the users is actually a "guest" user in my IdP (e.g. contractor) and I'd like to see this information in GitHub?

Let's see how we can do that.

## Custom mappings

