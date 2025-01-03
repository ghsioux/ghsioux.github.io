---
layout: post_deeper_toc
title: "7 Cool Things You Can Do with GitHub Rulesets"
description: "Explore GitHub rulesets and learn 7 cool things to improve your workflows, code quality, and security."
comments: false
tags: [rulesets, policies, compliance, best-practices]
minute: 25
toc: true
---

As a GitHub enthusiast, I strive to make my repositories as pristine, efficient, and secure as a shinobi’s blade 🥷⚔️. Enter GitHub [rulesets](https://docs.github.com/en/enterprise-cloud@latest/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets) - a potent tool for automating and enforcing best practices across repositories and organizations.

While GitHub already offers some [nice examples](https://docs.github.com/en/enterprise-cloud@latest/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/creating-rulesets-for-a-repository#useful-regular-expression-patterns) (like ensuring tags follow the [semantic versioning](https://semver.org/) scheme) and [ruleset recipes](https://github.com/github/ruleset-recipes), I wanted to take things a bit further. In this post, we’ll explore 7 powerful and practical ways to supercharge your repositories using GitHub rulesets.

What makes this special? These use cases come straight from real-world customer challenges and scenarios I’ve tackled.

> ⚡️ For each example, you’ll find the corresponding ruleset JSON in the associated section and in [this repository](https://github.com/ghsioux/cool-rulesets/), ready for [import](https://docs.github.com/en/enterprise-cloud@latest/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/managing-rulesets-for-a-repository#importing-a-ruleset) into your own environment. Let’s get started! 

### 1. Prevent Unauthorized CI/CD Changes

Protecting your CI/CD specifications from unauthorized changes is critical for maintaining a secure and reliable pipeline. This push ruleset prevents direct pushes to the `.github/workflows/` directory using [a path restriction](https://docs.github.com/en/enterprise-cloud@latest/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets#restrict-file-paths), ensuring that only repository administrators can modify GitHub Actions workflows. This adds an additional layer of protection to your CI/CD processes.

<details class="details-container">
  <summary class="details-summary"> 📜 Ruleset JSON</summary>

{% highlight json %}
{
  "id": 2949246,
  "name": "forbid-cicd-changes",
  "target": "push",
  "source_type": "Repository",
  "source": "ghsioux/cool-rulesets",
  "enforcement": "active",
  "conditions": null,
  "rules": [
    {
      "type": "file_path_restriction",
      "parameters": {
        "restricted_file_paths": [
          ".github/workflows/*"
        ]
      }
    }
  ],
  "bypass_actors": [
    {
      "actor_id": 5,
      "actor_type": "RepositoryRole",
      "bypass_mode": "always"
    }
  ]
}
{% endhighlight %}

</details>

##### Other Use-Cases

- **Lock deployment scripts**: Protect files like `scripts/deploy.sh` to prevent unauthorized modifications.
- **Secure infrastructure-as-code files**: Restrict changes to critical files such as `terraform/*.tf`.
- **Protect configuration files**: Limit updates to sensitive configurations, e.g., `config/*.yml`.
- **Control dependency updates**: Restrict updates to files like `package.json` or `requirements.txt` for tighter dependency management.


### 2. Enforce Shorter File Paths for Legacy System Compatibility

Legacy systems often have strict file path length constraints, and exceeding these can cause deployment failures or runtime errors. This push ruleset ensures all file [paths remain within a defined limit](https://docs.github.com/en/enterprise-cloud@latest/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets#restrict-file-path-length), preventing longer paths from being committed. It’s an essential safeguard for repositories interacting with older or constrained systems.

<details class="details-container">
  <summary class="details-summary"> 📜 Ruleset JSON</summary>

{% highlight json %}
{
  "id": 2950733,
  "name": "legacy-system-filepath",
  "target": "push",
  "source_type": "Repository",
  "source": "ghsioux/cool-rulesets",
  "enforcement": "active",
  "conditions": null,
  "rules": [
    {
      "type": "max_file_path_length",
      "parameters": {
        "max_file_path_length": 255
      }
    }
  ],
  "bypass_actors": []
}
{% endhighlight %}

</details>

##### Other Use-Cases

- **Deployment reliability**: Avoid issues during ZIP file generation or deployment processes caused by excessively long paths.  
- **Repository organization**: Maintain clean, manageable, and well-structured repositories by limiting overly nested file paths.  

### 3. Block Binary Files from Being Pushed into the Repository

Preventing binary files from being pushed into your repository avoids unnecessary bloat, ensures faster cloning, and maintains better compatibility with Git versioning tools. This push ruleset [restricts common binary file extensions](https://docs.github.com/en/enterprise-cloud@latest/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets#restrict-file-extensions), such as `.exe`, `.bin`, `.dll`, and more.

<details class="details-container">
  <summary class="details-summary"> 📜 Ruleset JSON</summary>

{% highlight json %}
{
  "id": 3003094,
  "name": "forbid-binary-files",
  "target": "push",
  "source_type": "Repository",
  "source": "ghsioux/cool-rulesets",
  "enforcement": "active",
  "conditions": null,
  "rules": [
    {
      "type": "file_extension_restriction",
      "parameters": {
        "restricted_file_extensions": [
          "*.exe", "*.bin", "*.dll", "*.so", "*.dmg", "*.msi", "*.app",
          "*.zip", "*.tar", "*.gz", "*.rar", "*.7z", "*.xz", "*.bz2",
          "*.iso", "*.vmdk", "*.qcow2", "*.img",
          "*.dat", "*.db", "*.sqlite", "*.bak", "*.ldf", "*.mdf",
          "*.ttf", "*.otf", "*.woff", "*.woff2", "*.eot",
          "*.dwg", "*.dxf", "*.stl", "*.3ds", "*.obj",
          "*.doc", "*.docx", "*.xls", "*.xlsx", "*.ppt", "*.pptx", 
          "*.pdf", "*.swp", "*.tmp", "*.bak"
        ]
      }
    }
  ],
  "bypass_actors": []
}
{% endhighlight %}

</details>

##### Other Use-Cases

- **Media file management**: For file types like `.jpg`, `.png`, `.mp4`, or `.mp3`, consider creating a specific ruleset or leveraging [Git LFS](https://git-lfs.github.com/) for efficient storage.  
- **Optimize code review processes**: Restrict binary files to ensure that code analysis and review tools function without interference, focusing exclusively on source code.
- **Prevent malware spread**: Block potentially harmful files (e.g., `.exe`, `.dll`, `.vbs`, `.js`, `.bat`) from being pushed to the repository, reducing the risk of malware distribution within the codebase and ensuring a safer environment for contributors.

### 4. Restrict Commits to Authors with Company Email Addresses

Ensuring that commits come from authorized sources is crucial for compliance, accountability, and security. By restricting commits to authors with company email addresses, you can maintain organizational standards and prevent unauthorized contributions. This push ruleset [validates the commit author’s email pattern](https://docs.github.com/en/enterprise-cloud@latest/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets#metadata-restrictions) before allowing the commit to proceed.

<details class="details-container">
  <summary class="details-summary">📜 Ruleset JSON</summary>

{% highlight json %}
{
  "id": 3003187,
  "name": "commit-with-company-email",
  "target": "branch",
  "source_type": "Repository",
  "source": "ghsioux/cool-rulesets",
  "enforcement": "active",
  "conditions": null,
  "rules": [
    {
      "type": "commit_author_email_pattern",
      "parameters": {
        "operator": "regex",
        "pattern": "^[a-zA-Z0-9._%+-]+@company-domain\\.com$",
        "negate": false,
        "name": ""
      }
    }
  ],
  "bypass_actors": []
}
{% endhighlight %}
</details>

##### Other Use-Cases
- **Audit and compliance**: Maintain clear traceability of contributions to ensure that only internal team members are making changes, which is especially useful for security audits and compliance checks.
- **Enforce code ownership and responsibility**: Helps track and maintain clear ownership, ensuring accountability for code contributions.

### 5. Enforce Branch Naming Patterns

Maintaining a clear and consistent branch naming structure is vital for effective collaboration and repository organization. Enforcing branch naming conventions, such as `feature/*`, `bugfix/*`, or `release/*`, ensures that branches align with your team's development workflows. This branch ruleset helps streamline collaboration, making it easier to manage and identify different types of work, whether it's new features, bug fixes, or preparation for releases.

<details class="details-container">
  <summary class="details-summary">📜 Ruleset JSON</summary>

{% highlight json %}
{
  "id": 3139547,
  "name": "enforce-branch-naming-convention",
  "target": "branch",
  "source_type": "Repository",
  "source": "ghsioux/cool-rulesets",
  "enforcement": "active",
  "conditions": {
    "ref_name": {
      "exclude": [
        "refs/heads/releases/**/*",
        "refs/heads/feature/**/*",
        "refs/heads/hotfix/**/*"
      ],
      "include": []
    }
  },
  "rules": [
    {
      "type": "creation"
    }
  ],
  "bypass_actors": []
}
{% endhighlight %}
</details>

##### Additional Use Cases
- **Track specific work areas**: Use naming conventions like `ui/*`, `backend/*`, or `api/*` to categorize branches based on the type of work being done. This helps teams easily identify which parts of the application are being worked on and fosters better cross-functional collaboration.
- **Support multiple environments**: Enforce patterns such as `dev/*`, `qa/*`, `prod/*` for branches associated with different environments, ensuring each stage of the deployment pipeline is clearly defined and tracked.
- **Improved automated pipeline integration**: Name branches like `release/*` or `feature/*` to trigger specific actions in CI/CD pipelines, such as deployment to staging or triggering automated tests.

### 6. Protecting The Production Branch

One of the most common and important rulesets in GitHub Enterprise accounts is protecting the production branch (e.g. `main`). Ensuring its integrity is essential for stable deployments and maintaining a reliable codebase. This branch ruleset enforces strict requirements for merging feature branches, guaranteeing that only thoroughly reviewed, high-quality, and secure code makes it to production.

This ruleset [requires the use of a pull request](https://docs.github.com/en/enterprise-cloud@latest/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets#require-a-pull-request-before-merging), which enforces the following safeguards:

- **Code review**: At least 3 approvals are required, including from [Code Owners](https://docs.github.com/en/enterprise-cloud@latest/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners).
- **Build and test**: A GitHub Actions [job](https://github.com/ghsioux/cool-rulesets/blob/main/.github/workflows/build-and-test.yml#L14) must successfully execute a build and run tests - this is what we call a [required status check](https://docs.github.com/en/enterprise-cloud@latest/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets#require-status-checks-to-pass-before-merging).
- **Security scanning**: A CodeQL scan must succeed, with no high or critical security alerts (this requires [GitHub Advanced Security](https://docs.github.com/en/enterprise-cloud@latest/get-started/learning-about-github/about-github-advanced-security)).

A bypass list has been added to this ruleset, enabling repository owners to bypass it in critical situations, providing flexibility when needed.

<details class="details-container">
  <summary class="details-summary"> 📜 Ruleset JSON</summary>

{% highlight json %}
{
  "id": 106399,
  "name": "production-branch-protection",
  "target": "branch",
  "source_type": "Repository",
  "source": "ghsioux/cool-rulesets",
  "enforcement": "active",
  "conditions": {
    "ref_name": {
      "exclude": [],
      "include": [
        "~DEFAULT_BRANCH" 
      ]
    }
  },
  "rules": [
    {
      "type": "pull_request",
      "parameters": {
        "required_approving_review_count": 3,
        "dismiss_stale_reviews_on_push": false,
        "require_code_owner_review": true,
        "require_last_push_approval": true,
        "required_review_thread_resolution": true
      }
    },
    {
      "type": "required_status_checks",
      "parameters": {
        "strict_required_status_checks_policy": true,
        "do_not_enforce_on_create": true,
        "required_status_checks": [
          {
            "context": "🚧 - Build & Test",
            "integration_id": 15368
          }
        ]
      }
    },
    {
      "type": "code_scanning",
      "parameters": {
        "code_scanning_tools": [
          {
            "tool": "CodeQL",
            "security_alerts_threshold": "high_or_higher",
            "alerts_threshold": "errors"
          }
        ]
      }
    }
  ],
  "bypass_actors": [
    {
      "actor_id": 5,
      "actor_type": "RepositoryRole",
      "bypass_mode": "always"
    }
  ]
}
{% endhighlight %}

</details>

##### Other Use-Cases

- **Enforce quality gates**: Add more required status checks, e.g. to ensure specific code coverage thresholds.
- [**Control merge methods**](https://docs.github.com/en/enterprise-cloud@latest/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets#additional-settings): Restrict merge methods (e.g., squash-only) for a consistent Git history.
- [**Require deployment validation**](https://docs.github.com/en/enterprise-cloud@latest/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets#require-deployments-to-succeed-before-merging): Ensure changes are successfully deployed and tested in staging [environment](https://docs.github.com/en/enterprise-cloud@latest/actions/writing-workflows/choosing-what-your-workflow-does/using-environments-for-deployment) before merging to production.


### 7. Ensure Code Adheres to Linting Standards Before Merging

Maintaining consistent code quality across an organization requires enforcement of linting and formatting standards. This **organization-level** ruleset leverages [repository custom properties](https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-organization-settings/managing-custom-properties-for-repositories-in-your-organization) and [required workflows](https://docs.github.com/en/enterprise-cloud@latest/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets#require-workflows-to-pass-before-merging) to ensure that branches in critical JavaScript/TypeScript repositories adhere to an organization-provided linting workflow before changes can be merged into the default branch.

Unlike repository-specific rulesets, this organization-wide approach scales effortlessly, applying uniform rules across all repositories meeting the defined criteria.

<details class="details-container">
  <summary class="details-summary">📜 Ruleset JSON</summary>

{% highlight json %}

{
  "id": 3135923,
  "name": "enforce-org-linting-standard",
  "target": "branch",
  "source_type": "Organization",
  "source": "ghsioux-example-org",
  "enforcement": "active",
  "conditions": {
    "ref_name": {
      "exclude": [],
      "include": [
        "~DEFAULT_BRANCH"
      ]
    },
    "repository_property": {
      "exclude": [],
      "include": [
        {
          "name": "language",
          "source": "system",
          "property_values": [
            "JavaScript",
            "TypeScript"
          ]
        },
        {
          "name": "criticality",
          "source": "custom",
          "property_values": [
            "high",
            "medium"
          ]
        }
      ]
    }
  },
  "rules": [
    {
      "type": "workflows",
      "parameters": {
        "do_not_enforce_on_create": true,
        "workflows": [
          {
            "repository_id": 586945170,
            "path": ".github/workflows/lint.yaml",
            "ref": "refs/heads/main"
          }
        ]
      }
    }
  ],
  "bypass_actors": [
    {
      "actor_id": 1,
      "actor_type": "OrganizationAdmin",
      "bypass_mode": "always"
    }
  ]
}
{% endhighlight %}
</details>

##### Other Use-Cases

You can extend the concept of organization rulesets to implement compliance and consistency across repositories in your organization, such as by enforcing test automation, security scanning, and even the previous rulesets, ensuring that all projects follow the same processes.


### 🌀 Bonus 🌀

Congratulations on making it to the end! That’s the spirit of a true shinobi. To start the new year on a high note, here are some bonus tips just for you.

#### Bonus 1: Enterprise Rulesets

Until now, we’ve focused on repository and organization-level rulesets. With [a recent update](https://github.blog/changelog/2024-12-04-enterprise-repository-properties-policies-and-rulesets-public-preview/), you can now also create **enterprise-level rulesets**. If you're an [enterprise owner](https://docs.github.com/en/enterprise-cloud@latest/admin/managing-accounts-and-repositories/managing-users-in-your-enterprise/roles-in-an-enterprise#enterprise-owners) on GitHub Enterprise, this is a powerful game-changer that streamlines governance and ensures consistency at scale across your organizations.

#### Bonus 2: Enterprise Repository Policies  
Building on the same update, you can now define **repository policies** at the enterprise level. This gives you the ability to enforce consistent rules across repositories throughout your entire enterprise, improving centralized management and governance across organizations.

Here are a few examples (click to expand):

<details class="details-container">
    <summary class="details-summary">📸 Protect Critical Repositories from Being Deleted</summary>
    
    This policy ensures that only organization admins can delete repositories labeled with the custom property <code>"criticality": "high"</code>. This safeguard prevents both accidental and malicious deletions. It is enforced across all organizations within the enterprise, helping to protect repositories that are essential for your organization's operations and security.

    <img src="/assets/images/2025-01-07-7-cool-things-with-rulesets/prevent-critical-repos-deletion.png" alt="prevent critical repos deletion in enterprise"/>
</details>

<details class="details-container">
    <summary class="details-summary">📸 Restrict Public Repository Creation </summary>    

    This policy restricts the creation of public repositories within your enterprise. It ensures that repositories can only be created as private or internal to promote innersourcing. This is an important safeguard to protect sensitive data and intellectual property from being exposed to the public. Note that this is the default behavior with <a href="https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/understanding-iam-for-enterprises/about-enterprise-managed-users">Enterprise Managed Users</a>.
    <img src="/assets/images/2025-01-07-7-cool-things-with-rulesets/prevent-public-repo.png" alt="prevent public repos in enterprise"/>
</details>

<details class="details-container">
    <summary class="details-summary">📸 Enforce Repository Naming Conventions for IaC-specific Organizations</summary>
    
    This policy targets organizations with names starting with <code>iac-*</code> and enforces a naming convention for their repositories. The repository names must begin with the environment (e.g., <code>`prod-*`</code>, <code>`test-*`</code>, <code>`staging-*`</code>).

    <img src="/assets/images/2025-01-07-7-cool-things-with-rulesets/enforce-iac-repos-naming-convention.png" alt="enforce iac repos naming convention"/>
</details>


## Wrapping Up and Looking Ahead

We’ve explored how GitHub rulesets help streamline workflows, enforce policies, and ensure consistency across repositories and organizations. From blocking binary files to enforcing commit author patterns, these tools foster better collaboration and security.

Like a Zen master refining their craft through meditation, the consistent application of these rules cultivates balance, focus, and discipline in your processes. Keep your practices sharp and your mind clear, and you’ll master the art of streamlined, secure development.

Like a Zen master refining their craft through meditation, the consistent application of these rules cultivates balance, focus, and discipline in your workflows. Keep your practices sharp and your mind clear, and you'll master the art of streamlined, secure development.

Gasshō 🙏