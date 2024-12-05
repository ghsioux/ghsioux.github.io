---
layout: post
title: Securing Actions workflows with Immutable Actions
description: A deep dive into the power of immutable actions, wildcard versioning, and build provenance using artifact attestations in GitHub Actions.
comments: false
tags: actions immutable-actions artifact-attestations security supply-chain-security
minute: 30
---

**Long time no see!**  

I've been honing my skills in the AI peaks in the DevSecOps realm ⛰️🏔️⛰️.  Now I'm back stronger and with new quests - hope you have practiced too. 

Along the way, a customer raised an intriguing concern: **Could immutable actions cause issues with how they reference actions in their workflows?** This sent me on a deep dive into the lore of GitHub Actions. The answer? **All good.** Why? **Immutable actions resolution and wildcard versioning** hold the key. 

But before we dive into that mystery, let’s set the stage: What are immutable actions? Why do they matter? And how do artifact attestations fit into this puzzle? Alright, ninja, let's resume the journey 🥷.

> ⚠️ **Note:** At the time of writing, Immutable Actions are in [limited public preview](https://github.com/features/preview/immutable-actions) and subject to change.


## 1. The Genesis: Artifact Attestations

[Artifact attestations in GitHub](https://docs.github.com/en/enterprise-cloud@latest/actions/security-for-github-actions/using-artifact-attestations/using-artifact-attestations-to-establish-provenance-for-builds) [[1](https://github.blog/security/supply-chain-security/introducing-npm-package-provenance/)] [[2](https://github.blog/security/supply-chain-security/configure-github-artifact-attestations-for-secure-cloud-native-delivery/)] enhance supply chain security by cryptographically linking [artifacts](https://docs.github.com/en/enterprise-cloud@latest/actions/writing-workflows/choosing-what-your-workflow-does/storing-and-sharing-data-from-a-workflow#about-workflow-artifacts) to their source code and build process. They ensure both integrity and provenance, reducing the risk of tampered or malicious software:

- **Integrity:** By using cryptographic signatures, artifact attestations ensure that the artifact remains untampered with since its creation. GitHub utilizes Sigstore for these cryptographic signatures.

- **Origin:** Provenance, or the origin of the artifact, is verified through detailed metadata contained in the attestations. This includes for instance the specific commit, the GitHub Actions workflow, and the build environment, allowing full traceability back to the original source. GitHub uses the Provenance feature from OpenSSF's SLSA (Supply Chain Levels for Software Artifacts) framework to provide this.

The following table compares the security implications of artifacts with and without cryptographic signatures, and with and without provenance metadata:

|                           | **With Cryptographic Signature**                                  | **Without Cryptographic Signature**                             |
|---------------------------|--------------------------------------------------------------------|------------------------------------------------------------------|
| **With Provenance Metadata**            | 🟢 **Secure**<br>- Full traceability.<br>- Artifact integrity assured.<br>- Minimal risk (if implemented correctly). | 🟡 **Partial Security**<br>- Traceability available.<br>- Artifacts may be tampered post-build.<br>- Risks of distribution attacks (e.g., artifact substitution). |
| **Without Provenance Metadata**           | 🟡 **Partial Security**<br>- Artifact integrity assured.<br>- No traceability.<br>- Loss of reproducibility.<br>- Builds from unverified sources. | 🔴 **Insecure**<br>- No traceability or integrity assurance.<br>- High risk of malicious artifacts and insecure builds. |

As a last note, remember that attestations are only effective if [verified](https://docs.github.com/en/enterprise-cloud@latest/actions/security-for-github-actions/using-artifact-attestations/using-artifact-attestations-to-establish-provenance-for-builds#verifying-artifact-attestations-with-the-github-cli), making it crucial to validate signed artifacts, particularly for binaries, packages, and manifests. 



## 2. The Aftermath: Immutable Actions

Now that we know what are artifact attestations, let's take these concepts a step further and apply them to **GitHub Actions**. Just like build artifacts, it’s essential to ensure that the actions used in workflows are trustworthy and secure. The good news is that we can achieve this through **Immutable Actions**.

Immutable Actions are a new feature in GitHub that ensures the actions used in CI/CD pipelines are secured and untampered. Let's break down how Immutable Actions work with three essential features:

- **Secure Publishing Flow**  
    Immutable Actions are published through a secure process using an Actions token as part of a workflow run. This ensures that the action is firmly tied to its source repository. Furthermore, **artifact attestations** are generated to verify the publishing process, ensuring that actions are legitimate.

- **Package and Tag Immutability**  
    Once an action is published, its version becomes immutable. This means that tags and branches can't be overwritten or changed to point to different artifacts. This feature guarantees that the action version remains consistent over time, protecting against tampering and unexpected changes.

- **Locking Down Actions Resolution**  
    With Immutable Actions, GitHub enforces strict [semantic versioning](https://semver.org/) with automatic wildcard resolution (✦✦ see next section). This ensures that workflows use actions that have a stable and verifiable version. For instance, the action's reference cannot change to a different commit unless the version number reflects it. This prevents mutable references (like branches and tags) from altering the behavior of workflows unexpectedly.

Let’s explore how Immutable Actions compare to the mutable ones and how they mitigate common security risks:

| Aspect               | Before Immutable Actions                                | With Immutable Actions                                 |
|----------------------|----------------------------------------------------------|--------------------------------------------------------|
| **Source & Behavior** | Actions fetched **as code** from GitHub using mutable git tags, making them less traceable and prone to unexpected changes. | Actions served **as packages** from `ghcr.io`, resolved with immutable package versions, ensuring better traceability and consistency. |
| **(Im)mutability** | Vulnerable to tag mutations, allowing potentially malicious code changes. | Secure resolution with immutable tags and verified publishing flow, preventing tampering. |
| **Versioning**        | Wildcard versions (`v1`) manually maintained, leading to inconsistencies. | Reliable resolution of wildcard versions to the latest matching package, eliminating versioning issues (✦✦ see next section). |

## 3. A note on versioning and resolution

Before Immutable Actions, actions were referenced using mutable tags or branches, e.g.: 
```
uses: my-org/my-action@v1                                         # tag
uses: my-org/my-action@v1.2.9                                     # tag
uses: my-org/my-action@main                                       # branch
uses: my-org/my-action@817e909cb2f3398abaecfa3233967a94d6f6b1c8   # commit SHA
```

With Immutable Actions, the reference is resolved to an immutable package version, e.g.:
```
uses: my-org/my-action@1.2.9   # strict semver
uses: my-org/my-action@1.x     # latest version with major version 1
uses: my-org/my-action@1.1.x   # latest version with major & minor version 1
```

Immutable Actions introduce wildcard versioning to balance stability and flexibility in GitHub workflows. Wildcard versioning allows reference like `v1.2` to automatically resolve to the latest action package version in the major semantic version `1.2.x`. This ensures workflows benefit from updates while avoiding breaking changes across major versions.

Here's a diagram illustrating how wildcard versioning works:

```
graph TD
    A["Reference: v1"] -- Resolves to --> B["Latest 1.x Version Available"]
    B -- Chosen --> C["1.3.2"]
    B -.-> D["1.3.1"]
    B -.-> E["1.2.9"]

    F["Reference: v1.2"] -- Resolves to --> G["Latest 1.2.x Version Available"]
    G -- Chosen --> H["1.2.9"]
    G -.-> I["1.2.8"]
```


While wildcard versioning provides flexibility, it challenges the core principle of immutability. Immutable Actions ensure an action’s content and behavior remain consistent, making dynamic changes from wildcard references potentially problematic. For example, if a workflow references `v1.*`, it might unintentionally introduce behavior changes with a new minor release, like moving from `1.2` to `1.3`.

However, wildcard versioning still operates within a defined range, such as minor or patch updates, making it more stable than completely unrestricted versioning.

In summary, wildcard versions like `v1` or `v1.3` allow updates within a controlled set of releases, offering more predictability and security than dynamic, unpredictable version resolutions. For the highest level of stability and immutability, pinning to a specific version (e.g., `v1.3.2` or simply `1.3.2`) is recommended.


## 4 - Practical Example

Enough theory, let's now see how to use Immutable Actions in practice. We'll use [this demo repository](https://github.com/ghsioux/immutable-action-demo) which has Immutable Actions enabled to showcase the steps. It contains a typical structure for a simple custom action written in JavaScript:
  - `index.js`: the action code;
  - `action.yml`: the action's metadata file;
  - `package.json`: the action's package file.

In addition, we have [the `publish-immutable-actions.yml` workflow file](https://github.com/ghsioux/immutable-action-demo/blob/main/.github/workflows/publish-immutable-actions.yml), triggered when a new released is created to publish the immutable action:

```yaml
name: "Publish Immutable Action Version"
on:
  release:
    types: [created]
jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write
      packages: write
    steps:
      - name: Checking out
        uses: actions/checkout@v4
      - name: Publish
        id: publish
        uses: actions/publish-immutable-action@0.0.1
        with:
          github-token: {% raw %}${{ secrets.GITHUB_TOKEN }}{% endraw %}
```

All the magic happens in [the `publish-immutable-action` action](https://github.com/actions/publish-immutable-action).

I created and pushed the tag `v1.0.0`, created a release from this tag and the immutable action [has been published accordingly](https://github.com/ghsioux/immutable-action-demo/actions/runs/12175417462/job/33959091043) (you can see the attestation has been published too in the job's log). [The corresponding package](https://github.com/ghsioux/immutable-action-demo/pkgs/container/immutable-action-demo/317337038?tag=1.0.0) in GHCR has the label `Immutable`.

> screenshot here

I've then [updated the action code](https://github.com/ghsioux/immutable-action-demo/commit/f4b1295e2051fd43a5085dd9514f5ae823ce73aa) and [published a new version `1.0.1`](https://github.com/ghsioux/immutable-action-demo/actions/runs/12175547252) of the immutable action following the same process. 

I can verify the attestation of the immutable action by running the following command, e.g. for version 1.0.1:
```bash
$ gh attestation verify oci://ghcr.io/ghsioux/immutable-action-demo:1.0.1 --bundle-from-oci --owner ghsioux
Loaded digest sha256:e69e029690d0d0d221be231656d12a5160e9006bd458de195846177997ec66ce for oci://ghcr.io/ghsioux/immutable-action-demo:1.0.1
Loaded 1 attestation from oci://ghcr.io/ghsioux/immutable-action-demo:1.0.1
✓ Verification succeeded!

sha256:e69e029690d0d0d221be231656d12a5160e9006bd458de195846177997ec66ce was attested by:
REPO                           PREDICATE_TYPE                  WORKFLOW
ghsioux/immutable-action-demo  https://slsa.dev/provenance/v1  .github/workflows/publish-immutable-actions.yml@refs/tags/v1.0.1
```

And get the associated git commit hash using:
```bash
$ gh attestation verify oci://ghcr.io/ghsioux/immutable-action-demo:1.0.1 --bundle-from-oci --owner ghsioux --format json | jq '.[0].verificationResult.statement.predicate.buildDefinition.resolvedDependencies[0].digest.gitCommit'
"f4b1295e2051fd43a5085dd9514f5ae823ce73aa"
```

That indeed matches [the commit](https://github.com/ghsioux/immutable-action-demo/commit/f4b1295e2051fd43a5085dd9514f5ae823ce73aa) I made for the version `1.0.1`.

Let's now test the action with [the following workflow](https://github.com/ghsioux/immutable-action-demo/blob/653ed94bd9425e1e50c8de8002f16bbe22e84f38/.github/workflows/test-immutable-action.yml):
```yaml
name: "Test Immutable Action"
on:
  workflow_dispatch:
jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - name: Test immutable action (strict SemVer)
        uses: ghsioux/immutable-action-demo@1.0.0

      - name: Test tag resolution to package version (full tag)
        uses: ghsioux/immutable-action-demo@v1.0.0

      - name: Test tag resolution to package version (wildcard versioning patch version)
        uses: ghsioux/immutable-action-demo@v1.0

      - name: Test tag resolution to package version (wildcard versioning minor version)
        uses: ghsioux/immutable-action-demo@v1
```

In [the workflow logs](https://github.com/ghsioux/immutable-action-demo/actions/runs/12175629284/job/33959726210), we can see the following sequence of printed messages:
```bash
Hello immutable shinobi!
Hello immutable shinobi!
Goodbye immutable shinobi!
Goodbye immutable shinobi!
```

Proving the action has been correctly resolved from tag to package version, supporting also wildcard versioning (`v1` and `v1.0` tags both resolving to action package version `1.0.2`). 

That's nice. Now let's test some corner cases. What if I create [a tag `v1.0.2`](https://github.com/ghsioux/immutable-action-demo/releases/tag/v1.0.2) without publishing a corresponding immutable action and try to use it [in a workflow](https://github.com/ghsioux/immutable-action-demo/blob/231b99f91e2507a977f663da70404836f7064be3/.github/workflows/test-immutable-action-tag-without-release.yml)? As expected, that simply [does not work](https://github.com/ghsioux/immutable-action-demo/actions/runs/12176024772/job/33960889984) because there is no matching immutable action package version:

> screenshot

Now let's tackle immutability. I've created a new tag `v1.0.3` from [commit 231b99f](https://github.com/ghsioux/immutable-action-demo/commit/231b99f91e2507a977f663da70404836f7064be3)  and [published](https://github.com/ghsioux/immutable-action-demo/actions/runs/12176234738) the corresponding immutable action. I then updated the code of the action ([commit be580c03](https://github.com/ghsioux/immutable-action-demo/commit/be580c03ebd9ba926f6bdfe516b6635ea9f240c3)) and made the remote tag `v1.0.3` to point to this new commit. I removed the original `v1.0.3` release, created it again, and guess what? Trying to create a new immutable action with that same tag [failed](https://github.com/ghsioux/immutable-action-demo/actions/runs/12176310609/job/33961788301) with the error:

```bash
Error: Unexpected 400 Bad Request response from manifest upload. Errors: BAD_REQUEST - A container with version 1.0.3 has previously existed as an Actions Package and cannot be reused.
```

## 5 - Conclusion

In this journey, we've explored the importance of securing GitHub Actions workflows with Immutable Actions and artifact attestations. By ensuring that actions and artifacts are cryptographically signed and their provenance is verified, we can significantly enhance the security and integrity of our CI/CD pipelines.

By adopting these practices, we can mitigate common security risks associated with mutable actions and artifacts, ensuring that our software supply chain remains secure and trustworthy.

In the dojo, we learn the value of team spirit. So a big shoutout to the amazing team that helped me understand the internals of Immutable Actions: [@tinaheidinger](https://github.com/tinaheidinger), [@thyeggman](https://github.com/thyeggman), [@konradpabjan](https://github.com/konradpabjan), [@conorsloan](https://github.com/conorsloan) and [@jcambass](https://github.com/jcambass).


Stay vigilant, and keep your workflows secure, shinobi! 🥷

Gasshō 🙏