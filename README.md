# CID - Content IDentifier

[![](https://img.shields.io/badge/made%20by-Protocol%20Labs-blue.svg?style=flat-square)](http://ipn.io)
[![](https://img.shields.io/badge/project-ipld-blue.svg?style=flat-square)](http://github.com/ipld/ipld)
[![](https://img.shields.io/badge/freenode-%23ipfs-blue.svg?style=flat-square)](http://webchat.freenode.net/?channels=%23ipfs)

> self-describing content-addressed identifiers for distributed systems


<!--

## Table of Contents

TODO

-->

## Motivation

[**CID**](https://github.com/ipld/cid) is a format for referencing content in distributed information systems, like [IPFS](https://ipfs.io). It leverages [content addressing](https://en.wikipedia.org/wiki/Content-addressable_storage), [cryptographic hashing](https://simple.wikipedia.org/wiki/Cryptographic_hash_function), and [self-describing formats](https://github.com/multiformats/multiformats). It is the core identifier used by [IPFS](https://ipfs.io) and [IPLD](https://ipld.io).


## How does it work? - Protocol Description

CID is a self-describing content-addressed identifier. It uses cryptographic hashes to achieve content addressing. It uses several [multiformats](https://github.com/multiformats/multiformats) to achieve flexible self-description, namely [multihash](https://github.com/multiformats/multihash) for hashes, [multicodec-packed](https://github.com/multiformats/multicodec/blob/master/multicodec-packed.md) for data content types, and [multibase](https://github.com/multiformats/multibase) to encode the CID itself into strings.

Current version: CIDv1

A CIDv1 has four parts:

```sh
<cidv1> ::= <mb><version><mcp><mh>
# or, expanded:
<cidv1> ::= <multibase-prefix><cid-version><multicodec-packed-content-type><multihash-content-address>
```
Where

- `<multibase-prefix>` is a [multibase](https://github.com/multiformats/multibase) code (1 or 2 bytes), to ease encoding CIDs into various bases.
- `<cid-version>` is a [varint](https://github.com/multiformats/unsigned-varint) representing the version of CID, here for upgradability purposes.
- `<multicodec-packed-content-type>` is a [multicodec-packed](https://github.com/multiformats/multicodec/blob/master/multicodec-packed.md) code representing the content type or format of the data being addressed.
- `<multihash-content-address>` is a [multihash](https://github.com/multiformats/multihash) value, representing the cryptographic hash of the content being addressed. Multihash enables CIDs to use many different cryptographic hash function, for upgradability and protocol agility purposes.

That's it!

## Design Considerations

CIDs design takes into account many difficult tradeoffs encountered while building [IPFS](https://ipfs.io). These are mostly coming from the multiformats project.

- Compactness: CIDs are binary in nature to ensure these are as compact as possible, as they're meant to be part of longer path identifiers or URLs.
- Transport friendliness (or "copy-pastability"): CIDs are encoded with multibase to allow choosing the best base for transporting. For example, CIDs can be encoded into base58 to yield shorter and easily-copy-pastable hashes.
- Versatility: CIDs are meant to be able to represent values of any format with any cryptographic hash.
- Avoid Lock-in: CIDs prevent lock-in to old, potentially-outdated decisions.
- Upgradability: CIDs encode a version to ensure the CID format itself can evolve.

## Human Readable CIDs

It is advantageous to have a human readable description of a CID, solely for the purposes of debugging and explanation. We can easily transform a CID to a "Human Readable CID" as follows:

```
<hr-cid> ::= <hr-mbc> "-" <hr-cid-version> "-" <hr-mcp> "-" <hr-mh>
```
Where each sub-component is represented with its own human-readable form:

- `<hr-mbc>` is a human-readable multibase code (eg `base58`)
- `<hr-cid-version>` is the string `cidv#` (eg `cidv1` or `cidv2`)
- `<hr-mcp>` is a human-readable multicodec-packed code (eg `cbor`)
- `<hr-mh>` is a human-readanble multihash (eg `sha2-256-256-abcdef0123456789...`)

For example:

```
# TODO example
# example CID
# corresponding human readable CID
```

## Versions

### CIDv0

CIDv0 is a backwards-compatible version, where:
- the `multibase` is always `base58` and implicit (not written)
- the `multicodec` is always `protobuf-mdag` and implicit (not written)
- the `cid-version` is always `cidv0` and implicit (not written)
- the `multihash` is written as is.

```
cidv0 ::= <multihash-content-address>
```

### CIDv1

See the section: [How does it work? - Protocol Description](#how-does-it-work-protocol-description)

```
<cidv1> ::= <multibase-prefix><cid-version><multicodec-packed-content-type><multihash-content-address>
```

## Implementations

- [go-cid](https://github.com/ipld/go-cid)
- [js-cid](https://github.com/ipld/js-cid)
- [Add yours today!](https://github.com/multiformats/multicodec/edit/master/multicodec-packed.md)

## FAQ

> **Q. I have questions on multicodec, multibase, or multihash.**

Please check their repositories: [multicodec](https://github.com/multiformats/multicodec), [multibase](https://github.com/multiformats/multibase), [multihash](https://github.com/multiformats/multihash).

> **Q. Why does CID exist?**

We were using base58 encoded multihashes in IPFS, and then we needed to switch formats to IPLD. We struggled with lots of problems of addressing data with different formats until we created CIDs. You can read the history of this format here: https://github.com/ipfs/specs/issues/130

> **Q. Is the use of multicodec-packed similar to file extensions?**

Yes, kind of! like a file extension, the multicodec-packed identifier establishes the format of the data. Unlike file extensions, these are in the middle of the identifier and not meant to be changed by users. There is also a short table of supported formats.

> **Q. What formats (multicodec-packed codes) does CID support?**

We are figuring this out at this time. It will likely be a table of formats for secure distributed systems. So far, we want to address: IPFS's original protobuf format, the new IPLD CBOR format, git, bitcoin, and ethereum objects.

## Maintainers

Captain: [@jbenet](https://github.com/jbenet).

## Contribute

Contributions welcome. Please check out [the issues](https://github.com/multiformats/multicodec/issues).

Check out our [contributing document](https://github.com/multiformats/multiformats/blob/master/contributing.md) for more information on how we work, and about contributing in general. Please be aware that all interactions related to multiformats are subject to the IPFS [Code of Conduct](https://github.com/ipfs/community/blob/master/code-of-conduct.md).

## License

[MIT](LICENSE)
