+++
title = "Trusting AI Models in Kubernetes: Introducing the Sigstore Model Validation Operator"
date = "2025-06-23"
tags = ["sigstore","artificial intelligence","model signing","machine learning", "operator"]
draft = false
author = "Nina Bongartz, Red Hat Trusted Artificat Signer Team"
type = "post"
+++

---

As machine learning becomes deeply embedded in critical infrastructure, the question of trust in deployed models is increasingly critical. How can we be sure that an AI model running in a Kubernetes cluster is exactly what it claims to be?

The [OpenSSF AI/ML working group][workinggroup] believes the answer can be found in signing AI models. A long-standing practice in traditional software distribution is to leverage cryptographic signatures to help end-users verify provenance: that software is authentic, has not been tampered with, and was authored by the expected creator. Extending this concept naturally provides a path to ensure similar guarantees with AI models.

The Sigstore [model transparency][repomodeltransparency] project ([blog post][blogmodelauthenticity]),  one of the primary efforts from the  working group’s, has been laying the groundwork to tackle these types of challenges for over a year, and recently released version 1.0 ([blog post][bloglaunchv1-0]). This community-driven library and command-line interface (CLI) supports multiple signing and verification methods for AI models, ranging from traditional key pairs and certificates to modern identity-based signing and verification with [Sigstore][sigstore]. This work will help make model signing approachable with modern tooling, bringing cryptographic integrity and transparency to ML models.

But, while these tools make signing and verifying models easier, they leave it up to the user to enforce verification during the deployment phase. As a result, a need arose to mitigate such concerns.

The [model-validation-operator][repomodeloperator] is a Kubernetes native solution for automatically verifying AI models signed by the model-transparency CLI before they’re used by workloads within a cluster. Developed as an official Sigstore community project, the operator ensures that whether you’re running critical inference pipelines or deploying foundation models at scale, only authentic, trusted models are allowed to be executed.

In this post, we’ll walk through the workflow and architecture of the model-validation-operator and demonstrate how to deploy the operator into your own ML infrastructure - seamlessly enabling automatic verification of AI models.

⚠️ Note: This post showcases an early proof-of-concept (PoC) version of the model-validation-operator, which was recently donated to the Sigstore GitHub organization. While the operator follows the same versioning as the model-transparency tools (currently at v1.0.1), it is still considered alpha and remains under active, fast-paced development.

As adoption grows and the ecosystem matures, this operator is expected to become a foundational component of a trusted AI supply chain—enabling secure, policy-driven model usage directly within Kubernetes environments. Stay tuned, contribute, and help shape its future (more to come on that aspect later in this post).

---

## Why Model Validation Matters
When deploying AI models to production, how do you ensure that only trusted, untampered models are used?

The model-transparency CLI tools offer a great way to sign and verify models. However, they are tied to specific environments and Python dependencies and are not designed with containerized or Kubernetes-based workflows in mind. That’s where our operator comes in.

The model-validation-operator brings the verification process into the Kubernetes lifecycle and extends the model-transparency project into the cluster. It watches for specific labels and custom resources defined within the cluster, enabling automatic verification of signed models, before they are used.

---

## The Workflow: How It Comes Together 
Here’s how a typical workflow that uses the model-validation-operator might look:
![Model Validation Operator Workflow](/images/model-validation-operator-workflow.png)

1. Start with a trained AI model

Have a model artifact ready for deployment.

---

2. Sign the model using the model-transparency CLI

Use the [model-transparency tools][repomodeltransparency] to sign your model, either manually or automatically as part of a CI/CD pipeline, such as a GitHub Actions workflow. These tools are now packaged in a container for better portability, removing the need to worry about local Python environments or dependencies.

A model snapshot for testing can be found in the [sigstore/model-validation-operator][repomodeloperator] repository and can be used to demonstrate signing a model:

```bash
git clone git@github.com:sigstore/model-validation-operator.git
cd model-validation-operator
docker run -it --rm -v $(pwd)/testdata/tensorflow_saved_model:/tensorflow_saved_model:z -w /tensorflow_saved_model ghcr.io/sigstore/model-transparency-cli:v1.0.1 sign sigstore --signature="/tensorflow_saved_model/model.sig" /tensorflow_saved_model
```
If no valid OIDC token is provided to the CLI tool via `--identity_token "$OIDC_TOKEN"`, a browser-based authentication flow will be initiated. You will be redirected to a page similar to the following:

![Model Validation Operator Workflow](/images/model-validation-operator-oidc.png)


This flow will:

- **Verify your identity** with your chosen identity provider for example GitHub or Google

- **Obtain a short-lived identity token (ID token)** that proves who you are

- **Enable secure operations** such as signing artifacts or verifying signatures with your authenticated identity

Once authentication is complete, a verification code will be displayed in your browser. Paste this code back into the CLI prompt to proceed.

Go to the following link in your browser:

```
https://oauth2.sigstore.dev/auth/auth?response_type=code&client_id=sigstore&client_secret=&scope=openid+email&redirect_uri=urn%3Aietf%3Awg%3Aoauth%3A2.0%3Aoob&code_challenge=ajfn5BIb10C-XuQ6V7iFOPfW4TYt1focj2q2a0Ubhho&code_challenge_method=S256&state=81e6bbc6-786e-4791-8aa6-f6385d43f0b6&nonce=56e5433a-ab81-421c-8c0e-b2f4f76e6ad0
Enter verification code: 
```

This process results in the generation of a signature file (.sig) for your model.

---

3. Store the signed model

Upload both the model and its signature to a location accessible from your Kubernetes or OpenShift cluster. Any of the following options can be used:

- A Persistent Volume (PVC)
- An object storage bucket for example  S3 or MinIO, when mounted as file system
- Shared mounted volume

3.1 (Optional) Sign the Model In-Cluster

Alternatively, the CLI is also available within a container and can be deployed as a pod with access to the relevant PVC. You can then use the kubectl exec command to access the container and perform the signing steps directly within the cluster.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: model-validation-debug
spec:
  containers:
  - command: ["sleep", "infinity"]
    image: ghcr.io/sigstore/model-transparency-cli:v1.0.1
    name: debug
    resources:
      limits:
        cpu: "1"
        memory: 2Gi
      requests:
        cpu: 50m
        memory: 1Gi
    volumeMounts:
    - mountPath: /models
      name: hf-cache
```
---
4. Install Model Validation Operator and CRDs

To create a Model Validation Custom resource, the Custom Resource Definition (CRD) and the model-validation-operator must be installed first. This step can be accomplished by running the following command:

```bash
kubectl apply -f https://github.com/sigstore/model-validation-operator/releases/download/v1.0.1/manifests.yaml
```

---

5. Create a ModelValidation Custom Resource

To define how the model should be verified, a ModelValidation Custom Resource must be created next. This resource specifies:

- The model file path
- The signature path
- The verification method (for example, Sigstore)
- The OIDC identity and issuer used during the signing process

It's important to provide the **exact OIDC identity and issuer** that were used when the model was originally signed. This ensures that the verification step can correctly validate the model's provenance.

Example `ModelValidation` Custom Resource:
```yaml
apiVersion: rhtas.redhat.com/v1alpha1
kind: ModelValidation
metadata:
  name: model-validation-example
spec:
  config:
    # pkiConfig:
    #   certificateAuthority: /path/to/ca.crt
    # privateKeyConfig:
    #   keyPath: /root/pub.key
    sigstoreConfig:
      certificateIdentity: "certificateIdentity: "<GITHUB_WORKFLOW_URL>@refs/tags/<VERSION_TAG>"
      certificateOidcIssuer: "https://token.actions.githubusercontent.com"
  model:
    path: /data
    signaturePath: /data/model.sig
---
apiVersion: v1
kind: Pod
metadata:
  name: model-consuming-workload
  labels:
    validation.rhtas.redhat.com/ml: "true"
spec:
  containers:
 - name: app-container
  image: <your-application-image>
    ports:
    - containerPort: 80
    volumeMounts:
    - name: model-storage
      mountPath: /data
  volumes:
  - name: model-storage
    persistentVolumeClaim:
      claimName: models
```
💡 **Note**: Replace the `certificateIdentity` and `certificateOidcIssuer` values with those that were used during the signing process. For example:

- Google Signer:
	- Identity: "fake@gmail.com"
	- Issuer: "https://accounts.google.com"
- GitHub Actions Signer:
	- Identity: "fake@example.com"
	- Issuer: "https://token.actions.githubusercontent.com"



Example Consumer Pod (Workload):
``` yaml
apiVersion: rhtas.redhat.com/v1alpha1
kind: ModelValidation
metadata:
  name: demo
spec:
  config:
    sigstoreConfig:
certificateIdentity: "fake@gmail.com"
certificateOidcIssuer: "https://accounts.google.com"
…
```
This pod will consume the validated model once verification is complete.

---

6. Label your workload pod for validation

Add the following label to any pod or within the podSpec of another resource (such as a Deployment) that should have its model verified by the operator:

```yaml
metadata:
  labels:
    validation.rhtas.redhat.com/ml: "true"
```
Note: This label must be present at pod creation time. Adding it afterward won’t trigger validation until the next time the Pod is created.

---

7. Operator injects an init container

The model-validation-operator uses a mutating webhook to intercept pod creation. If the label matches, it injects an init container that:

- Locates the appropriate ModelValidation Custom Resource in the namespace
-- Currently, only one CR per namespace is supported [issue][issue].

- Mounts the model and signature paths

- Performs signature verification using the model-transparency CLI

---

8. Enforced policy before model execution

- ✅ If verification passes, the init container exits successfully and the main workload container starts.
- ❌ If verification fails, the init container errors out and the pod does not start — ensuring that only verified models are ever executed.

--- 

## What’s Next?
The model-validation-operator is still in its infancy. The current implementation focuses on verifying signed models from mounted volumes using Sigstore’s model-transparency CLI. Looking into the future, the long-term goal is evolving toward a more robust, cloud-native future for AI model security.

We’re working toward a flexible validation system that supports how models are delivered and deployed in modern clusters — including volumes, OCI registries, or remote APIs. Based on current feedback and exploration, our roadmap includes:

- OCI-Based Model Packaging and Verification
We're exploring support for packaging ML models as [OCI artifacts][ociartifacts], enabling lightweight, scalable verification using signed manifests — without downloading full model files. This could dramatically reduce startup latency and align with how containers are already verified in Kubernetes.

- Integration with Sigstore’s Policy Controller
We envision integrating with the [Sigstore Policy Controller ][repopolicycontroller] for synchronous admission-time model verification, similar to how container images are validated today. This would allow clusters to reject unsigned or tampered models before pods are admitted.

- Go-Based Verification Engine
To support deeper integration with Kubernetes admission workflows and enable synchronous validation, we’re exploring a Go-based implementation of the model-transparency CLI (vs the current Python CLI).

- Trust Policies at Scale
We're designing a policy engine for defining and enforcing model trust at scale — across teams, namespaces, or entire clusters — similar to image policy management.

- Improved Multi-Tenancy
Multi-namespace isolation and support for per-tenant model validation configurations are a priority to make this viable in multi-tenant Kubernetes environments.

We welcome feedback and contributions from the community as we iterate. Our goal is to make trusted AI deployment the default — not the exception.

Do you have questions, ideas or feedback? Feel free to reach out on the [Sigstore Slack][sigstoreslack] to engage with the community or file an issue within the [project repository][repomodeloperator]!

[workinggroup]: https://github.com/ossf/ai-ml-security
[blogmodelauthenticity]: https://next.redhat.com/2025/04/10/model-authenticity-and-transparency-with-sigstore/ 
[bloglaunchv1-0]: https://openssf.org/blog/2025/04/04/launch-of-model-signing-v1-0-openssf-ai-ml-working-group-secures-the-machine-learning-supply-chain/
[sigstore]: https://www.sigstore.dev/
[repomodeloperator]: https://github.com/sigstore/model-validation-operator
[repomodeltransparency]: https://github.com/sigstore/model-transparency/
[issue]: https://github.com/sigstore/model-validation-operator/issues/21
[ociartifacts]: https://opencontainers.org/
[repopolicycontroller]: https://github.com/sigstore/policy-controller
[sigstoreslack]: https://join.slack.com/t/sigstore/shared_invite/zt-2ub0ztl5z-PkWb_Ldwef5d6nb~oryaTA

