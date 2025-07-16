---
layout: post
title: Using Cost Centers and Budgets to Control GitHub Copilot Spending
description: A guide to controlling Copilot spending in your enterprise using budgets and cost centers
comments: false
tags: copilot cost-management budgets administration enterprise
minute: 20
toc: true
toc_h_min: 2
toc_h_max: 2
---

Welcome back to the dojo, fellow ninjas! 🥷 

Recently, several customers asked me: **"How can we authorize specific groups of users to consume premium requests while controlling the associated costs?"** The answer lies in mastering cost centers and budgets for granular spending control.

In this guide, we'll explore how to set up cost centers, apply budgets, and ensure your teams can leverage powerful AI models without breaking the bank.

> ⚠️ **Note:** Premium request billing and cost centers are available for GitHub Enterprise Cloud customers with Copilot Business or Enterprise licences.

## 1 - Understanding Copilot Premium Requests

[GitHub Copilot](https://github.com/features/copilot) offers two tiers of AI models: standard (unlimited) and premium (metered). Here's what you need to know:

- **Standard models** (GPT-4.1 and GPT-4o as of July 2025): Unlimited requests included with any paid Copilot plan;
- **Premium models** (e.g. o3-mini, Claude Opus 4, Gemini Pro): Limited monthly allowance, then $0.04 per request × [model multiplier](https://docs.github.com/en/enterprise-cloud@latest/copilot/concepts/copilot-billing/understanding-and-managing-requests-in-copilot#model-multipliers) based on the model complexity and resource usage;

It's important to note that premium requests aren't just about using advanced AI models in your IDE. They're also consumed by several other Copilot features such as **Copilot coding agent**, **Copilot code review** and **Copilot Spaces**. This means your premium request budget needs to account for both model usage and these additional features.

The critical detail: **GitHub sets a $0 default budget for premium requests beyond the included allowance**, effectively blocking them to prevent unexpected charges.


| Plan | Monthly Premium Allowance | Price per premium after allowance | Default Budget |
|------|---------------------------|-----------------------------------|----------------|
| Copilot Business | 300 requests/user | $0.04 × model multiplier | $0 (blocked) |
| Copilot Enterprise | 1000 requests/user | $0.04 × model multiplier | $0 (blocked) |

<p align="center"><em>Pricing as of July 2025</em></p>


The challenge becomes clear: How do you enable premium models for users that need them while preventing runaway costs? Enter **cost centers** and **budgets**—GitHub's answer to enterprise spending control.


## 2 - Setting Up Cost Centers

Cost centers in GitHub are logical containers that group resources (i.e. users, organizations, or repositories) for spending tracking and allocation. Think of them as financial dojos where specific teams train with allocated budgets. For Copilot, they serve two critical functions:

- **License Cost Attribution:** Track the monthly cost of Copilot licenses (Copilot Business or Enterprise plans)
- **Premium Request Tracking:** Monitor and control spending on premium AI model requests beyond the included allowance

### What Can You Allocate to Cost Centers?

Cost centers can contain three types of resources, each serving different billing purposes:

- **Users**: For license-based products like Copilot. Added via API. Takes priority for billing.
- **Organizations**: For usage-based products AND as a fallback for Copilot billing. Can be added via UI.
- **Repositories**: For usage-based products like GitHub Actions. Can be added via UI.

For Copilot, both **user assignments** and **organization assignments** work, following the billing hierarchy described below.

A resource can only belong to one cost center at a time. Plan your allocations carefully.

### The Billing Hierarchy

With this understanding of resource allocation, let's examine how GitHub determines which cost center gets charged for Copilot premium requests beyond the included monthly allowance:

1. **Direct User Assignment (Highest Priority)**  
   If a user is explicitly assigned to a cost center, ALL their Copilot charges—both license and premium requests—are tracked there. This assignment overrides any organizational membership.

2. **Organization Assignment (Fallback)**  
   If a user isn't directly assigned to any cost center, charges fall back to the cost center containing their organization (where they received their license).

3. **Enterprise Default (Last Resort)**  
   Without any cost center assignment, charges accumulate at the enterprise level with no granular tracking.

![Cost centers billing hierarchy](/assets/images/2025-07-06-copilot-cost-centers-and-budgets/billing-hierarchy.png "Cost center billing hierarchy"){: style="max-width: 75%; height: auto; display: block; margin: 0 auto;"}


> 💡 **Pro tip:** In Copilot-only enterprise accounts (without GitHub Enterprise), there are no organizations—only direct user assignments work. Plan your cost center strategy accordingly.

> 🔍 **Bonus:** I've created a [gist](https://gist.github.com/ghsioux/your-gist-id) that helps you determine which cost center a user belongs to and identify potential assignment conflicts across your enterprise.

### Creating Cost Centers

Let's create our first cost center. You have two approaches—via the UI or programmatically through the API.

**Method 1: GitHub UI**
1. Navigate to your enterprise settings
2. Click **Billing and licensing** > **Cost centers**
3. Click **New cost center**
4. Provide a descriptive name (e.g., `ai-ninjas-copilot`)
5. Add an optional description for clarity

**Method 2: API Automation**
```bash

ENTERPRISE_SLUG="octodemo" # Replace with your enterprise slug
TOKEN="ghp_4q7....O4" # Replace with your GitHub personal access token, must have `manage_billing:enterprise` scope

# Create cost center via curl
COST_CENTER_RESPONSE=$(curl -L \
  -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  https://api.github.com/enterprises/$ENTERPRISE_SLUG/settings/billing/cost-centers \
  -d '{"name":"ghsioux-ai-ninjas-copilot","description":"Cost center for ML team Copilot usage tracking"}')

# Extract the cost center ID for future operations
COST_CENTER_ID=$(echo $COST_CENTER_RESPONSE | jq -r '.id')
echo "Created cost center with ID: $COST_CENTER_ID"
```

### Populating Cost Centers

After creation, you must assign resources to your cost center. For this example, we'll use **direct user assignment** to ensure precise control over who can access premium requests, though organization assignment would also work as a fallback approach.

Currently (as of July 2025), GitHub does not support adding users to cost centers through the UI—only via API. Here's how to do it:

```bash
# Add users to the cost center
curl -L \
  -BX POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  https://api.github.com/enterprises/$ENTERPRISE_SLUG/settings/billing/cost-centers/$COST_CENTER_ID/resource \
  -d '{"users":["tgrall","ghsioux"]}'

# Verify the assignment
curl -L \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  https://api.github.com/enterprises/$ENTERPRISE_SLUG/settings/billing/cost-centers/$COST_CENTER_ID | \
  jq '.resources[] | select(.type=="User") | .name'
```

## 3 - Implementing Budget Controls

Once you have your cost center set up and populated with users, the next step is to apply a budget to it. This is the key mechanism for controlling spending on Copilot premium requests.

### Understanding Budget Architecture

GitHub's budget system operates on two distinct levels, each serving different control objectives:

| Budget Type | Scope | What It Controls | Examples |
|------------|-------|------------------|----------|
| **Product-Level** | Entire product category | All costs associated with the product | • **GitHub Actions** budget covering all runner types<br>• **GitHub Copilot** budget covering licenses + premium requests |
| **SKU-Level** | Specific billing component within a product | Only the specific SKU costs | • **Actions Linux 96-core** runner budget<br>• **Copilot Premium Request** budget (excludes license costs) |

In our context of Copilot Premium Requests spending, we therefore need to create a **SKU-level budget** specifically for the **Copilot Premium Request SKU**. This allows us to control costs associated with premium requests without affecting the budget with license costs.

### Creating a SKU-Level Budget

By default, your enterprise has a $0 budget for the **GitHub Copilot Premium Request SKU**, which prevents any spending on premium requests over the included allowance. To enable premium requests for a specific cost center, you need to create a new SKU-level budget scoped to that cost center.

Here are the steps to create a budget for your new cost center directly from the GitHub UI:

1.  In your enterprise settings, navigate to **<svg version="1.1" width="16" height="16" viewBox="0 0 16 16" class="octicon octicon-credit-card" aria-label="credit-card" role="img"><path d="M10.75 9a.75.75 0 0 0 0 1.5h1.5a.75.75 0 0 0 0-1.5h-1.5Z"></path><path d="M0 3.75C0 2.784.784 2 1.75 2h12.5c.966 0 1.75.784 1.75 1.75v8.5A1.75 1.75 0 0 1 14.25 14H1.75A1.75 1.75 0 0 1 0 12.25ZM14.5 6.5h-13v5.75c0 .138.112.25.25.25h12.5a.25.25 0 0 0 .25-.25Zm0-2.75a.25.25 0 0 0-.25-.25H1.75a.25.25 0 0 0-.25.25V5h13Z"></path></svg> Billing & Licensing** and click **Budgets and alerts**.
2.  Click **New budget**.
3.  Under "Budget Type," select **SKU-level budget**.
4.  From the "Product" dropdown, choose **Copilot**.
5.  From the "SKU" dropdown, choose **Copilot Premium Request**.
6.  Under "Budget scope," select the cost center you created earlier (e.g., `frontend-team-copilot`).
7.  Set your desired monthly budget amount.
8.  Crucially, to prevent runaway spending, select **Stop usage when budget limit is reached**.
9.  Under "Alerts," configure who should receive email notifications as the budget reaches 75%, 90%, and 100% of its limit.
10. Click **Create budget**.


With this in place, only the users within the `ai-ninjas-copilot` cost center will be able to make premium requests after their included monthly allowance has been reached, and their usage will be capped at the budget you defined.



### Budget Enforcement Mechanics

When a user attempts a premium request, GitHub's billing engine follows this decision tree:

![Budget mechanics decision tree](/assets/images/2025-07-06-copilot-cost-centers-and-budgets/budget-mechanics.png "Budget mechanics decision tree"){: style="max-width: 85%; height: auto; display: block; margin: 0 auto;"}

This enforcement happens in real-time, ensuring immediate feedback to developers when limits are reached.

### Budget Example

Let's walk through a concrete example to understand how budgets work in practice. Imagine you've created a **$100 monthly budget** for the `ai-ninjas-copilot` cost center containing 5 users.

Here's how the budget gets consumed throughout the month:

| Day | User | Model Used | Multiplier | Requests Made | Cost | Running Total |
|-----|------|------------|------------|---------------|------|---------------|
| 1 | Alice | o3-mini | 0.33× | 30 | 30 × 0.33 × $0.04 = $0.40 | $0.40 |
| 3 | Bob | Claude Opus 4 | 10× | 5 | 5 × 10 × $0.04 = $2.00 | $2.40 |
| 5 | Alice | Gemini 1.5 Pro | 1× | 20 | 20 × 1 × $0.04 = $0.80 | $3.20 |
| 8 | Carol | Copilot coding agent | 1× | 40 (avg) | 40 × 1 × $0.04 = $1.60 | $4.80 |
| 12 | Dave | Claude 3.5 Sonnet | 1.25× | 15 | 15 × 1.25 × $0.04 = $0.75 | $5.55 |
| 15 | Eve | o3-mini | 0.33× | 100 | 100 × 0.33 × $0.04 = $1.32 | $6.87 |
| ... | ... | ... | ... | ... | ... | ... |
| 28 | Team | Various | - | - | - | $98.50 |
| 29 | Bob | Claude Opus 4 | 10× | 5 | 5 × 10 × $0.04 = $2.00 | **$100.50** ❌ |

In this scenario:
- The team shares the $100 budget collectively (since budgets are assigned to cost centers, not individuals);
- Different models consume the budget at different rates based on their multipliers;
- Bob's final request on day 29 would exceed the budget and be **rejected**;
- All subsequent premium requests by any team member would be blocked until the budget resets on the 1st of the next month.


## 4 - Monitoring and Tracking Usage

Once your budgets are in place, effective monitoring becomes crucial for maintaining cost control and understanding usage patterns. GitHub provides comprehensive monitoring tools that transform raw data into actionable insights:

- **[Premium Requests Usage reports](https://docs.github.com/en/enterprise-cloud@latest/copilot/tutorials/rolling-out-github-copilot-at-scale/assigning-licenses/managing-your-companys-spending-on-github-copilot#tracking-premium-requests)** deliver per-user consumption metrics to identify power users and optimization opportunities;
- **[Budget alerts](https://docs.github.com/en/enterprise-cloud@latest/copilot/tutorials/rolling-out-github-copilot-at-scale/assigning-licenses/managing-your-companys-spending-on-github-copilot#receive-alerts-for-overspending)** trigger proactive notifications at 75%, 90%, and 100% thresholds, preventing surprise budget exhaustion;
- **[Spending visualizations](https://docs.github.com/en/enterprise-cloud@latest/copilot/tutorials/rolling-out-github-copilot-at-scale/assigning-licenses/managing-your-companys-spending-on-github-copilot#visualize-spending-trends)** reveal patterns across cost centers, enabling data-driven budget allocation decisions;
- for your developers, **[In-IDE usage monitoring](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/monitoring-your-copilot-usage-and-entitlements#viewing-usage-in-your-ide)** provides real-time visibility into your premium request consumption directly within VS Code, Visual Studio, JetBrains IDEs, Xcode, and Eclipse.


Beyond the tooling, successful budget management requires a strategic approach. GitHub's official documentation [recommends](https://docs.github.com/en/enterprise-cloud@latest/copilot/tutorials/rolling-out-github-copilot-at-scale/assigning-licenses/managing-your-companys-spending-on-github-copilot#tracking-premium-requests) to identify power users, understand their use cases, and evaluate whether upgrading to Copilot Enterprise might be more cost-effective. Based on field experience implementing these strategies, here are some complementary practices:

* 💡 **Start Conservative, Then Scale:** Launch with 50% of your estimated budget. Real usage data beats projections every time, and it's far easier to increase budgets based on demonstrated need than to claw back overspending.

* 🎯 **Developer Education Matters:** Create a simple decision tree with your teams regarding which model to use for which use-case. Share the [model selection guide](https://docs.github.com/en/enterprise-cloud@latest/copilot/reference/ai-models/choosing-the-right-ai-model-for-your-task) to help developers choose the right model and implement team guidelines for high-multiplier model usage to maximize budget efficiency.

* 🔧 **Administrative Controls:** Copilot admins can proactively manage costs by enabling or disabling specific models through Copilot policies, providing an additional layer of budget protection beyond user education.



## 5 - The Bigger Picture ☁️

Implementing Copilot budget controls enables teams to harness AI capabilities responsibly while ensuring sustainable innovation. In this guide, we've journeyed through creating cost centers, assigning users, implementing SKU-level budgets, and establishing monitoring practices—giving you the tools to enable AI-powered development while maintaining financial predictability.

Remember: These cost center and budget concepts extend far beyond Copilot to all GitHub Enterprise metered products—Actions minutes, Packages storage, Codespaces compute, and more. Master them here, apply them everywhere across your GitHub Enterprise ecosystem.

Until next time, keep your code clean and your budgets balanced like a zen master! 

Gasshō 🙏