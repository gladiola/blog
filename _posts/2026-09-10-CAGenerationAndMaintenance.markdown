---
title:  "CAGenerationAndMaintenance Builds an Air-Gapped CA Workflow"
date:   2026-09-10 13:50:00
description: Reviewing CAGenerationAndMaintenance, a shell-script toolkit for operating an offline OpenBSD certificate authority and synchronizing status data by USB.
---

[CAGenerationAndMaintenance](https://github.com/gladiola/CAGenerationAndMaintenance) is one of those repositories whose name is plain, but accurate. It is a shell-script toolkit for running an **offline, air-gapped certificate authority** on OpenBSD and then moving the revocation and responder data to an online OCSP system by USB.

That workflow is the interesting part. The repository is not just about minting certificates. It is about separating the most sensitive CA operations from the network while still keeping revocation information usable by online services.

# What the repository provides

The project organizes the CA lifecycle into a set of focused scripts.

It includes scripts to:

- initialize a **root CA**,
- create an **intermediate CA**,
- issue **server certificates**,
- issue **client certificates**,
- revoke certificates and regenerate **CRLs**,
- export CA artifacts to a USB drive,
- and import those updates onto an OCSP server machine.

This is not a toy collection of certificate commands pasted into a README. It is an opinionated workflow for running an offline CA with repeatable operational steps.

# What stands out

The strongest design choice is the explicit air-gap architecture.

The repository assumes:

- one **offline OpenBSD CA machine**,
- one **networked OCSP server machine**,
- and a **USB transfer path** between them.

That model gives the sensitive signing environment a clear boundary. Private CA keys stay off the network, while revocation and responder artifacts can still be published where clients need them.

The repository also pays attention to practical details that matter in real operations:

- OpenSSL configuration templates,
- checksum verification during USB transfer,
- naming conventions for keys, certificates, and bundles,
- CRL renewal guidance,
- and integration with a separate OpenBSD OCSP server service.

# Why the project is useful

Many PKI writeups stop at "generate a CA" and leave the reader alone with the hard parts: maintenance, rotation, revocation, and safe publication of status. This repository is better than that. It treats certificate authority work as an ongoing operational process.

That makes it useful for labs, internal enterprise PKI, security-conscious small deployments, or anyone who wants a more inspectable certificate workflow than a commercial black box.

# Who should read it

CAGenerationAndMaintenance seems best for readers who want:

- an OpenBSD-friendly offline CA workflow,
- repeatable certificate issuance and revocation steps,
- a physical transfer model for air-gapped environments,
- or a companion process for an OCSP responder deployment.

It is especially compelling when read alongside the OCSP server repository it is designed to feed.

# Final thoughts

What I like most about [CAGenerationAndMaintenance](https://github.com/gladiola/CAGenerationAndMaintenance) is that it respects the security boundary it is trying to protect. The repository assumes the CA should stay offline, then builds the rest of the process around that fact instead of weakening it for convenience.

If you want a concrete, script-driven model for operating an air-gapped OpenBSD CA, this repository is a solid example.
