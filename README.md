<h1 align="center">Security Advisory: Hard-coded Cloud Storage Credential in Cardo Update for Windows</h1>

CVE: Requested (MITRE CNA-LR), pending assignment

## Summary:

This document details a critical vulnerability in Cardo Update, the desktop firmware updater for Windows distributed by Cardo Systems. The shipped application contains a hard-coded credential that grants access to a cloud storage account operated by the vendor. Because the credential is embedded in the freely downloadable installer, anyone who obtains the installer can recover it and use it to read, modify, or delete data in that storage account over the internet. The specific location of the credential and the affected backend are withheld from this public document while coordinated disclosure is in progress; see section 6.

**1. Vulnerability Details**

Vulnerability Type:

   &nbsp;&nbsp;&nbsp;&nbsp;Use of Hard-coded Credentials (CWE-798)  
   &nbsp;&nbsp;&nbsp;&nbsp;Cleartext Storage of Sensitive Information (CWE-312)  
   &nbsp;&nbsp;&nbsp;&nbsp;Use of Hard-coded Cryptographic Key (CWE-321)  

**Vendor: Cardo Systems Ltd.**

**Affected Products:** Cardo Update, the desktop firmware updater application for Windows.

&nbsp;&nbsp;&nbsp;&nbsp;Affected Versions: 4.7.0.44542, and likely earlier 4.x builds.

&nbsp;&nbsp;&nbsp;&nbsp;Attack Vector: Remote. The credential is recovered from the public installer, then used against the storage account over the network.

&nbsp;&nbsp;&nbsp;&nbsp;Severity: CVSS v3.1 10.0 (Critical), AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H. CVSS v4.0 7.9 (High), CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:N/VI:N/VA:N/SC:H/SI:H/SA:H.

### 2. Technical Description

Cardo Update is an Electron desktop application that users download to update the firmware on their Cardo headsets. The shipped application contains a credential for a cloud storage account inside its own program code. The credential is a broadly scoped account credential, not a narrowly scoped or time-limited token, so it grants full read, write, and delete access across the entire storage account rather than access to a single resource.

The credential travels with every copy of the installer. The installer is distributed publicly from the vendor's website with no authentication, so recovering the credential requires only unpacking the installer and reading the application's own files. No access to the vendor's systems is needed to obtain it, and nothing prevents a copy that has already been downloaded from being read at any time.

The exact file, the credential value, and the name of the storage account are intentionally omitted from this public document. Publishing them before the vendor rotates the credential would expose a live secret. Full detail has been provided to the vendor privately.

### 3. Verification

This finding was identified through static analysis of the shipped, publicly distributed application only. The credential was not used to authenticate to the vendor's account, and no vendor data was accessed, altered, or deleted. No proof-of-concept or exploit code is published. The presence of the credential in the shipped build is sufficient to establish the finding.

### 4. Impact

Because the exposed credential is a full account credential, anyone who recovers it can act against the vendor's storage account with the same authority as the account owner:

   Confidentiality: read and download everything stored in the account, including any diagnostic data uploaded by users.

   Integrity: add or overwrite stored objects. Content that is served from the account to users can be replaced, and attacker-controlled files can be staged under a vendor-trusted domain.

   Availability: delete objects or entire containers, disrupting the service that depends on the account.

The full extent depends on what the account holds, which was not enumerated because doing so would require using the credential. At minimum the exposure covers the data the application uploads. At worst, if the account also serves content consumed by clients, it becomes a supply-chain foothold.

### 5. Suggested Mitigation

**Vendor-side:**

   Rotate the exposed account credential immediately. Treat the account as compromised until this is done. Rotation invalidates the credential that has already been distributed.

   Remove the credential from the client. Storage-account credentials should never ship in client-side code or installers.

   Re-architect the upload path so the client holds no storage credentials. An authenticated backend should issue short-lived, least-privilege, scoped tokens, or accept uploads through a server-side API.

   Audit the storage account's access logs for use of the credential that did not originate from vendor infrastructure during the exposure window.

   Review the data held in the account for anything that may have been exposed, and handle it per the applicable breach and privacy obligations.

**User-side:**

   No action is required beyond installing a fixed version once the vendor releases one. The exposure is of vendor-side data, not data on the user's own computer.

### 6. Disclosure

This issue was reported to Cardo Systems through their coordinated vulnerability disclosure channel on 2026-09-26, and a CVE identifier was requested from MITRE the same day. A 90-day disclosure window was proposed. The specific technical details are withheld from this document until the vendor confirms the credential has been rotated and removed from the client, or the disclosure window closes, whichever comes first. This document will be updated at that time with the full technical detail and the assigned CVE identifier.

Reported by Michael Cook (https://github.com/futileskills).