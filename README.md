#  Building a secure AWS Environment from the get-go

A guide to building an AWS environment with security built-in from the start: locking down the root account, applying org-wide guardrails, and designing a centralized, multi-account security architecture.

---

## 1. Secure the root account

When you create a brand-new AWS account, you automatically inherit an all-powerful root user.
With great power comes great responsibility, so lock that account down, and make sure access
to it is closely monitored and limited to a trusted party. It is essentially the keys to your
cloud kingdom.

**Best practices for the root account**

- Enable MFA.
- Use a strong, unique password.
- Use a dedicated, company-managed email address, and forward messages to a group of users so they're actioned promptly.
- Store root credentials in an AWS-independent system; monitor and log access to them.
- Use it only for the [tasks that genuinely require the root user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_root-user.html#root-user-tasks).
- Require multi-person approval for root sign-in: e.g. one person holds the password, another holds the MFA device.
- Perform daily administration with IAM users or roles, never with root.
- Do not deploy any workload in this account.
- Do not create access keys for the root user.
- Restrict recovery mechanisms: if you don't store the root password and MFA, make sure the registered phone number and email address stay under your governance and access at all times.

---

## 2. Enable AWS Organizations

Turn this on early as it makes management easier as you scale.

**When to use it**

- You run several accounts grouped by purpose and requirements.
- You want to apply guardrails to member accounts using Service Control Policies (SCPs) and Resource Control Policies (RCPs).

**Watch out for the management account**

- SCPs and RCPs do **not** apply to the management account, so never run any workload there.

**How to set it up**

- Enable **all features** in AWS Organizations (not just consolidated billing).
- Centrally manage root access for member accounts.
- Design your OU structure around workloads that share constraints and requirements: e.g. Financial, Security, Dev, Prod...
- Design your SCPs and RCPs as guardrails adapted to your environment. Start from a minimal baseline set ([example](#) <!-- add link -->).

---

## 3. Design a centralized security architecture

The centralized security architecture rests on the following AWS services:

| Service | Role |
|---|---|
| **AWS Organizations** | Centrally manage accounts and guardrails |
| **Amazon GuardDuty** | Threat detection |
| **Amazon CloudWatch** | Real-time monitoring of cloud apps and infrastructure |
| **AWS CloudTrail** | Records every API call across your accounts |
| **Amazon EventBridge** | Automate responses and generate alerts on specific activity |
| **AWS Config** | Evaluates resource changes against compliance rules |
| **AWS Security Hub** | Aggregates findings (GuardDuty, Config, …) into a unified view |

You *could* run all of this from a single security account inside your Security OU, but least
privilege and blast-radius reduction argue against that. Split the responsibilities across
dedicated accounts instead:

| Account | Purpose |
|---|---|
| **IAM** | Delegated administrator for IAM Identity Center : centralized human access |
| **SecurityAudit** | Audit & posture-management hub: cross-account scanning tools (e.g. Prowler) and delegated admin for AWS Security Hub |
| **LogArchive** | Hardened, dedicated repository consolidating CloudTrail and Config data. Protects log integrity |
| **SecurityOperations** | Incident-response command center: analyze active threats, triage alerts, and run containment/remediation |

---

# 4. Set up human access with IAM Identity Center

Now that the accounts exist, people need a safe way in. Don't hand out long-lived IAM users and
access keys because they're static secrets that can leak. Use **IAM Identity Center** to hand out short-lived credentials from a single front door.

**How to set it up**

- Delegate administration of IAM Identity Center to your dedicated **IAM** account
- Connect an identity source: the built-in Identity Center directory, or your corporate IdP (e.g. Entra ID, Okta) via SAML for sign-in and SCIM for automatic user/group provisioning.
- Define **permission sets** (bundles of policies) scoped to job functions: they're provisioned as short-lived IAM roles in each target account.
- Assign permission sets to **groups**, not individuals, and map them to accounts/OUs so access follows your org structure.
- Enforce **MFA** for every user.

**Watch out for**

- Overly-broad permission sets

---

## References

- AWS Security Reference Architecture (SRA)
- [AWS root user tasks](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_root-user.html#root-user-tasks)
