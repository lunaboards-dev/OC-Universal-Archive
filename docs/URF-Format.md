## Abstract

This document describes the URF, intended to make mass file exchange easier.

## Rationale

There are many competing methods of exchanging large amounts of files, and many are incomplete, such as TAR implementations, or proprietary, such as NeoPAKv1. With this in mind, a standard format will make file exchange less error prone.

## Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119).

All integers are unsigned arbitrary length integers as specified in the Document, unless otherwise specified.

All strings are encoded with UTF-8, prepended with an integer corresponding to the length of the string in bytes.

Character specifications such as `NULL` and `DC1` are part of US ASCII, unless otherwise specified.

## Concepts

  * Signature: Data at the beginning of the File, marking the File as a URF format archive, and specifying the version.
  * File Table: Data structure containing all file information.
  * Entry: Any data sub-structure in the File Table.
  * Entry Specifier byte: A byte describing how to decode the data contained in an Entry.
  * Object: A filesystem structure, i.e.. "file" or "directory".
  * Attributes: Data describing the size, offset, parent Object, and Object ID of an Object.
  * Extended Attributes: Data describing non-core attributes such as Permissions, Owner, Security, etc.
  * Producer: A program or process that generates data in this format.
  * Consumer: A program or process that consumes data in this format.

## Arbitrary length integers

Arbitrary length integers (ALIs) MAY be over 64-bits in precision. ALIs are little endian. For each byte, add the value of the first seven bits, shifted by 7 times the number of characters currently read bits, to the the read value and repeat until the 8th bit is 0.

## Signature

The Signature MUST be `URF` followed by a `DC1` character, or an unsigned 32-bit little endian integer equal to 1431455249. The next two bytes MUST be the version number, the first being the major version, the second being the minor version. The next two bytes MUST be `DC2` followed by a `NULL` character.

## File Table

The File Table MUST start after the Signature. This MUST contain all Entries, and MUST end with an EOH Entry. See EOH Entry.

## Entry

An Entry MUST start with an Entry Specifier byte. The Entry Specifier byte MUST be followed with an integer specifying the length. Data contained SHOULD be skipped if the Entry Specifier byte allows and the Entry Type is not understood.

## Entry Specifier byte

An Entry Specifier byte MUST have the 7th bit set to 1. If the Consumer comes across an entry with the 7th bit not set to 1, the Consumer MUST stop reading the file and raise a fatal error. The 6th byte specifies if the entry is critical. If the entry is critical and not understood, the Consumer should raise a fatal error; if the entry is non-critical, the Consumer SHOULD skip the entry and continue reading the file. If the 8th byte is set to 1, the Entry is a non-standard extension. Vendors MAY use this range for vendor-specific data.

## Filesystem Structure

Any Object MUST have a Parent ID and an Object ID. Object ID 0 is reserved for the root directory.

## File naming conventions

Object names MUST NOT contain any type of slash. Object names must also be of the 8.3 format, though the full name MAY be specified with Extended Attributes. Full names in Attributes MUST NOT contain any type of slash. 

## File offsets

File offsets are relative to the end of the File Table.

## Entry Type: File

The Entry Specifier byte MUST be `F`, and the data contained MUST be the name as a string, followed by the file offset and file size represented as an integer, then followed by the Object ID and Parent ID.

## Entry type: Directory

The Entry Specifier byte MUST be `F`, and the data contained MUST be the name as a string, followed by the Object ID and the Parent ID.

## Entry type: Extended Attributes

The Entry Specifier byte MUST be `x`, and the data contained MUST be the Object ID of the Entry the Entry is describing, followed by a four byte Attribute and the value.

Currently recognized attributes include:
  * `NAME`: The long name of the Object.
  * `PERM`: The POSIX-compatible permissions of the Object
  * `W32P`: Win32-compatible permissions of the Object, which override POSIX permissions.
  * `OTIM`: Creation time of the Object
  * `MTIM`: Modification time of the Object
  * `CTIM`: Metadata update time of the Object
  * `ATIM`: Access time of the Object

## Entry type: EOH

The Entry Specifier byte MUST be `Z`, and the data contained MUST be the offset required to reach the end of the file.

## Compressed URF naming convention

In an environment with long names, the file extension SHOULD be `urf` followed by a period (`.`) and the compression method (i.e. `gz`, `lzma`, `xz`, `deflate`)
In an environment with 8.3 names, the file extension MUST be one of the following:
  * `UMA` for LZMA
  * `UXZ` for XZ
  * `UGZ` for Gunzip
  * `UL4` for LZ4
  * `UB2` for BZip2
  
## TODO

Document is incomplete. Document should outline how to build the Filesystem structure, etc. Document should be checked for clarity and rewritten if needed.
