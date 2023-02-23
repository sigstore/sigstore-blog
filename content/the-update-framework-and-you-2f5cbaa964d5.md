+++
title = "The Update Framework and You"
date = "2021-05-18"
tags = ["sigstore","infosec","opensource","security"]
draft = false
author = "Dan Lorenc"
type = "post"

+++

Why does it need to be so TUF?

If you’re anything like me and spend time reading blog posts and GitHub discussions around how to securely package and release software, you’ve probably heard of [The Update Framework](https://theupdateframework.io/). Unfortunately, if you’re actually anything like me it probably seemed overwhelming and confusing at first. This blog post explains the mental model I’ve built up for TUF, and some of the concepts that finally made it understandable and digest-able for me.

# What is TUF?

Naming is hard, but TUF really isn’t a framework in the traditional sense. In my head, a framework is something like a library but also the opposite. Libraries provide useful functions you can call in your program however you see fit, frameworks give you, well, a framework to write your application inside of. You call libraries, frameworks call you.

While there are some TUF libraries that make working with TUF metadata and formats easier, TUF is neither a library nor a framework. So, what is it then? TUF is a set of defined attacks and threat models specific to software distribution systems, and a cleverly designed set of protocols to protect against them. Maybe it’s a “design framework” rather than an implementation one — but it’s definitely not a traditional batteries-included development framework like Django or Angular or Rails.

# Getting Concrete

Enough philosophy, what does it do? This is difficult to explain too. TUF addresses a lot of *really* subtle attacks and threat models that you might have never considered. I think you can split TUF up into a few mostly-independent pieces that make it easier to understand. You might not be building a full update system where you need TUF, but you might still be inspired by some of these parts and make use of them without the rest of the framework.

I like to break TUF up into three parts: key protection, role delegation and freeze-attack prevention. Let’s go through them one at a time.

## Key Protection

One of the hardest things in software distribution is protecting your keys. Once you’ve finally managed to convince your users to check signatures, you now have to make sure you don’t leak or (more likely) lose your private keys. Rather than assume this will never happen and say “game-over”, TUF takes a different, real-world approach by assuming you will lose or leak keys eventually. It’s still not a great situation — but TUF allows you to recover from it.

There’s no magic here — TUF works like everything else in software, by adding another level of indirection. Instead of having a single root key you have to protect at all costs, TUF recommends you create a set of root keys across different systems. You can spread these out however you like. The whole idea is that you won’t be using these keys often, so you can get really creative. Put them in hardware somewhere. Store them on paper in a bank vault. Grab a [Yubikey and swallow it.](https://twitter.com/p1nt1nh0/status/1369637197394153474) The important thing is to have a couple of them, ideally not stored next to each other.

Right before you get excited and scatter them around like horcruxes, you use all of these (or some kind of quorum you define) to sign the actual keys you’re going to use for releases. Then publish these signatures (the ones you get from using the root keys to sign your signing keys) as official proof that these are the real keys you’re going to be using for a little while.

When you decide to rotate the “daily-driver” keys, either because you’re ready for a change or because you lost/leaked some of them (there are actual schedules you should follow, we’ll get to that later though), you have to go collect all the root keys from the corners of the earth and sign a new set of keys. If you want to get meta, you can even use the remaining root keys (there’s no way you lost them all!) to sign a new root key and add it into your rotation.

So — to summarize, TUF “shards” out your root keys by allowing you to have multiple independent ones. The horcrux analogy is actually a really good one. You just have to keep adding new keys faster than an attacker can compromise them to stay protected. These root keys are not used frequently, so you can make them really inconvenient for yourself and attackers. You then assemble some kind of “quorum” of these that can “elect” the keys you actually use on a daily basis.

## Role Delegation

This one can be especially confusing if you’re looking at TUF from the perspective of “I just wan’t to sign a binary on GitHub”. If that’s you — feel free to skip this section. It’s still pretty interesting though, even if you may never have a real world need for it.

Role delegation allows you to delegate roles! There are some standard roles that will make sense later when you understand all the pieces of TUF, but you can ignore those for now. Imagine you’re on a large project with hundreds of contributors. Only a subset of these contributors are actual “maintainers”, and only some of these “maintainers” have the necessary permissions to cut a release and publish a build.

Most projects communicate and define these policies implicitly though README.md/MAINTAINER files and IAM permissions on their build/artifact systems. TUF allows you to define this in a standard format which can be verified. It does this with even more keys and signatures!

If you assume every individual on a project has some kind of public key, TUF allows you to publish formal “delegations” by signing those keys. Everything in the project eventually ties back into the root keys, so we call that the “root role” and start delegating from there.

The root role (set of keys) from the first part is used to define other roles (Designated Releaser, for example), and then “bind” this role to a set of individuals. (These can be robots, or anything that can sign things, but pretend they’re all people). You might be able to guess by now, but this “binding” is just another set of signatures from a quorum of the root keys.

You can get meta and allow these groups to create and designate other sub-groups, but if you need to design an actual system like this you should go ask the TUF maintainers for help and stop reading this post immediately.

In summary — there are a lot of keys flying around in TUF. Role delegations help you securely and verifiably group these keys into meaningful subsets and hierarchies that can be managed together.

## Heartbeats

In my opinion this is by far the most controversial part of TUF. It is designed to address a pretty narrow threat model— called a freeze attack or a rollback attack.

If you or your systems rely on some auto-update mechanism to apply security patches, and an attacker can trick you into installing an older version (or not updating to a newer version), you might be on a vulnerable build without realizing it. The attacker can take advantage of this, and you will be sad. This is a really tricky problem to address, but it can be very very very important for certain software distribution systems.

What build of software is running in your TV or your car? You probably don’t know off the top of your head. Software in these systems has bugs too, and some of them are very scary. If you’re not paying attention to the updates and versions, hopefully someone is.

But how can the manufacturer (or whoever is pushing updates) be sure you’re getting them, and someone hasn’t tricked you into not seeing the updates? What if an attacker is able to completely block access to the update server from your house? You could argue that’s outside the threat model, but at some point a car would happily go years without an update and bad things might happen to the car and/or driver.

TUF solves this with exploding timestamps. The update server promises to give you a fresh message containing the updated version information every week, whether there is an update or not. If there is no new build, it simply publishes a manifest with a fresh timestamp telling you you’re all set. If there is a new build, there’s a new signed timestamp with the new build info. If you don’t hear back that week -something is up and you should look into it. That’s a (simplified) version of TUF’s heartbeat or timestamp protocol.

If you’ve been paying careful attention, you might have noticed that this protocol is actually pretty scary! We haven’t really fixed a threat model, we made a tradeoff and replaced it with new ones. No tradeoff, including this one, is always a good idea. If it were, it would be called a Best Practice instead of a tradeoff.

In this timestamp protocol world, an attacker can no longer trick me into running an old, potentially unpatched version of software, but if they cut off my network they can prevent me from receiving updated timestamp snapshots. Which leaves an important question for the implementer of TUF — what happens if the timestamps expire without being refreshed? Should the software stop working? Pop up a giant warning? Users are very good at ignoring these. There are often no good options here.

Another downside is that continuously refreshing these timestamps also introduces an operation burden on the distributor. What if your system fails and you forget to publish an update that week? Now all your users get warnings or errors or are broken, and everyone involved will have a bad day.

This tradeoff is probably a good idea for critical systems where you already need to staff a 24–7 on-call rotation for your update system and your users have complete trust that you will push them updates in a timely manner without them taking any action. It’s probably not a good idea for a single developer with a cron job running on a Raspberry Pi under their desk signing timestamps.

Unfortunately most OSS development and infrastructure is WAY closer to the Raspberry Pi model than you would hope (insert relevant XKCD here), so I can’t really recommend this part in good faith for most OSS software. If you do decide to go ahead with this, at least give yourself a safety margin. Refresh the timestamps 2–3x as often as you need to, but alert on every failed update. Make sure you have to miss **multiple ** alerts before anything stops working. This is how Let’s Encrypt is typically used with short-lived TLS certificates.

These exploding timestamps aren’t only used for updates though! They’re also used as part of the key protection/rotation story I mentioned in part one. When you make some keys easy to access and use, you inherently make them more vulnerable to compromise. You mitigate this by giving these keys short lifetimes (using delegations and timestamps). You must rotate or re-sign your keys to keep using them past that time period. If you lose them, no need to revoke. Just wait for them to expire!

# Summary

That’s it! TUF in three simple parts (that admittedly form a complex graph by relying on each other). I’m sure I glossed over a bunch of details and other subtleties here, but this is a only high level overview of some of the parts of TUF that might make sense on their own. There are just a couple core cryptographic primitives (using keys to sign other keys, and signed, exploding timestamps), that are assembled in clever tree structures to express intent.

For example, you might consider using the root key system to protect the keys you use on a daily basis, even if you don’t want a timestamp server. You might even be doing this already without realizing it — PGP sub-keys, and even the entire Web PKI system work this way under the hood.

Or, you might use a delegation system to verifiably define the policy for tagging an official release. You can implement policy like “3 of the 5 release team members must approve”, by requiring multiple signatures from the “authorized” release members. The release member authorization is just another set of signatures, from the root keys!

The timestamp piece is tricky, many people and projects have been burned by letting certificates or keys expire. Don’t do this without a plan in place to prevent this. I don’t mean to completely scare you away from it, but it’s important to at least understand the security vs. reliability tradeoffs you’re making by implementing something like this.

I hope this post helped you get a better understanding of TUF, and that you can use this knowledge to better understand the threat models against the software you produce and consume.