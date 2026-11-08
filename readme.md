# AZ-500 Student Practical Lab – Custom Roles, PIM and Microsoft Entra Applications

**Course:** AZ-500 – Microsoft Azure Security Technologies  
**Audience:** Students  
**Estimated duration:** 2.5–3.5 hours (some exercises depend on tenant licensing)  
**Delivery:** Azure Portal and Microsoft Entra admin center; no application code or deployment required

---

## Lab objectives

By completing this lab, you will:

1. Create and assign an Azure custom RBAC role.
2. Explore Microsoft Entra custom roles and understand how they differ from Azure roles.
3. Review PIM settings and, if licensed, create and activate an eligible Azure resource role.
4. Create application registrations and identify their enterprise applications (service principals).
5. Expose a delegated permission scope from an API registration.
6. Request that scope from a client registration and review/grant consent.
7. Review OAuth permission grants on an enterprise application and understand revocation.

## Prerequisites

- A Microsoft Entra workforce tenant.
- An Azure subscription for Exercise 1.
- A user account with sufficient permissions for each exercise.
- A separate test user is recommended for role assignment and PIM activation.
- For Azure custom role creation: permission to create role definitions and role assignments at the selected scope (for example, Owner or User Access Administrator with the required actions).
- For PIM: Microsoft Entra ID P2 or Microsoft Entra ID Governance entitlement is generally required for the relevant features.
- For Entra custom roles: check the current license and administrative-role prerequisites in your tenant.
- For application registration: permission to register applications, or an administrator who can do so.
- For granting tenant-wide admin consent: an appropriately privileged administrator. The exact role required depends on the API and permission type.

## Safety requirements

1. Use a dedicated training tenant. Do not perform these exercises in production.
2. Do not grant broad permissions to an unknown application.
3. Do not create permanent Owner or Global Administrator assignments for convenience.
4. Do not change tenant-wide consent settings during this introductory lab.
5. Never create or expose a client secret for this lab. No credential is needed.
6. Use only lab-created app registrations when testing consent and revocation.
7. Do not delete pre-existing applications, service principals, role assignments or consent grants.

---

# Exercise 1 – Create and assign an Azure custom role

## Scenario

A networking support engineer needs to view virtual networks, network security groups and route tables in a lab resource group. The engineer should not receive broad Contributor permissions.

In this exercise, create a read-only custom role for selected networking resources.

## Step 1 – Create a resource group

1. Sign in to the [Azure portal](https://portal.azure.com/).
2. Open **Resource groups**.
3. Select **Create**.
4. Select your training subscription.
5. Enter resource group name: `rg-az500-identity-lab`.
6. Select a region.
7. Select **Review + create** and then **Create**.

You do not need to deploy a VM or application. You may create a VNet and NSG in this resource group if you want sample objects to inspect.

## Step 2 – Open custom role creation

1. Open **Subscriptions**.
2. Select your training subscription.
3. Open **Access control (IAM)**.
4. Select **Add** → **Add custom role**.
5. Choose **Start from scratch**.

Portal labels can change slightly over time.

## Step 3 – Define the role

Use the following details:

| Field | Value |
|---|---|
| Role name | `AZ500 Network Resource Reader` |
| Description | `Read selected networking resources for the AZ-500 lab` |
| Baseline permissions | Start from scratch |

Add these control-plane operations:

```text
Microsoft.Network/virtualNetworks/read
Microsoft.Network/networkSecurityGroups/read
Microsoft.Network/networkInterfaces/read
Microsoft.Network/routeTables/read
Microsoft.Resources/subscriptions/resourceGroups/read
```

Do not add write or delete operations.

Set **Assignable scopes** to your training subscription. Review the definition and create the role.

> `Actions` controls management-plane operations. `DataActions` controls supported data-plane operations, such as operations on data stored inside a resource. `NotActions` is not an explicit deny against permissions granted by other role assignments.

## Step 4 – Assign the role at resource-group scope

1. Open `rg-az500-identity-lab`.
2. Select **Access control (IAM)**.
3. Select **Add** → **Add role assignment**.
4. Search for `AZ500 Network Resource Reader`.
5. Select the role.
6. Select your dedicated test user.
7. Review the scope and complete the assignment.

The custom role definition is available at subscription scope, but this assignment is limited to the lab resource group.

## Step 5 – Validate the assignment

1. Return to the resource group's **Access control (IAM)** page.
2. Open **Role assignments**.
3. Confirm that the test user has the custom role.
4. If possible, sign in as the test user in a separate private browser session.
5. Confirm that the user can view permitted networking resources.
6. Do not test write operations against production resources.

## Check your understanding

- Does creating a custom role automatically assign it to a user? No.
- What is the difference between `Actions` and `DataActions`?
- Does `NotActions` explicitly deny a permission granted by another role? No.
- What is the difference between `AssignableScopes` and the scope of a role assignment?

---

# Exercise 2 – Explore Microsoft Entra custom roles

## Scenario

A helpdesk or application support team needs selected directory administration permissions without receiving Global Administrator.

## Steps

1. Open the [Microsoft Entra admin center](https://entra.microsoft.com/).
2. Navigate to **Entra ID** → **Roles & admins**.
3. Review built-in roles such as:
   - User Administrator
   - Application Administrator
   - Cloud Application Administrator
   - Privileged Role Administrator
4. Locate **New custom role**.
5. Review the available permission categories and permissions.
6. If your tenant is licensed and your account has the required privileges, create a training-only custom role with a narrow supported permission.
7. Review the assignment scope options. Entra custom roles can be assigned at directory scope or supported app registration resource scope.
8. Do not assign the role to your only administrator account.

> Microsoft Entra custom roles can contain only permissions enabled for custom use. The available permission set is not identical to every permission in built-in roles. This exercise can be completed as a review if your tenant does not have the required license or privileges.

## Check your understanding

- Azure custom roles control Azure resource operations.
- Entra custom roles control supported directory operations.
- Azure Owner and Entra Global Administrator are not equivalent.

---

# Exercise 3 – Review and manage PIM for Azure resources

## Scenario

A cloud administrator occasionally needs elevated access to an Azure subscription. The organization wants the administrator to activate privileges only when required.

## Step 1 – Open PIM

1. Open the Microsoft Entra admin center.
2. Search for **Privileged Identity Management**.
3. Open **Azure resources**.
4. Select the appropriate subscription, if it is available.

If the subscription is not visible, check whether it is onboarded to PIM and whether your account has the necessary resource-administrator permissions.

## Step 2 – Review role assignments

1. Open **Assignments** or **Roles**, depending on the portal view.
2. Review the assignment types:
   - Active
   - Eligible
3. Compare the role, principal, scope and assignment duration.
4. Open **Settings** for a suitable role, if permitted.
5. Review:
   - Activation maximum duration.
   - MFA requirement.
   - Justification requirement.
   - Approval requirement.
   - Ticket information requirement.
   - Notification settings.

## Step 3 – Create an eligible assignment (if licensed and authorized)

1. In PIM, select your training subscription.
2. Open **Assignments**.
3. Select **Add assignments**.
4. Choose a low-risk role such as **Reader** for the lab.
5. Select a dedicated test user.
6. Choose **Eligible** as the assignment type.
7. Choose a time-bound assignment if available.
8. Review and complete the assignment.

Do not grant permanent Owner access just to complete the exercise.

## Step 4 – Activate the eligible assignment

1. Sign in as the test user.
2. Open PIM → **My roles** → **Azure resource roles**.
3. Locate the eligible Reader assignment.
4. Select **Activate**.
5. Provide justification if prompted.
6. Complete MFA or approval if configured.
7. Confirm that the assignment becomes active for the selected duration.

After the activation expires, the temporary active assignment is removed. The eligible assignment can remain available for later activation.

## Step 5 – Review the result

Return to PIM and inspect the assignment status and audit/history information available to you.

## Check your understanding

- Active means the role is currently available.
- Eligible means activation is required.
- Assignment duration and activation duration are different.
- PIM governs privileged assignment and activation; Azure RBAC still determines the underlying resource permissions.

---

# Exercise 4 – Create an API app registration and expose a delegated scope

## Scenario

A company has an internal API called `AZ500-Lab-API`. A client application should be able to request permission to read information from this API.

No API code or endpoint is deployed in this lab. You are configuring the identity and permission definitions only.

## Step 1 – Create the API app registration

1. In the Microsoft Entra admin center, open **App registrations**.
2. Select **New registration**.
3. Enter name: `AZ500-Lab-API`.
4. Under supported account types, select **Accounts in this organizational directory only**.
5. Leave Redirect URI blank.
6. Select **Register**.

## Step 2 – Record the identifiers

On the Overview page, record:

- Application (client) ID.
- Directory (tenant) ID.
- Object ID.

These are different identifiers. The Application (client) ID identifies the application across its use, while the Object ID identifies this application object in the tenant.

## Step 3 – Configure the Application ID URI

1. Open **Expose an API**.
2. Select **Add** beside Application ID URI.
3. Accept the suggested URI, normally:

```text
api://<application-client-id>
```

4. Save the URI.

## Step 4 – Add a delegated permission scope

1. Under **Expose an API**, select **Add a scope**.
2. Enter:

| Field | Value |
|---|---|
| Scope name | `Employee.Read` |
| Who can consent? | Admins and users |
| Admin consent display name | Read employee information |
| Admin consent description | Allows the client to read employee information on behalf of the signed-in user |
| User consent display name | Read employee information |
| User consent description | Allows this application to read employee information on your behalf |
| State | Enabled |

3. Save the scope.

The complete scope will resemble:

```text
api://<application-client-id>/Employee.Read
```

## Step 5 – Understand what you configured

You created a delegated permission definition exposed by the API registration.

You did not create an API endpoint or implement authorization. A real API must validate access tokens and enforce the required scope before returning protected information.

---

# Exercise 5 – Create a client app registration and request the API scope

## Scenario

A second application called `AZ500-Lab-Client` needs to access the API from Exercise 4 on behalf of a signed-in user.

## Step 1 – Create the client registration

1. Open **App registrations** → **New registration**.
2. Enter name: `AZ500-Lab-Client`.
3. Select **Accounts in this organizational directory only**.
4. Leave Redirect URI blank.
5. Select **Register**.
6. Record the client Application (client) ID.

## Step 2 – Add the API permission

1. Open `AZ500-Lab-Client`.
2. Select **API permissions**.
3. Select **Add a permission**.
4. Select **My APIs**.
5. Select `AZ500-Lab-API`.
6. Select **Delegated permissions**.
7. Select `Employee.Read`.
8. Select **Add permissions**.

## Step 3 – Review the permission

1. Return to the client's **API permissions** page.
2. Confirm that `Employee.Read` appears.
3. Review its permission type and consent status.

> Adding the permission declares what the client wants to access. It does not by itself grant consent.

## Step 4 – Understand delegated and application permissions

For this lab, you selected a delegated permission because the scenario involves a signed-in user.

For a background service that runs without a signed-in user, an API may expose an application permission using an app role. Application permissions generally require administrator consent.

---

# Exercise 6 – Review enterprise applications and manage consent

## Scenario

The administrator needs to review the client application in the tenant, grant the required consent, and understand how to revoke a grant.

## Step 1 – Locate the enterprise applications

1. Open **Enterprise applications** in the Microsoft Entra admin center.
2. Search for `AZ500-Lab-API`.
3. Open the result.
4. Note its Object ID.
5. Return to Enterprise applications and search for `AZ500-Lab-Client`.
6. Open the result and note its Object ID.

The enterprise application is the tenant-local service principal. Its Object ID is different from the corresponding app registration's Object ID.

## Step 2 – Review the client enterprise application

1. Open `AZ500-Lab-Client` under Enterprise applications.
2. Review **Properties**.
3. Review **Users and groups**.
4. Review **Permissions**.
5. Review **Owners** and **Sign-in logs**, if available.

Do not change assignment requirements or other tenant-wide settings for this exercise.

## Step 3 – Grant admin consent (if authorized)

1. Return to **App registrations**.
2. Open `AZ500-Lab-Client`.
3. Select **API permissions**.
4. Review the requested `Employee.Read` permission.
5. If your account has the required administrative role, select **Grant admin consent for [tenant]**.
6. Confirm the operation.

If the button is unavailable, ask your tenant administrator to perform this step. The role required depends on the API and permission type. Do not elevate your own account simply to complete the exercise.

## Step 4 – Verify the granted permission

1. Open **Enterprise applications**.
2. Select `AZ500-Lab-Client`.
3. Open **Permissions**.
4. Review the granted permission.
5. Compare it with the requested permission shown in the app registration.

## Step 5 – Review consent settings (do not change them)

1. Open **Enterprise applications**.
2. Navigate to **Consent and permissions**.
3. Open **User consent settings**.
4. Review the current tenant setting.
5. Review the **Admin consent requests** workflow if it is configured.
6. Do not modify tenant-wide settings during this lab.

## Step 6 – Understand revocation

In a training tenant, you may revoke the lab's admin consent if the portal exposes the action:

1. Open the `AZ500-Lab-Client` enterprise application.
2. Select **Permissions**.
3. Locate the organization-wide consent grant.
4. Review the available revoke action.
5. If you want to test revocation, revoke only the grant created for this lab.
6. Return to the client app registration's **API permissions** page.

The requested permission may still appear in the app registration after consent is revoked. This demonstrates the difference between a permission declaration and an authorization grant.

> The portal's ability to revoke user-specific grants differs from organization-wide admin consent. Some user-specific grants require Microsoft Graph or PowerShell. Revocation also does not necessarily invalidate every already-issued access token immediately.

---

# Final validation checklist

- [ ] Created an Azure custom role.
- [ ] Assigned the role at resource-group scope.
- [ ] Reviewed Microsoft Entra custom roles.
- [ ] Compared Azure RBAC and Entra RBAC.
- [ ] Reviewed Active and Eligible PIM assignments.
- [ ] Reviewed PIM activation settings.
- [ ] Created an API app registration.
- [ ] Recorded its Application ID, Tenant ID and Object ID.
- [ ] Configured an Application ID URI.
- [ ] Exposed a delegated API scope.
- [ ] Created a client app registration.
- [ ] Requested the API's delegated permission.
- [ ] Located both enterprise applications/service principals.
- [ ] Reviewed permissions and consent.
- [ ] Granted admin consent if authorized.
- [ ] Reviewed the difference between requested and granted permissions.
- [ ] Reviewed consent revocation.

# Cleanup

Remove only the lab objects you created.

1. Revoke the lab consent grant if you want to test cleanup.
2. Delete `AZ500-Lab-Client` app registration.
3. Delete `AZ500-Lab-API` app registration.
4. Remove the eligible PIM assignment.
5. Remove the Azure custom role assignment.
6. Delete the Azure custom role if no longer required.
7. Delete `rg-az500-identity-lab` if it was created only for this exercise.

Deleting the app registrations also removes their associated service principals in the tenant. Do not delete unrelated enterprise applications or shared identities.

---

# Knowledge-check questions

1. What is the difference between Azure RBAC and Microsoft Entra RBAC?
2. What is the difference between a role definition and a role assignment?
3. What is the difference between `Actions` and `DataActions`?
4. Why is `NotActions` not an explicit deny?
5. What is the difference between Active and Eligible PIM assignments?
6. What is Just-In-Time privileged access?
7. What is the difference between an app registration and an enterprise application?
8. What is a service principal?
9. What is the difference between delegated and application permissions?
10. What is an OAuth permission grant?
11. What is the purpose of an Application ID URI?
12. What is a delegated permission scope?
13. What is the difference between a delegated scope and an app role?
14. What is the difference between adding an API permission and granting consent?
15. Why should administrators review and periodically revoke unnecessary OAuth grants?

## Expected learning outcome

Students should be able to explain how custom roles restrict administrative access, how PIM provides just-in-time privileges, how app registrations and service principals represent applications, and how API permissions and consent govern application access to protected resources—without deploying a complex application.
