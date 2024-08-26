+++
title = "Fulcio Streamlines Onboarding for CI Identity Providers"
author = "Javan Lacerda (@javanlacerda)"
date = "2024-08-26"
draft = true
type = "post"
tags = ["sigstore","fulcio","identity"]
+++

### TL;DR
[Fulcio](https://github.com/sigstore/fulcio), the Sigstore certificate authority, has introduced a new feature that simplifies the onboarding process for identity providers, mainly for continuous integration. This enhancement eliminates the need for complex, provider-specific logic implementations.

Traditionally, integrating a new identity provider for CI involved significant development effort, which required understanding the codebase and cutting a new release. However, with this update, onboarding is reduced to configuring a YAML file which houses essential provider information for building certificate extensions based on ID token claims. Additionally, the configuration allows a provider to use template strings for handling more complex logic.

### Key benefits of this streamlined approach
* Reduced complexity: Developers no longer need to delve into intricate provider-specific code.
* Faster onboarding: The YAML-based configuration significantly accelerates the integration process.
* Enhanced maintainability: A centralized configuration simplifies updates and modifications, improving the time to deployment as with this feature, we only need to update a configuration file and roll it out to support a new CI provider, like all of the other providers.

This improvement underscores Fulcio's commitment to developer-friendly solutions and its dedication to simplifying signing with the adoption of workload identities used to generate artifact signatures. By streamlining onboarding, Fulcio empowers organizations to seamlessly integrate diverse identity providers.For CI identity providers, all complexity of implementing provider-specific logic and the necessity of cutting a new release for modifications was totally removed.

New identity providers for CI should be able to use this feature by filling the config.yaml file by following the example [here](https://github.com/sigstore/fulcio/pull/1738/files). More information about how to fill it properly can be found in [Fulcio's documentation](https://github.com/sigstore/fulcio/blob/main/docs/oidc.md#integration-guide).

### The summary for adding a new CI identity provider is the following
1. Add your CI provider into the **define** section for defining a variable, like this:
```yaml
define:
  - &my-provider-type "my-provider-name"
```
2. Add your CI provider into the oidc-issuers section by filling the following fields:
```yaml
oidc-issuers:
  https://myissuer.url.com:
    issuer-url: https://myissuer.url.com
    client-id: sigstore 
    contact: mycontact@email.com
    description: "My OIDC auth"
    type: ci-provider
    ci-provider: *my-provider-type
```

Note that for CI providers, the field type should be set as **ci-provider** and the field **ci-provider** should point to your provider variable, which was created on step 1.

**NOTE: If your issuer URL has a regular expression, you should add it into the meta-issuers section**

```yaml
meta-issuers:
  https://*.myissuer.url.com/*:
    issuer-url: https://*.myissuer.url.com/*
    client-id: sigstore 
    type: ci-provider
    contact: mycontact@email.com
    description: "My OIDC auth"
    type: ci-provider
    ci-provider: *my-provider-type
```

3. Fill the **ci-issuer-metadata** section, using the variable you've set on the step one as key and by filling the 3 nested following fields:

**a. extension-templates:** here you will define templates for each extension field that applies to your certificate requirements, for example:
```yaml
ci-isser-metadata:
  *my-provider-type:
    extension-templates:
      run-invocation-uri: "{{ .claim_one }}/{{ .claim_two }}/actions/runs/{{ .claim_three }}/attempts/{{ .claim_four }}"
	.
	.
	.
```	
Note that the templates should be filled by claims that come from your id-token, or by default values that are defined on the **default-template-values**.

**b. default-template-values:** here you will define default values for claims or new information that you would like to use in the templates. Note: For defaults with the same name of claims, the claim value in the token will have priority over the defaults, then default values will be used only for cases where the claim is not found.

```yaml
ci-isser-metadata:
  *my-provider-type:
    default-template-values:
       url: "https://my-issuer-etc.com"
    extension-templates:
      run-invocation-uri: "{{ .claim_one }}/{{ .claim_two }}/actions/runs/{{ .claim_three }}/attempts/{{ .claim_four }}"
      build-config-uri: "{{ .url }}/{{ .claim_two }}"
	.
	.
	.
```	
**c. subject-alternative-name-template:** finally, you should set the template for the subject alternative name:
```yaml
ci-isser-metadata:
  *my-provider-type:
    subject-alternative-name-template: "{{ .url }}/{{ .claim_three }}" 
    default-template-values:
       url: "https://my-issuer-etc.com"
    extension-templates:
      run-invocation-uri: "{{ .claim_one }}/{{ .claim_two }}/actions/runs/{{ .claim_three }}/attempts/{{ .claim_four }}"
      build-config-uri: "{{ .url }}/{{ .claim_two }}"
	.
	.
	.
```	
