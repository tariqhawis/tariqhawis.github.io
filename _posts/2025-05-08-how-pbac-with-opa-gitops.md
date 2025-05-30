---
layout: post
hero_title: How PBAC with OPA & GitOps Secures Financial Data in Modern Banking Apps
title: How PBAC with OPA & GitOps Secures Financial Data in Modern Banking Apps
description: What is Man-in-The-Middle Attack? How it's implemented? And how to prevent it.
summary: |-
  Man in The Middle Attack is one of the most dangerous attacks, you will learn what is it, the known techniques, and implementing the attack on a real Minecraft PI Server using Kali Linux and Ettercap tool
permalink: /:title/
#hero_image: /img/secure-website-hero.webp
#hero_height: is-large
#image: /img/secure400x200.jpg
date: 2022-04-01
tags: PBAC policy-as-code OPA GitOps Kubernetes Azure CI/CD
show_sidebar: true
toc: true
toc_title: Index
category: Security
---

In today’s cloud-native banking environments, securing access to sensitive financial data requires more than traditional role-based controls. This blog explores how Policy-Based Access Control (PBAC), enforced using Open Policy Agent (OPA) and Rego, enables fine-grained, context-aware authorization across internal and vendor-facing applications. Set within a real-world scenario of a bank using Azure, Kubernetes, GitLab, and GitOps, we demonstrate how dynamic policies improve compliance, reduce risk, and streamline access governance. Learn how to architect, implement, and audit PBAC in a zero-trust environment.

As banks modernize their infrastructure with Azure, Kubernetes, and GitOps workflows, they face new challenges in securing access to sensitive applications. Static RBAC models fall short in scenarios involving internal teams and third-party vendors accessing shared services. This blog presents a detailed use case where a financial institution implements PBAC using Open Policy Agent (OPA) and Rego to control access at the application layer.

**You’ll learn:**
• Why traditional RBAC creates security blind spots in hybrid environments
• How PBAC enables granular, attribute-driven decisions (e.g., role, time, tenant, data classification)
• How to enforce and automate policies via GitLab CI/CD and OPA integration
• How to audit access and maintain compliance using OPA decision logs and Azure AD signals

This post is a practical guide for security engineers, architects, and DevSecOps teams looking to adopt PBAC as a scalable solution for modern access control.

Imagine a Transaction Monitoring Service running on AKS (Azure Kubernetes Service). Internal risk analysts access it to review suspicious transactions, while a vetted fraud-analytics vendor connects through a REST API to fetch aggregated transaction metrics. Without PBAC, static roles can’t easily prevent data leaks. A simple RBAC model might give vendors a single “Analyst” role or group membership. That often leads to excessive privilege – vendors may see full transaction records instead of only allowed summaries, or employees may retain permissions beyond their current duties. In practice this can mean vendor accounts or long-tenured staff inadvertently access highly sensitive PII or move money outside business hours. Hard-coding these constraints in application logic is error-prone and doesn’t scale: roles must be manually created and maintained for every combination of user type, data sensitivity, and environment. This gap invites security incidents (for example, breaches via third-party vendors) unless mitigated by dynamic controls ￼.

**Key risks without PBAC:**
• Over-broad vendor roles (e.g. one “vendor” role for all APIs) or tangled cross-role assignments.
• Static roles that don’t expire or adjust, leading to privilege creep for employees as they change jobs.
• Inability to enforce context (time, device, geography, dev/prod) in access decisions.
• Complex manual policy management that lags behind application changes (bad for compliance).

## RBAC vs Fine-Grained PBAC

Traditional RBAC assigns permissions to coarse roles (“Teller”, “Manager”, “Vendor*Analyst”). That model is inflexible: roles quickly proliferate as you try to cover every department or use case. By contrast, Policy-Based Access Control (PBAC) (often implemented as ABAC) makes decisions dynamically using rules and attributes ￼. Instead of “Alice is a Manager, so she sees everything in Finance,” a PBAC rule might say “allow access if user.department == Finance and resource.classification != ‘Top Secret’ and it’s between 9am–6pm.” This lets the bank express rich conditions: for example, “Deny a vendor account from querying production customer PII after hours” or “Allow a branch manager to approve loans only for their own branch” – without creating dozens of fixed roles. In effect, PBAC provides fine-grained authorization: policies can combine user attributes (department, role, tenure), resource tags (data sensitivity, project), and environmental context (network location, time, risk score) ￼ ￼.
• \_Example:* An Azure AD rule-condition might enforce that “John can only read customer records if the record’s tag = ProjectX” ￼. Similarly, OPA policies in Rego can reference claims like input.user.roles or input.resource.tags to allow or deny each API call.
• _Zero Trust Alignment:_ PBAC aligns with zero-trust principles by never trusting a user by default – every request is evaluated on its attributes. This helps satisfy regulatory requirements (SoX/PCI/DORA) by ensuring only necessary access is granted.

Overall, PBAC (policy-as-code) decouples authorization logic from applications ￼. Policy rules are managed separately (in Git repos) and evaluated at runtime by a policy engine. This enables continuous audits and rapid updates: change a Rego rule in Git, and the new policy takes effect across all services.

## GitOps Workflow with OPA on Kubernetes

The bank uses GitLab and GitOps (e.g. ArgoCD or Flux) to manage both application deployments and access policies. In practice, developers and security engineers store code and Rego policy files in GitLab repositories. For each service (e.g. the Transaction API), a Git merge triggers a CI/CD pipeline that includes automated policy checks: a tool like Conftest (OPA) or custom CI scripts runs Rego rules against new manifests or API definitions. Only if all policies pass will the change be merged and deployed.

An OPA policy engine (Rego) pulls rules from source control and evaluates queries from applications to produce allow/deny decisions. In GitOps, policies are stored as code in Git and deployed with apps. In this example, policy rules in Git and live request data flow into an OPA engine that outputs authorization decisions.

Once changes are merged to the main branch, ArgoCD (or another GitOps operator) automatically syncs the Kubernetes cluster. This means new application images and configurations, and updated OPA policies, are rolled out together. In our bank scenario:
• **Kubernetes setup:** The cluster runs the banking microservices and OPA. We deploy OPA as an admission controller (e.g. Gatekeeper) to enforce Kubernetes-level constraints (for example, preventing new pods from using privileged containers), but more critically we integrate OPA at the application layer. Each microservice includes a lightweight OPA sidecar or invokes the OPA REST API. When a user calls an API, the service passes the request context (user claims, request method, resource) to OPA for decision. OPA evaluates the preloaded Rego policy bundle and returns allow/deny in milliseconds ￼. The service then proceeds or rejects the action.
• **Identity integration:** Users (employees or external contractors) authenticate via Azure Active Directory (Entra ID). Vendor accounts can be invited as Azure AD B2B guests ￼. Applications receive JWT tokens with Azure AD claims (groups, roles, department, vendorID, etc). These claims are forwarded to OPA as part of the input document. For example, input.user.groups might include “FinanceManagers” or “ExternalVendorX”.
• **Policy-as-code in Git:** All Rego modules (OPA policies) live in a GitLab repo under a directory like policies/. For instance, one Rego rule might check input.user.attributes.Role == "Internal" and input.resource.classification == "Sensitive" to decide access. Because policies are just code, they go through the same Git review process as application code. This means any change (adding a new vendor, tightening a rule) is tracked, peer-reviewed, and auditable.

**GitLab CI and OPA:** Additionally, the bank runs policy tests in CI: before merging, a GitLab CI job uses OPA (or Conftest) to simulate requests and ensure they are allowed or denied as expected. This “shifts left” the enforcement, catching policy violations early. Once merged, the policies are deployed to the cluster, where the OPA engine keeps them in memory and uses them for live traffic ￼.

## Policy Decision Factors

In this scenario the PBAC rules consider multiple dimensions:
• **User attributes (from Azure AD):**
• _Type of user:_ Employees vs contractors vs guest vendors. Vendors have a specific claim (input.user.isExternal == true or a vendorID). Internal staff have attributes like department (Finance, IT, Audit) and role (Analyst, Manager, RiskOfficer).
• _Role & group membership:_ Azure AD groups (e.g. “FinanceManagers”, “OpsAuditors”) or custom extension attributes (like ClearanceLevel). These feed into policies, e.g. only “OpsAuditors” can view audit logs or full customer IDs.
• _User risk context:_ Possibly factors such as whether MFA was used, device compliance, or Azure AD Identity Protection risk level. A rule might say “if input.user.risk > High then deny sensitive data access.”
• **Environment and context:**
• _Cluster/namespace (env):_ The bank may have separate dev, test, and prod namespaces. A policy can check input.env so that, for example, external vendors are allowed in dev but automatically blocked from prod data.
• _Time, location, IP:_ Rules can incorporate time-based or IP-based conditions. E.g. “Allow large-amount transfers only during banking hours and from trusted IPs.”
• _Application metadata:_ The application can label resources (or include in request context) things like data classification (“PII”, “Confidential”, “Public”) or regulatory category. A policy might require classification = “Public” for vendors.
• **Resource attributes:**
• _Data sensitivity tags:_ Each financial data resource (a database row, or API endpoint) might carry a tag. For instance, a transaction endpoint could specify resource.classification: "Sensitive", which the policy checks. Internal users might access “Sensitive” data, but a rule can require input.user.groups to include “BankTeller” or “FraudAnalyst”.
• _Operations allowed:_ The policy can distinguish read vs write. Example: a rule could allow any teller to view a transaction in their branch, but only a branch manager to approve or flag it.

Sample pseudo-policy snippet (Rego-like logic):

```
allow {
    input.method == "GET"
    input.path == ["api","transactions",txn_id]
    # Only allow if user is internal and in same branch
    input.user.department == "Finance"
    input.user.branch == input.resource.branch
}
allow {
    input.method == "POST"
    input.path == ["api","transactions","report"]
    # Vendors can only post to reporting API, not raw data
    input.user.isExternal == true
    input.app == "FraudReportingService"
}
deny {
    input.resource.classification == "PII"
    input.user.clearance < 5
}
```

By mixing these factors, OPA implements bank policy rules that go well beyond simple roles. In effect, it enforces a zero-trust, need-to-know policy at the application layer.

Auditing and Monitoring

Every access request and policy decision is logged for audit. OPA’s decision logs record each authorization query, including the input (user, resource, action) and which rule fired ￼. These logs can be shipped (e.g. via OPA’s remote logging plugin) to a central logging system or SIEM. A log entry might look like “User X (FinanceManager) requested GET /api/transactions/12345 – decision=allowed by rule FinanceManager-Access”. Because OPA can tag logs with bundle and policy version, it’s easy to trace exactly which policy version was applied.
• **OPA Decision Logs:** Configured via OPA’s management API, decision logs can be sent to an HTTP endpoint or console ￼. In Kubernetes, logs might go to stdout and be collected by Fluentd/Datadog. Security teams can then query these logs to verify compliance (“Did any vendor ever access production data?”) or detect anomalies (“Denied requests outside business hours”).
• **Azure AD Logs:** All Azure AD sign-in and audit logs are exported via Azure Monitor or sent to a SIEM ￼. This covers authentication events and group/role changes. For example, if an employee’s manager role is revoked in AD, that event is recorded. Correlating AD logs with OPA logs lets auditors reconstruct who accessed what and when.
• **Kubernetes Audit Logs:** At the cluster level, Kubernetes audit logging captures changes to configuration (e.g. who deployed a new version of the Transaction API) and can integrate with OPA Gatekeeper logs. Together with OPA’s application-layer logs, this gives a full chain of custody.
• **Alerts and Dashboards:** The bank can set up alerts (in Azure Sentinel or Splunk) on important patterns from these logs. For instance, an alert if an external (vendor) account is ever allowed into a high-sensitivity endpoint, or if unusually large data queries are denied by policy.

In summary, the PBAC setup enables visibility and auditability: every access decision is made by code and logged. This satisfies auditors that “segregation of duties” and least-privilege are enforced by design.

## Benefits in Practice

In this example, PBAC with OPA/Rego solves the bank’s business and security problems: it enforces consistent, fine-grained access rules across all apps; it prevents misconfiguration of vendor permissions; and it adapts quickly to policy changes. Compared to a brittle RBAC system, the PBAC approach reduces manual role administration and gives Security teams a single place (Rego policies in Git) to reason about access. It also dovetails with Azure AD and Kubernetes native controls: Azure AD does the authentication and broad role assignments, while OPA enforces application-specific rules that RBAC cannot express.

By incorporating OPA into a GitOps workflow, the bank ensures policies are version-controlled, reviewed, and deployed atomically with code. Engineers write policy tests alongside feature code, and every policy change flows through the same CI/CD process. When the rules finally run in the cluster, OPA evaluates them in real time and logs all decisions for audit. This shifts authorization out of application code and into a flexible engine ￼. The end result is a robust authorization framework where, for example, an external fraud analysis firm can safely query only approved data in test environments, while bank employees’ access to production data is tightly gated by branch, role, and time-of-day.

## Sources

- **Policy-Based Access Control (PBAC)** allows dynamic, rule-based access decisions based on user attributes, resource tags, environment, and context, unlike traditional RBAC:

  - [NIST SP 800-162: Guide to Attribute-Based Access Control (ABAC)](https://csrc.nist.gov/publications/detail/sp/800-162/final)

- **Open Policy Agent (OPA)** is a lightweight, general-purpose policy engine that decouples policy from code and supports Rego for fine-grained access logic:

  - [Open Policy Agent – Official Documentation](https://www.openpolicyagent.org/docs/latest/)

- **OPA Decision Logging** provides audit trails of policy evaluations and is essential for traceability in financial services:

  - [OPA Decision Logs](https://www.openpolicyagent.org/docs/latest/management/#decision-logs)

- **GitOps & Policy-as-Code** tools like GitLab CI/CD, ArgoCD, and Conftest enable version-controlled, testable policy deployment pipelines:

  - [Styra Academy – GitOps for Policy Management](https://academy.styra.com/)
  - [Conftest – Testing Configuration Files using Rego](https://www.conftest.dev/)

- **Azure Active Directory (Entra ID)** is used for identity management and supports claims-based access and external collaboration:

  - [Azure AD Conditional Access](https://learn.microsoft.com/en-us/azure/active-directory/conditional-access/overview)
  - [Azure AD Logs to SIEM](https://learn.microsoft.com/en-us/azure/active-directory/reports-monitoring/concept-activity-logs-azure-monitor)

- **Kubernetes Policy Enforcement** with OPA and Gatekeeper for infrastructure-level controls:

  - [OPA Gatekeeper Project](https://github.com/open-policy-agent/gatekeeper)

- **Zero Trust Security Models** align with PBAC principles by enforcing least-privilege and just-in-time access:
  - [Microsoft Zero Trust Maturity Model](https://www.microsoft.com/security/blog/2021/06/14/the-new-microsoft-zero-trust-maturity-model/)
