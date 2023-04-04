+++
title = "Sigstore’s cosign and policy-controller with GKE, Artifact Registry and KMS"
date = "2023-02-10"
tags = ["cosign","sigstore","security","kubernetes","kms"]
draft = false
author = "Mathieu Benoit"
type = "post"

+++

As soon as I came back from [KubeCon NA 2022](https://www.youtube.com/playlist?list=PLj6h78yzYM2O5aNpRM71NQyx3WUe1xpTn), my first ever in-person KubeCon, I felt re-energized. What a community, full of people eager to share knowledge and expertise with each others, so inspiring. I mostly attended sessions about security best practices for containers and Kubernetes (that’s what excites me these days!). Secure Software Supply Chain (S3C) was almost mentioned everywhere, for good reasons.

Sigstore as a new standard for signing, verifying and protecting software, got its first own [SigstoreCon](https://www.youtube.com/playlist?list=PLj6h78yzYM2MUNId2hvHBnrGCCbmou_gl) as co-located event and [hit the General Availability (GA) milestone](https://openssf.org/press-release/2022/10/25/sigstore-announces-general-availability-at-sigstorecon/). It piqued my curiosity. I wanted to give [Cosign in Kubernetes clusters](https://docs.sigstore.dev/cosign/overview/#kubernetes-integrations) a try.

I did, and to be honest, within a few hours of research and tests I was able to sign my container images with Cloud KMS and Google Artifact Registry and then only allow my own signed images to run in my GKE cluster.

I thought I would share my learnings and show you how easy it is. Hope you’ll like it!

This blog article will walk you through two main concepts:
- [Sign a container image with Cosign, Google Artifact Registry and Cloud KMS](#sign-a-container-image-with-sigstores-cosign-google-artifact-registry-and-cloud-kms)
- [Enforce that only signed container images are allowed in a GKE cluster with Policy-controller](#enforce-that-only-signed-container-images-are-allowed-in-a-gke-cluster-with-sigstores-policy-controller)

![](/images/cosign-with-gke.png)

_Note: while learning and testing, it was also the opportunity for me to open my first issues and PRs in the `sigstore/docs` ([#63](https://github.com/sigstore/docs/pull/63)), `sigstore/policy-controller` ([#520](https://github.com/sigstore/policy-controller/pull/520)), and `sigstore/community` ([#220](https://github.com/sigstore/community/issues/220)) repos to fix some frictions I faced._

Define the common bash variables used throughout this blog article:
```bash
PROJECT_ID=FIXME-WITH-YOUR-EXISTING-PROJECT-ID
gcloud config set project ${PROJECT_ID}
REGION=northamerica-northeast1
```

## Sign a container image with Cosign, Google Artifact Registry and Cloud KMS

In this section you will:
- Create a key in Cloud KMS
- Create a Google Artifact Registry repository to store container images
- Push a simple `nginx` container image in this repository
- Install [Cosign](https://docs.sigstore.dev/cosign/overview/) locally
- Sign this remote private container image

Enable the Cloud KMS API in our current project:
```bash
gcloud services enable cloudkms.googleapis.com
```

Create a key in Cloud KMS:
```bash
KEY_RING=cosign
gcloud kms keyrings create ${KEY_RING} \
    --location ${REGION}
KEY_NAME=cosign
gcloud kms keys create ${KEY_NAME} \
    --keyring ${KEY_RING} \
    --location ${REGION} \
    --purpose asymmetric-signing \
    --default-algorithm ec-sign-p256-sha256
```

Enable the Artifact Registry API in our current project:
```bash
gcloud services enable artifactregistry.googleapis.com
```

Create a private Google Artifact Registry repository to store our container images:
```bash
REGISTRY_NAME=containers
gcloud artifacts repositories create ${REGISTRY_NAME} \
    --repository-format docker \
    --location ${REGION}
```

Push an `nginx` image in our own private Google Artifact Registry repository:
```bash
docker pull nginx
docker tag nginx ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REGISTRY_NAME}/nginx
gcloud auth configure-docker ${REGION}-docker.pkg.dev
SHA=$(docker push ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REGISTRY_NAME}/nginx | grep digest: | cut -f3 -d" ")
```
_Note: we are grabbing the `SHA` of this remote container image in order to sign this container image later._

Install [Cosign](https://docs.sigstore.dev/cosign/installation/) locally:
```bash
COSIGN_VERSION=$(curl -s https://api.github.com/repos/sigstore/cosign/releases/latest | jq -r .tag_name)
curl -LO https://github.com/sigstore/cosign/releases/download/${COSIGN_VERSION}/cosign-linux-amd64
sudo mv cosign-linux-amd64 /usr/local/bin/cosign
chmod +x /usr/local/bin/cosign
```

[Generage a key](https://docs.sigstore.dev/cosign/key-generation/#key-generation-and-management) and [sign](https://docs.sigstore.dev/cosign/sign/) this remote container image with Cloud KMS:
```bash
gcloud auth application-default login
cosign generate-key-pair \
    --kms gcpkms://projects/${PROJECT_ID}/locations/${REGION}/keyRings/${KEY_RING}/cryptoKeys/${KEY_NAME}
cosign sign \
    --key gcpkms://projects/${PROJECT_ID}/locations/${REGION}/keyRings/${KEY_RING}/cryptoKeys/${KEY_NAME} \
    ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REGISTRY_NAME}/nginx@${SHA}
```

We could now see that our Google Artifact Registry repository has two entries, one for the actual container image and the other for the associate `.sig` signature:
```bash
gcloud artifacts docker tags list ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REGISTRY_NAME}/nginx
```
Output similar to:
```plaintext
Listing items under project ${PROJECT_ID}, location ${REGION}, repository ${REGISTRY_NAME}.
TAG                                                                          IMAGE                                                             DIGEST
latest                                                                       ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REGISTRY_NAME}/nginx  sha256:4c1c50d0ffc614f90b93b07d778028dc765548e823f676fb027f61d281ac380d
sha256-4c1c50d0ffc614f90b93b07d778028dc765548e823f676fb027f61d281ac380d.sig  ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REGISTRY_NAME}/nginx  sha256:f02d7fef0df5c264e34b995a4861590bbdd7001631f6e5f23250f34202359a56
```
_Note: there is an [ongoing discussion](https://github.com/sigstore/cosign/issues/1397) to support the [reference types from the OCI spec](https://oras.land/cli/6_reference_types/) in order to just have the container image where the signature could be attached on._

Verify this signed container image:
```bash
cosign verify \
    --key gcpkms://projects/${PROJECT_ID}/locations/${REGION}/keyRings/${KEY_RING}/cryptoKeys/${KEY_NAME} \
    ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REGISTRY_NAME}/nginx@${SHA}
```
_Note: we are verifying with the container image digest `SHA`, it’s also working with the container image tag associated to this digest._

Output similar to:
```plaintext
Verification for ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REGISTRY_NAME}/nginx@sha256:4c1c50d0ffc614f90b93b07d778028dc765548e823f676fb027f61d281ac380d --
The following checks were performed on each of these signatures:
  - The cosign claims were validated
  - The signatures were verified against the specified public key

[{"critical":{"identity":{"docker-reference":"${REGION}-docker.pkg.dev/${PROJECT_ID}/${REGISTRY_NAME}/nginx"},"image":{"docker-manifest-digest":"sha256:4c1c50d0ffc614f90b93b07d778028dc765548e823f676fb027f61d281ac380d"},"type":"cosign container image signature"},"optional":null}]
```

## Enforce that only signed container images are allowed in a GKE cluster with Policy-controller

In this section you will:
- Create a GKE cluster with Workload Identity
- Create a dedicated least privilege Google Service Account for the Policy-controller’s `ServiceAccounts`
- Install [Policy-controller](https://docs.sigstore.dev/policy-controller/overview/) in this GKE cluster
- Deploy a policy to only allow signed container images
- Test this policy with both signed and unsigned container images

Enable the GKE API in our current project:
```bash
gcloud services enable container.googleapis.com
```

Create a GKE cluster with Workload Identity:
```bash
gcloud container clusters create ${CLUSTER_NAME} \
    --workload-pool=${PROJECT_ID}.svc.id.goog \
    --zone ${ZONE} \
    --scopes "gke-default,https://www.googleapis.com/auth/cloudkms"
```
_Note: we explicitly add the `https://www.googleapis.com/auth/cloudkms` scope needed by Policy-controller. [`https://www.googleapis.com/auth/cloud-platform`](https://cloud.google.com/kubernetes-engine/docs/how-to/access-scopes) instead is fine too._

Define a least privilege Google Service Account (GSA) for Policy-controller by granting the `cloudkms.viewer`, `cloudkms.verifier` and `artifactregistry.reader` roles and by enabling Workload Identity between the Kubernetes Service Account and the Google Service Account:
```bash
PC_GSA_NAME=policy-controller-sa
PC_GSA_ID=${PC_GSA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com

gcloud iam service-accounts create ${PC_GSA_NAME}

gcloud iam service-accounts add-iam-policy-binding ${PC_GSA_ID} \
    --role roles/iam.workloadIdentityUser \
    --member "serviceAccount:${PROJECT_ID}.svc.id.goog[cosign-system/policy-controller-policy-webhook]"
gcloud iam service-accounts add-iam-policy-binding ${PC_GSA_ID} \
    --role roles/iam.workloadIdentityUser \
    --member "serviceAccount:${PROJECT_ID}.svc.id.goog[cosign-system/policy-controller-webhook]"

gcloud projects add-iam-policy-binding ${PROJECT_ID} \
    --role roles/cloudkms.verifier \
    --member serviceAccount:${PC_GSA_ID}
gcloud projects add-iam-policy-binding ${PROJECT_ID} \
    --role roles/cloudkms.viewer \
    --member serviceAccount:${PC_GSA_ID}

gcloud artifacts repositories add-iam-policy-binding ${REGISTRY_NAME} \
    --location ${REGION} \
    --member "serviceAccount:${PC_GSA_ID}" \
    --role roles/artifactregistry.reader
```

Install the [Policy-controller Helm chart](https://github.com/sigstore/helm-charts/tree/main/charts/policy-controller) in this GKE cluster:
```bash
helm repo add sigstore https://sigstore.github.io/helm-charts
helm repo update
helm upgrade policy-controller \
    sigstore/policy-controller \   
    --install \
    -n cosign-system \
    --create-namespace \
    --set "policywebhook.serviceAccount.annotations.iam\.gke\.io/gcp-service-account=${PC_GSA_ID}" \
    --set "webhook.serviceAccount.annotations.iam\.gke\.io/gcp-service-account=${PC_GSA_ID}"
```

Deploy a policy only allowing signed container images with our Cloud KMS key:
```bash
cat << EOF | kubectl apply -f -
apiVersion: policy.sigstore.dev/v1alpha1
kind: ClusterImagePolicy
metadata:
  name: private-signed-images-cip
spec:
  images:
  - glob: "**"
  authorities:
  - key:
      kms: gcpkms://projects/${PROJECT_ID}/locations/${REGION}/keyRings/${KEY_RING}/cryptoKeys/${KEY_NAME}/cryptoKeyVersions/1
EOF
```

Enfore this policy for the `test` namespace:
```bash
kubectl create namespace test
kubectl label namespace test policy.sigstore.dev/include=true
```
_Note: you need to apply this label on the namespaces you want this policy to be enforced in._

Test with an unsigned container image and see that it’s blocked:
```bash
kubectl create deployment nginx \
    --image=nginx \
    -n test
```
Output similar to:
```plaintext
error: failed to create deployment: admission webhook "policy.sigstore.dev" denied the request: validation failed: failed policy: private-signed-images-cip: spec.template.spec.containers[0].image
index.docker.io/library/nginx@sha256:b8f2383a95879e1ae064940d9a200f67a6c79e710ed82ac42263397367e7cc4e signature key validation failed for authority authority-0 for index.docker.io/library/nginx@sha256:b8f2383a95879e1ae064940d9a200f67a6c79e710ed82ac42263397367e7cc4e: no matching signatures:
```

Test with our signed container image and see that it’s allowed:
```bash
kubectl create deployment nginx \
    --image=${REGION}-docker.pkg.dev/${PROJECT_ID}/${REGISTRY_NAME}/nginx@${SHA} \
    -n test
```
_Note: we are deploying with the container image digest `SHA`, it’s also working with the container image tag associated to this digest._

Output similar to:
```plaintext
deployment.apps/nginx created
```

That’s it, congrats! We just enforced our GKE cluster to only allow our private and signed container images on specific namespaces! Wow!

## Resources

- [Using Transparent Digital Signatures to Help Secure the Software SupplyChain- Bob Callaway](https://youtu.be/_HL_I5k_oP4)
- [How We Learned to Stop Trusting Registries and Love Signatures — Wojciech Kocjan & Tyson Kamp, InfluxData](https://youtu.be/mduvP92bhPs?list=PLj6h78yzYM2MUNId2hvHBnrGCCbmou_gl)
- [How to verify container images with Kyverno using KMS, Cosign, and Workload Identity](https://blog.sigstore.dev/how-to-verify-container-images-with-kyverno-using-kms-cosign-and-workload-identity-1e07d2b85061)

Happy signing, happy sailing!

_Originally posted on [Medium](https://medium.com/google-cloud/sigstores-cosign-and-policy-controller-with-gke-and-kms-7bd5b12672ea)._
