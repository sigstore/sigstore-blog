+++
title = "Trusted Time in Sigstore"
date = "2023-08-02"
tags = ["sigstore","timestamping","rekor"]
draft = false
author = "Hayden Blauzvern (Google), Meredith Lancaster (GitHub), Héctor Fernández (Chainguard)"
type = "post"

+++

![](/images/trustedtime.png)

# Time in Sigstore

Time is a critical component of Sigstore. It's used to verify that a short-lived certificate issued by Fulcio was valid at a previous point, when the artifact was signed.

As a reminder, the default signing flow for Sigstore clients includes the following:

* Signer requests an identity token from an OpenID Connect provider
* Signer generates an ephemeral keypair
* Signer sends the public key and identity token to Fulcio, Sigstore's certificate authority
* Fulcio issues a short-lived (10 minute expiration) code-signing certificate
* Signer signs the artifact, and uploads the artifact, the certificate, and signature to Rekor, Sigstore's transparency log

During artifact verification, a client must verify the certificate. Typically, certificate verification would require that the certificate not be expired. In this model for code signing, the certificate would need to be longer-lived, on the order of months or years, and the signer would periodically re-sign the artifact. This becomes a burden on both the signer and verifier, who has to periodically re-fetch the code signing certificate.

Instead, Sigstore relies on time provided by another service. When verifying the short-lived code signing certificate, Sigstore verifies that the provided timestamp falls within the certificate's validity period. Previously, Sigstore clients could only rely on Rekor to provide the timestamp, using the entry's inclusion time. However, this timestamp comes from Rekor's internal clock, which is not externally verifiable, and a timestamp is not a part of the node that goes into the append-only data structure that backs Rekor, meaning the timestamp is mutable without detection.

# Signed Timestamps

To mitigate this, Sigstore now supports [signed timestamps](https://en.wikipedia.org/wiki/Trusted_timestamping). Trusted Timestamp Authorities (TSAs) issue signed timestamps following the [RFC 3161](https://www.ietf.org/rfc/rfc3161.txt) specification. Since the timestamps are signed, the time becomes immutable and verifiable. During verification, verifiers will use the TSA's provided certificate chain to verify signed timestamps.

Leveraging signed timestamps from TSAs also distributes trust. Anyone can operate a TSA. If you represent an ecosystem that would like to integrate with Sigstore and leverage the public good instance but would like to have control over a part of the trust root, you can operate a TSA whose signed timestamps will be used during verification. Learn more [below](#more-information) on how to run a timestamp authority.

You also have options for public TSAs, such as:

* [FreeTSA](https://freetsa.org/index_en.php)
* [Digicert](https://knowledge.digicert.com/generalinformation/INFO4231.html)

Signed timestamps are associated with some value to bind the timestamp to the signing event. We recommend signing over a signature, a process called "countersigning", ensuring that the signature, not the artifact, was created at a certain time.

# Using a Timestamp Authority

Timestamp support has been added to [Cosign](https://github.com/sigstore/cosign). To use a TSA to fetch a signed timestamp during signing, pick a timestamp authority, and run:

```
export TSA_URL=https://freetsa.org/tsr
cosign sign --timestamp-server-url $TSA_URL <artifact>
```

To verify, retrieve the TSA's certificate chain, which must contain the root CA certificate, any number of intermediate CA certificates, and the issuing leaf TSA certificate. The chain could come from a trusted source such as [TUF metadata](https://theupdateframework.io/), from the TSA documentation, or through an API, `/api/v1/timestamp/certchain`, if the TSA is an instance of [the service we've implemented](https://github.com/sigstore/timestamp-authority). Run the following:

```
cosign verify --timestamp-certificate-chain ts_chain.pem <artifact>
```

# More Information

If you'd like to operate a timestamp authority, we have [implemented a service](https://github.com/sigstore/timestamp-authority) to issue RFC 3161 timestamps. We have also provided [a client](https://github.com/sigstore/timestamp-authority/releases) to query this service. See the [API specification](https://github.com/sigstore/timestamp-authority/blob/main/openapi.yaml) for more information on how to call the service.

If you have questions, come chat with us on Slack on the [#timestamping channel](https://sigstore.slack.com/archives/C047W7KEU6A).

