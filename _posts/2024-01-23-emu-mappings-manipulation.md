---
layout: post  
title: Customizing GitHub EMU Provisioning Mappings in Azure
description: A quick dive into customizing user provisioning mappings in GitHub Enterprise Managed Users (EMU), with a focus on username length limitations.  
comments: false  
tags: enterprise-managed-user emu scim provisioning azure entra-id
minute: 10  
toc: true  
---

As an EMU ([Enterprise Managed Users](https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/understanding-iam-for-enterprises/about-enterprise-managed-users)) administrator, you may have encountered challenges when provisioning users, especially when dealing with **username length limitations**. GitHub imposes a strict **39-character limit** on usernames for EMU, which can clash with long naming conventions in your IdP (Identity Provider).

In this post, I’ll show you how to customize provisioning mappings in Azure Entra ID to handle such scenarios, focusing on the username limit and how we can use **expression mappings** to create compliant usernames.

## 1 - EMU and User Provisioning

GitHub Enterprise Managed Users (EMU) is a feature that allows organizations to manage user accounts and access within GitHub, all through an Identity Provider (IdP). This allows your organization to provision, update, and decommission GitHub users automatically based on changes in your IdP, like Azure AD, Okta, or PingFederate.

User provisioning involves syncing user data (like usernames, email addresses, and roles) from your IdP to GitHub. This is automated to ensure that when users join, change roles, or leave your organization, their GitHub accounts are created, updated, or deleted accordingly. Under the hood, this is standardized through the [SCIM (System for Cross-domain Identity Management)](https://scim.cloud/) specification. 

A core aspect of this process is provisioning mappings, which define how attributes from your IdP map to [GitHub’s required SCIM properties](https://docs.github.com/en/enterprise-cloud@latest/rest/enterprise-admin/scim?apiVersion=2022-11-28#supported-scim-user-attributes). For example, Azure’s `userPrincipalName` (UPN) field may need to map to GitHub’s `userName` SCIM attribute. When default mappings fall short—like exceeding the username character limit—custom mappings come to the rescue.

## 2 - Handling Username Length Limitation

When using Azure Entra ID as the identity provider for EMU, the default mapping for the GitHub usernames will be the following:

![default userName mapping](/assets/images/2024-01-23-emu-mappings-manipulation/defaultmapping.png "default userName mapping")

Here’s the crux of the issue: GitHub usernames must include your [EMU shortcode](https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/iam-configuration-reference/username-considerations-for-external-authentication#shortcodes-on-githubcom) suffix, leaving less room for the main portion of the username. For instance, a user’s `userPrincipalName` like `firstname-lastname-external` might exceed the 39-character cap when the shortcode is added.

As a reminder, the shortcode is chosen when you create an enterprise with managed users on GitHub.com to represent your enterprise. For example, if the shortcode is `octo`, a provisioned user might have a GitHub username like `mona-cat_octo` where `mona-cat` is derived from the identity provider during provisioning.

Azure Entra ID provides a straightforward solution using the [`Left()` function](https://learn.microsoft.com/en-us/entra/identity/app-provisioning/functions-for-customizing-application-data#left), which trims strings to a specific length:

```java
Left(text, number_of_characters)
```

With:
* `text`: The string you want to shorten (in this case, `userPrincipalName`).
* `number_of_characters`: The number of characters to extract.


For example, if your shortcode plus the `_` is 5 characters (e.g. `_octo`), you can allow up to 34 characters for the UPN:

```java
Left([userPrincipalName], 34)
```

This ensures the entire username, including the shortcode, stays within GitHub’s limit. You can configure this as an expression mapping in Azure Entra ID to automate the transformation.

![custom userName mapping with Left() expression](/assets/images/2024-01-23-emu-mappings-manipulation/leftmapping.png "custom userName mapping with Left() expression")

## 3 - Broadening the Scope

While truncating usernames is a common use-case, there are other scenarios where provisioning mappings may need customization. For example, you might want to:

- **Use a custom field for the usernames**: Companies might have specific fields in their IdP - such as `employeeID` - that better suit their GitHub’s username strategy. By mapping these fields, you can ensure usernames are compliant without manual intervention.

- **Normalize email addresses**: Normalize the emails attribute to ensure a consistent domain suffix for all users. For instance, if your organization uses multiple domains, you might standardize email addresses so that all users appear with the same domain in GitHub (e.g., `john.doe@sub.example.com` becomes `john.doe@example.com`).

- **Customize display names**: Use the `name.givenName` and `name.familyName` attributes to create a more standardized or localized display name format for users in GitHub. For example, if your organization prefers full names in a "LastName, FirstName" format, you could map and combine these attributes accordingly.

## 4 - Wrapping Up and Next Steps

We’ve seen how to tackle GitHub’s username length limitations by customizing provisioning mappings in Azure Entra ID. By leveraging expression mappings like `Left`, you can ensure usernames stay compliant while maintaining automation and consistency. This approach can also be extended to normalize other attributes like email addresses or display names, adapting the provisioning process to fit your organizational needs.

What if you're not using Entra ID? No worries—GitHub supports various identity providers, including **[Okta](https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/provisioning-user-accounts-with-scim/configuring-scim-provisioning-with-okta)** and **[PingFederate](https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/provisioning-user-accounts-with-scim/configuring-authentication-and-provisioning-with-pingfederate)**, which offer similar customization options for SCIM attribute mappings. Additionally, for other IdPs, you can leverage GitHub’s **[Open SCIM Configuration](https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/provisioning-user-accounts-with-scim/configuring-scim-provisioning-for-users#configuring-provisioning-for-other-identity-management-systems)** and its **[SCIM API](https://docs.github.com/en/enterprise-cloud@latest/rest/enterprise-admin/scim?apiVersion=2022-11-28)** to build fully tailored integrations. These tools provide the flexibility to manage provisioning while adhering to GitHub’s constraints, no matter your setup.

That’s it for today, ninjas — time to resume practice in the Cloud Dojo. 

Gasshō 🙏
