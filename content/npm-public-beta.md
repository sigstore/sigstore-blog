+++
title = "Sigstore Support in NPM launches for Public Beta"
date = "2023-02-23"
tags = ["sigstore","npm","security","node"]
draft = true
author = "The Sigstore Techincal Steering Committee"
type = "post"
+++

![](/images/npm-beta.png)

We're thrilled to announce that the npm project has launched a public beta, bundling Sigstore support directly into the npm Command-Line Interface (CLI). This innovation brings strong origin guarantees to the npm ecosystem and marks the first time that a large package registry technology is using public software signatures to attest to a package’s originating source code and build instructions.

Sigstore is an OpenSSF project that enables developers to validate that the software they are using is exactly what it claims to be using cryptographic digital signatures and transparency log technologies. As the default package manager for the Node.js runtime, npm is the de facto standard for packaging and sharing code in the JavaScript ecosystem, and npmjs.com is the largest package registry on the internet by downloads-per-day. 

npm CLI tool

The CLI tool allows developers to construct, install, and manage packages that can be used in any Node.js project. It also includes features like version management, dependency tracking, and automated package installation, which make it easier for developers to manage their projects and share their code with others. In addition to Node.js packages, npm also supports the management of packages for front-end web development using tools like React, Angular, and Vue.js. npm is widely used by developers worldwide and hosts millions of packages in its registry.

“It’s incredible to see this adoption play out and testament to how serious the npm community takes the ever increasing threats to software supply chains. The work the npm community has carried out to integrate sigstore is a gold standard level of implementation and this work should lighten the path for other communities wanting to adopt sigstore and improve their supply chain security resilience.” Luke Hinds, Sigstore Technical Steering Committee (TSC) Chair.

Sigstore-signed Provenance

Using Sigstore, npm will start to generate signed cryptographic provenance records each time a package maintainer releases a new npm module. In turn users will be able to leverage the sigstore-signed provenance to verify the package is tamper free and generated from the correct source of origin.

“It's amazing what GitHub and npm are doing to provide cryptographically-signed provenance metadata using Sigstore. Securing such a critical ecosystem like npm is not an easy task, let alone trailblazing in the software supply chain security space. I hope this integration sets the tone for the conversation to secure other ecosystems in the same way." Said Santiago Torres-Arias, creator of in-toto, "in a way, this not only benefits npm, but the broader open source ecosystem — leading by example"

We're excited to see this public beta go live, and we're confident that the collaboration between these two open source projects will lead to even more innovations in the years to come.
