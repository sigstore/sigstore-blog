+++

title = "How to verify container images with Kyverno using KMS, Cosign, and Workload Identity"
date = "2022-04-25"
tags = ["sigstore","cosign","workload Identity","kms","kyverno"]
draft = false
author = "developer-guy"
type = "post"

+++

![](/images/verify.png)

Securing our software supply chains has become more critical with the rise of software supply chain attacks. Also, over the past few years, container adoption has increased too. In the light of these pieces of information, it has grown the need to sign container images to help prevent supply chain attacks. In addition, most of the containers we are using today, even if we use them in production environments, are vulnerable to supply chain attacks. In traditional CI/CD workflows, we build images and push them into the registry. An essential part of supply chain security is the integrity of the image we built, which means that we have to ensure that the image we built has not been tampered with, which means that guaranteeing the image which we pulled from a registry is the same image that we are going to deploy into production systems. One of the easiest and best ways (thanks to Sigstore) to prove that the image has not been tampered with is to sign it right after building it and verify it before allowing them to deploy into production systems. This is where Cosign and Kyverno come into play.

Kyverno is an open-source policy engine designed for Kubernetes that is managed as Kubernetes resources, with no new language required to write policies. *What is a policy engine?* It is software that allows users to define a set of policies that can be used to validate, mutate, and generate Kubernetes resources. As a CNCF Sandbox project, Kyverno is beginning to receive community support and awareness. Due to increased software supply chain attacks in recent years, Kyverno has grown in popularity. Kyverno has moved towards protecting workloads by supporting [verifying image signatures](https://kyverno.io/docs/writing-policies/verify-images/) and [in-toto attestations](https://github.com/in-toto/attestation). These workload protections are made possible through [cosign](https://github.com/sigstore/cosign) and the [SLSA](https://slsa.dev/) framework.

### Signing and verifying with Cosign

Cosign is a tool for container image signing and verifying maintained under the Project Sigstore in collaboration with the Linux Foundation. Among other features, Cosign supports KMS signing, built-in binary transparency, and timestamping service with Rekor and Kubernetes policy enforcement. In addition, Kyverno leverages Cosign to verify container image signatures, attestations, and more.

Software artifacts are generally opaque blobs that can’t easily be inspected for safety, so it’s more common to reason about *how they came to be* rather than *what is in them*. We can’t apply policy to individual lines of code, we apply policy on who built the software, how they built it, and where the code came from. This trail of breadcrumbs is typically referred to as the provenance of a piece of software.

> If you want to get more detail about provenance and attestations, please refer to the fantastic blog post from [Dan Lorenc](https://medium.com/u/d9ae980cd455?source=post_page-----1e07d2b85061--------------------------------) here.

Rather than signing an artifact directly, users create a document that captures their intent behind signing the artifact and any specific claims being made as part of this signature. Terminology varies, but the layering model defined by the [In-Toto](https://in-toto.io/) seems promising.

The[ in-toto attestation format](https://github.com/in-toto/attestation) provides a flexible scheme for metadata such as repository and build environment details, vulnerability scan reports, test results, code review reports, and other information to verify image integrity. Each attestation contains a signed statement with a ***predicateType\*** and a predicate.

It can be challenging to think about security holistically and to ensure that you are doing all that you can to make sure you are working towards greater security. The [SLSA](https://slsa.dev/) project can help with this. Standing for “Supply chain Levels for Software Artifacts,” SLSA is pronounced “salsa.” As a security framework, you can think of it as a checklist of standards and controls to prevent tampering, improve the integrity, and secure packages and infrastructure in your projects, businesses, or enterprises. It’s how you get from being safe enough to be as resilient as possible at any link in the chain. Bringing SLSA, Sigstore, and Kyverno together can provide you with a solid foundation for a secure software development life cycle.

Now that we covered the essential parts of the supply chain security features Kyverno provides, let’s dive into how it achieves all of that in a real environment.

### Kyverno and Cosign with Workload Identity

In the next section, we’ll be demonstrating on Google Cloud Platform (GCP) using services such as Google Kubernetes Engine (GKE) and Google Cloud Key Management Service (KMS). As mentioned in the Cosign section, cloud providers’ KMS systems are first-class citizens in [cosign](https://github.com/sigstore/cosign/blob/main/KMS.md), which means that Cosign works perfectly with GCP KMS.

GCP KMS is a cloud service for managing encryption keys for other Google cloud services so that enterprises can implement cryptographic functions. Cloud Key Management Service allows you to create, import, and manage cryptographic keys and perform cryptographic operations in a single centralized cloud service.

First, we need to create a Kubernetes cluster on GKE with the Workload Identity feature enabled. But before doing that, we should also understand more about Workload Identity and how Cosign can leverage this feature to make authorized calls to GCP services like GCP KMS.

GCP provides the Workload Identity feature, which allows applications running on GKE to access [Google Cloud APIs](https://cloud.google.com/apis) such as Compute Engine API, BigQuery Storage API, or Machine Learning APIs.

> [Workload Identity](https://cloud.google.com/kubernetes-engine/docs/concepts/workload-identity) allows a Kubernetes service account in your GKE cluster to act as an IAM service account. When accessing Google Cloud APIs, pods that use the configured Kubernetes service account automatically authenticate as the IAM service account. Using Workload Identity allows you to assign distinct, fine-grained identities and authorization for each application in your cluster.

Also, Workload Identity is the recommended way for your workloads running on Google Kubernetes Engine (GKE) to access Google Cloud services in a secure and manageable way. Fortunately, we don’t need to do anything extra to enable Workload Identity on GKE because Cosign can use this workload identity by providing [ambient credential detection](https://dlorenc.medium.com/a-bit-of-ambiance-comes-to-sigstore-f80d1d6b1c30) feature support use this workload identity. Again thanks to Dan Lorenc, he wrote another excellent blog post to explain the relationship between [Workload Identity and Ambient Credentials](https://dlorenc.medium.com/a-bit-of-ambiance-comes-to-sigstore-f80d1d6b1c30) terms.

In our case, Kyverno will be running on GKE, so we’ll apply a policy to verify container images. This type of rule in Kyverno is [verifyImages](https://kyverno.io/docs/writing-policies/verify-images/) which will fail if the signature is not found in the OCI registry or if the image was not signed using the specified key. It also mutates matching images to add the[ image digest](https://docs.docker.com/engine/reference/commandline/pull/#pull-an-image-by-digest-immutable-identifier) if the digest is not already specified. Using an image digest makes image references immutable.

In the above policy example, Kyverno uses the Cosign SDK internally to verify the given image against the specified key. Assuming we use GCP KMS, Kyverno must authenticate to that service to make API calls properly. Here, we use Workload Identity to achieve that.

> Instead of deploying a secret alongside your code (or god-forbid in it!), your code receives the credentials it needs from the environment. Of course, these have to come from somewhere — but the platform provider now manages the responsibility of storing, distributing, refreshing, and revoking secrets. Instead of provisioning long-lived secrets as part of the build/deployment process that needs to last as long as the binary might run for, your application can read ambient credentials on-demand, directly from the environment.

At the time of writing, Cosign supports providing ambient credential detection for four different systems, including GitHub, SPIFFE, Filesystem, and Google. You can see the support we added for GCP KMS in the following Issues:

- https://github.com/kyverno/kyverno/issues/2584.
- https://github.com/sigstore/sigstore/issues/138

For more detail, review the [providers directory](https://github.com/sigstore/cosign/tree/main/pkg/providers) in the GitHub Cosign repository.

Demo

This section will run through the demo described above of Kyverno running on GKE with a policy to verify container images. This rule — verifyImages — will fail if the signature is not found in the OCI registry or if it was not signed using the specified key. It also mutates matching images to add the image digest if the digest is not already specified. By using an image digest, our image references are immutable.

Prerequisites

- kubectl v1.20+
- gcloud v375.0.0
- cosign v1.6.0

First, we need to create a Kubernetes cluster on GKE with the Workload Identity feature enabled. We’ll be using a fixed workload identity pool in the form of **PROJECT_ID.svc.id.goog**.

When you enable Workload Identity on a cluster, GKE automatically creates a fixed *workload identity pool* for the cluster’s Google Cloud project. A workload identity pool allows IAM to understand and trust Kubernetes service account credentials. GKE uses this pool for all clusters in the project that use Workload Identity.

```shell
$ export PROJECT_ID=$(gcloud config get-value project)
$ export CLUSTER_NAME="gke-wif"
$ gcloud container clusters create $CLUSTER_NAME \
 --workload-pool=$PROJECT_ID.svc.id.goog --num-nodes=2
```

> https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity

Next, we need to create a GCP IAM Service Account to map a Kubernetes Service Account to make authorized calls to GCP services. Configuring Workload Identity includes using an IAM policy to bind the Kubernetes ServiceAccount member name to an IAM service account with the permissions your workloads need. Then, any Google Cloud API calls from workloads that use this Kubernetes ServiceAccount are authenticated as the bound IAM service account.

When you configure a Kubernetes ServiceAccount in a Namespace to use Workload Identity, IAM authenticates the credentials using the following member name:

**serviceAccount:PROJECT_ID.svc.id.goog[KUBERNETES_NAMESPACE/KUBERNETES_SERVICE_ACCOUNT]**

Let’s create it:

```shell
$ export GSA_NAME=kyverno-sa
$ gcloud iam service-accounts create $GSA_NAME
$ gcloud iam service-accounts add-iam-policy-binding \
 --role roles/iam.workloadIdentityUser \
 --member "serviceAccount:${PROJECT_ID}.svc.id.goog[kyverno/kyverno]" \
 ${GSA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com
```

We ensure that the IAM service account has the application’s roles. In addition, we can grant additional roles, such as **roles/cloudkms.viewer** and **roles/cloudkms.verifier** in this case.

> For more detail: [cloud.google.com/kms/docs/reference/permissions-and-roles](https://cloud.google.com/kms/docs/reference/permissions-and-roles)

```shell
$ gcloud projects add-iam-policy-binding ${PROJECT_ID} \
 --role roles/cloudkms.verifier \
 --member serviceAccount:${GSA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com

$ gcloud projects add-iam-policy-binding ${PROJECT_ID} \
 --role roles/cloudkms.viewer \
 --member serviceAccount:${GSA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com
```

We configured our IAM service account with the necessary roles and bound it to the Kubernetes ServiceAccount named kyverno in the kyverno namespace.

Next, we’ll be deploying Kyverno v1.6+ by using its Helm chart.

```shell
$ helm repo add kyverno https://kyverno.github.io/kyverno/
$ helm repo update
$ helm install kyverno kyverno/kyverno --namespace kyverno --create-namespace
```

Last but not least, we should annotate the Kubernetes ServiceAccount with the email address of the IAM service account.

```shell
$ kubectl annotate serviceaccount \
 --namespace kyverno \
 kyverno \
 iam.gke.io/gcp-service-account=${GSA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com
```

It is time to generate a key pair with Cosign stored in GCP KMS.

```shell
$ gcloud kms keyrings create test - location "global"
$ gcloud kms keys create "cosign" \
 - location "global" \
 - keyring "test" \
 - purpose=asymmetric-signing - default-algorithm=ec-sign-p256-sha256
$ cosign generate-key-pair - kms gcpkms://projects/$PROJECT_ID/locations/global/keyRings/test/cryptoKeys/cosign/versions/1
```

Let’s test all of these by applying a verifyImages policy.

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: check-image
spec:
  validationFailureAction: enforce
  background: false
  webhookTimeoutSeconds: 30
  failurePolicy: Fail
  rules:
    - name: check-image
      match:
        resources:
          kinds:
            - Pod
      verifyImages:
      - image: "gcr.io/shaped-shuttle-342907/alpine:*"
        key: "gcpkms://projects/shaped-shuttle-342907/locations/global/keyRings/test/cryptoKeys/cosign/versions/1"
```

> Be careful, **shaped-shuttle-342907** is the value of our $PROJECT_ID environment variable.

In order to verify the container image, we should sign it first. Let’s sign it:

```shell
$ cosign sign --key gcpkms://projects/$PROJECT_ID/locations/global/keyRings/test/cryptoKeys/cosign/versions/1 gcr.io/$PROJECT_ID/alpine:3.15.0
```

Now, let’s run our Pod with the signed container image. We should expect that Kyverno will let us create this Pod:

```shell
$ kubectl run signed --image=gcr.io/$PROJECT_ID/alpine:3.15.0
pod/signed created
```

Congratulations! You have verified container images with Kyverno using KMS, Cosign, and Workload Identity!