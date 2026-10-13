# Cardo Update for Windows: Hard-coded Azure Storage Account Key

**CVE:** Requested via MITRE CNA-LR (2026-10-13), pending assignment  
**Researcher:** Michael Cook ([@futileskills](https://github.com/futileskills))  
**Vendor:** Cardo Systems Ltd.  
**Product:** Cardo Update (Windows desktop firmware updater)  
**Affected version:** 4.7.0.44542, likely earlier 4.x builds  
**CWE:** CWE-798 (Use of Hard-coded Credentials), CWE-312 (Cleartext Storage of Sensitive Information)  
**CVSS v3.1:** 10.0 Critical (AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H)  
**CVSS v4.0:** 7.9 High (CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:N/VI:N/VA:N/SC:H/SI:H/SA:H)

## Summary

Cardo Update ships with a full Azure Storage account key baked into its application code. The installer is a public download on Cardo's website with no login required, so anyone who grabs it can pull the key out and get owner-level read, write, and delete access to Cardo's storage account.

I reported this to Cardo twice. Both reports were ignored and marked as spam, so I'm publishing without vendor coordination. The key is still live, so the key value, the storage account name, and where it sits in the app are left out of this writeup.

## Details

Cardo Update is an Electron app that riders use to update firmware on their Cardo headsets. Somewhere in the shipped app code is an Azure Storage connection string containing the account's shared key.

This isn't a scoped SAS token or anything time-limited. It's the account key itself, which means it works against every container and object in the account, not just whatever the app actually needs. Getting it out takes nothing more than unpacking the installer and reading the app's own files. No interaction with Cardo's systems is needed, and every copy of the installer that's already been downloaded has the key in it.

## Proof of Concept

I found this through static analysis of the public installer only. I did not use the key, did not connect to Cardo's storage account, and did not access, change, or delete any data.

**Installer examined:**

    Version:  4.7.0.44542
    SHA-256:  df46c09d67161adf43dd5c11b123a6340df31ebbd42f14cc9db4d75ec0effb7e
    Source:   Public download from cardosystems.com, no authentication

**What's in the build (redacted):**

    DefaultEndpointsProtocol=https;
    AccountName=[REDACTED];
    AccountKey=[REDACTED];
    EndpointSuffix=core.windows.net

**Hash of the full unredacted connection string:**

        SHA-256: 1557bd4ad5e9a1d5521892f7391cc67659c37aa1284b7566be33a17fc3e0da07

That hash lets Cardo confirm exactly which key was exposed, and lets me prove I had it as of this date, without putting the key itself out there.

## Impact

Anyone holding the key effectively owns the storage account:

- **Read:** download everything in it, including any diagnostic data the app uploads from users.
- **Write:** upload or overwrite files. If the account serves anything back to the app or to users, that content can be swapped out for something malicious under a Cardo-trusted domain.
- **Delete:** wipe objects or whole containers and break whatever depends on them.

I didn't enumerate the account's contents, because that would mean using the key. Worst case, if the account serves content to clients, this is a supply-chain problem, not just a data exposure.

## Recommendations for Cardo

1. Rotate the account key now and treat the account as compromised until you do. Rotating kills the copy that's already out in the wild.
2. Pull storage credentials out of the client entirely. Account keys should never ship in an installer.
3. Have the client request short-lived, scoped SAS tokens from an authenticated backend, or route uploads through a server-side API.
4. Check storage access logs for requests using this key that didn't come from your own infrastructure.
5. Review what was stored in the account and handle any exposed user data per your breach notification obligations.

Users don't need to do anything beyond updating once a fixed version is released. The exposure is on Cardo's side, not on users' machines.

## Disclosure Timeline

Cardo doesn't have a vulnerability disclosure program or a security contact, so I used their email support portal.

| Date | Event |
|---|---|
| 2026-09-26 | First report via Cardo Report Security Issue portal (case CES-US317) |
| 2026-10-12 | Second report via Cardo support portal (case ZTK25129259) |
| 2026-10-13 | Report ZTK25129259 closed as spam |
| 2026-10-13 | CVE requested through MITRE CNA-LR; Cardo notified of public disclosure via support portal (case ZTK25129507); advisory published |
