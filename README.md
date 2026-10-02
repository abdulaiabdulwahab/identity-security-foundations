# identity-security-foundations

# Identity Security Foundations

## Project Overview

This project demonstrates the fundamentals of **Identity Security in Microsoft Azure** using **Microsoft Entra ID, Service Principals, and Azure Role-Based Access Control (RBAC)**.

The goal is to understand how workload identities authenticate to Azure and how least-privilege access can be enforced by assigning only the permissions required at a specific scope.

---

## Objectives

In this project, I:

- Created an Azure Resource Group for testing.
- Created a Microsoft Entra Service Principal.
- Identified the difference between a Client ID and Object ID.
- Assigned the built-in **Reader** role using Azure RBAC.
- Scoped the permission to a single Resource Group.
- Authenticated to Azure using the Service Principal.
- Verified that read operations were allowed.
- Verified that write operations were denied.
- Inspected Azure RBAC role assignments.
- Practiced least-privilege identity security.

---

## Architecture

```text
Microsoft Entra ID
        |
        | Authentication
        v
Service Principal
        |
        | Azure RBAC
        | Reader Role
        v
Resource Group
        |
        +---- Read Operations  -> Allowed
        |
        +---- Write Operations -> Denied
```

---

## Technologies Used

- Microsoft Azure
- Microsoft Entra ID
- Azure CLI
- Azure Role-Based Access Control (RBAC)
- Service Principals

---

## Create the Resource Group

```bash
LOCATION="canadacentral"
RG="rg-identity-security-lab"

az group create \
  --name "$RG" \
  --location "$LOCATION"
```

---

## Create the Service Principal

```bash
SP_NAME="sp-identity-security-lab"

az ad sp create-for-rbac \
  --name "$SP_NAME"
```

The command returns values including:

```text
appId
password
tenant
```

The password is sensitive and should never be committed to GitHub.

---

## Get the Service Principal Object ID

```bash
CLIENT_ID="<APP-ID>"

SP_OBJECT_ID=$(az ad sp show \
  --id "$CLIENT_ID" \
  --query id \
  --output tsv)
```

Verify:

```bash
echo "$SP_OBJECT_ID"
```

---

## Get the Resource Group ID

```bash
RG_ID=$(az group show \
  --name "$RG" \
  --query id \
  --output tsv)
```

Verify:

```bash
echo "$RG_ID"
```

---

## Assign Least-Privilege Access

Assign the Service Principal the **Reader** role only at the Resource Group level.

```bash
az role assignment create \
  --assignee-object-id "$SP_OBJECT_ID" \
  --assignee-principal-type ServicePrincipal \
  --role "Reader" \
  --scope "$RG_ID"
```

Verify the assignment:

```bash
az role assignment list \
  --assignee "$SP_OBJECT_ID" \
  --scope "$RG_ID" \
  --output table
```

---

## Authenticate as the Service Principal

```bash
az login \
  --service-principal \
  --username "$CLIENT_ID" \
  --password "$CLIENT_SECRET" \
  --tenant "$TENANT_ID"
```

Set the subscription:

```bash
az account set \
  --subscription "$SUB_ID"
```

---

## Test Allowed Access

The Reader role should allow the Service Principal to view the Resource Group.

```bash
az group show \
  --name "$RG" \
  --output table
```

Expected result:

```text
SUCCESS
```

---

## Test Denied Access

Attempt to modify the Resource Group:

```bash
az group update \
  --name "$RG" \
  --set tags.SecurityTest="ShouldFail"
```

Expected result:

```text
AuthorizationFailed
```

This confirms that the Service Principal has read access but does not have permission to modify Azure resources.

---

## Troubleshooting

### AuthorizationFailed

If role assignment creation fails, verify that your account has sufficient permissions to assign Azure roles.

### PrincipalNotFound

Confirm that the Service Principal exists:

```bash
az ad sp show \
  --id "$CLIENT_ID"
```

### Unexpected Write Access

Check whether the Service Principal has inherited permissions from another scope:

```bash
az role assignment list \
  --assignee "$SP_OBJECT_ID" \
  --all \
  --include-inherited \
  --output table
```

Look for roles such as:

```text
Contributor
Owner
```

---

## Cleanup

Delete the role assignment:

```bash
az role assignment delete \
  --assignee "$SP_OBJECT_ID" \
  --role "Reader" \
  --scope "$RG_ID"
```

Delete the Entra application:

```bash
az ad app delete \
  --id "$CLIENT_ID"
```

Delete the Resource Group:

```bash
az group delete \
  --name "$RG" \
  --yes \
  --no-wait
```

---

## Key Concepts Learned

This project demonstrates the relationship between:

```text
Identity
   +
Authentication
   +
Azure RBAC
   +
Resource Scope
   =
Secure Azure Access
```

The main security principle demonstrated is **least privilege**: an identity should receive only the permissions it requires and only at the scope where those permissions are needed.

---

## Project Outcome

By completing this project, I gained hands-on experience with:

- Microsoft Entra workload identities
- Service Principal authentication
- Azure RBAC
- Resource-level access control
- Least-privilege security
- Testing allowed and denied authorization scenarios
- Azure identity troubleshooting

This project provides a foundation for more advanced identity security concepts such as **Managed Identities, Workload Identity Federation, Azure DevOps service connections, Privileged Identity Management, and passwordless authentication**.