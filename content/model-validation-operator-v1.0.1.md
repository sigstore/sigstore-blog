+++
title = "Trusting AI Models in Kubernetes: Introducing the Sigstore Model Validation Operator"
date = "2025-02-06"
tags = ["sigstore","artificial intelligence","model signing","machine learning", "operator"]
draft = false
author = "Nina Bongartz, Red Hat Trusted Artificat Signer Team"
type = "post"
+++

---

# Trusting AI Models in Kubernetes: Introducing the Sigstore Model Validation Operator
As machine learning becomes deeply embedded in critical infrastructure, the question of trust in deployed models is increasingly critical. How can we be sure that an AI model running in your Kubernetes cluster is exactly what it claims to be?

The OpenSSF AI/ML working group believes the answer can be found in signing AI models. A long-standing practice in traditional software distribution, signatures help end-users verify provenance: that software is authentic, has not been tampered with, and was authored by the expected developer. Extending this concept naturally provides a path to similar guarantees with AI models.

The Sigstore [model transparency][modeltransperencyrepo] project ([blog post][blogmodelauthenticity]), one of the working group’s primary efforts, has been laying the groundwork to tackle these problems for over a year, and recently released version 1.0 ([blog post][bloglaunchv1-0]). This community-driven library and command-line interface (CLI) supports multiple signing and verification methods for AI models, ranging from traditional key pairs and certificates to modern identity-based signing and verification with Sigstore. This work will help make model signing approachable with modern tooling, bringing cryptographic integrity and transparency to ML models.
But while these tools make signing and verifying models easier, they leave it up to the user to enforce these checks during deployment.

The [model-validation-operator][repomodeloperator] runs in Kubernetes and automatically verifies AI models signed by the model-transparency CLI before they’re used by workloads in the cluster. Developed as an official Sigstore community project, it ensures that whether you’re running critical inference pipelines or deploying foundation models at scale, only authentic, trusted models are allowed to execute.

In this post, we’ll walk through the workflow and architecture of the model-validation-operator, then demonstrate how to deploy it into your own ML infrastructure - seamlessly enabling automatic verification of AI models.

    ⚠️ Note: What we’re showcasing here is an early proof-of-concept (PoC) version of the model-validation-operator, which was recently donated to the Sigstore GitHub organization. While the project follows the same versioning as the model-transparency tools (currently at v1.0.1), it is still considered alpha and remains under active, fast-paced development.

As adoption grows and the ecosystem matures, this operator is expected to become a foundational piece of the trusted AI supply chain—enabling secure, policy-driven model usage directly within Kubernetes environments. Stay tuned, contribute, and help shape its future.

---

## Why Model Validation Matters
When deploying AI models to production, how do you ensure that only trusted, untampered models are used?

The model-transparency CLI tools offer a great way for signing and verifying models, but they’re tied to specific environments and Python dependencies — and weren’t designed with containerized or Kubernetes-based workflows in mind. That’s where our operator comes in.

The model-validation-operator brings the verification process into the Kubernetes lifecycle and extends the model-transparency project into the cluster. It watches for specific labels and custom resources, enabling automatic verification of signed models before they run in your cluster.

---

## The Workflow: How It Comes Together 
Here’s how a typical workflow that uses the model-validation-operator might look:
![Model Validation Operator Workflow](/images/modmodel-validaiton-operator-workflow.png)

1. Start with a trained AI model

Have a model artifact ready for deployment.

---

2. Sign the model using the model-transparency CLI

Use the [model-transparency tools][repomodeltransparency] to sign your model, either manually or automatically as part of a CI/CD pipeline, such as a GitHub Actions workflow. These tools are now packaged in a container for better portability, removing the need to worry about local Python environments or dependencies.

A model snapshot for testing can be found in the [sigstore/model-validation-operator][repomodeloperator] repository and sign a model:

```bash
git clone git@github.com:sigstore/model-validation-operator.git
cd model-validation-operator
docker run -it --rm -v $(pwd)/testdata/tensorflow_saved_model:/tensorflow_saved_model:z -w /tensorflow_saved_model ghcr.io/sigstore/model-transparency-cli:v1.0.1 sign sigstore --signature="/tensorflow_saved_model/model.sig" /tensorflow_saved_model
```
If no valid OIDC token is provided to the CLI tool via `--identity_token "$OIDC_TOKEN"`, it will automatically launch a browser-based authentication flow. You’ll be redirected to a page like the one shown below:

![Model Validation Operator Workflow](/images/modmodel-validaiton-operator-oidc.png)


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

This generates a signature file (.sig) for your model.

---

3. Store the signed model

Upload both the model and its signature to a location accessible from your Kubernetes or OpenShift cluster. This could be:
- A Persistent Volume (PVC)
- An object storage bucket for example  S3 or MinIO, when mounted as file system
- Shared mounted volume

3.1 (Optional) Sign the Model In-Cluster

Alternatively, the CLI container can be deployed as a pod with access to the relevant PVC. You can then use `kubectl exec` to access the container and perform the signing steps directly within the cluster.

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

In order to create a Model Validation Custom resource, the controller and the corresponding resource must first be installed. This can be done as follows:

```bash
kubectl apply -f https://github.com/sigstore/model-validation-operator/releases/download/v1.0.1/manifests.yaml
```

---

5. Create a ModelValidation Custom Resource

To define how the model should be verified, you need to create a ModelValidation Custom Resource. This resource specifies:
- The model file path
- The signature path
- The verification method for example  Sigstore
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

💡 **Note**: Replace the `certificateIdentity` and `certificateOidcIssuer` values with those that were used during signing. For example:

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

Add the following label to any pod that should have its model verified:
```yaml
metadata:
  labels:
    validation.rhtas.redhat.com/ml: "true"
```
Note: This label must be present at pod creation time. Adding it later won’t trigger validation.

---

7. Operator injects an init container

The model-validation-operator uses a webhook to intercept pod creation. If the label matches, it injects an init container that:

- Locates the appropriate ModelValidation Custom Resource in the namespace

- Mounts the model and signature paths

- Performs signature verification using the model-transparency CLI

---

8. Enforced policy before model execution

- ✅ If verification passes, the init container exits successfully and the main workload container starts.
- ❌ If verification fails, the init container errors out and the pod does not start — ensuring that only verified models are ever executed.

--- 

## What’s Next?
The model-validation-operator is just getting started. The current implementation focuses on verifying signed models from mounted volumes using Sigstore’s model-transparency CLI — but the long-term goal is evolving toward a more robust, cloud-native future for AI model security.

We’re working toward a flexible validation system that supports how models are delivered and deployed in modern clusters — including volumes, OCI registries, or remote APIs. Based on current feedback and exploration, our roadmap includes:
OCI-Based Model Packaging and Verification
We're exploring support for packaging ML models as OCI artifacts, enabling lightweight, scalable verification using signed manifests — without downloading full model files. This could dramatically reduce startup latency and align with how containers are already verified in Kubernetes.


- Integration with Sigstore’s Policy Controller
We envision integrating with the Sigstore Policy Controller for synchronous admission-time model verification, similar to how container images are validated today. This would allow clusters to reject unsigned or tampered models before pods are admitted.


- Go-Based Verification Engine
To support deeper integration with Kubernetes admission workflows and enable synchronous validation, we’re exploring a Go-based implementation of the model-transparency CLI (vs the current Python CLI).


- Trust Policies at Scale
We're designing a policy engine for defining and enforcing model trust at scale — across teams, namespaces, or entire clusters — similar to image policy management.


- Improved Multi-Tenancy
Multi-namespace isolation and support for per-tenant model validation configurations are a priority to make this viable in multi-tenant Kubernetes environments.

We welcome feedback and contributions from the community as we iterate. Our goal is to make trusted AI deployment the default — not the exception.

Do you have questions, ideas or feedback? Feel free to reach out on the Sigstore Slack or file an issue!


[modeltransparencyrepo]: https://github.com/sigstore/model-transparency
[blogmodelauthenticity]: https://next.redhat.com/2025/04/10/model-authenticity-and-transparency-with-sigstore/ 
[bloglaunchv1-0]: https://openssf.org/blog/2025/04/04/launch-of-model-signing-v1-0-openssf-ai-ml-working-group-secures-the-machine-learning-supply-chain/
[repomodeloperator]: https://github.com/sigstore/model-validation-operator
[repomodeltransparency]: https://github.com/sigstore/model-transparency/
