# LAB 61 – Cloud Advisor | OCI Resource Optimization

## 1. Objective

Understand how Oracle Cloud Infrastructure (OCI) Cloud Advisor identifies opportunities to reduce cost, improve resource performance, and strengthen availability—and how an administrator reviews and acts on its recommendations.

## 2. What is OCI Cloud Advisor?

**OCI Cloud Advisor is a tenancy-level optimization service that analyzes OCI resources and provides actionable recommendations to help optimize cost, performance, and high availability.**

It helps administrators identify underutilized or misconfigured resources and take informed corrective actions.

## 3. How it works – high-level flow

```text
OCI Tenancy
    |
    v
OCI Cloud Advisor (periodic analysis)
    |
    +--> Cost Management recommendations
    |
    +--> Performance recommendations
    |
    +--> High Availability recommendations
    |
    v
Administrator reviews recommendation and affected resources
    |
    v
Validate impact, dependencies and change requirements
    |
    +--> Implement through Cloud Advisor (where supported)
    |
    +--> Perform the change manually / through automation
    |
    v
Review work request and recommendation status
```

## 4. Prerequisites

- Access to the OCI Console and the target tenancy.
- Appropriate IAM permissions to view recommendations and the affected resources.
- Additional compartment- or resource-level permissions may be required to view resource details or implement certain recommendations.
- Follow organizational change-management and approval procedures before modifying production resources.

Example policy for an administrator group:

```text
Allow group OptimizerAdmins to manage optimizer-api-family in tenancy
```

Use least-privilege policies and validate the permissions required for the specific recommendation.

## 5. Step-by-step implementation

### Step 1 – Open Cloud Advisor

1. Sign in to the OCI Console.
2. Open the **Navigation menu**.
3. Go to **Governance & Administration → Cloud Advisor**.
4. Open the **Recommendations** page.

### Step 2 – Review recommendations

Review the recommendations shown for the tenancy. Open a recommendation to understand:

- What issue or optimization opportunity was identified.
- Which OCI resources are affected.
- The suggested action.
- Any estimated cost benefit or operational impact shown.

### Step 3 – Understand the recommendation categories

| Category | What it helps identify | Examples |
|---|---|---|
| **Cost Management** | Opportunities to reduce unnecessary spend | Underutilized Compute instances, idle resources, unattached Block Volumes, storage lifecycle improvements |
| **Performance** | Opportunities to improve resource utilization and performance | Compute or database right-sizing, Load Balancer sizing, Block Volume performance auto-tuning |
| **High Availability** | Opportunities to improve resilience and data protection | Fault-tolerance improvements, Object Storage versioning and replication |

**Security note:** Cloud Advisor can surface built-in Cloud Guard recommendations that help address security vulnerabilities. Security is not a separate top-level Cloud Advisor recommendation category; use Cloud Guard for security posture findings.

### Step 4 – Filter and inspect

1. Filter recommendations by **Category**, **Service**, and **Status**.
2. Narrow the results by compartment, tag, or region where those filters are available.
3. Open the recommendation and inspect its affected resources.
4. Download the filtered recommendation list as CSV if required for reporting.

### Step 5 – Implement a recommendation

Where the recommendation supports implementation from Cloud Advisor:

1. Open the recommendation.
2. Go to **Resource recommendations**.
3. Select the resources you intend to change.
4. Select **Implement selected**.
5. Review the proposed changes and their impact.
6. Confirm **Implement**.
7. Monitor the associated work request until it completes.

Some recommendations require administrators to perform the change manually or through their existing automation process. Follow the instructions provided with the individual recommendation.

> **Production safety:** Do not delete, resize, or modify a production resource solely because Cloud Advisor recommends it. Validate utilization history, application dependencies, capacity requirements, backup and recovery readiness, maintenance windows, and change approvals first.

### Step 6 – Track recommendation status

Review the recommendation status and history to track the outcome.

| Status | Meaning |
|---|---|
| **Pending** | Recommendation is awaiting action or status confirmation |
| **Implemented** | Recommendation has been marked as implemented |
| **Postponed** | Action has been deferred |
| **Dismissed** | Recommendation has been dismissed |

For changes performed outside Cloud Advisor, the status may update after a subsequent scan or after the recommendation is otherwise refreshed.

## 6. Important operational points

- Cloud Advisor scans the tenancy once per day.
- Some recommendations can be generated immediately; others require up to seven days of resource data.
- CPU-based Compute recommendations depend on Compute Agent monitoring being enabled and supplying the required metrics.
- Estimated savings are estimates, not guaranteed savings.
- Not every recommendation can be implemented directly from the Cloud Advisor page.
- Recheck the recommendation and resource state after implementation.

## 7. Cloud Advisor vs. related OCI services

| OCI service | Primary purpose |
|---|---|
| **Cloud Advisor** | Recommends cost, performance, and availability optimizations |
| **Cloud Guard** | Detects security posture issues and risky configurations |
| **OCI Monitoring** | Collects metrics and supports alarms for operational monitoring |
| **Cost Analysis** | Helps analyze and report OCI spending and usage costs |

## 8. Administrator validation checklist

- [ ] Opened Cloud Advisor in the correct tenancy.
- [ ] Reviewed Cost Management recommendations.
- [ ] Reviewed Performance recommendations.
- [ ] Reviewed High Availability recommendations.
- [ ] Checked the affected compartments and resources.
- [ ] Validated application impact, dependencies, and capacity.
- [ ] Confirmed backup, recovery, and change approvals where applicable.
- [ ] Implemented or deferred the recommendation appropriately.
- [ ] Checked the work request and final recommendation status.
- [ ] Recorded the result for operational reporting.

## 9. Interview questions

**Q1. What is OCI Cloud Advisor?**  
OCI Cloud Advisor analyzes tenancy resources and provides recommendations to optimize cost, performance, and high availability.

**Q2. What are its main recommendation categories?**  
Cost Management, Performance, and High Availability.

**Q3. Does Cloud Advisor automatically change all OCI resources?**  
No. It provides recommendations. Some actions can be implemented through Cloud Advisor, while others require manual or automated changes by an administrator.

**Q4. How frequently does Cloud Advisor scan the tenancy?**  
It scans the tenancy once per day.

**Q5. Why might a Compute CPU recommendation not appear?**  
CPU-based recommendations require Compute Agent monitoring to be enabled and the necessary metrics to be available.

**Q6. Is estimated cost saving guaranteed?**  
No. It is an estimate. Actual savings depend on resource usage, configuration, and the changes implemented.

**Q7. What should an administrator verify before implementing a recommendation?**  
Resource utilization, application dependencies, capacity, backup and recovery, business impact, and change approvals.

## 10. Expected outcome

After completing this lab, an OCI Administrator should be able to navigate Cloud Advisor, interpret its recommendation categories, identify affected resources, assess operational risk, implement supported recommendations, and track their status.

## 11. Official Oracle documentation

- [About Cloud Advisor](https://docs.oracle.com/en-us/iaas/Content/CloudAdvisor/Concepts/cloudadvisoroverview.htm)
- [Categories and Recommendations](https://docs.oracle.com/en-us/iaas/Content/CloudAdvisor/Concepts/recommendations.htm)
- [Viewing Categories and Recommendations](https://docs.oracle.com/en-us/iaas/Content/CloudAdvisor/Tasks/view-recommendations.htm)
- [Implementing Cloud Advisor Recommendations](https://docs.oracle.com/en-us/iaas/Content/CloudAdvisor/Tasks/implement-recommendations.htm)

---
**Lab:** 61  
**Service:** Oracle Cloud Infrastructure – Cloud Advisor  
**Focus:** Cost Optimization | Performance Optimization | High Availability
