+++
title = "New repository: model-transparency"
date = "2024-03-20"
tags = ["sigstore","artificial intelligence"]
draft = false
author = "mihaimaruseac"
type = "post"
+++

In order to strengthen the supply chain of ML, we have adopted a new
repository,
[model-transparency](https://github.com/sigstore/model-transparency). Incubated
at Google, the project aims to offer solutions for signing ML models using
Sigstore.

Our plan is for the repository to evolve into a library that can be used by
model frameworks to sign models and checkpoints during training runs. We also
aim for the library to be used by model hubs to sign models on upload (if model
has not been signed previously). Furthermore, the library will also allow
verification of model signatures. This can be performed by end users or by
model hubs, in order to display a "verified" badge on the model card.

We think that having the repository under the Sigstore umbrella will help with
engagement with OSS communities and organisations that produce, host, or
consume ML models.
