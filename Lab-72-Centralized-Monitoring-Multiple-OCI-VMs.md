# Lab-72-Centralized-Monitoring-Multiple-OCI-VMs

**Category:** OCI Monitoring & Observability  
**Level:** Intermediate — OCI Administrator (5+ years)  
**Objective:** Centralize visibility of multiple OCI compute instances using OCI Monitoring, dashboards, alarms, and notifications.

---

## 1. Lab objectives

- View CPU utilization for multiple OCI VMs from one dashboard.
- Understand the difference between platform metrics and guest OS metrics.
- Configure dashboard widgets for quick operational visibility.
- Create monitoring alarms and route alarm messages through Notifications.
- Troubleshoot missing metrics, IAM access, agent telemetry, and notification delivery.

## 2. Architecture

```text
 OCI Compute VM-01 ─┐
 OCI Compute VM-02 ─┼──> OCI Monitoring (metric namespace: oci_computeagent / oci_compute, as applicable)
 OCI Compute VM-03 ─┘                         │
                                             ├──> OCI Monitoring Dashboard
                                             │      ├── CPU utilization
                                             │      ├── Memory utilization*
                                             │      └── Per-instance metric views
                                             │
                                             └──> Monitoring Alarms
                                                       │
                                                 OCI Notifications Topic
                                                       │
                                                Email / HTTPS / supported
                                                subscription protocol

*Memory and filesystem metrics require guest-level telemetry, commonly via the Oracle Cloud Agent
Compute Instance Monitoring plugin or Management Agent/custom telemetry, depending on the metric.
```

**Important:** OCI platform monitoring provides standard compute metrics such as CPU utilization. Memory and filesystem usage are guest OS metrics and are not automatically available from basic platform metrics alone. Confirm the metric namespace, agent/plugin prerequisites, and supported dimensions for the telemetry source you use.

## 3. Prerequisites

1. An OCI tenancy and permission to view the target compute instances.
2. Two or more running compute instances in a supported region.
3. Access to OCI Monitoring and Dashboards.
4. For guest metrics, the relevant monitoring agent/plugin enabled and running, with required network access and permissions.
5. An email address or other supported endpoint for alarm notifications.

Use a non-production environment for testing and avoid creating very frequent alarms without reviewing costs and operational impact.

## 4. Step A — Validate CPU metrics

1. Sign in to the OCI Console and select the region containing your instances.
2. Open **Observability & Management → Monitoring → Metrics Explorer** (menu labels can vary).
3. Select the appropriate compartment.
4. Choose the compute metric namespace available for the instance:
   - **Platform metrics:** typically `oci_compute_infrastructure_health` for infrastructure health metrics and `oci_compute` for standard compute metrics, depending on the metric and console experience.
   - **Guest/agent metrics:** commonly `oci_computeagent` when the Compute Instance Monitoring plugin publishes guest metrics.
5. Select the metric **CPU Utilization** (metric name commonly `CpuUtilization`).
6. Set a time range such as the last 1 hour and an aggregation interval such as 1 minute or 5 minutes.
7. Filter/group by the instance resource dimension (often `resourceId`) to distinguish the VMs.
8. Confirm each expected instance has recent data.

If the namespace or metric is not listed, verify the compartment, region, instance state, metric dimensions, and telemetry source before proceeding.

## 5. Step B — Enable guest-level metrics for memory/filesystem

1. Open **Compute → Instances → [instance] → Oracle Cloud Agent**.
2. Locate **Compute Instance Monitoring** and enable it where supported.
3. Wait several minutes, then refresh the agent/plugin status. An **Enabled** configuration does not necessarily mean the plugin is actively running.
4. Check the instance's OS and agent logs if the plugin remains stopped or metrics do not appear.
5. Return to Metrics Explorer and look for the relevant guest metric namespace and supported memory/filesystem metrics.
6. Repeat for each instance, or use an approved automation method for consistent configuration.

Exact metric names and availability can vary by agent/plugin version and operating system. Use the metric list shown in your tenancy and the current Oracle documentation rather than assuming every guest metric is present.

## 6. Step C — Create a centralized dashboard

1. Open **Observability & Management → Dashboards**.
2. Create a dashboard in the desired compartment, or open an existing dashboard you are authorized to edit.
3. Add a widget and choose a metric/chart visualization.
4. Configure the widget with the correct region, compartment, metric namespace, metric name, time range, and aggregation.
5. Group or filter by instance resource ID so each VM can be compared.
6. Add separate widgets for CPU, memory, and filesystem metrics where those metrics are available.
7. Give each widget a clear title, for example:
   - `CPU Utilization — All Lab VMs`
   - `Memory Utilization — Agent Metrics`
   - `Filesystem Usage — Agent Metrics`
8. Save the dashboard and verify that the data refreshes for the intended time range.

**Dashboard design tip:** Keep CPU and memory as separate widgets if they come from different namespaces or have different dimensions. This makes missing guest telemetry easier to identify.

## 7. Step D — Configure alarms and notifications separately

A dashboard visualizes metrics; it does not automatically notify an operator when a threshold is exceeded.

### Create a Notifications topic and subscription

1. Open **Developer Services → Application Integration → Notifications** (navigation may vary).
2. Create a topic in the appropriate compartment.
3. Create an email subscription and confirm the subscription using the message sent to the recipient.
4. Keep the topic OCID available for alarm configuration.

### Create a Monitoring alarm

1. Open **Observability & Management → Monitoring → Alarm Definitions**.
2. Create an alarm in the appropriate compartment.
3. Select the metric compartment, namespace, metric, and aggregation.
4. Define a threshold appropriate to the workload. Example: CPU utilization greater than 80% over a sustained period.
5. Configure the evaluation interval, trigger delay or pending duration if available, and severity to reduce transient-noise alerts.
6. Under notifications, select the topic created above.
7. Save the alarm and verify its state.

Example threshold concept (not a universal production recommendation):

```text
Metric: CpuUtilization
Condition: mean() > 80
Evaluation period: sustained threshold for several minutes
Destination: OCI Notifications topic
```

Use the alarm query builder and syntax supported by your OCI console. Tune thresholds to the application baseline and operational SLA.

## 8. Validation checklist

- [ ] CPU metric is visible for every intended VM.
- [ ] Instance IDs/dimensions identify the correct VM.
- [ ] Memory/filesystem metrics are present only where guest telemetry is configured.
- [ ] Dashboard is saved and accessible to the intended team.
- [ ] Alarm evaluates without query or permission errors.
- [ ] Email subscription is confirmed.
- [ ] A controlled test or approved threshold simulation confirms end-to-end notification delivery.
- [ ] Alarm returns to its normal state after the condition clears.

Do not deliberately overload production instances just to test an alarm. Prefer a safe test environment or a temporary threshold change approved by the service owner.

## 9. Troubleshooting

| Symptom | Likely causes | Checks / resolution |
|---|---|---|
| CPU metric missing | Wrong region/compartment, wrong namespace, no recent data, incorrect dimensions | Confirm region and compartment; inspect available namespaces and metric dimensions in Metrics Explorer. |
| CPU appears but memory is missing | Guest telemetry/plugin not enabled or running | Check Oracle Cloud Agent plugin status, OS support, and agent logs; confirm the guest metric namespace. |
| Plugin says Enabled but Stopped | Configuration enabled but plugin/service is not running, startup issue, or delayed status update | Refresh after a few minutes; inspect instance agent status/logs; restart only after assessing impact and following operational procedures. |
| Only some VMs appear | Different compartments, plugin versions, instance states, or filters | Compare instance configuration and dashboard filters; validate each VM individually. |
| Dashboard widget is blank | Wrong metric query, time range, compartment, or dimensions | Test the same query in Metrics Explorer and widen the time range. |
| Access denied | Missing IAM policy or wrong compartment scope | Ask an administrator to verify least-privilege policies for Monitoring, Dashboards, and Notifications in the relevant compartments. |
| Alarm is not firing | Threshold not met, aggregation/evaluation settings, query mismatch, or missing metric data | Review alarm history, query, metric data, threshold, and evaluation window. |
| Alarm fires but email is not received | Subscription unconfirmed, wrong topic, email filtering, or notification delivery issue | Confirm subscription, verify alarm topic, inspect notification delivery/status, and check spam/quarantine. |
| Memory values look wrong | Metric unit or aggregation misunderstood | Verify the metric's unit, statistic, dimensions, and time interval before comparing values. |

## 10. IAM considerations

Use least privilege and scope permissions to the smallest practical compartment. Exact policy statements depend on the actions and resources involved, tenancy setup, and whether users only view metrics or also create dashboards, alarms, and topics.

Review access for:
- Reading metrics and monitoring resources.
- Viewing/editing dashboards.
- Creating and managing alarm definitions.
- Creating topics and subscriptions in OCI Notifications.
- Agent/plugin operation and telemetry publishing, where applicable.

Do not copy broad tenancy-wide policies into production without reviewing Oracle's current IAM policy reference.

## 11. Interview questions and answers — 5 years' experience

**Q1. How would you monitor CPU utilization across 20 OCI VMs centrally?**  
Use OCI Monitoring metrics and a dashboard with the correct compartment and namespace. Group or filter by instance resource ID, then create separate alarms for actionable thresholds.

**Q2. Is memory utilization available by default for every OCI VM?**  
No. Standard platform metrics include CPU utilization, but memory and filesystem usage generally require guest-level telemetry, such as the supported Compute Instance Monitoring plugin or another configured agent/telemetry pipeline.

**Q3. What is the difference between a dashboard and an alarm?**  
A dashboard provides a visual summary of metrics. An alarm evaluates a metric query against a condition and can publish a notification when the condition is met.

**Q4. What is the role of an OCI Notifications topic?**  
It is the destination to which alarm messages are published. Subscriptions, such as confirmed email subscriptions, determine where those messages are delivered.

**Q5. A plugin is Enabled but Stopped. What would you check?**  
I would verify the actual plugin state, instance and OS compatibility, agent logs, network reachability to required OCI services, and any applicable permissions. I would not assume that the Enabled setting means telemetry is running.

**Q6. CPU data is visible for one VM but not another. What is your troubleshooting sequence?**  
Compare region, compartment, instance state, metric namespace, dimensions, time range, IAM access, and agent/plugin configuration where guest metrics are involved.

**Q7. How do you avoid alert fatigue?**  
Use workload-based thresholds, sustained evaluation periods, appropriate aggregation, meaningful severity levels, and alarms tied to actionable response procedures.

**Q8. How would you prove that alerting works end to end?**  
Verify metric data, alarm query and state, selected Notifications topic, confirmed subscription, delivery status, and receipt of the test message. Record the test result.

**Q9. Can an OCI dashboard replace an external monitoring platform?**  
It can cover many OCI-native visualization needs, but the decision depends on cross-cloud visibility, log/APM requirements, retention, integrations, alert routing, and operational standards.

**Q10. What evidence would you retain for a monitoring lab?**  
Screenshots of metric queries, dashboard widgets, alarm configuration/state, confirmed notification subscription, and a successful test notification—with sensitive IDs and addresses redacted before public sharing.

## 12. Lab completion summary

This lab demonstrates centralized visibility and alerting for multiple OCI compute instances. The key operational distinction is between **platform metrics** and **guest-level metrics**: CPU utilization is commonly available from OCI compute monitoring, while memory and filesystem telemetry require the appropriate agent/plugin or another configured telemetry source.

**Lab status:** Mark complete only after the validation checklist passes in your environment.

## References

- OCI Monitoring documentation: https://docs.oracle.com/en-us/iaas/Content/Monitoring/home.htm
- OCI Monitoring Metrics Explorer: https://docs.oracle.com/en-us/iaas/Content/Monitoring/Tasks/exploringmetrics.htm
- OCI Monitoring alarms: https://docs.oracle.com/en-us/iaas/Content/Monitoring/Tasks/monitoringalarms.htm
- OCI Notifications: https://docs.oracle.com/en-us/iaas/Content/Notification/Concepts/notificationoverview.htm
- Oracle Cloud Agent: https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/manage-plugins.htm

---
*Prepared for the OCI lab wiki — by TK*
