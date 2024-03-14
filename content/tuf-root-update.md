+++
title = "Sigstore Announcement: New TUF Trust Root and Client Compatibility"
date = "2024-03-14"
tags = ["sigstore","cosign","tuf"]
draft = false
author = "Sigstore TSC"
type = "post"
+++

## New TUF Trust Root

We are planning to publish a new TUF trust root for Sigstore. This update does not contain any functional changes,
but it does update to the latest version of the TUF specification.
This means that older clients may not be able to load it properly. The current compatibility is as follows:

* Cosign
   - **v2.2.0+** (Released Aug 31st 2023) works, older Cosign v2 clients will not work
   - **v1.x** will not work, though we are backporting support with an upcoming v1.13.3 release. We strongly encourage updating to Cosign v2 for the latest bug and security fixes
* **sigstore-js**: no known issues
* **sigstore-python**: no known issues
* **sigstore-java**: no known issues
* **sigstore-rust**: the TUF client it uses does not support the latest TUF spec. See [this issue](https://github.com/awslabs/tough/issues/754) for more information. We are actively working on fixing this.

The updated TUF trust root will be deployed within the next week.

## Do I need to do anything?

If you're using one of the compatible clients, the update will happen seamlessly when you sign or verify, as new TUF metadata is automatically fetched and verified.

If you're using Cosign v1.x, please update to Cosign v2 or download the upcoming v1.13.3 release. If you're using the Rust client, we'll have a fix out shortly.

## How to reach out?

If you have any concerns, please let us know. You can reach out on Slack on #sigstore-keyholders.
