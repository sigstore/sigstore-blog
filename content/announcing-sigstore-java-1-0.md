+++
title = "Announcing sigstore-java 1.0"
author = "Appu Goundan (@loosebazooka)"
date = "2024-08-28"
draft = false
type = "post"
tags = ["sigstore","java","clients"]
+++

# sigstore-java
The sigstore-java project brings the Sigstore signing and verification paradigm to the Java ecosystem. It is built natively in Java and is easy to integrate with your Maven and Gradle builds or custom workflows.

## 1.0 Stable Release
Today, thanks to the support of the community and work of our contributors, we’re excited to announce a 1.0 stable release of the sigstore-java client. This includes a library([`dev.sigstore:sigstore-java`](https://central.sonatype.com/artifact/dev.sigstore/sigstore-java)) for programmatic access to the sigstore signing and verification APIs. It also includes signing plugins for maven([`dev.sigstore:sigstore-maven-plugin`](https://central.sonatype.com/artifact/dev.sigstore/sigstore-maven-plugin)) and gradle([`dev.sigstore.sign`](https://plugins.gradle.org/plugin/dev.sigstore.sign)) that integrate effortlessly with your build and publishing workflows.

## Signatures on Maven Central
Thanks to the team at Sonatype, Sigstore signature bundles (`*.sigstore.json`) are an accepted signature format on Maven Central. So you can start signing your Maven Central artifacts with Sigstore today. To ensure existing verification workflows and tooling continue to work for Java workflows PGP signing is currently still required for publishing to Maven Central.

## Features
The `sigstore-java` library is meant to provide a comprehensive API to sigstore infrastructure. It gives users access to both simple high level APIs while also making it possible to tweak the behavior according to your needs.

The 1.0 ships with these features today
- Programmatic signing and verifying using sigstore-java APIs
- Easy signing using Maven and Gradle Sigstore plugins
- Signing identity via browser workflow or from Github Actions id-token
- Sigstore signature bundle support (`*.sigstore.json`)

The client is already in use by some early adopters, and we attach sigstore signature bundles to all our sigstore-java artifacts on Maven Central. Head over to our github page ([sigstore-java](https://github.com/sigstore/sigstore-java), [maven](https://github.com/sigstore/sigstore-java/tree/main/sigstore-maven-plugin), [gradle](https://github.com/sigstore/sigstore-java/tree/main/sigstore-gradle)) for further details on accessing the API or integrating directly with your Java builds.

## Development, Testing and Contributing
Sigstore is an open source project developed directly on GitHub and made possible by a dedicated and inclusive community.

In addition to standard tests, sigstore-java is fuzzed with [oss-fuzz](https://github.com/sigstore/sigstore-java/tree/main/fuzzing) and tested with the [sigstore-conformance](https://github.com/sigstore/sigstore-conformance) test suite to ensure compatibility within the sigstore ecosystem and all other language clients.

Feel free to file issues directly on the sigstore-java [issue tracker](https://github.com/sigstore/sigstore-java/issues). You can also catch on the [sigstore#java](https://app.slack.com/client/T01CP44M5K9/C03239XUL92) slack channel.

Special thanks to our core contributors [Patrick Flynn](https://github.com/patflynn) and [Vladimir Sitnikov](https://github.com/vlsi) and also to [Hervé Boutemy](https://github.com/hboutemy) from Maven, [Louis Jacomet](https://github.com/ljacomet) from Gradle, and [Joel Orlina](https://github.com/jorlina) for their help with Maven Central.
