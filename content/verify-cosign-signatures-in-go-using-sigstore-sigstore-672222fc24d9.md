+++

title = "Verify cosign signatures in go using sigstore/sigstore"
date = "2022-08-09"
tags = ["sigstore","go","golang","cosign","Edgeless Systems"]
draft = false
author = "Fabian Kammel"
type = "post"

+++

![](/images/gocosign.png)

After integrating [cosign](https://docs.sigstore.dev/cosign/overview/) into the release process of [Constellation](https://www.edgeless.systems/products/constellation/)’s CLI, I also wanted to improve the supply chain security of our metadata that are used for attestation.

Using cosign CLI for signing and verifying [blobs](https://docs.sigstore.dev/cosign/working_with_blobs) or [container images](https://docs.sigstore.dev/cosign/sign) is a well documented process. **The** [**sigstore/sigstore**](https://github.com/sigstore/sigstore) **project is the common go library for all sigstore services and clients** and has [documented public functions](https://pkg.go.dev/github.com/sigstore/sigstore), but I was unable to find examples on how to use them together.

Here is a working example to get you started quickly:

```shell
#!/bin/bash
set -e
echo "Any artifact we want to distribute." > artifact.txt
export COSIGN_PASSWORD=$(openssl rand -hex 32)
cosign generate-key-pair
cosign sign-blob --key cosign.key artifact.txt > artifact.txt.sig
cosign verify-blob --key cosign.pub --signature $(cat artifact.txt.sig) artifact.txt
```

```go
package main

import (
	"bytes"
	"crypto"
	"encoding/base64"
	"fmt"

	"github.com/sigstore/sigstore/pkg/cryptoutils"
	"github.com/sigstore/sigstore/pkg/signature"
)

// VerifySignature checks if the signature of content can be verified
// using publicKey.
// signature is expected to be base64 encoded.
// publicKey is expected to be PEM encoded.
func VerifySignature(content, sig, publicKey []byte) error {
	sigRaw := base64.NewDecoder(base64.StdEncoding, bytes.NewReader(sig))

	pubKeyRaw, err := cryptoutils.UnmarshalPEMToPublicKey(publicKey)
	if err != nil {
		return fmt.Errorf("unable to parse public key: %w", err)
	}
	if err := cryptoutils.ValidatePubKey(pubKeyRaw); err != nil {
		return fmt.Errorf("unable to validate public key: %w", err)
	}

	verifier, err := signature.LoadVerifier(pubKeyRaw, crypto.SHA256)
	if err != nil {
		return fmt.Errorf("unable to load verifier: %w", err)
	}

	if err := verifier.VerifySignature(sigRaw, bytes.NewReader(content)); err != nil {
		return fmt.Errorf("unable to verify signature: %w", err)
	}

	return nil
}

func main() {
	content := "Any artifact we want to distribute.\n"
	sig := "MEQCIEJhWr+c5bCAcWv5gNsUn4Fkjo+qV2ss59SLuS0Gmh8LAiBPoLGy8KkQcV+rDjGN757WE2QS1ujnkuScd9+md+Fzhw=="
	pubKey := "-----BEGIN PUBLIC KEY-----\nMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE+3ZBZe8lNJ4oA5TAPCIQX3IMStdi\nUO6/xsAoZQc3lEKxFu+r1QDWyMS8D/fqLUbMoIW1lPPJV1M3jPiBAhhqPA==\n-----END PUBLIC KEY-----"

	if err := VerifySignature([]byte(content), []byte(sig), []byte(pubKey)); err != nil {
		fmt.Println("Verification failed!", err)
	} else {
		fmt.Println("Verification success!")
	}
}
```

