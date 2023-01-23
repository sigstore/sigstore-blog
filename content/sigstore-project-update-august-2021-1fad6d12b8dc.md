title = "Sigstore Project Update — August 2021"
date = "2021-08-31"
tags = ["sigstore","security","devsecops"]
draft = false
author = "Luke Hinds"
type = "post"

+++

Welcome to our project update for August.

As always the community is continuing to expand. We are now close to 600 members in our [slack workspace](https://join.slack.com/t/sigstore/shared_invite/zt-mhs55zh0-XmY3bcfWn4XEyMqUUutbUQ). We now have [46 contributors](https://insights.lfx.linuxfoundation.org/projects/sigstore/dashboard) and are growing each day.

The month of August saw 254 commits and 526k lines of code were changed!

Lots of exciting things are happening, read more for our project updates and our presence at KubeCon, NA.

![](/images/aug21.png)

### cosign

Cosign hit 1.0 and adoption is coming on great. It seems almost daily that we spot someone new using cosign.

- go-releaser now has the [ability to sign](https://carlosbecker.com/posts/goreleaser-cosign/) using cosign
- flagger, a delivery kubernetes operator is now [signing its images](https://twitter.com/stefanprodan/status/1430491008752623617) using cosign
- Kubernetes is [seeking to expand](https://github.com/kubernetes/release/issues/2227) its signing efforts using cosign and sigstore

There is some exciting work happening around TUF integration with a Fulcio root. See here for a more [detailed update](https://dlorenc.medium.com/using-the-update-framework-in-sigstore-dc393cfe6b52?source=user_profile---------2----------------------------), and expect to see this start landing next month!

### fulcio

Fulcio continues on its path to maturity. A first release is in the planning stage and will come shortly after rekor 1.0 (since rekor is a dependency of fulcio). We’re also working to directly integrate fulcio IDs with The Update Framework, which should be [landing soon](https://github.com/theupdateframework/go-tuf/pull/141).

### rekor

Rekor will soon be reaching it’s 1.0 release which means we consider it ready for production! We are currently working on a sharding approach and considering its relevant API design.

### New Website!

sigstore has a cool new website, check it out [https://sigstore.dev](https://sigstore.dev/)

![](/images/how.png)

### Other Projects

The Helm Plugin is almost ready to [publish](https://github.com/sigstore/sigstore-helm-operator/pull/2)

[k8s-manifest-sigstore](https://github.com/sigstore/k8s-manifest-sigstore) is developing at a fast space and will bring many protections to the kubernetes space with its ability to leverage admission controller based signing and validation of workloads.

We now have [homebrew taps](https://github.com/sigstore/homebrew-tap) for cosign and rekor-cli!

### Events and talks!

Sigstore will be at KubeCon, NA, 2021 where we will have a booth! Come and say hi and let’s chat signing all the things! Community members will be present and there to help you get set up using sigstore and there will be swag!

There will also be lots of talks. Project co-founders Dan Lorenc and Luke Hinds will be providing a [history of sigstore and what’s next](https://events.linuxfoundation.org/kubecon-cloudnativecon-north-america/program/schedule/)

### Supply Chain Security Con

The Continuous Delivery Foundation is hosting a colocated event with KubeCon North America called Supply Chain Security Con, and Sigstore is featured in a number of talks. Check out the full agenda [here](https://events.linuxfoundation.org/supplychainsecuritycon-north-america/).

[Kubelist Podcast](https://www.heavybit.com/library/podcasts/the-kubelist-podcast/ep-20-sigstore-with-dan-lorenc-of-google/?utm_campaign=coschedule&utm_source=twitter&utm_medium=heavybit&utm_content=Ep. %2320, Sigstore with Dan Lorenc of Google)

[OSS Supply Chain Security Deep Dive with Dan Lorenc](https://www.youtube.com/watch?v=JhFr5_PQS_Y)

[Luke Hinds OpenShift Commons — sigstore](https://www.twitch.tv/videos/1127122758)

### Getting Involved!

As always, we truly welcome contributors and users to our community. We take pride in being friendly to new folks and fostering a welcome and safe environment. Being a large open source project, there is always lots to do and it’s not always complex coding tasks, helping with documentation, general testing or just telling others about sigstore are all valued contributions. Come and join our [slack workspace](https://join.slack.com/t/sigstore/shared_invite/zt-mhs55zh0-XmY3bcfWn4XEyMqUUutbUQ) and say hello!