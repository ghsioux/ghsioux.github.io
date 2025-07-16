---
layout: post
title: Getting Started with GitHub Copilot Coding Agent
description: A curated guide to mastering GitHub Copilot Coding Agent, from creating PRs to delegating issues like a true code ninja
comments: false
tags: copilot coding-agent ai automation pull-request issues
minute: 10
toc: true
toc_h_min: 2
toc_h_max: 2
---

Greetings, code ninjas! 🥷

After helping a customer unlock the power of [Copilot Coding Agent](https://docs.github.com/en/enterprise-cloud@latest/copilot/concepts/about-copilot-coding-agent#overview-of-copilot-coding-agent) this morning, I realized this knowledge should be shared with the entire dojo. Consider this your essential scroll of wisdom for getting started with your new AI coding companion.

## 1 - The Two Paths of the Coding Agent

Like choosing between the way of the sword or the bow, Copilot Coding Agent offers two primary approaches to enhance your coding journey:

### Path 1: Direct PR Creation 
Ask Copilot to create a Pull Request directly. This is the swift strike approach - perfect when you know exactly what needs to be done.
- 📖 [Full documentation on PR creation](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/agents/copilot-coding-agent/asking-copilot-to-create-a-pull-request)

### Path 2: Issue Delegation
Create an issue and assign it to Copilot. This is the strategic approach - ideal for more complex tasks that benefit from structured planning. [Copilot can even help you create well-structured issues](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/github-flow/using-github-copilot-to-create-issues) before delegating them.
- 📖 [Full documentation on issue delegation](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/agents/copilot-coding-agent/using-copilot-to-work-on-an-issue)

## 2 - Essential Scrolls of Knowledge

Before embarking on your journey, study these core teachings:

- 🎯 **[About Copilot Coding Agent](https://docs.github.com/en/enterprise-cloud@latest/copilot/concepts/about-copilot-coding-agent)** - Learn how Copilot works autonomously to fix bugs, implement features, improve tests, update docs, and address technical debt;
- 📚 **[Complete Documentation](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/agents/copilot-coding-agent)** - The comprehensive guide covering everything from setup to advanced usage:
  - [Enabling Copilot coding agent](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/agents/copilot-coding-agent/enabling-copilot-coding-agent) for your repository
  - [Best practices](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/agents/copilot-coding-agent/best-practices-for-using-copilot-to-work-on-tasks) for optimal results
  - [Tracking sessions](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/agents/copilot-coding-agent/tracking-copilots-sessions) to monitor progress
  - [Reviewing PRs](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/agents/copilot-coding-agent/reviewing-a-pull-request-created-by-copilot) created by Copilot
  - [Extending with MCP](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/agents/copilot-coding-agent/extending-copilot-coding-agent-with-mcp) (Model Context Protocol) for advanced scenarios
  - [Customizing the environment](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/agents/copilot-coding-agent/customizing-the-development-environment-for-copilot-coding-agent) with additional tools
  - [Customizing the firewall](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/agents/copilot-coding-agent/customizing-or-disabling-the-firewall-for-copilot-coding-agent) to control domain access
  - [Troubleshooting guide](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/agents/copilot-coding-agent/troubleshooting-copilot-coding-agent) for common issues


## 3 - The Art of Issue Crafting

When following Path 2 (issue delegation), remember: [a well-structured issue](https://github.blog/developer-skills/github/how-to-create-issues-and-pull-requests-in-record-time-on-github/#h-the-anatomy-of-a-great-github-issue) is like a perfectly balanced kata. Your AI companion performs best with clear, detailed context and specifications.

Leveraging [issue templates](https://docs.github.com/en/enterprise-cloud@latest/communities/using-templates-to-encourage-useful-issues-and-pull-requests/configuring-issue-templates-for-your-repository) is a great idea to ensure consistent structure and let [**Copilot help create issues**](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/github-flow/using-github-copilot-to-create-issues) for you with the right content.

As you master the art of issue creation, know that GitHub Issues itself is rapidly evolving with powerful new features:

- [Issue Types](https://github.blog/changelog/2025-04-09-evolving-github-issues-and-projects/#%f0%9f%8f%97%ef%b8%8f-bring-structure-to-your-issues-with-issue-types) - Bring structure to your issues with customizable types;
- [Sub-Issues](https://github.blog/changelog/2025-04-09-evolving-github-issues-and-projects/#%f0%9f%94%a8-break-it-down-with-sub-issues) - Break down complex tasks into manageable sub-issues;
- [Issue Dependencies](https://github.com/orgs/community/discussions/165749) - Manage relationships between issues for better workflow orchestration.

While, as of today, Copilot doesn't leverage these new issue capabilities directly, we can imagine a future where it could intelligently break down complex requests into sub-issues, respect dependencies, and orchestrate sophisticated multi-phase workflows.


## 4 - The Way of Custom Instructions

Just as a dojo has its own training methods and philosophies, you can teach Copilot Coding Agent to follow your team's specific practices and coding standards in your repository. This is achieved through [**repository custom instructions**](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/agents/copilot-coding-agent/best-practices-for-using-copilot-to-work-on-tasks#adding-custom-instructions-to-your-repository) - a powerful feature that ensures your AI companion adheres to your project's unique requirements.

You can have a look at the `instructions` folder of the [awesome-copilot repository](https://github.com/github/awesome-copilot/) for inspiration.

For VS Code users, you can automatically generate a `.github/copilot-instructions.md` file tailored to your project ([learn more here](https://code.visualstudio.com/docs/copilot/copilot-customization#_generate-an-instructions-file-for-your-workspace)).


## 5 - The Path Forward

As you begin your journey with Copilot Coding Agent, remember that mastery comes through practice. Start with simple tasks, observe how your AI companion approaches problems, and gradually increase complexity.

For those seeking to push beyond current boundaries, keep an eye on [GitHub Next](https://githubnext.com/) where the future of AI-assisted development is being forged. 

May your code be bug-free and your deployments swift!

Gasshō 🙏

#### Additional Readings from the GitHub Blog

> **Note:** GitHub Copilot Coding Agent is rapidly evolving! Some content in these posts reflects the development phase and may be outdated. Always check the [official documentation](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/agents/copilot-coding-agent) for the most current information.

- **July 15, 2025** - **[From chaos to clarity: Using GitHub Copilot agents to improve developer workflows](https://github.blog/ai-and-ml/github-copilot/from-chaos-to-clarity-using-github-copilot-agents-to-improve-developer-workflows/)** - Learn how to set up Copilot coding agent for success.
- **July 1, 2025** - **[From idea to PR: A guide to GitHub Copilot's agentic workflows](https://github.blog/ai-and-ml/github-copilot/from-idea-to-pr-a-guide-to-github-copilots-agentic-workflows/)** - A practical guide demonstrating how to use Copilot's coding agent.
- **June 6, 2025** - **[Assigning and completing issues with coding agent in GitHub Copilot](https://github.blog/ai-and-ml/github-copilot/assigning-and-completing-issues-with-coding-agent-in-github-copilot/)** - A detailed walkthrough of the issue delegation workflow.
- **June 5, 2025** - **[How to create issues and pull requests in record time](https://github.blog/developer-skills/github/how-to-create-issues-and-pull-requests-in-record-time-on-github/)** - Learn the anatomy of well-structured issues and how Copilot amplifies your workflow.

