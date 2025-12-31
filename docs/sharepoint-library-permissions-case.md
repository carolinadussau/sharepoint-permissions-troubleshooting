# Troubleshooting Case: SharePoint Online Library – Preventing Deletions Without Breaking Usability (Audit-Sensitive Content)

## Background and Context
As part of an audit review, a SharePoint Online document library containing
sensitive business data (for example HR, Legal, or Finance content) required
changes to reduce the risk of accidental deletion. The goal was to prevent
standard members from deleting files or folders, while still allowing normal
editing, renaming, and day-to-day content management.

The library had grown organically over time, with deep folder structures and
historical manual sharing, making permission behavior non-trivial.

This was classified as a **minor non-conformity**, with usability and operational
continuity being important considerations.

---

## Initial State
- Standard members were assigned the default **Edit** permission level.
- This allowed normal editing and renaming, but also allowed deletions.
- The library contained thousands of nested folders and legacy sharing entries.

---

## Initial Remediation Attempt
To address the audit requirement, the following approach was taken:

- The built-in **Contribute** permission level was modified to remove:
  - Delete Items
  - Delete Versions
- All other edit-related permissions were preserved.
- A custom SharePoint group using this modified permission level was applied.
- Site Owners retained **Full Control**.

This appeared correct from a permission-model perspective, but users immediately
reported functional issues.

---

## Symptoms Observed
- Users could no longer rename folders in some or all locations.
- Permission behavior appeared inconsistent across different folders.
- The modern SharePoint UI continued to display “Can edit,” even when delete
  actions were blocked.
- Changes did not reliably propagate across the entire library structure.

---

## Investigation and Findings

### 1. Permission Inheritance and Unique Permissions
The library and many subfolders had **unique permissions** created over time
through manual sharing. SharePoint requires consistent permission inheritance
across the entire folder path for actions such as rename to function correctly.
A single folder with unique permissions can break rename operations for all
nested items.

### 2. Multiple Permission Sources Overriding Intent
Users were receiving permissions from multiple sources:
- A custom SharePoint group with the modified Contribute role.
- A default Members group still granting Edit permissions.

SharePoint always applies the **most permissive permission** a user receives,
causing the custom restrictions to be silently overridden.

### 3. SharePoint Groups vs Microsoft 365 Groups
The same group name appeared in permission dialogs as both:
- A classic **SharePoint Group**
- A **Microsoft 365 (Azure AD) Group**

Selecting the incorrect group reintroduced Edit permissions at the site level.
The modern UI does not clearly distinguish between these group types, increasing
the likelihood of misconfiguration.

### 4. Hidden Permission Dependencies
Folder rename operations rely on less obvious permissions and internal list
behavior overrides that are not clearly documented by Microsoft and are not
surfaced in the modern permission interface. These dependencies only became
visible once default permission roles were customized.

### 5. Scale and Risk Constraints
The library contained **thousands of nested folders**. SharePoint does not
provide a native, safe way to recursively reset permissions across all items.
A PowerShell-based reset was considered but rejected due to risk:
- Removal of intentional one-off access
- No rollback mechanism
- Potential exposure of sensitive business data
- High operational disruption

---

## Decision and Outcome
Given:
- the scale and complexity of the library,
- the unpredictability of custom permission behavior at depth,
- and the minor severity of the audit finding,

the decision was made to revert to the original **Edit-based permission model**
and mitigate risk using platform-level safeguards rather than enforcing complex
custom permissions at scale.

The following approach was adopted:
- Restore original Edit permissions for standard members.
- Rely on SharePoint’s **Recycle Bin** (up to 93 days) to recover accidental
  deletions.
- Implement **deletion notifications** so responsible users are alerted quickly
  and can restore content before permanent loss.

This approach balanced audit intent with usability and operational safety.

---

## Key Takeaways
- SharePoint permission behavior is influenced by inheritance, group type, and
  UI abstraction—not just role definitions.
- Customizing default permission levels can introduce non-obvious side effects
  at scale.
- Automation (for example, PowerShell permission resets) is not always the safest
  solution in sensitive environments.
- In some cases, leveraging built-in platform safeguards (Recycle Bin, alerts)
  provides a more reliable risk-control strategy than enforcing granular access
  controls.

This case reinforced the importance of validating platform behavior in real
conditions and clearly communicating trade-offs to stakeholders.
