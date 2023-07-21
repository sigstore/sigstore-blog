+++
title = "Sigstore Announcement: No Longer Publishing Cosign Releases to GCS Bucket"
date = "2023-07-31"
tags = ["sigstore","cosign"]
draft = false
author = "sigstore TSC"
type = "post"
+++

We are announcing that we will stop publishing [Cosign](https://github.com/sigstore/cosign) releases to the GCS bucket named `cosign-releases`. The current v2.1.1 release of Cosign is the last release that will be pushed to the bucket, and public access to the GCS bucket will be removed on October 31st, 2023.

# Why are we deprecating the GCS bucket?

We are deprecating the GCS bucket because we already use GitHub in the Sigstore community, and it is a reliable and secure platform for hosting release artifacts. GitHub has a proven track record of uptime and security. Deprecating this bucket also simplifies our release processes which lowers costs and administrative toil on our community members.

# What do I need to do to prepare for the deprecation?

If you currently download Cosign releases from the GCS bucket, you will need to update your installation instructions to download releases from GitHub. You can find the latest Cosign releases on the Cosign GitHub releases page: https://github.com/sigstore/cosign/releases.

In 90 days, on October 31, 2023, we will revoke public access to the `cosign-releases` GCS bucket.

You should inspect any scripts or instructions where you may be downloading releases via the following URLs:
- `gs://cosign-releases/`
- https://storage.googleapis.com/cosign-releases/{version}/{artifact}
- https://cosign-releases.storage.googleapis.com/{version}/{artifact}

Instead, you can download Cosign releases from Cosign's GitHub repository. Please use 
- https://github.com/sigstore/cosign/releases/download/{version}/{artifact}

(For example: https://github.com/sigstore/cosign/releases/download/v2.1.1/cosign-linux-amd64).

# Update your GitHub Actions

If you are using `cosign-installer`, our GitHub Action that installs Cosign, you may need to update the action __only if you are pinned by hash to a version earlier than v3.1.0__. The following example shows how to update the action:

```
uses: sigstore/cosign-installer@6e04d228eb30da1757ee4e1dd75a0ec73a653e06 # v3.1.1
```
or
```
uses: sigstore/cosign-installer@v3.1.1
```

We apologize for any inconvenience this may cause. Thank you for your continued support of sigstore!
