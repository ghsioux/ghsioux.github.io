> \[!IMPORTANT]
>
> * Billing for premium requests began on June 18, 2025 for all paid Copilot plans, and the request counters were only set to zero for paid plans.
> * Billing for premium requests is **not available** if you use GitHub Enterprise Cloud with data residency. This means premium requests are not counted and you will not be charged for them. Support for premium request billing in data residency environments is coming soon.
> * Premium request counters reset on the 1st of each month at 00:00:00 UTC. See [Monitoring your Copilot usage and entitlements](/en/copilot/managing-copilot/understanding-and-managing-copilot-usage/monitoring-your-copilot-usage-and-entitlements).
> * Certain requests may experience rate limits to accommodate high demand. Rate limits restrict the number of requests that can be made within a specific time period.

## What is a request?

A request is any interaction where you ask Copilot to do something for you—whether it’s generating code, answering a question, or helping you through an extension. Each time you send a prompt in a chat window or trigger a response from Copilot, you’re making a request.

## What are premium requests?

Some Copilot features use more advanced processing power and count as premium requests. The number of premium requests a feature consumes can vary depending on the feature and the AI model used.

### Premium features

The following Copilot features can use premium requests.

| Feature                                                                                                                               | Premium request consumption                                                                                                                                                                                                                                                                                                                                                           |
| ------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Copilot Chat](/en/copilot/using-github-copilot/copilot-chat)                                                                         | Copilot Chat uses **one premium request** per user prompt, multiplied by the model's rate.                                                                                                                                                                                                                                                                                            |
| [Copilot coding agent](/en/copilot/concepts/about-copilot-coding-agent)                                                               | Copilot coding agent will make multiple premium requests to complete a single task. On average, Copilot coding agent uses **30-50 premium requests** each time it is invoked. The exact number of premium requests will vary depending on the task’s complexity and the number of required steps. Copilot coding agent uses a fixed multiplier of 1 for the premium requests it uses. |
| [Agent mode in Copilot Chat](/en/copilot/using-github-copilot/copilot-chat/asking-github-copilot-questions-in-your-ide#copilot-edits) | Agent mode uses **one premium request** per user prompt, multiplied by the model's rate.                                                                                                                                                                                                                                                                                              |
| [Copilot code review](/en/copilot/using-github-copilot/code-review/using-copilot-code-review)                                         | When you assign Copilot as a reviewer for a pull request, **one premium request** is used each time Copilot posts comments to the pull request.                                                                                                                                                                                                                                       |
| [Copilot Extensions](/en/copilot/building-copilot-extensions/about-building-copilot-extensions)                                       | Copilot Extensions uses **one premium request** per user prompt, multiplied by the model's rate.                                                                                                                                                                                                                                                                                      |
| [Copilot Spaces](/en/copilot/using-github-copilot/copilot-spaces/about-organizing-and-sharing-context-with-copilot-spaces)            | Copilot Spaces uses **one premium request** per user prompt, multiplied by the model's rate.                                                                                                                                                                                                                                                                                          |

## How do request allowances work per plan?

If you use **Copilot Free**, your plan comes with up to 2,000 code completion requests and up to 50 premium requests per month. All chat interactions count as premium requests.

If you're on a **paid plan**, you get unlimited code completions and unlimited chat interactions using the included models (GPT-4.1 and GPT-4o). Rate limiting is in place to accommodate for high demand. See [Rate limits for GitHub Copilot](/en/copilot/troubleshooting-github-copilot/rate-limits-for-github-copilot).

Paid plans also receive a monthly allowance of premium requests, which can be used for advanced chat interactions, code completions using premium models, and other premium features. For an overview of the amount of premium requests included in each plan, see [Plans for GitHub Copilot](/en/copilot/about-github-copilot/subscription-plans-for-github-copilot#comparing-copilot-plans).

> \[!NOTE]
> If a user has licenses from multiple enterprises, or standalone organizations, they must make a selection using the "Usage billed to" drop down in order to utilize premium requests. The billing entity selected will be billed for any premium requests they make. See [Monitoring your Copilot usage and entitlements](/en/copilot/managing-copilot/monitoring-usage-and-entitlements/monitoring-your-copilot-usage-and-entitlements#managing-premium-request-billing-with-multiple-copilot-licenses).

## What happens to unused requests at the end of the month?

Unused requests for the previous month do not carry over to the following month.

## What if I run out of premium requests?

> \[!NOTE]
> Additional premium requests are not available to:
>
> * Users on Copilot Free. To access more premium requests, upgrade to a paid plan.
> * Users who subscribe, or have subscribed, to Copilot Pro or Copilot Pro+ through GitHub Mobile on iOS or Android.

If you're on a **paid plan** and use all of your premium requests, you can still use Copilot with one of the included models for the rest of the month. This is subject to change. Response times for the included models may vary during periods of high usage. Requests to the included models may be subject to rate limiting. See [Rate limits for GitHub Copilot](/en/copilot/troubleshooting-github-copilot/rate-limits-for-github-copilot).

If you need more premium requests beyond your monthly allowance, you can:

* Set a spending limit for additional premium requests. See [Using budgets to control spending on metered products](/en/billing/managing-your-billing/using-budgets-control-spending).
* Upgrade to a higher plan.

These actions can be taken by organization owners, billing managers, and personal account users.

> \[!IMPORTANT] By default, all budgets are set to zero and premium requests over the allowance are rejected unless a budget has been created. Additional premium requests beyond your plan’s included amount are billed at $0.04 USD per request.

## Model multipliers

The available models vary depending on your Copilot plan. See [Plans for GitHub Copilot](/en/copilot/about-github-copilot/plans-for-github-copilot#models).

> \[!NOTE]
> The models included with Copilot plans are subject to change.

Each model has a premium request multiplier, based on its complexity and resource usage. If you are on a paid Copilot plan, your premium request allowance is deducted according to this multiplier.

GPT-4.1 and GPT-4o are the included models, and do not consume any premium requests if you are on a **paid plan**.

If you use **Copilot Free**, you have access to a limited number of models, and each model will consume one premium request when used. For example, if you make a request using the o3-mini model, your interaction will consume **one premium request**, not 0.33 premium requests.

<div class="ghd-tool rowheaders">

| Model                      | Multiplier for **paid plans** | Multiplier for **Copilot Free** |
| -------------------------- | ----------------------------- | ------------------------------- |
| GPT-4.1                    | 0                             | 1                               |
| GPT-4o                     | 0                             | 1                               |
| GPT-4.5                    | 50                            | Not applicable                  |
| Claude Sonnet 3.5          | 1                             | 1                               |
| Claude Sonnet 3.7          | 1                             | Not applicable                  |
| Claude Sonnet 3.7 Thinking | 1.25                          | Not applicable                  |
| Claude Sonnet 4            | 1                             | Not applicable                  |
| Claude Opus 4              | 10                            | Not applicable                  |
| Gemini 2.0 Flash           | 0.25                          | 1                               |
| Gemini 2.5 Pro             | 1                             | Not applicable                  |
| o1                         | 10                            | Not applicable                  |
| o3                         | 1                             | Not applicable                  |
| o3-mini                    | 0.33                          | 1                               |
| o4-mini                    | 0.33                          | Not applicable                  |

</div>

## Examples of premium request usage

Premium request usage is based on the model’s multiplier and the feature you’re using. For example:

* **Using GPT-4.5 in Copilot Chat**: With a 50× multiplier, one interaction counts as 50 premium requests.
* **Using GPT-4.1 on Copilot Free**: Each interaction counts as 1 premium request.
* **Using GPT-4.1 on a paid plan**: No premium requests are consumed.