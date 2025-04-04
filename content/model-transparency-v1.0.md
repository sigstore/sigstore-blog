+++
title = "Taming the Wild West of ML: Practical Model Signing with Sigstore"
date = "2025-03-31"
tags = ["sigstore","artificial intelligence","model signing","machine learning"]
draft = false
author = "Mihai Maruseac, Google Open Source Security Team"
type = "post"
+++

Over the past year, in collaboration with [OpenSSF][openssf], [NVIDIA][nvidia]
and [HiddenLayer][hiddenlayer], we have worked on bringing Sigstore signatures
to the world of machine learning, making ML models tamper resistant via
transparent signatures.  Today we are pleased to announce the launch of version
1.0 of the [`model-signing`][model-signing-pypi] project, built on top of
[`sigstore-python`][sigstore-python-github]. After installing via
`pip install model-signing`, users can use the CLI to sign and verify models,
as per the following examples:

```bash
# Sign model at S{MODEL_PATH} using Sigstore
[...]$ model_signing sign ${MODEL_PATH}

# Verify model at  ${MODEL_PATH} against signature ${SIGNATURE} with
# claimed identity ${IDENTITY} and OIDC provider ${OIDC_PROVIDER}
[...]$ model_signing verify ${MODEL_PATH} --signature ${SIGNATURE} \
       --identity ${IDENTITY} --identity_provider ${OIDC_PROVIDER}
```

The same commands can be run via `python -m model_signing` instead of just
`model_signing`. Or, developers can run `hatch run python -m model_signing`
after cloning the repository, without having to install the wheel.

In fact, the package is also a library which can be incorporated directly into
various workflows and projects. An example of using the API is presented in this
[demo notebook][demo-notebook]. We created this API such that the library can be
directly integrated with model hubs (such as [Kaggle][kaggle] and
[HuggingFace][huggingface]) and ML frameworks (such as [TensorFlow][tensorflow]
and [PyTorch][pytorch]).  Integrating with the model hubs allows verifying the
model signatures as soon as the model upload occurs, or allows signing models
during the upload time, if an unsigned model is being uploaded.  Integrating
with the ML frameworks allows deploying artifact integrity to ML in a
transparent way, since an ML developer would write `model.save()` in the code
for the training pipeline and the signing of the model would happen
automatically behind the scenes.

With this launch, we are recommending that every ML pipeline would sign the
models using Sigstore, as the default approach to ensuring model integrity –
even though the library also supports signing using public key infrastructure or
self-signed certificates. The proposed integration looks like the following
diagram:

![Signing and verification flow for ML using Sigstore](/images/sigstore-model-signing-diagram.png)

During training, the training workflow will connect to the Sigstore Certificate
Authority to obtain a short-lived certificate bound to the OpenID Connect token
that refers to the workload or developer identity. This certificate is valid
only for a short period of time, sufficient to sign one model. The signature of
the model and the certificate are stored in a Sigstore transparency log to help
with monitoring signing events such that a rogue insider cannot release new
models as if they are signed by the company. When the model is uploaded to
storage, the workflow would also upload the certificate and the proof of
inclusion in the transparency log. With these, the model hub and users can
verify the integrity of the model and the binding to the workload identity that
produced the model, at any time, even after the short-lived certificate has
expired. If the verification passes, model hubs can display that information on
the model page. Users can download the signed model and can decide between
running the integrity verification themselves or delegating to trusting the
model hub, according to their own trust model.

We can view model signing as laying out the foundation of trust in the ML
ecosystem. We envision extending this approach to also include datasets and
other ML-related artifacts. Trusted provenance for ML workflows is a future step
that can be taken. Finally, we plan to build on top of signatures, towards fully
tamper-proof metadata records, that can be read by both humans and machines, as
illustrated in [this RFC][cosai-rfc] from [Coalition for Secure AI][cosai]. We
are aiming to automate a significant fraction of the work needed to perform
incident response in case of a compromise in the ML world. If you are interested
in this work, join us in the [OpenSSF project meeting][openssf-notes].

[cosai-rfc]: https://github.com/cosai-oasis/ws1-supply-chain/issues/4
[cosai]: https://www.coalitionforsecureai.org/
[demo-notebook]: https://colab.sandbox.google.com/drive/18IB_uipduXYq0ohMxJv2xHfeihLIcGMT
[hiddenlayer]: https://hiddenlayer.com/
[huggingface]: https://huggingface.co/
[kaggle]: https://kaggle.com/
[model-signing-pypi]: https://pypi.org/project/model-signing/
[nvidia]: https://www.nvidia.com/en-us/
[openssf-notes]: https://docs.google.com/document/d/1L3ZEIN3mHNORB9xBw0yuVUe6T_yrlumnXK5_iIw3wkk/edit?tab=t.0
[openssf]: http://openssf.org/
[pytorch]: https://pytorch.org/
[sigstore-python-github]: https://github.com/sigstore/sigstore-python
[tensorflow]: https://www.tensorflow.org/
