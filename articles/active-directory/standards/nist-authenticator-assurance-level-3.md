---
title: Achieve NIST AAL3 by using Azure Active Directory
description: This article provides guidance on achieving NIST authenticator assurance level 3 (AAL3) by using Azure Active Directory.
services: active-directory 
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: how-to
author: gargi-sinha
ms.author: gasinh
manager: martinco
ms.reviewer: martinco
ms.date: 09/13/2022
ms.custom: it-pro
ms.collection: M365-identity-device-management
---

# NIST authenticator assurance level 3 by using Azure Active Directory

Use the information in this article for National Institute of Standards and Technology (NIST) authenticator assurance level 3 (AAL3). 

Before obtaining AAL2, you can review the following resources:

* [NIST overview](nist-overview.md): Understand AAL levels
* [Authentication basics](nist-authentication-basics.md): Terminology and authentication types
* [NIST authenticator types](nist-authenticator-types.md): Authenticator types
* [NIST AALs](nist-about-authenticator-assurance-levels.md): AAL components and Azure Active Directory (Azure AD) authentication methods

## Permitted authenticator types

Use Microsoft authentication methods to meet required NIST authenticator types. 

| Azure AD authentication methods| NIST authenticator type |
| - | -|
| **Recommended methods**|    |
| FIDO 2 security key, or <br> Smart card (Active Directory Federation Services), or <br> Windows Hello for Business with hardware TPM| Multi-factor cryptographic hardware |
| **Additional methods**|   |
| Password and <br> Hybrid Azure AD joined with hardware TPM or, <br> Azure AD joined with hardware TPM)| Memorized secret and <br> single-factor cryptographic hardware |
| Password and <br> single-factor one-time password OTP hardware, from an OTP manufacturer and <br> Hybrid Azure AD joined with software TPM or <br> Azure AD joined with software TPM or <br> [Compliant managed device](/mem/intune/protect/device-compliance-get-started))| Memorized secret and <br> single-factor OTP hardware and <br> single-factor cryptographic software |

### Recommendations 

For AAL3, we recommend using a multi-factor cryptographic hardware authenticator. Passwordless authentication eliminates the greatest attack surface, the password. Users have a streamlined authentication method. If your organization is cloud based, we recommend you use FIDO 2 security keys.

> [!NOTE]
> Windows Hello for Business is not validated at the required FIPS 140 Security Level. Federal customers should conduct risk assessments and evaluation before accepting this service as AAL3.

For guidance, see [Plan a passwordless authentication deployment in Azure Active Directory](../authentication/howto-authentication-passwordless-deployment.md). See also [Windows Hello for Business deployment guide](/windows/security/identity-protection/hello-for-business/hello-deployment-guide).

## FIPS 140 validation

### Verifier requirements

Azure AD uses the Windows FIPS 140 Level 1 overall validated cryptographic module for its authentication cryptographic operations, making Azure AD a compliant verifier.

### Authenticator requirements

Single-factor and multi-factor cryptographic hardware authenticator requirements. 

#### Single-factor cryptographic hardware

Authenticators are required to be:

* FIPS 140 Level 1 Overall, or higher

* FIPS 140 Level 3 Physical Security, or higher

Azure AD joined and Hybrid Azure AD joined devices meet this requirement when: 

* You run [Windows in a FIPS-140 approved mode](/windows/security/threat-protection/fips-140-validation)

* On a machine with a TPM that's FIPS 140 Level 1 Overall, or higher, with FIPS 140 Level 3 Physical Security

  * Find compliant TPMs: search for Trusted Platform Module and TPM on [Cryptographic Module Validation Program](https://csrc.nist.gov/Projects/cryptographic-module-validation-program/validated-modules/Search).

Consult your mobile device vendor to learn about their adherence with FIPS 140.

#### Multi-factor cryptographic hardware

Authenticators are required to be: 

* FIPS 140 Level 2 Overall, or higher

* FIPS 140 Level 3 Physical Security, or higher

FIDO 2 security keys, smart cards, and Windows Hello for Business can help you meet these requirements.

* FIDO2 key providers are in FIPS certification. We recommend you review the list of [supported FIDO2 key vendors](../authentication/concept-authentication-passwordless.md#fido2-security-key-providers). Consult with your provider for current FIPS validation status.

* Smart cards are a proven technology. Multiple vendor products meet FIPS requirements.

  * Learn more on [Cryptographic Module Validation Program](https://csrc.nist.gov/Projects/cryptographic-module-validation-program/validated-modules/Search)

**Windows Hello for Business**

FIPS 140 requires the cryptographic boundary, including software, firmware, and hardware, to be in scope for evaluation. Windows operating systems can be paired with thousands of combinations of hardware. Microsoft can't maintain FIPS certifications for each combination. 

Evaluate the following component certifications in your risk assessment of using Windows Hello for Business as an AAL3 authenticator:

* **Windows 10 and Windows Server** use the [US Government Approved Protection Profile for General Purpose Operating Systems Version 4.2.1](https://www.niap-ccevs.org/Profile/Info.cfm?PPID=442&id=442) from the National Information Assurance Partnership (NIAP). This organization oversees a national program to evaluate commercial off-the-shelf (COTS) information technology products for conformance with the international Common Criteria. 

* **Windows Cryptographic Library** [has FIPS Level 1 Overall in the NIST Cryptographic Module Validation Program](https://csrc.nist.gov/Projects/cryptographic-module-validation-program/Certificate/3544) (CMVP), a joint effort between NIST and the Canadian Center for Cyber Security. This organization validates cryptographic modules against FIPS standards. 

* Choose a **Trusted Platform Module (TPM)** that's FIPS 140 Level 2 Overall, and FIPS 140 Level 3 Physical Security. Your organization ensures hardware TPM meets the AAL level requirements you want.   

To determine the TPMs that meet current standards, go to [NIST Computer Security Resource Center Cryptographic Module Validation Program](https://csrc.nist.gov/Projects/cryptographic-module-validation-program/validated-modules/Search). In the **Module Name** box, enter **Trusted Platform Module** for a list of hardware TPMs that meet standards.

## Reauthentication 

For AAL3, NIST requirements are reauthentication every 12 hours, regardless of user activity. Reauthentication is required after a period of inactivity 15 minutes or longer. Presenting both factors is required.

To meet the requirement for reauthentication, regardless of user activity, Microsoft recommends configuring [user sign-in frequency](../conditional-access/howto-conditional-access-session-lifetime.md) to 12 hours. 

Use NIST for compensating controls to confirm subscriber presence:

* Set a session inactivity time out of 15 minutes: Lock the device at the OS level by using Microsoft Endpoint Configuration Manager, Group Policy Object (GPO), or Intune. For the subscriber to unlock it, require local authentication.

* Set timeout, regardless of activity, by running a scheduled task using Configuration Manager, GPO, or Intune. Lock the machine after 12 hours, regardless of activity.

## Man-in-the-middle resistance 

Communications between the claimant and Azure AD are over an authenticated, protected channel for resistance to man-in-the-middle (MitM) attacks. This configuration satisfies the MitM resistance requirements for AAL1, AAL2, and AAL3.

## Verifier impersonation resistance

Azure AD authentication methods that meet AAL3 use cryptographic authenticators that bind the authenticator output to the session being authenticated. The methods use a private key controlled by the claimant. The public key is known to the verifier. This configuration satisfies the verifier-impersonation resistance requirements for AAL3.

## Verifier compromise resistance

All Azure AD authentication methods that meet AAL3:

- Use a cryptographic authenticator that requires the verifier store a public key corresponding to a private key held by the authenticator
- Store the expected authenticator output by using FIPS-140 validated hash algorithms

For more information, see [Azure AD Data Security Considerations](https://aka.ms/AADDataWhitepaper).

## Replay resistance

Azure AD authentication methods that meet AAL3 use nonce or challenges. These methods are resistant to replay attacks because the verifier can detect replayed authentication transactions. Such transactions won't contain the needed nonce or timeliness data.

## Authentication intent

Authentication makes it more difficult for directly connected physical authenticators, like multi-factor cryptographic devices, to be used without the subject's knowledge. For example, by malware on the endpoint.

Use NIST compensating controls for mitigating malware risk. Any Intune-compliant device that runs Windows Defender System Guard and Windows Defender ATP meets this mitigation requirement.

## Next steps 

[NIST overview](nist-overview.md)

[Learn about AALs](nist-about-authenticator-assurance-levels.md)

[Authentication basics](nist-authentication-basics.md)

[NIST authenticator types](nist-authenticator-types.md)

[Achieving NIST AAL1 by using Azure AD](nist-authenticator-assurance-level-1.md)

[Achieving NIST AAL2 by using Azure AD](nist-authenticator-assurance-level-2.md)
