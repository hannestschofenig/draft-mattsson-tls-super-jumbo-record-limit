---
title: Super Jumbo Record Size Limit Extension for TLS
abbrev: Super Jumbo TLS Records
docname: draft-mattsson-tls-super-jumbo-record-limit-00

ipr: trust200902
cat: std

coding: utf-8
pi: # can use array (if all yes) or hash here
  toc: yes
  sortrefs: yes
  symrefs: yes
  tocdepth: 2

author:
      -
        ins: J. Preuß Mattsson
        name: John Preuß Mattsson
        org: Ericsson
        email: john.mattsson@ericsson.com
        
normative:

  RFC2119:
  RFC5246:
  RFC8174:
  RFC8446:
  RFC8447:
  RFC8449:

informative:

  RFC6083:
  RFC6347:
  RFC8201:
  I-D.ietf-tls-dtls13:

--- abstract

An extension "super_jumbo_record_size_limit" to Transport Layer Security (TLS) is defined that allows endpoints to negotiate a 2^16 bytes maximum size of protected records. This is larger than the default limit of around 2^14 bytes.

--- middle

# Introduction

The records in all version of TLS records has an uint16 length field that could theoretically allow records 65536 octets in size. TLS does however have a lower protocol-defined limit for maximum plaintext record size. For TLS 1.2 {{RFC5246}}, that limit is 2^14 = 16384 octets. TLS 1.3 {{RFC8446}} uses a limit of 2^14 + 1 = 16385 octets. In addition, TLS 1.2 allow expansion from compression and protection up to 2048 octets (though typically this expansion is only 16 octets).  TLS 1.3 reduces the allowance for expansion to 256 octets.

The "record_size_limit" extension {{RFC8449}} enables endpoints to negotiate a lower limit for the maximum plaintext record size, but does not allow endpoints to increase the limits enforced by TLS 1.3 {{RFC8446}}, TLS 1.2 {{RFC5246}}, DTLS 1.3 {{I-D.ietf-tls-dtls13}}, and DTLS 1.2 {{RFC6347}}.

In some use cases such as DTLS over SCTP {{RFC6083}} the 2^14 bytes limit is a severe limitation. DTLS over SCTP is for example used in 3GPP networks to provide end-to-end-protection. Some SCTP messages are already exceeding 16 kB and this problem is expected to grow over time as messages get bigger.

This document defines a "super_jumbo_record_size_limit" extension ({{ex}}). The extension allows endpoints to negotiate a 2^16 bytes maximum size of protected records, which is larger than the default limit of 2^14 bytes. This extension is valid in all versions of TLS.

# Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.

# The "super_jumbo_record_size_limit" Extension {#ex}

The "super_jumbo_record_size_limit" extension does not have any ExtensionData. When the "super_jumbo_record_size_limit" extension is negotiated, an endpoint MUST be prepared to accept protected records with ciphertexts of length 2^16 bytes and protected record with plaintext of length 2^16 - the allowed expansion. The maximum length of a protected record plaintext is therefore 2^16 - 2^11 = 63488 octets in TLS 1.2 and 2^16 - 2^8 = 65280 octets in TLS 1.3. Unprotected messages are still subject to the lower default limits in TLS 1.2 and 1.3.

The "super_jumbo_record_size_limit" extension MUST NOT be negotiated together with the "record_size_limit" extension or the "max_fragment_length" extension. A client MUST treat receipt
of "super_jumbo_record_size_limit" together with "record_size_limit" or "max_fragment_length" as a fatal error, and it SHOULD generate an "illegal_parameter" alert.

In TLS 1.3, the server sends the "super_jumbo_record_size_limit" extension in the EncryptedExtensions message.

During renegotiation or resumption, the record size limit is renegotiated.  Records are subject to the limits that were set in the handshake that produces the keys that are used to protect those records.  This admits the possibility that the extension might not be negotiated when a connection is renegotiated or resumed.

For DTLS 1.3 {{I-D.ietf-tls-dtls13}} or DTLS 1.2 {{RFC6347}} over UDP or DCCP, the Path Maximum Transmission Unit (PMTU) also limits the size of records.  The record size limit does not affect PMTU discovery and SHOULD be set independently. The record size limit is fixed during the handshake and so should be set based on constraints at the endpoint and not based on the current network environment. In comparison, the PMTU is determined by the network path and can change dynamically over time. See PMTU {{RFC8201}} and Section 4.1 of DTLS 1.3 {{I-D.ietf-tls-dtls13}} for more detail on PMTU discovery. For DTLS over TCP or SCTP, which automatically fragment and reassemble datagrams, there is no PMTU limitation.

# AEAD Limits

The maximum record size limit is an input to the AEAD limits calculations in TLS 1.3 {{RFC8446}} and DTLS 1.3 {{I-D.ietf-tls-dtls13}}. Increasing the maximum record size to 2^16 bytes while keeping the same confidentiality and integrity advantage therefore requires lower AEAD limits. When the "super_jumbo_record_size_limit" has been negotiated, existing AEAD limits shall be decreased by a factor of 4. For example, when AES-CGM is used in TLS 1.3 {{RFC8446}} with a 64 kB record limit, only 2^22.5 records (about 6 million) may be encrypted on a given connection.

# Security Considerations

Large record sizes might require more memory allocation for senders and receivers. Large record sizes also means that more processing is done before verification of non-authentic records fails.

# IANA Considerations

This document registers the "super_jumbo_record_size_limit" extension in the "TLS ExtensionType Values" registry established in {{RFC5246}}. The "super_jumbo_record_size_limit" extension has been assigned a code point of TBD. The IANA registry {{RFC8447}} \[\[will list\|lists\]\] this extension as "Recommended" (i.e., "Y") and indicates that it may appear in the ClientHello (CH) or EncryptedExtensions (EE) messages in TLS 1.3 {{RFC8446}}.

--- back

# Acknowledgments
{: numbered="no"}

TDB

--- fluff