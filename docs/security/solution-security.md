#### Overview
 
This page provides a complete reference of the security capabilities built into CelestaCX. It is intended for security reviewers, IT administrators, compliance teams, and enterprise procurement teams evaluating the platform's security posture.
 
CelestaCX is a self-hosted platform — meaning your organisation retains full data sovereignty and infrastructure ownership. The security controls described here are provided by the platform itself. Infrastructure-level security (host hardening, antivirus, physical access, network firewalls) is the responsibility of the operator or hosting partner.
 
---
 
#### 1. Identity & Authentication
 
CelestaCX centralises all authentication through Keycloak — an enterprise-grade, open-source identity and access management platform supporting OpenID Connect and SAML 2.0.
 | Capability | Detail |
| --- | --- |
| Keycloak  IAM | All  user  authentication  is  handled  by  Keycloak.  Supports  OpenID  Connect  (OIDC)  and  SAML  2.0.  Used  for  all  platform  roles  —  agents,  supervisors,  administrators,  and  evaluators. |
| Role-Based  Access  Control  (RBAC) | Roles  —  Agent,  Supervisor,  Admin,  Quality  Manager,  Evaluator,  Routing  Manager  —  enforce  least-privilege  access  to  data  and  platform  functions.  Users  can  only  access  what  their  role  permits. |
| Group-Based  Access  Control  (GBAC) | Access  is  further  scoped  by  team  and  tenant.  Agents  see  only  their  own  conversations  and  assigned  queues.  Supervisors  see  only  agents  in  their  team. |
| Single  Sign-On  (SSO) | Integration  with  corporate  identity  providers  (Azure  AD,  Okta,  Google  Workspace)  via  SAML  2.0  or  OIDC.  Users  authenticate  through  your  corporate  IdP  and  are  granted  CelestaCX  access  based  on  Keycloak  role  assignments. |
| Two-Factor  Authentication  (2FA) | OTP  delivery  via  email.  Configurable  parameters  include  OTP  expiry  (60–120  seconds),  maximum  failed  attempts,  and  automatic  cooldown/lockout  after  repeated  failures. |
| User  Inactivity  Logout | Sessions  are  automatically  terminated  after  a  configurable  idle  period.  Prevents  unattended  sessions  from  remaining  active. |
| Password  Policy  Enforcement | Keycloak  enforces  configurable  password  complexity,  expiry,  and  history  rules  across  all  user  accounts. |
| Brute-Force  Protection | Keycloak  includes  built-in  brute-force  detection.  Accounts  are  subject  to  lockout  after  a  configurable  number  of  failed  login  attempts. |
| Multi-Tenant  Isolation | Each  customer  organisation  operates  in  its  own  dedicated  Keycloak  realm.  There  is  complete  data  and  access  isolation  between  tenants  —  no  cross-tenant  visibility  is  possible  by  design. | 
---
 
#### 2. Encryption
 | Capability | Detail |
| --- | --- |
| Encryption  in  Transit | TLS  is  enforced  on  all  external  communication  —  Agent  Desk,  public  APIs,  channel  connectors,  and  inter-service  communication  within  the  platform.  No  unencrypted  channels  are  exposed. |
| Encryption  at  Rest | Message  content  and  call  recordings  are  encrypted  at  rest.  Configurable  via  the  Encryption  at  Rest  configuration. |
| Secrets  Management  (HashiCorp  Vault) | HashiCorp  Vault  is  supported  for  centralised  secrets  management  across  platform  services.  Vault  stores  and  controls  access  to  database  credentials,  API  keys,  and  other  sensitive  configuration  values. |
| Dynamic  Credential  Rotation | Vault  issues  short-lived,  dynamically  generated  credentials  for  MongoDB,  Redis,  and  ActiveMQ.  Credentials  expire  automatically  and  are  rotated  without  service  interruption  —  no  static  long-lived  passwords  are  used  for  data  stores. |
| SSL/TLS  Certificate  Lifecycle | cert-manager  integration  supports  automated  certificate  provisioning  and  renewal.  Procedures  are  documented  for  manual  certificate  renewal  and  RKE2  cluster  certificate  rotation. | 
---
 
#### 3. Data Privacy & PII Protection
 | Capability | Detail |
| --- | --- |
| PII  Data  Masking | Personally  Identifiable  Information  can  be  masked  in  two  scopes:  (1)  within  system  and  application  logs,  preventing  PII  from  appearing  in  log  output;  and  (2)  within  the  Agent  Desk  UI,  masking  customer  attributes  from  agent  view  where  configured. |
| Pause-and-Resume  Recording | Agents  can  pause  call  and  screen  recordings  from  within  the  Agent  Desk  during  any  point  in  a  live  interaction.  This  allows  sensitive  information  —  payment  card  numbers,  health  data,  identity  documents  —  to  be  disclosed  verbally  without  being  captured  in  the  recording. |
| Configurable  Data  Retention  &  Archival | Interaction  data  retention  periods  are  configurable.  Older  data  can  be  automatically  archived  to  long-term  storage  (S3-compatible  object  stores  or  secondary  databases)  based  on  defined  retention  policies. |
| Do  Not  Contact  (DNC)  Lists | DNC  lists  are  managed  within  the  platform  and  checked  automatically  before  any  outbound  dialling  attempt.  Supports  compliance  with  telecoms  regulations  governing  unsolicited  contact. |
| Multi-Tenant  Data  Isolation | All  customer  data  is  logically  separated  at  the  Keycloak  realm  and  database  tenant  level.  No  mechanism  exists  —  by  design  —  for  one  tenant's  data  to  be  accessed  from  another  tenant's  session. | 
---
 
#### 4. Audit Logging & Traceability
 
CelestaCX uses OpenTelemetry Protocol (OTLP)-based logging and tracing across all platform services. Logs are compatible with OpenSearch (the recommended SIEM) and other OTLP-capable observability platforms.
 | Capability | Detail |
| --- | --- |
| OTLP-Based  Audit  Logging | All  platform  services  emit  structured,  OTLP-formatted  logs  and  traces.  Covers  application  events,  authentication  events,  routing  decisions,  and  system  operations. |
| Immutable  Audit  Records | Audit  records  are  append-only.  Once  written,  they  cannot  be  altered  or  deleted  —  preserving  evidentiary  integrity  for  compliance  and  incident  investigation. |
| Chronological  Integrity | Events  are  stored  in  the  order  they  occurred,  with  accurate  timestamps. |
| Trace  Correlation | Logs  are  linked  to  distributed  execution  traces  via  trace  IDs  and  span  IDs.  A  single  customer  interaction  can  be  traced  end-to-end  across  all  microservices  that  handled  it. |
| IAM  Event  Logging | Keycloak  records  all  authentication  and  authorisation  events  —  successful  logins,  failed  attempts,  role  assignments,  and  administrative  changes. |
| OpenSearch  Integration | Centralised  log  aggregation  and  search  via  OpenSearch,  with  Fluentbit  as  the  log  shipping  agent.  Supports  full-text  search,  alerting,  and  dashboard  visualisation  of  audit  events. |
| Non-Repudiation | The  audit  trail  ensures  users  cannot  deny  having  performed  a  specific  action  within  the  system. |
| Incident  Response  Support | OTLP  traces  and  logs  provide  the  telemetry  needed  to  reconstruct  the  full  timeline  of  a  security  event  or  service  disruption. | 
#### Compliance Framework Coverage
 
The audit logging capability maps to the following regulatory requirements:
 | Framework | Requirement  Addressed |
| --- | --- |
| SOC  2 | Traceability  of  administrative  actions;  availability  logging;  confidentiality  of  log  storage |
| ISO  27001 | Control  A.12.4  —  event  traceability  across  all  platform  resources |
| HIPAA | §164.312(b)  —  recording  and  examining  access  to  systems  containing  ePHI |
| GDPR | Articles  30  &  33  —  accountability  in  data  processing;  72-hour  breach  notification  support |
| PCI  DSS | Requirement  10  —  tracking  all  access  to  cardholder  data  environments | 
---
 
#### 5. Infrastructure Security
 | Capability | Detail |
| --- | --- |
| Kubernetes  Network  Policies | Inter-service  traffic  within  the  cluster  can  be  restricted  using  Kubernetes  network  policies.  This  limits  the  blast  radius  of  a  compromised  pod  and  prevents  lateral  movement  between  services. |
| Namespace  Isolation | Platform  services  and  per-tenant  components  are  deployed  in  dedicated  Kubernetes  namespaces,  providing  an  additional  layer  of  logical  separation. |
| No  Default  Credentials  in  Production | Default  passwords  for  all  system  components  —  including  Keycloak  admin,  ActiveMQ,  and  database  services  —  must  be  changed  before  go-live.  This  is  enforced  as  a  mandatory  pre-launch  step  in  the  operational  checklist. |
| Helm-Based  Deployment | All  platform  components  are  deployed  via  Helm  charts  with  configurable  values.  No  credentials  are  hardcoded  in  manifests.  Sensitive  values  are  managed  via  Kubernetes  Secrets  or  Vault. |
| CI/CD  Security  Scanning | Security  checks  are  applied  throughout  the  build  and  release  pipeline.  Each  release  package  is  scanned  against  known  vulnerability  databases  before  publication. |
| Security  Maintenance  Releases | When  new  vulnerabilities  are  discovered  in  platform  components  or  dependencies,  security  maintenance  releases  are  issued.  Customers  are  notified  and  upgrade  procedures  are  documented. |
| Cluster  State  Backup | etcd  snapshots  capture  the  full  Kubernetes  control  plane  state.  Combined  with  MongoDB  and  PostgreSQL  backups,  this  enables  full  platform  restoration  in  a  disaster  recovery  scenario. | 
---
 
#### 6. Compliance Posture Summary
 | Framework | Posture | Notes |
| --- | --- | --- |
| GDPR | ✅  Compliant | Encryption  in  transit  and  at  rest,  RBAC/GBAC,  PII  masking,  pause-and-resume  recording,  data  retention  controls,  DPA  available  on  request |
| HIPAA | ✅  Compliant  (Technical  Safeguards) | Access  control,  authentication  (including  2FA),  TLS  transmission  security,  PII  masking.  Business  Associate  Agreement  (BAA)  available  on  request.  Physical  and  Administrative  Safeguards  are  customer  responsibility. |
| PCI  DSS | ⚠️  Partially  compliant  Call  transcript  auto-redaction  is  not  natively  available | Network  isolation  capable,  no  default  passwords,  TLS  in  transit,  RBAC,  2FA,  CI/CD  scanning.  Pause-and-resume  recording  supports  cardholder  data  handling. | 
#### Related Documentation
 
- [Identity & Access Management](#) — Keycloak realm setup, user creation, role assignments, 2FA configuration
- [Security & Certificates](#) — TLS configuration, certificate management, Vault setup
- [Encryption at Rest Configuration](#) — MongoDB, file storage, and backup encryption
- [Audit Logging](#) — OTLP log access, OpenSearch integration, compliance framework mapping
- [Logging & Monitoring](#) — Kubernetes log access, cluster health monitoring
- [Platform Administration](#) — Credential rotation, certificate renewal, backup procedures