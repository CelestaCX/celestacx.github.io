CelestaCX uses a role-based access control (RBAC) model to manage who can log in to the platform, what they can see, and what actions they can take. This page covers user provisioning, roles and permissions, Single Sign-On (SSO), and multi-tenancy boundaries.
 
---
 
### Core Concepts
 
Before configuring access, it helps to understand how CelestaCX structures identity:
 | Concept | Description |
| --- | --- |
| Tenant | A  logically  isolated  environment  within  the  platform.  Each  customer  or  business  unit  operates  within  its  own  tenant.  Users,  queues,  and  configurations  do  not  cross  tenant  boundaries. |
| User | An  individual  with  a  login  account.  Users  are  always  scoped  to  a  tenant  and  assigned  one  or  more  roles. |
| Role | A  named  set  of  permissions  that  controls  access  to  features  and  data.  Roles  are  assigned  to  users  at  the  tenant  level. |
| Service  Account | A  non-human  account  used  by  integrations,  bots,  or  external  systems  to  authenticate  with  the  CelestaCX  API. | 
---
 
### Built-In Roles
 
CelestaCX ships with a set of pre-defined roles covering the most common operational profiles. These cannot be deleted, but their default permissions can be reviewed by a platform administrator.
 | Role | Typical  User | Access  Summary |
| --- | --- | --- |
| Platform  Admin | IT  /  Ops  team | Full  access  across  all  tenants.  Manages  infrastructure-level  settings,  SSO,  and  tenant  provisioning. |
| Tenant  Admin | Customer  admin | Full  access  within  a  single  tenant.  Manages  users,  channels,  queues,  routing,  and  configuration. |
| Supervisor | Team  lead | Access  to  live  monitoring,  queue  management,  agent  state  control,  and  reporting  within  assigned  teams. |
| Agent | Front-line  staff | Access  to  the  Agent  Desk  only.  Cannot  access  admin  or  reporting  areas. |
| Quality  Manager | QA  team | Access  to  interaction  recordings,  evaluation  forms,  and  quality  reports.  No  routing  or  admin  access. |
| Evaluator | QA  reviewer | Access  to  assigned  evaluations  only.  Read-only  on  recordings  and  completed  evaluation  forms. |
| Reporting  User | Analyst  /  manager | Access  to  historical  reports  and  dashboards.  No  operational  or  admin  access. | **Note:** If your operation requires custom roles with granular permission combinations, contact your CelestaCX implementation partner. Custom role configuration is available at the platform-admin level. 
---
 
### User Provisioning
 
#### Creating Users Manually
 
1. Log in to the CelestaCX Admin Portal as a Tenant Admin or Platform Admin.
2. Navigate to **Administration → Users → Add User** .
3. Enter the user's details: full name, email address, and preferred language.
4. Assign one or more roles.
5. Assign the user to one or more agent teams (required for agents and supervisors).
6. Set the user's status to **Active** and save.
 
The user will receive a welcome email with a link to set their password (if local authentication is enabled).
 
#### Bulk User Import
 
For larger deployments, users can be imported via CSV. The import template is available in the Admin Portal under **Administration → Users → Import Users** . The CSV must include: `first_name` , `last_name` , `email` , `role` , `team` (optional).
 
Validation errors are reported per row after upload. Successful rows are provisioned immediately.
 
#### Deactivating Users
 
Deactivating a user prevents login without deleting their history or interaction records. To deactivate, navigate to **Administration → Users** , locate the user, and set their status to **Inactive** . Active interactions assigned to that agent are not automatically reassigned — supervisors should handle any open work before deactivating.
 
---
 
### Single Sign-On (SSO)
 
CelestaCX supports SSO via **SAML 2.0** , allowing users to authenticate through your organisation's existing identity provider (IdP) rather than managing a separate CelestaCX password.
 
#### Supported Identity Providers
 
CelestaCX has been tested with the following IdPs:
 
- Microsoft Azure Active Directory / Entra ID
- Okta
- Google Workspace
- Any standards-compliant SAML 2.0 provider
 
#### Configuring SSO
 
SSO is configured at the tenant level by a Tenant Admin or Platform Admin.
 
1. Navigate to **Administration → Security → Single Sign-On** .
2. Select **SAML 2.0** as the authentication method.
3. Download the CelestaCX **Service Provider (SP) Metadata** file from this page.
4. Register CelestaCX as an application in your IdP using the SP metadata.
5. From your IdP, obtain the **IdP Metadata URL** or upload the IdP metadata XML.
6. Paste or upload the IdP metadata into CelestaCX and save.
7. Map the required SAML attributes:
 | SAML  Attribute | CelestaCX  Field | Required |
| --- | --- | --- |
| email | User  email  (primary  identifier) | Yes |
| firstName | User  first  name | Yes |
| lastName | User  last  name | Yes |
| role | CelestaCX  role  assignment | Optional | 
1. Test the configuration using the **Test SSO Login** button before enabling it for all users.
2. Set the authentication mode to **SSO Only** or **SSO + Local** depending on your policy.
 **Important:** If SSO is set to **SSO Only** , local password login is disabled for all tenant users. Ensure at least one break-glass admin account is configured before switching to SSO Only mode. 
#### SSO and Just-in-Time (JIT) Provisioning
 
When JIT provisioning is enabled, users who authenticate via SSO for the first time are automatically created in CelestaCX without needing to be pre-provisioned. The role assigned to JIT-provisioned users defaults to **Agent** unless a role mapping is configured via SAML attributes.
 
To enable JIT provisioning, toggle **Auto-provision users on first login** under **Administration → Security → Single Sign-On** .
 
---
 
### Multi-Tenancy
 
CelestaCX is a multi-tenant platform. Each tenant is fully isolated — users, configurations, queues, interactions, and reports are never shared across tenants.
 
**Platform Admins** can create and manage tenants. **Tenant Admins** operate exclusively within their own tenant and cannot view or modify other tenants.
 
#### Creating a New Tenant
 
This is a Platform Admin action performed from the platform-level administration interface, not from within a tenant.
 
1. Log in as a Platform Admin.
2. Navigate to **Platform Administration → Tenants → New Tenant** .
3. Provide a tenant name, subdomain or identifier, and primary admin user details.
4. Select the feature set and channel entitlements for the tenant.
5. Save. The tenant is immediately active and the admin user receives a setup email.
 
---
 
### Password Policy
 
For tenants using local authentication (not SSO Only), CelestaCX enforces the following default password policy:
 
- Minimum 8 characters
- At least one uppercase letter, one number, and one special character
- Password expiry: 90 days (configurable)
- Maximum failed login attempts before lockout: 5 (configurable)
- Account lockout duration: 15 minutes (configurable)
 
Password policy settings are available under **Administration → Security → Password Policy** .
 
---
 
### Audit Logging
 
All user access events — logins, failed login attempts, role changes, user creation and deactivation — are recorded in the platform audit log. The audit log is accessible to Platform Admins under **Platform Administration → Audit Log** and is retained for 90 days by default.
 
---
 
*What's next: With users and access configured, proceed to* [*Channels & Routing Configuration*](#) *to define how interactions enter the platform and reach your agents.*