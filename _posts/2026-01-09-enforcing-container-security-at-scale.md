---
layout: post_deeper_toc
title: "Enforcing Container Security at Scale"
description: "How to ensure all Docker images in your organization are scanned with Trivy and attested before merge using custom properties, security configurations, and org rulesets"
comments: false
tags: [rulesets, security, docker, trivy, ghas, artifact-attestation, custom-properties, supply-chain, org-workflows]
minute: 20
toc: true
---

Hey ninjas! 🥷 Today we're diving into a challenge that keeps many platform engineers awake at night: **how do you ensure that every single Docker image built across your organization is properly scanned for vulnerabilities and attested before it reaches production?**

In this dojo session, we'll combine several GitHub features into a powerful security framework:
- **Security Configurations** to enforce code scanning across all repositories
- **Custom Properties** to identify repositories that build and publish container images
- **Organization Rulesets** to require specific workflows on targeted repositories
- **Organization Workflows** to build, scan with Trivy, and attest images

The result? A supply chain security setup that leaves no container behind. Let's get started! ⛩️

> ⚡️ All the code and configurations shown in this post are available in [this repository](https://github.com/ghsioux/container-security-at-scale-demo), ready to be adapted to your own organization.

## 1 - The Challenge

Picture this: your organization has dozens (or hundreds) of repositories. Some build Docker images, some don't. Some push to GitHub Container Registry, others to DockerHub or ECR. How do you ensure that:

1. **Every** repository has code scanning enabled?
2. **Every** Docker image is scanned for vulnerabilities before a PR is merged?
3. **Every** published image has a cryptographic attestation proving its provenance?

Doing this manually? That's a path to madness. Relying on developers to remember? Good luck with that. What we need is **enforcement at the organization level** - a way to automatically apply these security requirements to all relevant repositories.

## 2 - The Strategy

Our approach combines four GitHub features working in harmony:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           ORGANIZATION LEVEL                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────────────┐    ┌──────────────────────┐                       │
│  │  Security Config     │    │  Custom Properties   │                       │
│  │  ──────────────────  │    │  ──────────────────  │                       │
│  │  • Code Scanning: ON │    │  • builds-container  │                       │
│  │  • Secret Scanning   │    │    images: true      │                       │
│  │  • Dependabot        │    │                      │                       │
│  └──────────┬───────────┘    └──────────┬───────────┘                       │
│             │                           │                                   │
│             │    Applied to all repos   │  Tags repos that build images     │
│             ▼                           ▼                                   │
│  ┌──────────────────────────────────────────────────────────────────┐      │
│  │                      Organization Ruleset                        │      │
│  │  ────────────────────────────────────────────────────────────────│      │
│  │  Target: repos where builds-container-images == true             │      │
│  │  Rule: Require org workflow "container-security.yml"             │      │
│  └──────────────────────────┬───────────────────────────────────────┘      │
│                             │                                               │
│                             │ Required on PRs                               │
│                             ▼                                               │
│  ┌──────────────────────────────────────────────────────────────────┐      │
│  │                    Organization Workflow                         │      │
│  │  ────────────────────────────────────────────────────────────────│      │
│  │  1. Build Docker image                                           │      │
│  │  2. Scan with Trivy → Upload SARIF to Code Scanning              │      │
│  │  3. Push to registry                                             │      │
│  │  4. Generate artifact attestation                                │      │
│  └──────────────────────────────────────────────────────────────────┘      │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

Let's break down each component.

## 3 - Security Configuration: The Foundation

[Security configurations](https://docs.github.com/en/enterprise-cloud@latest/code-security/securing-your-organization/enabling-security-features-in-your-organization/configuring-global-security-settings-for-your-organization) allow you to define a baseline security posture that applies to all (or selected) repositories in your organization. Think of it as the foundation of your security pyramid.

Navigate to your organization's **Settings → Code security → Configurations** and create a new configuration:

![Security Configuration](/assets/images/2026-01-09-enforcing-container-security-at-scale/security-configuration.png "Security Configuration")

The key settings we want enabled:

| Feature | Setting | Why |
|---------|---------|-----|
| **Code Scanning** | Enabled (default setup) | Automatically analyzes code for vulnerabilities |
| **Secret Scanning** | Enabled | Detects accidentally committed secrets |
| **Push Protection** | Enabled | Blocks commits containing secrets |
| **Dependabot Alerts** | Enabled | Alerts on vulnerable dependencies |

Once configured, apply this to all repositories in your organization. This ensures that every repo - existing and future - has a consistent security baseline.

> 💡 **Pro tip**: You can set a configuration as the "default" for new repositories, ensuring that security is enforced from day one.

## 4 - Custom Properties: Tagging Your Container Builders

Not all repositories build Docker images. We need a way to identify those that do, and that's where [custom properties](https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-organization-settings/managing-custom-properties-for-repositories-in-your-organization) come in.

Navigate to **Settings → Custom properties** and create a new property:

| Property Name | Type | Allowed Values | Description |
|---------------|------|----------------|-------------|
| `builds-container-images` | Single select | `true`, `false` | Indicates if this repository builds and publishes container images |

![Custom Property Definition](/assets/images/2026-01-09-enforcing-container-security-at-scale/custom-property-definition.png "Custom Property Definition")

Now, for each repository that builds Docker images, set this property to `true`. You can do this:
- Manually via the repository settings
- In bulk via the organization settings
- Programmatically via the [REST API](https://docs.github.com/en/enterprise-cloud@latest/rest/repos/custom-properties?apiVersion=2022-11-28)

![Custom Property Assignment](/assets/images/2026-01-09-enforcing-container-security-at-scale/custom-property-assignment.png "Custom Property Assignment")

This tagging mechanism is the bridge between our security requirements and the repositories they apply to.

## 5 - The Organization Workflow: Build, Scan, Attest

Here's where the magic happens. We'll create a [centralized workflow](https://docs.github.com/en/enterprise-cloud@latest/actions/sharing-automations/creating-workflow-templates-for-your-organization) in a dedicated repository (e.g., `.github-workflows`) that will be required by our ruleset.

This workflow does three critical things:
1. **Builds** the Docker image
2. **Scans** it with Trivy and uploads results to GitHub Code Scanning
3. **Pushes** the image and generates an **artifact attestation**

```yaml
# File: .github/workflows/container-security.yml
# Repository: <YOUR-ORG>/.github-workflows
name: Container Security Scan

on:
  pull_request:
    branches: [main]
  workflow_call:
    inputs:
      dockerfile:
        description: 'Path to Dockerfile'
        required: false
        default: 'Dockerfile'
        type: string
      context:
        description: 'Build context'
        required: false
        default: '.'
        type: string
      image-name:
        description: 'Docker image name'
        required: false
        type: string

permissions:
  contents: read
  security-events: write
  packages: write
  id-token: write
  attestations: write

jobs:
  container-security:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Determine image name
        id: image-name
        run: |
          if [ -n "${{ inputs.image-name }}" ]; then
            echo "name=${{ inputs.image-name }}" >> $GITHUB_OUTPUT
          else
            echo "name=ghcr.io/${{ github.repository }}:${{ github.sha }}" >> $GITHUB_OUTPUT
          fi

      - name: Log in to GitHub Container Registry
        if: github.event_name != 'pull_request'
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build Docker image
        id: build
        uses: docker/build-push-action@v5
        with:
          context: ${{ inputs.context }}
          file: ${{ inputs.dockerfile }}
          push: ${{ github.event_name != 'pull_request' }}
          load: ${{ github.event_name == 'pull_request' }}
          tags: ${{ steps.image-name.outputs.name }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ steps.image-name.outputs.name }}
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH,MEDIUM'

      - name: Upload Trivy results to GitHub Security
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: 'trivy-results.sarif'
          category: 'container-scan'

      - name: Check for vulnerabilities
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ steps.image-name.outputs.name }}
          format: 'table'
          exit-code: '1'
          severity: 'CRITICAL,HIGH'

      - name: Attest image
        if: github.event_name != 'pull_request'
        uses: actions/attest-build-provenance@v1
        with:
          subject-name: ${{ steps.image-name.outputs.name }}
          subject-digest: ${{ steps.build.outputs.digest }}
          push-to-registry: true
```

This workflow is powerful because it:
- **Runs on pull requests** to catch issues before merge
- **Is reusable** via `workflow_call`, allowing repositories to customize paths
- **Fails the build** if critical or high vulnerabilities are found
- **Uploads results** to GitHub Security tab for visibility
- **Generates attestations** for supply chain verification

## 6 - Organization Ruleset: Enforcement Time

Now we need to ensure this workflow runs on every PR in repositories that build containers. Enter [organization rulesets](https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-organization-settings/managing-rulesets-for-repositories-in-your-organization).

Navigate to **Settings → Rules → Rulesets** and create a new ruleset:

![Organization Ruleset](/assets/images/2026-01-09-enforcing-container-security-at-scale/org-ruleset.png "Organization Ruleset")

Key configuration:

| Setting | Value |
|---------|-------|
| **Enforcement status** | Active |
| **Bypass list** | Organization admins (for emergencies) |
| **Target repositories** | Dynamic list based on custom property |
| **Property filter** | `builds-container-images` == `true` |
| **Rules** | Require status checks to pass before merging |
| **Required checks** | `container-security` |

<details class="details-container">
  <summary class="details-summary"> 📜 Ruleset JSON</summary>

{% highlight json %}
{
  "name": "require-container-security-scan",
  "target": "branch",
  "enforcement": "active",
  "conditions": {
    "ref_name": {
      "include": ["~DEFAULT_BRANCH"],
      "exclude": []
    },
    "repository_property": {
      "include": [
        {
          "name": "builds-container-images",
          "property_values": ["true"]
        }
      ],
      "exclude": []
    }
  },
  "rules": [
    {
      "type": "required_status_checks",
      "parameters": {
        "required_status_checks": [
          {
            "context": "container-security",
            "integration_id": 15368
          }
        ],
        "strict_required_status_checks_policy": true
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

This ruleset dynamically applies to repositories based on their custom property value. Add a new repository with `builds-container-images: true`? It automatically gets the security requirements. No manual updates needed.

## 7 - Putting It All Together

Let's see this in action with a sample repository that builds a Docker image.

### 7.1 - Repository Setup

In your repository that builds containers, create a simple workflow that calls the organization workflow:

```yaml
# .github/workflows/build.yml
name: Build and Scan

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  container-security:
    uses: <YOUR-ORG>/.github-workflows/.github/workflows/container-security.yml@main
    with:
      dockerfile: 'Dockerfile'
      context: '.'
    permissions:
      contents: read
      security-events: write
      packages: write
      id-token: write
      attestations: write
```

### 7.2 - The Enforcement Flow

When a developer opens a PR:

1. **The workflow runs** automatically because it's triggered on `pull_request`
2. **Docker image is built** from the PR code
3. **Trivy scans** the image for vulnerabilities
4. **Results upload** to GitHub Security tab for review
5. **Build fails** if critical/high vulnerabilities are found
6. **PR is blocked** by the ruleset until the check passes

![PR Check Required](/assets/images/2026-01-09-enforcing-container-security-at-scale/pr-check-required.png "PR Check Required")

When merged to main:

1. **Image is pushed** to GitHub Container Registry
2. **Attestation is generated** linking the image to its source
3. **Provenance is established** for downstream verification

![Security Scan Results](/assets/images/2026-01-09-enforcing-container-security-at-scale/security-scan-results.png "Security Scan Results")

## 8 - Verifying Attestations

The final piece of the puzzle: verification. Anyone can verify the provenance of your images using the GitHub CLI:

```bash
# Verify an image's attestation
gh attestation verify oci://ghcr.io/your-org/your-image:tag \
  --owner your-org

# Output shows:
# ✓ Verification succeeded!
# 
# Attestation verified against owner: your-org
# 
# PREDICATE:
#   Repository: your-org/your-repo
#   Workflow: .github/workflows/build.yml
#   Commit: abc123...
```

This verification proves:
- The image was built by your organization
- It came from a specific repository and commit
- It was built using a specific workflow
- It hasn't been tampered with since creation

## 9 - Advanced: Multi-Registry Support

What if you publish to multiple registries (DockerHub, ECR, ACR)? The workflow can be extended:

```yaml
- name: Push to multiple registries
  if: github.event_name != 'pull_request'
  run: |
    # GitHub Container Registry
    docker push ghcr.io/${{ github.repository }}:${{ github.sha }}
    
    # DockerHub
    docker tag ${{ steps.image-name.outputs.name }} docker.io/${{ secrets.DOCKERHUB_ORG }}/${{ github.event.repository.name }}:latest
    docker push docker.io/${{ secrets.DOCKERHUB_ORG }}/${{ github.event.repository.name }}:latest
    
    # Amazon ECR
    docker tag ${{ steps.image-name.outputs.name }} ${{ secrets.ECR_REGISTRY }}/${{ github.event.repository.name }}:latest
    docker push ${{ secrets.ECR_REGISTRY }}/${{ github.event.repository.name }}:latest
```

You can generate attestations for each registry:

```yaml
- name: Attest GHCR image
  uses: actions/attest-build-provenance@v1
  with:
    subject-name: ghcr.io/${{ github.repository }}
    subject-digest: ${{ steps.build.outputs.digest }}
    push-to-registry: true

- name: Attest DockerHub image
  uses: actions/attest-build-provenance@v1
  with:
    subject-name: docker.io/${{ secrets.DOCKERHUB_ORG }}/${{ github.event.repository.name }}
    subject-digest: ${{ steps.build.outputs.digest }}
    push-to-registry: false
```

## 10 - Monitoring and Reporting

With this setup, you get centralized visibility:

1. **Security Dashboard**: View all container scan results across the organization
2. **Compliance Reports**: Track which repositories have security checks enabled
3. **Custom Properties View**: Quickly identify all repositories building containers
4. **Ruleset Insights**: Monitor bypass requests and enforcement status

Navigate to **Security → Overview** to see a consolidated view of all security findings, including container vulnerabilities.

## 11 - Best Practices

Here are some ninja tips 🥷 from the trenches:

### 11.1 - Vulnerability Thresholds

Don't fail builds on every vulnerability. Be pragmatic:

```yaml
# Fail on critical/high during PR
exit-code: '1'
severity: 'CRITICAL,HIGH'

# Report all severities to Security tab
severity: 'CRITICAL,HIGH,MEDIUM,LOW'
```

### 11.2 - Caching Strategy

Use GitHub Actions cache to speed up builds:

```yaml
cache-from: type=gha
cache-to: type=gha,mode=max
```

This can reduce build times by 50%+ for subsequent runs.

### 11.3 - Custom Trivy Policies

Create organization-specific security policies:

```yaml
- name: Run Trivy with custom policy
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: ${{ steps.image-name.outputs.name }}
    format: 'sarif'
    output: 'trivy-results.sarif'
    severity: 'CRITICAL,HIGH,MEDIUM'
    vuln-type: 'os,library'
    trivyignores: '.trivyignore'
```

### 11.4 - Gradual Rollout

Don't enforce everything at once. Roll out gradually:

1. **Week 1**: Enable security configuration (baseline)
2. **Week 2**: Add custom properties to tag repositories
3. **Week 3**: Deploy organization workflow (non-required)
4. **Week 4**: Enable ruleset in "evaluate" mode
5. **Week 5**: Activate enforcement

This gives teams time to adapt and fix issues.

### 11.5 - Exception Handling

Some repositories may need exceptions (legacy apps, vendors):

```json
"bypass_actors": [
  {
    "actor_id": 5,
    "actor_type": "RepositoryRole",
    "bypass_mode": "always"
  },
  {
    "actor_id": 123,
    "actor_type": "Team",
    "bypass_mode": "pull_request"
  }
]
```

## 12 - Troubleshooting Common Issues

### Issue: Workflow not required

**Symptom**: PR merges without security check

**Solution**: Verify the repository has `builds-container-images: true` set and the ruleset is active.

### Issue: Trivy scan fails with timeout

**Symptom**: Job times out during vulnerability scanning

**Solution**: Increase timeout and use caching:

```yaml
timeout-minutes: 30
cache-from: type=gha
```

### Issue: Attestation fails

**Symptom**: `Error: failed to generate attestation`

**Solution**: Ensure permissions are correctly set:

```yaml
permissions:
  id-token: write
  attestations: write
  packages: write
```

## 13 - Conclusion

By combining Security Configurations, Custom Properties, Organization Rulesets, and Organization Workflows, we've built a comprehensive container security framework that:

✅ **Scales** automatically to new repositories  
✅ **Enforces** security requirements without manual intervention  
✅ **Provides** visibility into vulnerabilities before production  
✅ **Establishes** cryptographic proof of image provenance  
✅ **Maintains** flexibility for different repository needs  

This approach shifts security left while maintaining developer velocity. The platform team defines the guardrails once, and they automatically apply across the entire organization.

Remember: security at scale isn't about more tools, it's about **smart orchestration** of the tools you already have. 

Now go forth and secure those containers, ninja! 🥷⛩️

---

## Further Reading

- [GitHub Security Configurations Documentation](https://docs.github.com/en/enterprise-cloud@latest/code-security/securing-your-organization/enabling-security-features-in-your-organization/configuring-global-security-settings-for-your-organization)
- [Custom Properties API Reference](https://docs.github.com/en/enterprise-cloud@latest/rest/repos/custom-properties)
- [Organization Rulesets Guide](https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-organization-settings/managing-rulesets-for-repositories-in-your-organization)
- [Artifact Attestations Documentation](https://docs.github.com/en/actions/security-for-github-actions/using-artifact-attestations)
- [Trivy GitHub Action](https://github.com/aquasecurity/trivy-action)

_Happy scanning! 🔍🐳_
