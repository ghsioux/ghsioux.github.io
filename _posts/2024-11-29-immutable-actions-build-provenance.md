---
layout: post
title: Securing Actions workflows with Immutable Actions
description: A deep dive into the power of immutable actions, wildcard versioning, and build provenance using artifact attestations in GitHub Actions.
comments: false
tags: actions devsecops immutable-actions wildcard-versioning build-provenance artifact-attestations security supply-chain
minute: 25
---

**Long time no see!**

I was practicing my skills in the AI hills of the DevSecOps realm. Now I'm back stronger and with new quests, hope you have practiced too. ⚔️

Last time, one of my customers was concerned about immutable actions breaking its tagging strategy. He wanted to know if the tags would be resolved using semantic versioning. The short answer is all good, and this is thanks to splat versioning.

But first, what is immutable action? And why should I need this? What about build provenance?

All right, ninja, the journey resumes here. 🏁

---

**Long time no see!**  
I've been scaling the AI peaks in the DevSecOps realm 🏔️, sharpening my skills and uncovering new treasures to share.  Along the way, a customer raised an intriguing concern: **Could immutable actions cause issues with moving tag references in workflows?** This sent me on a deep dive into the lore of GitHub Actions.

The answer? **All good.** Why? **Immutable Actions wildcard versioning** holds the key. 

But before we dive into that mystery, let’s set the stage: What are immutable actions? Why do they matter? And how do artifact attestations fit into this puzzle?


Alright, ninja, let's resume the journey 🥷.

## 1. Artifact Attestations

Artifact attestations in GitHub serve as cryptographic proof of an artifact's **integrity** and **origin** within GitHub Actions workflows. They enable users to verify that artifacts have not been tampered with and are from a trusted source (the GitHub repository). These attestations generate cryptographically signed metadata, which includes vital information such as the commit the artifact was built from, the build environment, and other relevant details.

- **Integrity:** By using cryptographic signatures, artifact attestations ensure that the artifact remains untampered with since its creation. GitHub utilizes Sigstore for these cryptographic signatures.

- **Origin:** Provenance, or the origin of the artifact, is verified through detailed metadata contained in the attestations. This includes for instance the specific commit, the GitHub Actions workflow, and the build environment, allowing full traceability back to the original source. GitHub uses the Provenance feature from OpenSSF's SLSA (Supply Chain Levels for Software Artifacts) framework to provide this.


Here's a matrix that compares the security implications of artifacts with and without cryptographic signatures, and with and without provenance metadata:

|                           | **With Cryptographic Signature**                                  | **Without Cryptographic Signature**                             |
|---------------------------|--------------------------------------------------------------------|------------------------------------------------------------------|
| **With Provenance Metadata**            | 🟢 **Secure**<br>- Full traceability.<br>- Artifact integrity assured.<br>- Minimal risk (if implemented correctly). | 🟡 **Partial Security**<br>- Traceability available.<br>- Artifacts may be tampered post-build.<br>- Risks of distribution attacks (e.g., artifact substitution). |
| **Without Provenance Metadata**           | 🟡 **Partial Security**<br>- Artifact integrity assured.<br>- No traceability.<br>- Loss of reproducibility.<br>- Builds from unverified sources. | 🔴 **Insecure**<br>- No traceability or integrity assurance.<br>- High risk of malicious artifacts and insecure builds. |

By securing both integrity and provenance, artifact attestations help ensure that the software running in production is exactly as intended. This adds a crucial layer of defense against supply-chain attacks.



## 2. Immutable Actions

Now that we've explored artifact attestations, let's take these concepts a step further and apply them to **GitHub Actions**. Just like build artifacts, it’s essential to ensure that the actions used in workflows are trustworthy and secure. The good news is that we can achieve this through **Immutable Actions**.

Immutable Actions are a new feature in GitHub that ensures the actions used in CI/CD pipelines are secured and untampered. Let's break down how Immutable Actions work with three essential features:

- **Secure Publishing Flow**  
    Immutable Actions are published through a secure process using an Actions token as part of a workflow run. This ensures that the action is firmly tied to its source repository. Furthermore, artifact attestations are generated to verify the publishing process, ensuring that actions are legitimate.

- **Package and Tag Immutability**  
    Once an action is published, its version becomes immutable. This means that tags and branches can't be overwritten or changed to point to different artifacts. This feature guarantees that the action version remains consistent over time, protecting against tampering and unexpected changes.

- **Locking Down Actions Resolution**  
    With Immutable Actions, GitHub enforces strict semantic versioning and wildcard versioning. This ensures that workflows use actions that have a stable and verifiable version. For instance, the action's reference cannot change to a different commit unless the version number reflects it. This prevents mutable references (like branches and tags) from altering the behavior of workflows unexpectedly.

Let’s explore how Immutable Actions compare to the old method and how they mitigate common security risks:

| Aspect               | Before Immutable Actions                                | With Immutable Actions                                 |
|----------------------|----------------------------------------------------------|--------------------------------------------------------|
| **Source & Behavior** | Actions fetched from `codeload.github.com`, using mutable git tags, making them less traceable and prone to unexpected changes. | Actions served as packages from `ghcr.io`, resolved with immutable package versions, ensuring better traceability and consistency. |
| **(Im)mutability** | Vulnerable to tag mutations, allowing potentially malicious code changes. | Secure resolution with immutable tags and verified publishing flow, preventing tampering. |
| **Versioning**        | Wildcard versions (`v1`) manually maintained, leading to inconsistencies. | Reliable resolution of wildcard versions to the latest matching package, eliminating versioning issues. |

About the last row, guess it's time to introduce wildcard versioning.   

## 3. Wildcard Versioning

Immutable Actions introduce wildcard versioning, a mechanism that automatically resolves references like `v1` to the latest action package version within the major version 1 range, based on Semantic Versioning (SemVer) ordering.

Here's a diagram illustrating how wildcard versioning works:

```mermaid
graph TD
    B[4.*] -- resolves to --> C[Latest 4.x Version Available]
    C -- chosen --> D[4.5.1]
    C -.-> K[4.5.0] 
    E[4.2.*]
    E -- resolves to --> F[Latest 4.2.x Version Available]
    F -- chosen --> G[4.2.3]
    F -.-> H[4.2.2]
    F -.-> I[4.2.1]


While wildcard versioning offers flexibility, it poses challenges for Immutable Actions, which are designed to guarantee that an action’s content and behavior remain constant over time. Wildcard references like `4.*` will resolve to the latest version within the specified range (e.g., from 4.1 to 4.2), potentially introducing changes to the action's behavior, which conflicts with the immutability principle.

However, wildcard versioning still operates within a defined range, such as minor or patch updates, making it more stable than completely unrestricted versioning.

In summary, wildcard tags like `4` or `4.*` allow updates within a controlled set of releases, offering more predictability and security than dynamic, unpredictable version resolutions. For the highest level of stability and immutability, pinning to a specific version (e.g., `4.2.1`) is recommended.


## 4 - Practical Example


## 5 - Next steps

## References
https://github.blog/security/supply-chain-security/introducing-npm-package-provenance/
https://github.blog/security/supply-chain-security/configure-github-artifact-attestations-for-secure-cloud-native-delivery/


Gasshō 🙏