# Network File Shares and Permissions

**Windows Server • SMB file shares • Active Directory security groups • permission validation**

## Overview

Built and tested SMB file shares in an Azure-hosted Windows Active Directory lab. Configured four folders with different sharing permissions, validated read-only, writable, and restricted access from a standard domain user, and used an Active Directory security group to authorize access to an accounting share.

## Environment

| Component | Configuration |
| --- | --- |
| Domain | `mydomain.com` |
| Domain controller and file server | `DC-1` — Windows Server VM |
| Domain-joined workstation | `Client-1` — Windows client VM |
| Test account | `MYDOMAIN\bame.siv` — standard domain user |
| Administration tools | Active Directory Users and Computers (ADUC), File Explorer, Windows folder sharing |
| Security group | `ACCOUNTANTS` — Global / Security group in `_GROUPS` |
| Network protocol | SMB via UNC paths (`\\DC-1\...`) |

## Sharing permissions and test results

| Share | Share permission configured | Observed result for the standard domain user |
| --- | --- | --- |
| `\\DC-1\read-access` | `Domain Users`: Read | Folder accessible; file creation denied |
| `\\DC-1\write-access` | `Domain Users`: Read/Write | Test text file created |
| `\\DC-1\no-access` | `Domain Admins`: Read/Write | Access denied |
| `\\DC-1\accounting` | `ACCOUNTANTS`: Read/Write | Denied before group membership; permitted after adding the user and signing in again |

> **Permission note:** Windows effective file access also depends on NTFS permissions; a share-level grant cannot override a more restrictive NTFS setting. This walkthrough shows the sharing settings and the outcomes observed during the tests, not a separate NTFS permission audit.

## Screenshot walkthrough

The following **30 screenshots** are embedded directly in this README, in the order supplied, with no collapsed/expandable sections.


### Set up domain sessions and the four shared folders


**01. Administrator account on DC-1**  
Signed into the Windows Server domain controller using the administrative domain account.

![01 — Administrator account on DC-1](screenshots/01-dc1-domain-admin-login.jpg)


**02. Standard domain user on Client-1**  
Verified the separate, non-administrative domain user used for the permission tests.

![02 — Standard domain user on Client-1](screenshots/02-client1-standard-user-login.jpg)


**03. Four test folders on the server**  
Prepared `read-access`, `write-access`, `no-access`, and `accounting` on the `C:` drive.

![03 — Four test folders on the server](screenshots/03-create-four-test-folders.jpg)


### Configure read, write, and restricted sharing permissions


**04. Open read-access sharing settings**  
Reviewed the `read-access` folder’s Sharing tab before configuring access.

![04 — Open read-access sharing settings](screenshots/04-read-share-properties.jpg)


**05. Grant Domain Users read-only sharing**  
Assigned the `Domain Users` group **Read** sharing permissions.

![05 — Grant Domain Users read-only sharing](screenshots/05-domain-users-read-permission.jpg)


**06. Confirm read-access share**  
Verified the network path `\\DC-1\read-access` was published.

![06 — Confirm read-access share](screenshots/06-read-share-confirmed.jpg)


**07. Configure the writable share**  
Assigned `Domain Users` **Read/Write** permissions on the `write-access` share.

![07 — Configure the writable share](screenshots/07-domain-users-write-permission.jpg)


**08. Confirm write-access share**  
Verified the network path `\\DC-1\write-access` was published.

![08 — Confirm write-access share](screenshots/08-write-share-confirmed.jpg)


**09. Restrict a share to administrators**  
Selected `Domain Admins` for the `no-access` share rather than general domain users.

![09 — Restrict a share to administrators](screenshots/09-domain-admins-restricted-permission.jpg)


**10. Confirm restricted share creation**  
Verified that `\\DC-1\no-access` was available as a network share.

![10 — Confirm restricted share creation](screenshots/10-restricted-share-confirmed.jpg)


### Test access from the domain-joined client


**11. Connect from Client-1 using a UNC path**  
Entered `\\DC-1` in File Explorer to browse the domain controller’s network shares.

![11 — Connect from Client-1 using a UNC path](screenshots/11-client-unc-navigation.jpg)


**12. Review published network shares**  
Viewed `read-access`, `write-access`, and `no-access` from Client-1 alongside Windows system shares.

![12 — Review published network shares](screenshots/12-client-network-shares.jpg)


**13. Read-only share prevents a write**  
Confirmed that a standard user could open `read-access` but was denied a file-creation operation.

![13 — Read-only share prevents a write](screenshots/13-read-share-write-denied.jpg)


**14. Write permissions permit file creation**  
Created a test text file in `write-access` to demonstrate write capability.

![14 — Write permissions permit file creation](screenshots/14-write-share-file-created.jpg)


**15. Restricted share denies standard-user access**  
Confirmed an **access denied** message when attempting to open `no-access`.

![15 — Restricted share denies standard-user access](screenshots/15-restricted-share-access-denied.jpg)


### Create an Active Directory security group


**16. Launch Active Directory Users and Computers**  
Opened ADUC on DC-1 to configure group-based access.

![16 — Launch Active Directory Users and Computers](screenshots/16-launch-aduc.jpg)


**17. Choose a new organizational unit**  
Used ADUC to create a dedicated container for security groups.

![17 — Choose a new organizational unit](screenshots/17-create-organizational-unit-menu.jpg)


**18. Define the _GROUPS organizational unit**  
Entered `_GROUPS` and retained protection against accidental deletion.

![18 — Define the _GROUPS organizational unit](screenshots/18-configure-groups-ou.jpg)


**19. Start creating an Active Directory group**  
Selected **New → Group** inside `_GROUPS`.

![19 — Start creating an Active Directory group](screenshots/19-new-security-group-menu.jpg)


**20. Configure the ACCOUNTANTS group**  
Created `ACCOUNTANTS` as a **Global / Security** group.

![20 — Configure the ACCOUNTANTS group](screenshots/20-configure-accountants-global-security.jpg)


**21. Verify ACCOUNTANTS exists**  
Confirmed the security group was present in the `_GROUPS` organizational unit.

![21 — Verify ACCOUNTANTS exists](screenshots/21-verify-accountants-group.jpg)


### Configure the accounting share and verify initial denial


**22. Review the file-server folder structure**  
Located the original `accounting` folder with the other test shares.

![22 — Review the file-server folder structure](screenshots/22-review-accounting-folder.jpg)


**23. Grant ACCOUNTANTS read/write share access**  
Assigned the `ACCOUNTANTS` group **Read/Write** sharing permission for `accounting`.

![23 — Grant ACCOUNTANTS read/write share access](screenshots/23-assign-accountants-read-write.jpg)


**24. Publish the accounting share**  
Confirmed the network share `\\DC-1\accounting` was created.

![24 — Publish the accounting share](screenshots/24-accounting-share-created.jpg)


**25. Verify access denied before membership**  
Demonstrated that the standard user could not open the `accounting` share before joining the group.

![25 — Verify access denied before membership](screenshots/25-accounting-access-denied.jpg)


### Add group membership and verify access


**26. Locate the domain user for membership**  
Selected `bame.siv` for addition to the `ACCOUNTANTS` security group.

![26 — Locate the domain user for membership](screenshots/26-select-user-for-group.jpg)


**27. Confirm ACCOUNTANTS group membership**  
Verified `bame.siv` appears on the **Members** tab of `ACCOUNTANTS`.

![27 — Confirm ACCOUNTANTS group membership](screenshots/27-verify-accountants-membership.jpg)


**28. Reconnect with the standard domain account**  
Reauthenticated to Client-1 with the `mydomain.com\bame.siv` account.

![28 — Reconnect with the standard domain account](screenshots/28-reconnect-client-user.jpg)


**29. Verify the client sign-in context**  
Confirmed that Client-1 was now signed in using `MYDOMAIN\bame.siv`.

![29 — Verify the client sign-in context](screenshots/29-verify-standard-user-login.jpg)


**30. Verify accounting share write access**  
Opened `accounting` and created a file after group membership was updated.

![30 — Verify accounting share write access](screenshots/30-accounting-write-access-verified.jpg)


## What this lab demonstrates

- Configuration and validation of SMB shares using standard Windows administration tools.
- Differences between **read**, **read/write**, and **restricted** file sharing.
- Access control through an **Active Directory security group** rather than individual share permissions.
- A reproducible permissions troubleshooting workflow: **verify denial → change group membership → sign in again → retest**.

**Outcome:** The standard user initially could not access `\\DC-1\accounting`; after joining `ACCOUNTANTS` and signing back in, the user opened the shared folder and created a test file.
