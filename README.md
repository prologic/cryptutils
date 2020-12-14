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
$ go get github.com/prologic/cryptutils
```

And import it:

```#!go
import "github.com/prologic/cryptutils"
```

See the [Godoc](https://pkg.go.dev/github.com/prologic/cryptutils) for reference.

To install the `salt` command-line tool:

```#!console
$ go get github.com/prologic/cryptutils/cmd/salt/...
```

## Usage (command-line)

Usage of the `salt` command-line tool is as follows.

The first time you run `salt` it will generate a password-protected set of
private keys (_Curve25519 and Ed25519_). The public key can be exported with
`salt -export -`.

### Exporting your public key:

```#!console
$ salt -export -
```

### Adding keys

Exported keys include a signature. When exporting your own public key, the key
will be self-signed. The keystore operates on a web-of-trust, however,
and `salt` won't import self-signed keys by default. To bootstrap a key into
the web-of-trust, you'll need to allow untrusted keys for import.

Importing a key with a trusted signature:

```#!console
$ salt -import <file>
```

Importing a self-signed key (or a key with a signature not in the keystore):

```#!console
$ salt -u -import <file>
```

Both of these will prompt for a label to store the key under; the special
label "self" can't be used for imported keys, as it refers to the owner's key.

A public key can be removed with the `-r` flag; a label will be prompted for.
The current public keys can be listed with the `-k` flag:

```#!console
$ salt -k
Key store was last updated 2020-12-14 02:03 AEST
2 keys stored
Owner public key:
	Fingerprint: d90baf16a9312fc89bf4bbf2927958932e3e03b8bd5764084a4dc55aa3e60230
Key store:
	Adrian Grigore
		Last update: 2020-12-14 02:03 AEST
		  Signed at: 2020-12-14 02:02 AEST
		  Signed by: Jamess-MacBook
		Fingerprint: 18a69295ba5cdee713086df02a7df295491f1b4135bfa1d1d59a6b2c2320a7a8
	Jamess-MacBook
		Last update: 2020-12-14 01:10 AEST
		  Signed at: 2020-12-14 01:10 AEST
		  Signed by: self
		Fingerprint: 5e10172d37ba4ea8e24f0424888b1825fc06fdf96b6b83f8f2d496ce891df1c8
```

### Encrypting and Decrypting

Encrypting a file requires selecting a label (which defaults to
"self", i.e. because the most common operation I do is encrypting
files to myself) using the `-l` argument, and passing the `-e`
flag. For example,

```#!console
$ salt -e backup.tgz backup.tgz.enc
```

The same thing can be done with a label to encrypt to a different public key:

```#!console
$ salt -e -l James-MacBook backup.tgz remote-backup.tgz.enc
```

Alternatively, the encrypted file can be ASCII-armoured with the `-a` flag,
and you can use stdin and stdout as files by specifying `-` arguments:

```#!console
$ echo 'Hello World' | salt -a -e - -
-----BEGIN CRYPTUTIL ENCRYPTED MESSAGE-----
+HkkRtpDKNKArweaVsW0oyzwPV/E1z4fVMjfobR3UW2cVcPfI6lbZaF+rAn47h81
wz4UpvOMcNcf3d1G9pJJ+QpvYLpIpPgpEuPBiCXFfU3vgFF71Gm6HOM=
-----END CRYPTUTIL ENCRYPTED MESSAGE-----
```

Encryption uses ephemeral Curve25519 keys, and doesn't require
unlocking the keystore.

Decrypting is performed with the `-d` flag. The armour flag has no
effect, as decrypt will handle armoured files transparently.

```#!console
$ salt -d backup.tgz.enc backup.2.tgz
keystore passphrase> 
diff backup.tgz backup.2.tgz 
```

### Signing and Verifying

A file can be signed with the `-s` flag.

```#!console
$ salt -s backup.tgz backup.tgz.sig
```

The signature can be verified with the `-v` flag:

```#!console
$ salt -v backup.tgz backup.tgz.sig
Signature: OK
```

If the file was signed by another key, the `-l` argument is needed to
identify the signing public key. For example, if I wanted to verify
the previous file on my server, I need to tell `salt` to use the key
for my laptop (labeled "James-MacBook") for verification:

```#!console
$ salt -v -l James-MacBook backup.tgz backup.tgz.sig
Signature: OK
```

### Signing and Encryptiong

The `-s` and `-e` flags can be combined to perform both encryption and
signing at the same time; the message is signed and then encrypted to the
label named by the `-l` argument. This will automatically armour the file so
that `salt` can deal with it appropriately.

```#!console
$ salt -s -e - -
keystore passphrase>
Hello World
-----BEGIN CRYPTUTIL SIGNED AND ENCRYPTED MESSAGE-----
1gLhsS1l2km6esKrapAcmunRW82h7bKoR3AO/flfVkWDOraI/5AAcZJwtxVZh+FU
iYJcfs5SXsOn8JkZReq9f/RzuS7No/1EzPfbcHMISk8lCPocFWaKephBRRQNHfEZ
saKqN4oiBYmUP0U36rRQz3K7rDXHGS2wqg9/VWd1XmBntVyx+BrsjmhpxnrIicBi
gsRJyHiqCSIyc+egRaU=
-----END CRYPTUTIL SIGNED AND ENCRYPTED MESSAGE-----
```

The `-d` flag will also handle this case transparently, but requires
the `-l` argument to be set appropriately to verify the signature.

### Miscellanea

The integrity of the keystore can be checked with `-check`: it will
ensure the keystore unlocks properly, ensures the keystore is valid,
and performs a key audit. Key audits ensure that every public key has
a signature chain leading back to "self".

The keystore can be changed using the `-f` option to name a file.

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
