+++
title = "Using rekor-monitor to Scan Your Transparency Logs"
date = "2024-11-21"
tags = ["sigstore", "rekor-monitor", "security", "rekor"]
draft = false
author = "Linus Sun (Google)"
type = "post"
+++

## Overview

As part of the tool suite within Sigstore that focuses on providing transparency in the software supply chain, Rekor, Sigstore's signature transparency log, and Fulcio's certificate transparency log provides discoverability and auditability for  signed artifact metadata and code-signing certificates. These immutable read-only logs help secure the software supply chain by making it easier to show what actions have been performed by a compromised identity.

A variety of different improvements have recently been integrated into [rekor-monitor](https://github.com/sigstore/rekor-monitor) to make it easier to use. You can run rekor-monitor either locally from the CLI or remotely via a GitHub Actions reusable workflow and verify the consistency of your transparency log, as well as scanning it for specific identities and their associated entries in the transparency log; the output of these found identity entries can be sent as a notification via a GitHub issue or through email. Newly implemented, rekor-monitor also supports consistency verification and identity monitoring for Fulcio’s certificate transparency log. 

## Consistency Checking

As transparency logs are tamper-evident but not tamper-proof, it becomes important to have a tool to monitor the transparency log for any evidence of tampering, ensuring that the transparency log satisfies the desired properties of immutability and being append-only. rekor-monitor can be used to scan these transparency logs; by reading in a previously saved log checkpoint and computing a consistency proof using the previous and current log checkpoints, it can verify the consistency of the log and ensure that the transparency log has not been tampered with.

## Identity Monitoring

In the event that someone suspects that their account has been compromised, rekor-monitor provides a tool for them to check their associated signatures in the transparency log; this can potentially help a user better identify the window during which their account was compromised, as well as viewing any maliciously generated signatures. rekor-monitor can scan a specified range of log indices to view all entries and report any found entries matching specified identities. In the case that one is not signing with an identity in a certificate, rekor-monitor also supports monitoring for key fingerprints. 

## Demo

To run rekor-monitor locally, one can use the CLI to run the corresponding \`cmd\` package and \`[main.go](http://main.go)\` file with their specified flags, including a server URL to query the transparency log of, a log checkpoint file to read from and write to, and options pertaining to the consistency check and identity monitor, such as what log indices to scan, given identities to monitor for, and any desired form of notification handling from among the supported notification platforms. 

One can also run rekor-monitor via the GitHub Actions reusable monitoring workflow, for which an example can be shown below:

1. First, set up a workflow that uses the rekor-monitor reusable monitoring workflow, as follows:

````
name: Rekor log and identity monitor
on:
  schedule:
    - cron: '0 * * * *' # every hour

permissions: read-all

jobs:
  run_consistency_proof:
    permissions:
      contents: read # Needed to checkout repositories
      issues: write # Needed if you set "file_issue: true"
      id-token: write # Needed to detect the current reusable repository and ref
    uses: sigstore/rekor-monitor/.github/workflows/reusable_monitoring.yaml@main
````

2. This monitor, as set up, will run every hour and verify the transparency log consistency using the reusable monitoring workflow.  
3. If you wish to monitor for given identities, you can input given identities into the created workflow configuration as follows:

````
name: Rekor log and identity monitor
on:
  schedule:
    - cron: '0 * * * *' # every hour

permissions: read-all

jobs:
  run_consistency_proof:
    permissions:
      contents: read # Needed to checkout repositories
      issues: write # Needed if you set "file_issue: true"
      id-token: write # Needed to detect the current reusable repository and ref
    uses: sigstore/rekor-monitor/.github/workflows/reusable_monitoring.yaml@main
    with:
      file_issue: true # Strongly recommended: Files an issue on monitoring failure
      artifact_retention_days: 14 # Optional, default is 14: Must be longer than the cron job frequency
      config: |
	 monitoredValues: |
          certIdentities:
            - certSubject: user@domain\.com
            - certSubject: otheruser@domain\.com
              issuers:
                - https://accounts\.google\.com
                - https://github\.com/login
	   oidMatchers: 
	     fulcioExtensions:
              build-config-uri:
                - https://example.com/owner/repository/build-config.yml
	     customExtensions:
            - objectIdentifier: 1.3.6.1.4.1.57264.1.9
              extensionValues: 
		   - https://example.com/owner/repository/build-signer.yml
        gitHubIssue: 
          assigneeUsername: linus-sun
          repositoryOwner: linus-sun
          repositoryName: rekor-log-monitor
          authenticationToken: <PAT with repo write and push access>

````

This workflow configuration will monitor for any certificate identities containing a SAN (Subject Alternate Name) of either \`[user@domain.com](mailto:user@domain.com)\` or \`[otheruser@domain.com](mailto:otheruser@domain.com)\`, as well as looking for the issuers \`[https://accounts.google.com](https://acocunts.google.com)\` or \`[https://github.com/login](https://github.com/login)\`. For those who want to specifically monitor object identifiers, the configuration supports monitoring for object identifier extensions supported by Fulcio (in this case, any build config URI matching \`[https://example.com/owner/repository/build-config.yml](https://example.com/owner/repository/build-config.yml)\`), as well as custom provided object identifier extensions, for those who want to monitor custom OIDs, in the case of a private deployment (in this case, the custom object identifier is monitoring for the OID extension value matching the build config URI of \`[https://example.com/owner/repository/build-signer.yml](https://example.com/owner/repository/build-signer.yml)\`).

With the given \`gitHubIssue\` configuration of a repo owner and username, as well as an assignee username and defined personal authentication token, the identity monitor workflow will also write any found identities to the body of a newly created GitHub issue as shown below:

![](/images/rekor-monitor-github-issue.png)

## Wrapup

Using a reusable monitoring workflow, rekor-monitor can periodically and automatically verify both the consistency of a log and search for any found identities, better ensuring the security of both a log and any identities signing entries in the log. Happy monitoring\!