# CID (Content IDentifier) Specification

[![](https://img.shields.io/badge/made%20by-Protocol%20Labs-blue.svg?style=flat-square)](https://protocol.ai/)
[![](https://img.shields.io/badge/project-ipld-blue.svg?style=flat-square)](https://github.com/ipld/ipld)
[![](https://img.shields.io/badge/freenode-%23ipfs-blue.svg?style=flat-square)](https://webchat.freenode.net/?channels=%23ipfs)

> Self-describing content-addressed identifiers for distributed systems

## Table of Contents

  - [Motivation](#motivation)
  - [What is it?](#what-is-it)
  - [How does it work?](#how-does-it-work)
  - [Design Considerations](#design-considerations)
  - [Variant Representation - Internal CID](#variant-representation---internal-cid)
  - [Variant Representation - Human Readable CID](#variant-representation---human-readable-cid)
  - [Versions](#versions)
    - [CIDv0](#cidv0)
    - [CIDv1](#cidv1)
  - [Decoding Algorithm](#decoding-algorithm)
  - [Implementations](#implementations)
  - [FAQ](#faq)
  - [Maintainers](#maintainers)
  - [Contribute](#contribute)
  - [License](#license)

## Motivation

[**CID**](https://github.com/ipld/cid) is a format for referencing content in distributed information systems, like [IPFS](https://ipfs.io). It leverages [content addressing](https://en.wikipedia.org/wiki/Content-addressable_storage), [cryptographic hashing](https://simple.wikipedia.org/wiki/Cryptographic_hash_function), and [self-describing formats](https://github.com/multiformats/multiformats). It is the core identifier used by [IPFS](https://ipfs.io) and [IPLD](https://ipld.io). It uses a [multicodec](https://github.com/multiformats/multicodec) to indicate its version, making it fully self describing.

**You can read an in-depth discussion on why this format was needed in IPFS here: https://github.com/ipfs/specs/issues/130 (first post reproduced [here](./original-rfc.md))**

## What is it?

A CID is a self-describing content-addressed identifier. It uses cryptographic hashes to achieve content addressing. It uses several [multiformats](https://github.com/multiformats/multiformats) to achieve flexible self-description, namely [multihash](https://github.com/multiformats/multihash) for hashes, [multicodec](https://github.com/multiformats/multicodec) for data content types, and [multibase](https://github.com/multiformats/multibase) to encode the CID itself into strings.

Concretely, it's a *typed* content address: a tuple of `(content-type, content-address)`.

## How does it work?

Current version: CIDv1

A CIDv1 has four parts:

```text
<cidv1> ::= <mb><multicodec-cidv1><mc><mh>
# or, expanded:
<cidv1> ::= <multibase-prefix><multicodec-cidv1><multicodec-content-type><multihash-content-address>
```

Where

- `<multibase-prefix>` is a [multibase](https://github.com/multiformats/multibase) code (1 Unicode codepoint), to ease encoding CIDs into various bases. **NOTE:** *Binary-only* protocols and transport-limited contexts MAY omit the multibase prefix when the encoding is unambiguous. See the [internal CID section](#variant-representation---internal-cid) below for context.
- `<multicodec-cidv1>` is a [multicodec](https://github.com/multiformats/multicodec) representing the version of CID, here for upgradability purposes.
- `<multicodec-content-type>` is a [multicodec](https://github.com/multiformats/multicodec) code representing the content type or format of the data being addressed.
- `<multihash-content-address>` is a [multihash](https://github.com/multiformats/multihash) value, representing the cryptographic hash of the content being addressed. Multihash enables CIDs to use many different cryptographic hash function, for upgradability and protocol agility purposes.

That's it!

## Design Considerations

CIDs design takes into account many difficult tradeoffs encountered while building [IPFS](https://ipfs.io). These are mostly coming from the multiformats project.

- Compactness: CIDs are binary in nature to ensure these are as compact as possible, as they're meant to be part of longer path identifiers or URIs.
- Transport friendliness (or "copy-pastability"): CIDs are encoded with multibase to allow choosing the best base for transporting. For example, CIDs can be encoded into base58btc to yield shorter and easily-copy-pastable hashes.
- Versatility: CIDs are meant to be able to represent values of any format with any cryptographic hash.
- Avoid Lock-in: CIDs prevent lock-in to old, potentially-outdated decisions.
- Upgradability: CIDs encode a version to ensure the CID format itself can evolve.

## Variant Representation - Internal CID

The multibase prefix minimizes risk of data being base-encoded for transport or interoperability purposes and then being divorced from its context, rendering its base-encoding difficult to detect and reconstruct.
Furthermore, many transports and export formats require binary to be encoded as text according to one or more base-encodings.
For this reason, the multibase "prefix" should ONLY be omitted in binary-only contexts, where base-encoding won't be required and context drift is not a concern.

In these binary-only contexts, such as the binary CIDs used as links in the binary data structure [DAG-CBOR](https://ipld.io/specs/codecs/dag-cbor/spec/#links)), CIDs can be stored and consumed as "internal CIDs," i.e. internal to a binary system.
Where such binary-only systems interface with mixed systems, such as exposing internal CIDs to HTTP-based gateways or sending them over text-based transports, an appropriate base-encoding should be chosen and the prefix attached to make them full CIDs, warmly dressed and ready for the outside world.

## Variant Representation - Human Readable CID

It is often advantageous to have a human readable description of a CID, such as for  debugging purposes, unit tests, and documentation. We can easily transform a CID to a "Human Readable CID" by translating and segmenting its constituent parts as follows:

```text
<hr-cid> ::= <hr-mbc> "-" <hr-cid-mc> "-" <hr-mc> "-" <hr-mh>
```
Where each sub-component is replaced with its own human-readable form from the relevant registry:

- `<hr-mbc>` is the name of the multibase code (eg `z`--> `base58btc`)
- `<hr-cid-mc>` is the name of the multicodec for the version of CID used (eg `0x01` --> `cidv1`)
- `<hr-mc>` is the name of the multicodec code (eg `0x51` --> `cbor`)
- `<hr-mh>` is the name of the multihash code (eg `sha2-256-256`) followed by a final dash and the hash itself `-abcdef0123456789...`)

For example:

```text
# example CID
zb2rhe5P4gXftAwvA4eXQ5HJwsER2owDyS9sKaQRRVQPn93bA
# corresponding human readable CID
base58btc - cidv1 - raw - sha2-256-256-6e6ff7950a36187a801613426e858dce686cd7d7e3c0fc42ee0330072d245c95
```

See: https://cid.ipfs.io/#zb2rhe5P4gXftAwvA4eXQ5HJwsER2owDyS9sKaQRRVQPn93bA

## Versions

### CIDv0

CIDv0 is a backwards-compatible version, where:
- the `multibase` of the string representation is always `base58btc` and implicit (not written)
- the `multicodec` is always `dag-pb` and implicit (not written)
- the `cid-version` is always `cidv0` and implicit (not written)
- the `multihash` is written as is but is always a full (length 32) sha256 hash.

```text
cidv0 ::= <multihash-content-address>
```

### CIDv1

See the section: [How does it work?](#how-does-it-work)

```text
<cidv1> ::= <multibase-prefix><multicodec-cidv1><multicodec-content-type><multihash-content-address>
```

## Decoding Algorithm

To decode a CID, follow the following algorithm:

1. If it's a string (ASCII/UTF-8):
  * If it is 46 characters long and starts with `Qm...`, it's a CIDv0. Decode it as base58btc and continue to step 2.
  * Otherwise, decode it according to the multibase spec and:
    * If the first decoded byte is 0x12, return an error. CIDv0 CIDs may not be multibase encoded and there will be no CIDv18 (0x12 = 18) to prevent ambiguity with decoded CIDv0s.
    * Otherwise, you now have a binary CID. Continue to step 2.
2. Given a (binary) CID (`cid`):
   * If it's 34 bytes long with the leading bytes `[0x12, 0x20, ...]`, it's a CIDv0.
     * The CID's multihash is `cid`.
     * The CID's multicodec is DagProtobuf
     * The CID's version is 0.
   * Otherwise, let `N` be the first varint in `cid`. This is the CID's version.
     * If `N == 0x01` (CIDv1):
       * The CID's multicodec is the second varint in `cid`
       * The CID's multihash is the rest of the `cid` (after the second varint).
       * The CID's version is 1.
     * If `N == 0x02` (CIDv2), or `N == 0x03` (CIDv3), the CID version is reserved.
     * If `N` is equal to some other multicodec, the CID is malformed.

## Implementations

- [go-cid](https://github.com/ipfs/go-cid)
- [java-cid](https://github.com/ipld/java-cid)
- [js-multiformats](https://github.com/multiformats/js-multiformats)
- [rust-cid](https://github.com/multiformats/rust-cid)
- [py-multiformats-cid](https://github.com/pinnaculum/py-multiformats-cid)
- [elixir-cid](https://github.com/nocursor/ex-cid)
- [dart_cid](https://github.com/dwyl/dart_cid)
- [Add yours today!](https://github.com/multiformats/cid/edit/master/README.md)

## FAQ

> **Q. I have questions on multicodec, multibase, or multihash.**

Please check their repositories: [multicodec](https://github.com/multiformats/multicodec), [multibase](https://github.com/multiformats/multibase), [multihash](https://github.com/multiformats/multihash).

> **Q. Why does CID exist?**

We were using base58btc encoded multihashes in IPFS, and then we needed to switch formats to IPLD. We struggled with lots of problems of addressing data with different formats until we created CIDs. You can read the history of this format here: https://github.com/ipfs/specs/issues/130

> **Q. Is the use of multicodec similar to file extensions?**

Yes, kind of! like a file extension, the multicodec identifier establishes the format of the data. Unlike file extensions, these are in the middle of the identifier and not meant to be changed by users. There is also a short table of supported formats.

> **Q. What formats (multicodec codes) does CID support?**

We are figuring this out at this time. It will likely be a subset of [multicodecs](https://github.com/multiformats/multicodec/blob/master/table.csv) for secure distributed systems. So far, we want to address IPFS's UnixFS and raw blocks ([`dag-pb`](https://ipld.io/specs/codecs/dag-pb/spec/), [`raw`](https://www.iana.org/assignments/media-types/application/vnd.ipld.raw)), IPNS's [`libp2p-key`](https://github.com/libp2p/specs/blob/master/RFC/0001-text-peerid-cid.md), and IPLD's [`dag-json`](https://ipld.io/specs/codecs/dag-json/spec/)/[`dag-cbor`](https://ipld.io/specs/codecs/dag-cbor/spec/) formats.

> **Q. What is the process for updating CID specification (e.g., adding a new version)?**

CIDs are a well established standard. IPFS uses CIDs for content-addressing and IPNS.
Making changes to such key protocol requires a careful review which should include feedback from implementers and stakeholders across ecosystem.

Due to this, changes to CID specification MUST be submitted as an improvement proposal to [ipfs/specs](https://github.com/ipfs/specs/tree/main/IPIP) repository (PR with [IPIP document](https://github.com/ipfs/specs/blob/main/IPIP/0000-template.md)), and follow the IPIP process described there. 


## Maintainers

Captain: [@jbenet](https://github.com/jbenet).

## Contribute

Contributions welcome. Please check out [the issues](https://github.com/ipld/cid/issues).

Check out our [contributing document](https://github.com/ipld/ipld/blob/master/contributing.md) for more information on how we work, and about contributing in general. Please be aware that all interactions related to IPLD are subject to the IPFS [Code of Conduct](https://github.com/ipfs/community/blob/master/code-of-conduct.md).

Small note: If editing the README, please conform to the [standard-readme](https://github.com/RichardLitt/standard-readme) specification.

## License

This repository is only for documents. These are licensed under a [CC-BY 3.0 Unported](LICENSE) License © 2016 Protocol Labs Inc.
