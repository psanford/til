


# TPM Notes


## Overview

TPMs have a number of different facilities that they support for different use cases. This document is intended to be a non-comprehensive set of notes on those facilities. These notes will focus on the TPM 2.0 api.


## Hardware Facilities

- NVRam

- Key Hierarchy

- PCR (Platform Configuration Registers)

### PCRs

Platform Configuration Registers are a set of registers intended to be used to measure and validate the state of the system's configuration. They are primarily used for Secure Boot.

The idea of the PCRs is that each stage of the boot process will perform a measurement over its state and update the relevant PCR. The PCR values cannot be set directly, but are instead updated using a hash function along with private data stored within the TPM. This makes the values unpredictable but consistent.

Certain functionality in the TPM can be locked to the state of the PCRs. Thus you can prevent the disk encryption key from being accessible unless the hardware is running a specific BIOS version and known good bootloader and OS. This is known as secrets being sealed or unsealed to one or more PCR values.

PCRs are numbered and have specific meaning in the Windows environment that other OSes have adopted.

| PCR Index | Purpose                         |
|-----------|---------------------------------|
| 0         | BIOS                            |
| 1         | BIOS Configuration              |
| 2         | Option ROMs                     |
| 3         | Option ROM Config               |
| 4         | Master Boot Record              |
| 5         | MBR Config                      |
| 6         | State Transitions / Wake Events |
| 7         | Platform mfgr specific          |
| 8-15      | Operating System specific       |
| 14        | Boot Authorities (Linux Grub)   |
| 16        | Debug                           |
| 23        | Application Support             |


For attestation purposes, you may want to know about previous states of PCRs prior to more data being mixed into them. The kernel keeps a log of the history of the PCRs so you can see the previous measurements:

```
/sys/kernel/security/tpm0/binary_bios_measurements
```

A go tool that can parse the measurement log: https://github.com/cshari-zededa/eve-tpm2-tools/

### NVRam

TPMs contain a small amount of non-volatile storage. This storage space is used by both the TPM itself as well as for user definable sections.

The TPM ships with its EK public key stored in NVRam at 0x01c0000a

| Index      | Stored Item                  |
|------------|------------------------------|
| 0x01c00002 | RSA 2048 EK Certificate      |
| 0x01c00003 | RSA 2048 EK Nonce            |
| 0x01c00004 | RSA 2048 EK Template         |
| 0x01c0000a | ECC NIST P256 EK Certificate |
| 0x01c0000b | ECC NIST P256 EK Nonce       |
| 0x01c0000c | ECC NIST P256 EK Template    |

Users can allocate NVRam indexes and store data into them. This can one of the following type:
- Ordinary: unstructured data
- Counter: monotonic counter
- Bit-Field: 64 bit, bit indexed
- Extend: Extend Index (can be used to make hybrid, flexible PCRs)


### Key Hierarchy

TPMs have a set of Key Hierarchies for performing cryptographic operations. Keys in these hierarchies are protected by the TPM (they can only perform operations within the TPM). They support both symmetric and asymmetric operations. TPM 2.0 supports both RSA and ECC asymmetric keys.

There are 3 persistent hierarchies:

- Platform
- Storage
- Endorsement

Fundamentally, these are 3 separate root keys from which child keys can be derived.

There is a 4th hierarchy called the Null Hierarchy. This is non-persistent and resets on reboots.


#### Platform Hierarchy

The Platform Hierarchy is under the control of the platform manufacturer. It is used for early boot verification.


#### Storage Hierarchy

The Storage Hierarchy is intended to be used by the owner (either the IT department or the end user). The storage hierarchy is intended for non-privacy sensitive operations. This means keys from this hierarchy could be used to uniquely identity the system.

#### Endorsement Hierarchy

The Endorsement Hierarchy is intended to be used by the owner (IT or end user) for privacy sensitive applications.

Keys under the Endorsement Hierarchy should not be able to be correlated back to the same TPM. If application A and application B create separate keys under the Endorsement Hierarchy, a thirdparty seeing those keys should not be able to tell if they are from the same TPM or not.

Certificates for keys under the Endorsement Hierarchy confirm that the key came from a genuine TPM from a certain manufacturer.

Note that if you have EK -> Singing Key -> Children Signing Keys, the Children Signing Keys can be correlated to the Signing Key and thus to the same TPM. "For this reason, primary keys in the endorsement hierarchy are typically encryption keys, not signing keys."


## References

TPM 101:
https://web.archive.org/web/20151028130757/http://tiw2013.cse.psu.edu/slides/tiw-2013-martin.pdf

A Root of Trust for Measurement:
https://webcache.googleusercontent.com/search?q=cache:JBax1SIvqwwJ:security.hsr.ch/mse/projects/2011_Root_of_Trust_for_Measurement.pdf%20&cd=1&hl=en&ct=clnk#25kAHUgoCw21h7xea1j1CsCQ

TCG EK Credential Profile
https://trustedcomputinggroup.org/wp-content/uploads/Credential_Profile_EK_V2.1_R12_PUBLIC_REVIEW.pdf

NVRam
https://link.springer.com/chapter/10.1007/978-1-4302-6584-9_11
