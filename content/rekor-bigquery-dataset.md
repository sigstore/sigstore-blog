+++
title = "Announcing the Sigstore Transparency Log Research Dataset"
date = "2025-08-15"
tags = ["sigstore","rekor","transparency-log"]
draft = false
author = "Hayden Blauzvern & Eve Martin-Jones, Google Open Source Security Team"
type = "post"
+++

We're pleased to announce the creation of a new [BigQuery public dataset](https://console.cloud.google.com/marketplace/product/bigquery-public-data/rekor), `rekor`.
The `rekor` dataset is an easily-queryable mirror of the public good instance of Sigstore's transparency log, Rekor.

As a reminder, signing events are recorded in Rekor, Sigstore's append-only transparency log.
Software consumers rely on cryptographic proofs of log inclusion to verify that software artifacts are recorded to the log.
Software producers can verify metadata in the log, verifying that the recorded signature metadata was produced as expected
when their identities or keys were used to sign artifacts, using a [Rekor monitor](https://github.com/sigstore/rekor-monitor).
While software producers should monitor the log directly, researchers had to run similar monitoring tooling to ingest all entries,
which added complexity and cost for research.

This dataset will allow open source supply chain researchers and other interested parties to gather aggregate data on
how artifacts are being signed with Sigstore, answering questions like "what is the most common CI provider used to sign artifacts?"
or "how many artifacts are signed each month?".

Sample queries can be found at the [BigQuery marketplace listing](https://console.cloud.google.com/marketplace/product/bigquery-public-data/rekor).

If you have any questions or feedback, please contact us at rekor-dataset@google.com.
