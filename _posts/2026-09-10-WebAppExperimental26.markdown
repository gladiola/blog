---
title:  "Introducing WebAppExperimental26"
date:   2026-09-10 08:30:00
description: A look at WebAppExperimental26, an ASP.NET Core 9 web app that combines identity, certificate, security-header, localization, and multi-cloud integration experiments in one repository.
---

[WebAppExperimental26](https://github.com/gladiola/WebAppExperimental26) is a useful kind of experiment: not a toy app, not a polished product brochure, but a working ASP.NET Core 9 web application that tries to gather several hard web-programming problems into one place. Identity, certificates, TLS, secrets management, storage integrations, localization, and HTTP hardening all show up in the same codebase.

That makes the repository interesting for two different reasons. First, it is a program with a real shape: a Razor Pages application with authentication, protected routes, configuration, and supporting services. Second, it is a study repository: one that exposes how those moving parts can be wired together across Azure, AWS, and Google Cloud without pretending every feature is equally complete or equally production-ready.

# What the program is

At its core, WebAppExperimental26 is an ASP.NET Core 9 web app built on Razor Pages and standard dependency injection. It uses Microsoft Identity for Azure AD sign-in, protects application routes, and layers on security controls that are often discussed separately but rarely shown together in one small, inspectable project.

The program is especially focused on secure web deployment concerns. It can authenticate users with Azure AD, and it also includes optional identity pathways for AWS Cognito and GCP Identity Platform. It can require mutual TLS for client certificates. It can load a server certificate from Azure Key Vault instead of depending on a certificate file committed or deployed alongside the app. And it can attach a hardened set of HTTP response headers, including a Content Security Policy built around per-request nonces.

In other words, this is the kind of project you open when you want to see several enterprise-flavored concerns living side by side in one place.

# What stands out in the feature set

The strongest part of WebAppExperimental26 is how broad its security surface is without collapsing into total chaos.

- **Authentication and authorization:** Azure AD is the central sign-in path, with authorization protecting the experimental area of the site. The repository also includes OpenID Connect integrations for AWS Cognito and Google's identity platform.
- **Certificate-driven security:** The application can enforce mutual TLS and is designed to fetch its HTTPS certificate material from Azure Key Vault.
- **HTTP hardening:** CSP nonces, HSTS, anti-framing headers, content-type protections, cache restrictions, and related headers are all treated as first-class concerns instead of afterthoughts.
- **Cloud-backed services:** Azure Blob Storage and Cosmos DB appear alongside Amazon DynamoDB, AWS Secrets Manager, Google Cloud Secret Manager, and Google Cloud Firestore.
- **Localization:** The project supports 25 languages and even accounts for right-to-left layout behavior when Arabic is selected.
- **PII-aware logging and feature flags:** Sensitive logging is treated carefully, and major subsystems are controlled through feature flags so the application can expose experimental capabilities without forcing them all on at once.

Just as important, the repository is honest about which parts are templates or stubs. Some of the AWS, GCP, and OCSP plumbing is intentionally scaffolded rather than presented as finished production logic. That honesty is a strength. It makes the repo easier to trust as a learning artifact because it distinguishes between implemented behavior and extension points.

# Why the repository itself is worth browsing

The repo is not only "an app folder with some code in it." It is organized as a working project plus an explanation set.

At the top level you get the solution file, the main web project, a test project, documentation, scripts, and a dedicated `skills` directory. That `skills` directory is one of the most interesting parts of the repository because it breaks the project into seven capability areas:

1. ASP.NET Core and Razor Pages
2. Authentication and authorization
3. HTTP security hardening
4. Certificate and PKI management
5. Azure integration
6. Multi-cloud secrets and storage
7. Localization and documentation

That framing turns the repo into more than source code. It becomes a guided map of the competencies behind the source code.

The documentation is also unusually broad. The README does not merely say how to build the app; it explains feature flags, deployment expectations, prerequisites, and security notes. On top of that, the docs are translated into a large set of languages, which matches the application's own localization goals. For anyone interested in how software, documentation, and accessibility discipline can reinforce one another, that is a nice touch.

# Who this project is for

WebAppExperimental26 will be most useful to readers who want examples of secure-by-design web application thinking in ASP.NET Core. If you are trying to understand how identity, certificate handling, CSP nonces, Key Vault usage, and cloud service abstractions can coexist in one codebase, this repository gives you a compact place to start.

It is also useful for people who like to learn from structure rather than snippets. The feature flags, service boundaries, multilingual docs, and skill guides all help explain not only what the program does, but how the repository invites future extension.

# Where to start in the repo

If you open the repository for the first time, the best entry points seem to be:

- the main `README.md` for the overall feature map,
- `Program.cs` for the application pipeline and service registration,
- the `skills/` directory for a capability-by-capability tour,
- the `docs/` directory for translated documentation,
- and the test project for clues about what behavior is being verified.

That path gives a quick sense of both the application and the intent behind it.

# Final thoughts

What I like most about WebAppExperimental26 is that it treats modern web security and cloud integration as things to be explored concretely. Instead of talking abstractly about mutual TLS, CSP, OpenID Connect, secret managers, localization, and storage services, it puts them together in one repository and lets the reader inspect the seams.

If you want a clean, approachable program that doubles as a catalog of secure ASP.NET Core ideas, [WebAppExperimental26](https://github.com/gladiola/WebAppExperimental26) is worth your time.
