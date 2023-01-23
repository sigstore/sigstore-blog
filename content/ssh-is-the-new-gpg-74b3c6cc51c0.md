+++
title = "SSH is the new GPG"
date = "2021-01-25"
tags = ["sigstore","GitHub","open source","Security"]
draft = false
author = "Dan Lorenc"
type = "post"

+++

Not really. But Kind of?

Did you know that you probably already have a working PKI system for signing artifacts on your laptop today, with no keyservers, web-of-trust, or configuration? You can use it to sign files, and to find the public keys for other people and use them to verify files they signed.

So why aren’t more people using this? I think it’s just gone overlooked because it’s a relatively new feature in apretty old piece of software. I’ve only been able to find one [other blog post](https://seankhliao.com/blog/12020-01-09-signing-and-encrypting/) explaining how it works (outside of the man pages).

![](/images/ssh1.jpg)

### Signing and Verifying

Since early 2020, OpenSSH has supported creating and verifying file signatures natively in the `ssh-keygen` binary. You can create signatures with:

```
ssh-keygen -Y sign -n file -f $HOME/.ssh/id_rsa.pub <FILE-TO-SIGN>
```

This signs the file specified in <FILE-TO-SIGN> using your SSH public key at the standard location `$HOME/.ssh/id_rsa.pub` . The signature ends up at `<FILE-TO-SIGN>.sig` by default, and looks roughly like:

```
-----BEGIN SSH SIGNATURE-----
U1NIU0lHAAAAAQAAAhcAAAAHc3NoLXJzYQAAAAMBAAEAAAIBAPDRlY/jNksQtIlV6dBN<...>
-----END SSH SIGNATURE-----
```

If you have a signature, a file, and the SSH public key, you can verify it with `ssh-keygen` as well! First you have to create an `allowed_signers`file with the public key you want to verify against and a fake `Principal` name:

```
echo "thesigner $(cat id_rsa.pub)" > allowed_signers
```

Then, use that to verify:

```
cat <FILE-TO-VERIFY> ssh-keygen -Y verify -n file -f allowed_signers -s <SIGNATURE> -I thesigner
```

### Finding Keys

But how do you find public keys to check against, or distribute yours? PGP uses the web-of-trust and key-servers for this. Web PKI uses Certificate Authorities. Here, we can just use GitHub! Almost everyone using GitHub already has an SSH key configured to push code with, and GitHub exposes these keys for every user.

To see mine, you can hit https://github.com/dlorenc.keys. Replace `dlorenc` with your username to see what keys you use, or any other username to see the public keys for that user.

To see them in `json` form for scripting, you can access: https://api.github.com/users/dlorenc/keys

### Putting It All Together

The `allowed_signers` file from before basically contains a map of `Principal` to `Public Key`. Here, `Principal` is just a fancy word for “a string that represents a person”. So we can easily make an allowed_signers file that maps GitHub user names to public keys!

```
USERNAME="dlorenc"
curl https://github.com/${USERNAME}.keys | while read key; do
  echo "$USERNAME $key" >> allowed_signers.github
done
```

You can run this for any GitHub user, and append into your public key database file.

Then, to check a signature/file against a Github user:

```
cat FILE | ssh-keygen -Y verify -n file -f allowed_signers.github -I USER -s FILE.sig
```

### Caveats

Obviously there are some caveats here. This requires that users use SSH public keys to push to GitHub. This requires that GitHub users remove old or compromised public keys regularly. This little script does not handle revocation or removing old keys as users clean up their GitHub accounts.

Take a look at the documentation for `ssh-keygen` to see more information on how this all works before using it for anything serious.

Thanks to [Damien Miller](https://twitter.com/damienmiller) for designing and implementing this feature, and for maintaining OpenSSH!

Follow [me](http://twitter.com/lorenc_dan) on Twitter for more posts like this one.