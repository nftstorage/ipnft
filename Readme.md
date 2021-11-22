# InterPlanetary NFT (IPNFT)

[![hackmd-github-sync-badge](https://hackmd.io/34u90DYHQ86zBFclQUKmVw/badge)](https://hackmd.io/34u90DYHQ86zBFclQUKmVw)


## Simple Summary

A standard encoding and addressing for non-fungible tokens (NFTs) for upholding chain immutablity gurantees off chain.


## Abstract

Following standard defines a standard encoding for [ERC-721][] Non-Fungible Token assets & metadata (addressed by `tokenURI`) which is compatible with the existing specification, while it attempts to address some of the issues found in the wild.

## Motivation

[ERC-721][]: Non-Fungible Token Standard defines `tokenURI` as a distinct Uniform Resource Identifier (URI) for a given asset where it **may** point to a JSON file that conforms to the "ERC721 Metadata JSON Schema".

Metadata schema in turn defines `name`, `description` and `image` string fields, where `image` is a URI pointing to a resource with mime type `image/*` representing the asset to which this NFT represents.

Loose requirements lead to number of problems in the wild:

- **Autheticity** - Many NFTs use URIs to web 2.0 host that
    1. Can go down and make assets inaccessible (404)
    2. Serve unauthentic resource for maliciously or unintentionally.
- **Ambiguity** - While specification only defines `image` field for the asset, in practice many new fields with URI are used. This introduces ambiguity in which URIs are part of NFT asset and which are not.


IPNFT specification defines [IPLD][] format for encoding NFT metadata and all the assets into a hash linked [DAG][] which can be addressed via `ipfs://` URLs and used in `tokenURI` in ERC-721 compatible way while addressing outlined problems as follows:

- **Autheticity** - Content addressing of IPFS guarantees that content can be retreived from arbitrary nodes in the network and that content is verifiably be authentic.
- **Ambiguity** - All sssets included in the DAG are part of NFT any URL inside the metadata is not.

## Specification

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this document are to be interpreted as described in RFC 2119.

## Asset Link

MUST be an IPLD link to [DAG-PB] directory, which SHOULD contain exactly one named file.

## IPNFT

Every IPNFT node MUST be [DAG-CBOR][] encoded data structure which MUST have:

1. `type` property with `"nft"` value.
1. `name` property with a string vaule.
1. `description` property with a string value.
1. Have `image` [asset link](#Asset-Link).


IPFNT data structure MAY contain arbitrary other properties however it is RECOMMENDED to put those under `properties` as per "ERC721 Metadata JSON Schema".

All NFT assets MUST be included in IPNFT data structure as [asset links](#Asset-Link) which MAY be any top level property except (`type`, `name`, `description`, `image`, `properties`, `metadata.json`) or any nested property.


### `metadata.json` content

It is RECOMMENDED for IPNFT node to have a `metadata.json` IPLD link to a JSON source in [DAG-PB][] or RAW encoding mirroring IPNFT data structure with following caveats:

1. `type` property is OPTIONAL and can be omitted. If present it MUST have value `"nft"`.
2. Asset links MUST be substituted for corresponding `ipfs://{asset_cid}/file.png` URLs in string format _(substitute `file.png` with actual file name)_.
3. SHOULD NOT have a property named `metadata.json` so it does not diverge from IPNFT data structure.


## Caveats

1. IPNFT root block should not exceed 1MiB in size.

   > Larger blocks are not well supported by IPFS. If CBOR encoded metadata still exceeds this block limit author shoud consider refatoring metadata into multiple block e.g. taking JSON substructure, encoding it in DAG-CBOR and linking to it from metadata.


[ERC-721]:https://eips.ethereum.org/EIPS/eip-721
[IPLD]:https://ipld.io/
[DAG]:https://en.wikipedia.org/wiki/Directed_acyclic_graph
[DAG-CBOR]:https://ipld.io/docs/codecs/known/dag-cbor/
[DAG-PB]:https://ipld.io/docs/codecs/known/dag-pb/