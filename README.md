<div align="center">

# 🛡️ AWS WAF — Complete Setup Guide

### Day 6 | AWS Zero to Hero · Cloud & DevOps Learning Journey

**Protect web applications from common web exploits with AWS Web Application Firewall.**

<p>
  <img src="https://img.shields.io/badge/AWS-WAF-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white" alt="AWS WAF" />
  <img src="https://img.shields.io/badge/Focus-Web%20Security-6A1B9A?style=for-the-badge" alt="Web Security" />
  <img src="https://img.shields.io/badge/Notes-Cornell%20Method-1565C0?style=for-the-badge" alt="Cornell Notes" />
  <img src="https://img.shields.io/badge/Level-Beginner%20to%20Intermediate-2E7D32?style=for-the-badge" alt="Beginner to Intermediate" />
</p>

</div>

---

## 📌 Table of Contents

- [Overview](#-overview)
- [Learning Objectives](#-learning-objectives)
- [What Is AWS WAF?](#-what-is-aws-waf)
- [How AWS WAF Works](#-how-aws-waf-works)
- [Architecture](#-architecture)
- [Core Components](#-core-components)
- [Common Use Cases](#-common-use-cases)
- [Step-by-Step Setup with an ALB](#-step-by-step-setup-with-an-alb)
- [Example: Rate-Based Rule](#-example-rate-based-rule)
- [Logging and Monitoring](#-logging-and-monitoring)
- [Security Best Practices](#-security-best-practices)
- [Testing Checklist](#-testing-checklist)
- [Troubleshooting](#-troubleshooting)
- [Key Takeaways](#-key-takeaways)
- [Useful Resources](#-useful-resources)

---

## 🎯 Overview

**AWS WAF (Web Application Firewall)** helps protect web applications and APIs from common web exploits and unwanted HTTP/HTTPS traffic. You create a **Web ACL (Access Control List)**, add rules, and associate the Web ACL with a supported AWS resource.

This guide covers the concepts and a practical setup workflow for protecting an application behind an **Application Load Balancer (ALB)**.

> **Learning note:** This is a hands-on study guide. Adapt rule thresholds, scope, region, and actions to your own application before using them in production.

## 🧠 Learning Objectives

By the end of this guide, you should be able to:

- Explain what AWS WAF does and where it fits in a web architecture.
- Understand Web ACLs, rules, rule groups, managed rule groups, and IP sets.
- Create a Web ACL and associate it with an ALB.
- Configure rate-based and IP-based protections.
- Enable logging and review AWS WAF metrics.
- Test rules safely and tune them to reduce false positives.

---

## 🔐 What Is AWS WAF?

AWS WAF is a **web application firewall** that inspects HTTP/HTTPS requests and evaluates them against rules you configure.

It can help protect against patterns associated with:

- SQL injection
- Cross-site scripting (XSS)
- Automated bots and unwanted traffic
- Excessive request rates
- Requests from specified IP addresses or ranges

AWS WAF evaluates requests using a Web ACL. Depending on the rule and its action, a request can be **allowed, blocked, counted, challenged, or CAPTCHA-checked**. Available actions depend on the rule type and configuration.

**Important:** AWS WAF complements—not replaces—secure application code, authentication and authorization, patching, network controls, and monitoring.

## ⚙️ How AWS WAF Works

1. A user sends an HTTP/HTTPS request to your application.
2. The request reaches the AWS resource associated with a Web ACL.
3. AWS WAF evaluates the request against the Web ACL's rules in priority order.
4. The matching rule's action is applied, subject to rule evaluation and terminating/non-terminating behavior.
5. You review logs and metrics to understand traffic and tune the rules.

### Request flow

```text
Client / User
     |
     v
Internet
     |
     v
AWS WAF (Web ACL + Rules)
     |
     v
Application Load Balancer (ALB)
     |
     v
Application / Target Group
```

AWS WAF can also protect other supported resources, including Amazon CloudFront distributions, Amazon API Gateway REST APIs, AWS AppSync GraphQL APIs, and Amazon Cognito user pools. **Scope and region requirements vary by resource type**; CloudFront-scoped Web ACLs are managed through the required global scope, while regional resources use their applicable AWS Region.

---

## 🧩 Core Components

| Component | What it does |
|---|---|
| **Web ACL** | A collection of rules used to inspect and control web requests for an associated resource. |
| **Rule** | Defines a condition to inspect requests and an action to take when the condition matches. |
| **Rule group** | A reusable collection of rules. |
| **AWS Managed Rule groups** | Rule groups maintained by AWS to help detect common threat patterns. Review each group's behavior and cost before enabling it. |
| **Custom rules** | Rules you create for your application's specific requirements. |
| **IP set** | A reusable list of IPv4 and/or IPv6 addresses or CIDR ranges. |
| **Rule action** | Common actions include Allow, Block, Count, CAPTCHA, and Challenge, subject to rule support and configuration. |
| **Logging** | Records request details for investigation and tuning when logging is configured. |
| **Metrics and alarms** | CloudWatch metrics and alarms help monitor request patterns and rule activity. |

### Rule priority

Rules are evaluated according to their configured priority, generally from the **lowest numeric priority to the highest**. A terminating action such as Allow or Block can stop further evaluation. Review AWS documentation for the behavior of each action and rule group.

---

## 🧪 Common Use Cases

| Threat or requirement | Possible AWS WAF approach |
|---|---|
| SQL injection patterns | AWS Managed Rules for SQL injection or carefully tested custom rules |
| Cross-site scripting (XSS) | AWS Managed Rules for common threats or custom match conditions |
| Excessive requests | Rate-based rule |
| Known unwanted IP addresses | IP set referenced by a rule |
| Geographic restrictions | Geo-match statement, where appropriate for your business and compliance needs |
| Suspicious automation | AWS Managed Bot Control or other bot-management rules, subject to feature availability and pricing |
| Observe before enforcing | Count mode and log review |

No single rule can guarantee protection against every attack. Test rules against your application's real traffic and requirements.

---

## 🚀 Step-by-Step Setup with an ALB

This example outlines how to protect an existing application behind an Application Load Balancer.

### Step 1 — Check prerequisites

Before starting, confirm that:

- You have access to the AWS Management Console and permission to configure AWS WAF.
- Your ALB and target application are deployed and healthy.
- You know the ALB's AWS Region.
- You understand which requests should be allowed and which should be blocked.

**Safety first:** Start with Count mode where practical, review logs, and only then move to Block or other enforcement actions.

### Step 2 — Open AWS WAF

1. Sign in to the AWS Management Console.
2. Open **AWS WAF & Shield**.
3. Choose **Web ACLs** and start creating a Web ACL.
4. Select the scope appropriate for your protected resource.
5. For an ALB, select the **regional** scope and the Region where the ALB exists.

### Step 3 — Create the Web ACL

1. Give the Web ACL a clear name, such as `my-app-web-acl`.
2. Select the applicable resource type and Region.
3. Configure a default action. Choose this based on your intended security policy; do not blindly set a default Block on a live application.
4. Continue to rule configuration.

**Naming tip:** Use a consistent pattern such as `environment-application-purpose`, for example `prod-shop-web-acl`.

### Step 4 — Add AWS Managed Rules

1. Add an AWS Managed Rule group suitable for common web threats.
2. Review the rule group's documentation, included rules, exclusions, and pricing.
3. Consider starting in **Count** mode or using rule overrides while you validate legitimate traffic.
4. Add only the rule groups that fit your application's needs.

Examples of commonly used AWS managed rule group names include:

- `AWSManagedRulesCommonRuleSet`
- `AWSManagedRulesSQLiRuleSet`
- `AWSManagedRulesKnownBadInputsRuleSet`
- `AWSManagedRulesAmazonIpReputationList`
- `AWSManagedRulesAnonymousIpList`
- `AWSManagedRulesBotControlRuleSet` (additional charges may apply)

Availability, behavior, names, and costs can change. Check the current AWS documentation and console before deploying.

### Step 5 — Add custom rules

Examples of useful custom protections:

- **Rate-based rule:** Detects clients sending requests above a configured rate threshold.
- **IP set rule:** Blocks or allows specified IPv4/IPv6 addresses or CIDR ranges.
- **Geo-match rule:** Matches requests based on country of origin when appropriate.
- **Path or header rule:** Matches selected URI paths, headers, or other supported request attributes.

Be careful with shared networks, NAT gateways, corporate proxies, and mobile carriers: many legitimate users can share a public IP address.

### Step 6 — Set priorities and actions

1. Review the order of your rules.
2. Use Count mode while investigating uncertain matches.
3. Use Block only after validating that legitimate requests will not be disrupted.
4. Confirm the default action matches your security policy.
5. Save the Web ACL configuration.

### Step 7 — Associate the Web ACL with the ALB

1. In the Web ACL configuration, find the associated AWS resources section.
2. Add the target Application Load Balancer.
3. Confirm the ALB and Web ACL are in the correct Region.
4. Save and verify the association.

### Step 8 — Enable logging

1. Open the Web ACL's logging and metrics settings.
2. Choose a supported logging destination, such as Amazon CloudWatch Logs, Amazon S3, or Amazon Data Firehose, as available for the configuration.
3. Set up any required destination permissions and resource naming.
4. Review logging redaction and data-handling requirements before collecting request details.
5. Generate test traffic and confirm logs arrive.

### Step 9 — Test and monitor

1. Test normal application flows, including sign-in, forms, search, and API calls.
2. Review AWS WAF logs for matching rules and actions.
3. Check CloudWatch metrics for allowed, blocked, counted, and other relevant requests.
4. Tune rules if legitimate traffic is matched unexpectedly.
5. After validation, move suitable rules from Count to enforcement actions.

---

## 🚦 Example: Rate-Based Rule

A rate-based rule can help mitigate bursts of requests from a client. The example below is **illustrative**, not a universal recommended threshold.

> **Example policy:** If a client exceeds 2,000 requests in a 5-minute evaluation window, apply a Block action.

Configuration concept:

| Setting | Example |
|---|---|
| Rule name | `RateLimit-Example` |
| Statement | Rate-based |
| Evaluation window | Select an available window supported by AWS WAF |
| Rate limit | Example: `2000` requests |
| Aggregation | Per source IP for this example |
| Action | Start with Count for testing; use Block only after validation |

**Important details:**

- AWS WAF rate-based rules use a configurable evaluation window and aggregation settings supported by the service; they do not guarantee an exact fixed-window cutoff.
- A rate limit is not a precise per-user identity limit. Clients behind the same NAT or proxy may share an IP.
- The appropriate threshold depends on application traffic, endpoints, and business needs.
- AWS WAF pricing and rate-based rule capabilities may change. Check current documentation before implementation.

---

## 📈 Logging and Monitoring

Use logs and metrics to understand what AWS WAF is doing and whether rules are affecting legitimate users.

### What to monitor

- Allowed and blocked request counts
- Counted requests during rule tuning
- Which rules are matching most often
- Unexpected spikes in traffic
- False positives affecting normal application behavior
- Logging delivery failures and destination permissions

### Monitoring workflow

```text
AWS WAF Web ACL
      |
      +----> CloudWatch Metrics
      |          |
      |          +----> Dashboard / Alarm
      |
      +----> WAF Logs
                 |
                 +----> Investigation
                 +----> Rule tuning
```

Create CloudWatch alarms around meaningful changes in traffic or blocking rates. Set thresholds based on your application's normal baseline rather than copying arbitrary values.

---

## ✅ Security Best Practices

- **Start with managed rules:** Use relevant AWS Managed Rule groups as a baseline, then customize.
- **Use Count mode first:** Observe matches before enforcing uncertain rules.
- **Apply least privilege:** Limit who can create, modify, or delete Web ACLs and logging destinations.
- **Enable logging:** Use an appropriate destination and protect access to logs.
- **Use IP sets carefully:** Keep addresses and CIDR ranges current; avoid broad blocks that affect legitimate users.
- **Tune rate limits:** Base thresholds on real traffic and endpoint behavior.
- **Monitor continuously:** Review metrics, alarms, and logs after changes.
- **Test changes safely:** Validate application flows in a non-production environment when possible.
- **Control costs:** Check current pricing for managed rule groups, bot features, requests, and logging.
- **Use layered security:** Combine WAF with secure coding, identity controls, patching, TLS, and appropriate network configuration.
- **Review changes regularly:** Revisit rules as the application and threat patterns evolve.

---

## 🧾 Testing Checklist

Use this checklist before considering the setup complete:

- [ ] Web ACL created with the correct scope and Region.
- [ ] Intended ALB or other supported resource associated.
- [ ] Default action reviewed and tested.
- [ ] Managed rule groups reviewed and configured.
- [ ] Custom rate-based and IP-based rules tested where needed.
- [ ] Count mode used for uncertain rules before enforcement.
- [ ] Logging destination configured and verified.
- [ ] CloudWatch metrics reviewed.
- [ ] Normal application flows tested.
- [ ] False positives investigated.
- [ ] Permissions and costs reviewed.
- [ ] Configuration documented for future maintenance.

---

## 🛠️ Troubleshooting

| Issue | What to check |
|---|---|
| Web ACL cannot be associated | Verify resource type, scope, Region, IAM permissions, and service support. |
| Legitimate requests are blocked | Inspect WAF logs, identify the matching rule, and consider Count mode or a narrowly scoped exception. |
| Expected requests are not blocked | Check rule priority, statement conditions, action, Web ACL association, and whether the test request matches the rule. |
| Logs are missing | Verify logging configuration, destination permissions, destination requirements, and delivery status. |
| Rate rule triggers unexpectedly | Review traffic baselines, aggregation settings, evaluation window, shared IPs, and application endpoint patterns. |
| Unexpected charges | Review AWS WAF pricing, managed rule groups, bot features, request volume, and logging destination costs. |

---

## 📝 Key Takeaways

- AWS WAF helps protect web applications and APIs by evaluating HTTP/HTTPS requests against rules.
- A **Web ACL** contains the rules used to inspect and control traffic.
- AWS Managed Rules provide a useful baseline; custom rules address application-specific needs.
- Rate-based rules and IP sets can help manage excessive or unwanted traffic.
- Logging and CloudWatch metrics are essential for investigation and tuning.
- Test carefully, avoid unnecessary disruption, and review your protections regularly.

> **Remember:** Security is not a one-time setup—it is an ongoing process of testing, monitoring, and improvement.

---

## 📚 Useful Resources

- [AWS WAF Developer Guide](https://docs.aws.amazon.com/waf/latest/developerguide/what-is-aws-waf.html)
- [AWS WAF Rule Statements](https://docs.aws.amazon.com/waf/latest/developerguide/waf-rule-statements.html)
- [AWS Managed Rules](https://docs.aws.amazon.com/waf/latest/developerguide/aws-managed-rule-groups.html)
- [AWS WAF Logging](https://docs.aws.amazon.com/waf/latest/developerguide/logging.html)
- [AWS WAF Pricing](https://aws.amazon.com/waf/pricing/)

---

<div align="center">

### ☁️ Cloud & DevOps Learning Journey

**Day 6 — AWS WAF: Complete Setup & Web Application Security**

*Learn • Build • Test • Monitor • Improve*

</div>
