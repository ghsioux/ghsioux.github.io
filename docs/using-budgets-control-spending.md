Budgets and alerts allow you to track spending on metered products for your accounts, organizations, cost centers (enterprise only), and repositories. By setting a monthly budget, you can monitor your spending and receive notifications by email when your spending exceeds certain preset percentages of your budget threshold. This can help you stay within your budget and avoid overspending.

If your account does not have a valid payment method on file, usage is blocked once you use up your quota.

By default, if you have a valid payment method on file, spending is limited to $0 USD until you set a budget. You can set and manage a budget to limit spending for a product or SKU.

<!--Billing: default budget-->

## About budgets

Each budget has a type and a scope that define which paid use contributes to spending against the budget.

* **Type**: Defines which metered product or SKU is measured.
* **Scope**: Defines whether the budget applies to the whole account, or to a subset of repositories, organizations, or cost centers (enterprise only).

### Your first billing cycle after creating a budget

When you first create a budget, be aware that the budget applies only to metered usage from the date of its creation onwards. Any use made before you created the budget, is not included in the calculations. This means that you may exceed your budget in the first billing cycle after you create your budget, even if you select the option stop usage when the limit is reached.

### Budget limitations

For license-based products such as GitHub Copilot, Advanced Security, GitHub Team, and GitHub Enterprise, setting a budget does not prevent usage over the limit but does provide alerts.
Budgets and alerts are not available for pre-paid volume licenses.

## Deciding on the type and scope for a budget

When deciding on the type and scope for a budget, remember that the use of metered products is applied towards **all applicable** budgets. If any applicable budget with "Stop usage when budget limit is reached" enabled is exhausted, additional usage is blocked.

![Screenshot of budgets for "octo-org": "Actions" budget is $50 and "Actions Linux 96-core" budget is $100. All the "Actions" budget has been used.](/assets/images/help/billing/org-budget-example.png)

In this example, the organization has set a budget of $50 for the "Actions" product and a budget of $100 for one of the SKUs within the "Actions" product. The organization has used all the included quota of actions minutes and an extra $50 of billed minutes. Some of the extra use was for Linux 96-core runners so it is applied to both budgets. Overall, the organization has used the full budget for the "Actions" product of $50. Members are now blocked from using all GitHub-hosted runners until the next billing cycle or until the "Actions" product budget is increased. The SKU budget for Linux 96-core runners serves no purpose and is confusing, so should be deleted.

We recommend that you avoid creating overlapping budgets for the use of a product and a SKU, or an organization and a repository, so that users are not unexpectedly blocked from using a feature that they rely on. Alternative, you may prefer to monitor use without blocking users by disabling the "Stop usage when budget limit is reached" option.

## Managing budgets for your personal account

You can set budgets and receive alerts when your usage reaches 75%, 90%, or 100% of your defined budget. Budgets can be scoped at the repository or product level, depending on the product.

1. Open your billing overview page: <https://github.com/settings/billing>.

2. Click **Budgets and alerts**.

3. To create a new budget, click **New budget**.

4. Under "Budget Type" select either **Product-level budget** or **SKU-level budget**.

   * To create a Product-level budget, choose a metered product from the dropdown.
   * To create a SKU-level budget, choose a SKU from the dropdown. This limits spending for an individual SKU.

5. Under "Budget scope", set the scope of spending for this budget.

6. Under "Budget", set a budget amount.

   To stop any usage and further spending once the budget limit is reached, select **Stop usage when budget limit is reached**, if available.

   > \[!IMPORTANT] If you do not select **Stop usage when budget limit is reached**, you will be notified by email if you exceed your budget, but usage **will not** be stopped.

7. To receive an alert if your budget has reached 75%, 90% and 100% thresholds, select **Receive budget threshold alerts** under "Alerts". When the budget has reached the specific threshold, you will be notified via email and a banner on GitHub. You may opt out at any time.

8. Click **Create budget**.

To edit or delete a budget, on the "Budget and alerts page", click **Edit** or **Delete** next to the budget you want to edit or delete. Follow the prompts.

## Managing budgets for your organization or enterprise

You can manage budgets for your organization or enterprise by setting a budget, viewing budgets, and editing or deleting budgets.

### Viewing budgets

If you are an organization owner, enterprise owner, or billing manager, your budget is listed at the top of the "Budgets and alerts" page, followed by budgets for smaller scopes.

1. Display the settings for the organization or enterprise account you want to view data for. For example, using the context switcher shown on all personal and organization account settings pages.

   ![Screenshot of the "Public profile" settings for The Octocat. Next to "Your personal profile," a "Switch settings context" link is outlined in orange.](/assets/images/help/settings/context-switcher-button.png)

2. Click **<svg version="1.1" width="16" height="16" viewBox="0 0 16 16" class="octicon octicon-credit-card" aria-label="credit-card" role="img"><path d="M10.75 9a.75.75 0 0 0 0 1.5h1.5a.75.75 0 0 0 0-1.5h-1.5Z"></path><path d="M0 3.75C0 2.784.784 2 1.75 2h12.5c.966 0 1.75.784 1.75 1.75v8.5A1.75 1.75 0 0 1 14.25 14H1.75A1.75 1.75 0 0 1 0 12.25ZM14.5 6.5h-13v5.75c0 .138.112.25.25.25h12.5a.25.25 0 0 0 .25-.25Zm0-2.75a.25.25 0 0 0-.25-.25H1.75a.25.25 0 0 0-.25.25V5h13Z"></path></svg> Billing & Licensing** to display the billing and licensing overview for the account:
   * **Organization** accounts: under "Access" in the sidebar for settings.
   * **Enterprise** accounts: a separate tab at the top of the page.

3. Click **Budgets and alerts**.

4. Optionally, in the enterprise view only, to filter by scope, select **Scope**, then choose a scope.

### Creating a budget

As the onwer of an enterprise or organization account, or as a billing manager, you can set a budget at the account level, or at any level below this.

1. In the "Budgets and alerts" view, click **New budget**.

2. Under "Budget", set a budget amount.

   To stop any usage and further spending once your  reaches the budget limit, select **Stop usage when budget limit is reached**, if available.

   > \[!IMPORTANT] If you do not select **Stop usage when budget limit is reached**, you will be notified by email if you exceed your budget, but usage **will not** be stopped.

3. To receive an alert if your budget has reached 75%, 90% and 100% thresholds, select **Receive budget threshold alerts** under "Alerts". When the budget has reached the specific threshold, you will be notified via email and a banner on GitHub. You may opt out at any time.

   Under "Alert Recipients", select the people who will receive the alerts.

4. Click **Create budget**.

### Editing or deleting a budget

> \[!IMPORTANT] Deleting a budget may remove any limits on spending, depending on your other existing budgets. For example, deleting the default $0 budget for Copilot premium requests allows for unlimited usage.

You can edit or delete a budget at any time, but you cannot change the scope of a budget after creating it.

1. In the "Budgets and alerts" view, click **New budget**.
2. Click **Budgets and alerts**.
3. To edit a budget, in the list of budgets, click <svg version="1.1" width="16" height="16" viewBox="0 0 16 16" class="octicon octicon-kebab-horizontal" aria-label="View actions" role="img"><path d="M8 9a1.5 1.5 0 1 0 0-3 1.5 1.5 0 0 0 0 3ZM1.5 9a1.5 1.5 0 1 0 0-3 1.5 1.5 0 0 0 0 3Zm13 0a1.5 1.5 0 1 0 0-3 1.5 1.5 0 0 0 0 3Z"></path></svg> next to the budget you want to edit, and click **<svg version="1.1" width="16" height="16" viewBox="0 0 16 16" class="octicon octicon-pencil" aria-label="pencil" role="img"><path d="M11.013 1.427a1.75 1.75 0 0 1 2.474 0l1.086 1.086a1.75 1.75 0 0 1 0 2.474l-8.61 8.61c-.21.21-.47.364-.756.445l-3.251.93a.75.75 0 0 1-.927-.928l.929-3.25c.081-.286.235-.547.445-.758l8.61-8.61Zm.176 4.823L9.75 4.81l-6.286 6.287a.253.253 0 0 0-.064.108l-.558 1.953 1.953-.558a.253.253 0 0 0 .108-.064Zm1.238-3.763a.25.25 0 0 0-.354 0L10.811 3.75l1.439 1.44 1.263-1.263a.25.25 0 0 0 0-.354Z"></path></svg> Edit** or **<svg version="1.1" width="16" height="16" viewBox="0 0 16 16" class="octicon octicon-trash" aria-label="trash" role="img"><path d="M11 1.75V3h2.25a.75.75 0 0 1 0 1.5H2.75a.75.75 0 0 1 0-1.5H5V1.75C5 .784 5.784 0 6.75 0h2.5C10.216 0 11 .784 11 1.75ZM4.496 6.675l.66 6.6a.25.25 0 0 0 .249.225h5.19a.25.25 0 0 0 .249-.225l.66-6.6a.75.75 0 0 1 1.492.149l-.66 6.6A1.748 1.748 0 0 1 10.595 15h-5.19a1.75 1.75 0 0 1-1.741-1.575l-.66-6.6a.75.75 0 1 1 1.492-.15ZM6.5 1.75V3h3V1.75a.25.25 0 0 0-.25-.25h-2.5a.25.25 0 0 0-.25.25Z"></path></svg> Delete**.
4. Follow the prompts.