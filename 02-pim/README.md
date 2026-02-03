# Lab 02 — Azure PIM: Just-in-Time Data Plane Access

## Goal

This lab demonstrates the use of Microsoft Entra Privileged Identity Management (PIM) to enforce just-in-time access for sensitive data-plane operations in Azure.

The objective is to remove standing privileges while preserving operational capability through time-bound, auditable role activation.

---

## Scenario

An application stores sensitive data in an Azure Storage Account.

Operational requirements:
- Operators must not have permanent access
- Access must be requested only when required
- Permissions must be time-bound and auditable
- Management-plane access must remain restricted

Security requirements:
- deny-by-absence by default
- minimal blast radius
- separation of management and data planes

---

## Architecture

- Resource Group: `rg-az500-02-pim`
- Storage Account: Storage A
- User: `ops.operator`
- Entra ID Security Group: `grp-az500-02-staA-blob-contrib`
- Role: Storage Blob Data Contributor
- Assignment type: Eligible (PIM)

The role is scoped directly to the storage account and controlled via PIM.

---

## Threats Mitigated

- Standing privileged access
- Uncontrolled data access windows
- Excessive role inheritance
- Lack of accountability for elevated access

---

## Implementation

### 1. Create eligible role assignment

A group-based role assignment is created with **eligible** access only.

- Role: Storage Blob Data Contributor  
- Scope: Storage Account A  
- Principal: `grp-az500-02-staA-blob-contrib`  
- Assignment type: Eligible  

This ensures no permissions are granted without explicit activation.

`01-eligible-assignment.png`

---

### 2. Configure activation settings

Activation is configured with:
- approval required
- justification required
- time-bound activation
- audit logging enabled

`02-activation-settings.png`

---

### 3. Validate access before activation

Before activation:
- the operator has no effective permissions
- the storage account is not visible
- data-plane operations are denied

This confirms deny-by-absence and the absence of standing access.

`03-before-activation-denied.png`

---

### 4. Request role activation

The operator submits an activation request with justification.

`04-activation-request-submitted.png`

---

### 5. Approve activation

An administrator reviews and approves the request.

`05-approval-completed.png`

---

### 6. Validate data-plane access after activation

After activation:
- the role becomes active
- management-plane access remains restricted
- data-plane access is validated using Azure CLI with Entra ID authentication

Actions performed:
- create a blob container
- upload a test file

`06-creation-success.png`

---

### 7. Review audit logs

Activation, approval, and role usage events are recorded and reviewed via PIM and activity logs.

`07-audit-logs.png`

---

## Validation Summary

| Scenario | Result |
|--------|--------|
| Standing access | Denied |
| Visibility before activation | Denied |
| Activation required | Yes |
| Data-plane access after activation | Allowed |
| Management-plane access | Denied |
| Audit trail | Complete |

---

## Notes

- Eligible assignments do not grant visibility or permissions until activated
- The Azure Portal operates in the management plane and is intentionally restricted
- Data-plane access is validated through direct service interaction
- Group-based eligibility simplifies access governance

---

## Lab Status

Completed.  
All resources were removed after validation.
