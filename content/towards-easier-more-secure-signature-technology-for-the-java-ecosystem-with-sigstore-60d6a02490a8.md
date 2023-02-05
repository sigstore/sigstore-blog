+++
title = "Towards Easier, More Secure Signature Technology for the Java Ecosystem with Sigstore"
date = "2023-02-04"
tags = ["sigstore","java","gradle","maven"]
draft = false
author = "Sigstore"
type = "post"
+++

![](/images/sigstore_maven_gradle.png)

In October 2022, the Sigstore project announced the General Availability of its free software signing service giving open source communities access to production-grade services for artifact signing and verification. As the project matures, so do the language client integrations that are actively being developed. In January 2023, sigstore-python announced the 1.0 version of Sigstore for Python.

The Java community has always taken a mature approach to security. So it should come as no surprise that there is plenty of activity towards integrating Sigstore into the existing ecosystem and offering first-class support for software signing and verification with Sigstore.

For an overview of signing, including the evolution from PGP to Sigstore, this talk by Hervé Boutemy is highly recommended:

{{< youtube id="gFRt4fC7cMU" >}}

In short, many in the Java ecosystem are looking to Sigstore as a replacement for PGP signing with these particular benefits in mind:

* Users don’t manage keys; keys are single use
* Email addresses associated with signing are verified by cert authority/oidc provider
* Auditing via transparency logs

### Sigstore-java

The Sigstore Java client has been making steady progress through the collaboration of the Sigstore & Java working group. Most recently the client has adopted the new Sigstore bundle format which will promote interoperability amongst client implementations. The [sigstore-java client library](https://github.com/sigstore/sigstore-java) will be the foundation piece on which integration with Maven, Gradle, and other Java ecosystems will build on. It will provide a native Java implementation of signing and verification services.

Special thanks to Appu Goundan (Google), Vladimir Sitnikov, and Patrick Flynn (Chainguard) for their work on this client library.

### Sigstore & Sonatype’s OSSRH Service (aka “Maven Central”)

The OSSRH Service is the primary public repository for Java artifacts. It is an essential component of open source Java development making it simple and easy to consume third-party artifacts. Back in March 2022 Sonatype, the organization behind Maven Central, announced the intent to [adopt Sigstore as part of the Maven Central platform](https://central.sonatype.org/news/20220310_sigstore/).

Maven Central currently requires and uses PGP for signing artifacts. Each artifact must be accompanied by a detached PGP signature file. Work is underway to use Sigstore as a replacement for PGP in Maven Central. The work has been broken down into multiple stages.

*Stage 0 [complete]*- Maven Central historically has required a PGP signature for all uploaded artifacts. That requirement is being dropped for Sigstore signature artifacts.

*Stage 1* — Include Sigstore signature verification in Maven Central repository pre-release checks.

Supported workflows will be:

- artifact set + PGP signature + Sigstore signature
- artifact set + PGP signature
- artifact set + Sigstore signature

In all workflows, all provided cryptographic signatures will be verified.

*Stage 2 & Beyond* — Verification & more!

Thank you Damian Bradicich (Sonatype) and Joel Orlina (Sonatype) for your work enabling Sigstore support on Central.

### Sigstore & Java Build Tools

Gradle is one of the most popular build automation tools and it is used extensively in the Java and Android ecosystems. An early community proof of concept of signing artifacts with Sigstore was demoed at Sigstore Office hours back in September 2022 by Vladimir Sitnikov. Since then key members of the Gradle team have begun investigating what would be required for a full signing and verification Sigstore workflow to work within Gradle. The intent will be for Gradle versions 7.3 or higher to support Sigstore signing. Verification will require 8.2+. Gradle currently supports PGP signature verification in addition to signing publications.

Credit goes to Vladimir Sitnikov, Sterling Greene (Gradle), Louis Jacomet (Gradle), and Octavia Togami (Gradle).

### Maven

Jason Van Zyl, the original creator of Maven, was the first to build our first testing version of the [Maven Sigstore plugin](https://github.com/sigstore/sigstore-maven) that supports generating and publishing Sigstore signatures to Central. As the sigstore-java client library completes its support for the new Sigstore blob signing bundle the Maven plugin will be updated with this support and we expect will soon after be ready for production use.

### Join The Community

We are making rapid progress towards first-class support for Sigstore in the Java ecosystem. If you’d like to get involved please join us on the Sigstore #java slack and come to our weekly [community meeting](https://calendar.google.com/calendar/u/0?cid=ZnE0a2dvbTJjZTQzaG5jbmJjZmphMmNrMjBAZ3JvdXAuY2FsZW5kYXIuZ29vZ2xlLmNvbQ); the next one is on Wednesday, February 8.
