## cryptutils

`cryptutils ` is a Go library for common crypto operations using NaCL's
primitives and supports asymetric and symetic encryption as well as sigital
signatures using ED25519.

Included is a command-line utility called `salt` which can also be used as
a command-line tool similar in concept to GPG.

> Code originally borrowed from [cryptoutils](https://github.com/kisom/cryptutils)

## Install

To use the library in your project simply install it:

```#!console
go get github.com/prologic/cryptutils
```

And import it:

```#!go
import "github.com/prologic/cryptutils"
```

See the [Godoc](https://pkg.go.dev/github.com/prologic/cryptutils) for reference.

To install the `salt` command-line tool:

```#!console
go get github.com/prologic/cryptutils/cmd/salt/...
```

## The cryptography

`cryptutils` uses NaCl's secretbox (Salsa20 and Poly1305) for
secret-key encryption, NaCl's box (Curve25519, Salsa20, and Poly1305)
for public-key encryption, and Ed25519 for digital
signatures. Typically, secret keys are derived in one of ways: via
Scrypt, or via an ECDH exchange. For Scrypt, the parameters N=1048576,
r=8, and p=1 are used. This makes generating keys using this expensive
(typically, around 5 seconds on my 2.6 GHz i5 machine with 6G of
RAM). When encrypting messages using public keys, an ephemeral key is
generated for the encryption and a shared key is derived from
this. The public key is prepended to the message for extraction by the
recipient. When signing and encrypting using public keys, the message
is signed before encrypting. The recipient will decrypt, then validate
the signature.

## Dependencies

- crypto/ed25519
- golang.org/x/crypto
- github.com/gokyle/readpass 

## License

`cryptutils` is licensed under the terms of the [MIT License](/LICENSE).

The code this was originally based on (_see above_) was licensed under the
terms of the ISC license.
