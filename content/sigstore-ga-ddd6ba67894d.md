+++
title = "Sigstore Announces General Availability for Rekor and Fulcio"
date = "2022-10-25"
tags = ["sigstore","python","sigstorecon","ga","Rekor"]
draft = false
author = "Priya Wadhwa"
type = "post"

+++

![](/images/ga.png)

Sigstore is excited to announce general availability (GA) for the Rekor transparency log and Fulcio certificate authority public benefit services! The community has been working hard all year to accomplish this milestone, and we are thrilled that open source communities can now confidently rely on Sigstore for production grade stable services for artifact signing and verification.

While the Sigstore community has maintained a public instance since early 2021, the services were operated on a best-effort basis and maintainers periodically had to make breaking changes or reset data. End users needed to accept the warnings around data persistence, potential production outages, or breaking changes. This experimentation period ultimately helped the community harden the services and shape the APIs.

To enable wider adoption, the Sigstore community came together and worked to stabilize the infrastructure and APIs for Rekor and Fulcio all year. The team has set up a staging environment to allow testing new features before rolling them out to production, [codified](https://github.com/sigstore/scaffolding/tree/main/terraform/gcp/modules) our infrastructure with Terraform and set up ArgoCD for CI/CD.

This work has culminated in v1.0.0 releases for both Rekor and Fulcio, meaning the APIs are stable and will be supported long term. We’ve also set up a [status page](https://status.sigstore.dev/) where users can check on the availability of these services. The community will continue to operate the service with a 99.5% uptime SLO and round-the-clock pager support. This was possible thanks to the dedicated, multi-vendor Sigstore open source community, who fixed major bugs and added key features in both services over the past few months. A [third party security audit](https://openssf.org/blog/2022/07/18/results-of-sigstore-and-slf4j-security-audits/) was also conducted to catch any potential vulnerabilities and all findings have been addressed.

We hope that with GA the open source community will feel more confident in relying on the public benefit services. To try out these new features, download the latest version of one of our client tools like [Cosign](https://github.com/sigstore/cosign), [sigstore-python](https://github.com/sigstore/sigstore-python), or [sigstore-java](https://github.com/sigstore/sigstore-java) to sign your software without any keys. To learn more about using the Rekor or Fulcio APIs directly, see the GitHub repositories [here](https://github.com/sigstore/rekor) and [here](https://github.com/sigstore/fulcio). If you want to learn more about Sigstore, check out our [website](https://www.sigstore.dev/).

Looking forward, the Sigstore community will continue developing new features and improving reliability. To improve the trustworthiness of the transparency log, Rekor will support user-provided signed timestamps, and timestamp authorities will be operated by community members. Additionally, the Sigstore community will create a network of log monitors to maintain log integrity and immutability. Multiple package repositories are planning to integrate Sigstore into their workflows, such as npm and Ruby, and the Sigstore community will work closely with package repositories to make those integrations successful.