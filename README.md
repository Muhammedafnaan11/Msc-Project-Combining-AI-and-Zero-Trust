# Project Introduction and Workflow Architecture

## 1. Introduction

The rapid adoption of cloud computing has changed the way modern Security Operations Centres (SOCs) monitor and respond to cybersecurity threats. Unlike traditional environments, cloud infrastructures are highly distributed and continuously generate large quantities of security telemetry from endpoints, authentication mechanisms, operating systems, and cloud services. Although security monitoring platforms can collect and analyse this information, conventional SOC operations still depend heavily on manually configured detection rules, predefined conditions, and threshold-based alerts.

This dependency can create several operational challenges. As the volume of security events increases, security analysts may be required to investigate a large number of alerts manually. This can contribute to alert fatigue, increased workload, analyst burnout, and delays in identifying and responding to potentially malicious activity. Furthermore, predefined detection rules may have limited flexibility when dealing with changing attack techniques or behaviours that were not anticipated when the rules were created.

This project therefore investigates the use of **Artificial Intelligence (AI)-driven workflow automation** to improve the processing of security alerts within a cloud-based SOC environment. Rather than replacing the detection mechanism, AI is introduced after the security event has been detected, allowing the workflow to automatically transform technical alert information into a more structured and understandable summary for security analysts.

Alongside AI-driven automation, the project incorporates **Zero Trust architecture** as an independent endpoint security layer. While the AI workflow focuses primarily on the analysis and handling of security alerts, the Zero Trust component focuses on enforcing security policies at the endpoint. This creates two complementary security layers: one designed to improve security operations and alert handling, and another designed to restrict and control endpoint behaviour.

The proposed architecture integrates **Amazon Web Services (AWS), Elastic Security, Tines, and ThreatLocker** to demonstrate how cloud endpoint monitoring, security information and event management, AI-driven workflow automation, and Zero Trust enforcement can operate together within a single security model.

---

## 2. Technologies Used

### 2.1 Amazon Web Services (AWS)

**Amazon Web Services (AWS)** provides the cloud infrastructure used to host the Windows-based endpoint within the proposed security environment.

For this project, an **AWS EC2 instance running Windows Server 2025** was used as the cloud endpoint. The virtual machine provides an environment in which security events and endpoint activities can be generated and monitored.

The AWS endpoint therefore represents the cloud workstation/server being protected by the proposed architecture. Security telemetry generated within this endpoint is subsequently collected and forwarded to the monitoring layer.

---

### 2.2 Elastic Security and Elastic Defend

**Elastic Security** provides the central security monitoring and detection component of the architecture. It allows endpoint telemetry and security events to be collected, analysed, and monitored through its Security platform.

Within the project, **Elastic Defend** is deployed on the Windows Server endpoint to collect endpoint security telemetry. The collected information is then made available to Elastic Security, where detection rules can be configured to identify security-relevant activity.

A customised detection rule was created around **Windows Security Event ID 4672**. This event indicates that special privileges have been assigned to a new logon session and can therefore provide an indicator of privileged or administrative account activity.

The detection rule was configured to identify this activity and generate a security alert when the specified event occurred.

Elastic therefore performs the **initial detection and alert-generation stage** of the architecture.

---

### 2.3 Tines

**Tines** is used as the workflow automation platform within the project. It provides a mechanism for connecting security events to automated actions without requiring the development of a standalone software application.

Once the Elastic detection rule identifies the relevant security event, the generated alert is transmitted to Tines through a **webhook integration**.

Tines then processes the received information through a linear workflow consisting of a webhook action, an AI Agent action, and a Send Email action.

The main purpose of Tines in this architecture is therefore to automate the transition between:

**security detection → alert processing → AI analysis → analyst notification.**

---

### 2.4 Tines AI Agent

The **Tines AI Agent** provides the artificial intelligence component of the workflow.

The AI Agent does not perform the initial detection of the security event. Instead, Elastic Security first identifies the event using the configured detection rule. The resulting alert information is then provided to the Tines AI Agent through the webhook action data.

The AI Agent was configured using a specific prompt instructing it to produce a concise and professional SOC security alert. The instructions require the AI Agent to summarise the detected activity, identify important alert information such as the detection rule, event type or ID, source information and timestamp where available, and provide appropriate next steps for the SOC analyst.

This allows raw or technically detailed alert information to be transformed into a more structured form of investigation intelligence.

The AI component therefore acts as an **alert interpretation and response-guidance layer**, rather than as the primary detection mechanism.

---

### 2.5 ThreatLocker

**ThreatLocker** provides the Zero Trust endpoint security component of the architecture.

While Elastic and Tines concentrate on monitoring, detection, automation, and alert interpretation, ThreatLocker provides an enforcement mechanism designed to control what applications and activities are permitted to operate on the endpoint.

The Zero Trust approach follows the principle that access and execution should not automatically be trusted. Instead, activity is controlled through explicit security policies and restrictions.

Within the project, ThreatLocker contributes controls such as **application allowlisting, default-deny enforcement, and policy-based access restrictions**. These controls provide an additional defensive layer on the Windows cloud endpoint.

Consequently, ThreatLocker operates independently from the AI alert-processing workflow while complementing it by strengthening endpoint security and enforcing security policies.

---

# 3. Proposed Workflow Architecture

The overall architecture consists of two complementary security paths.

The first path is the **SOC monitoring and AI automation workflow**, which detects, processes, analyses, and communicates security alerts.

The second path is the **Zero Trust enforcement layer**, which applies endpoint security policies and restricts unauthorised application or system behaviour.

The overall architecture can be represented as:

```text
                    CLOUD ENDPOINT
              AWS EC2 Windows Server 2025
                         │
              ┌──────────┴──────────┐
              │                     │
              ▼                     ▼
       Elastic Defend          ThreatLocker
              │                     │
              │                     │
              ▼                     ▼
        Elastic Security       Zero Trust
              │                Enforcement
              │
      Event ID 4672 Detection
              │
              ▼
       Detection Alert
              │
              ▼
       Tines Webhook
              │
              ▼
        Tines AI Agent
              │
       ┌──────┴──────┐
       │             │
       ▼             ▼
 Alert Summary   Investigation
                  Steps
       │
       └──────┬──────┘
              ▼
        Send Email Action
              │
              ▼
        SOC Analyst
```

---

# 4. End-to-End Workflow

### Step 1: Cloud Endpoint

The workflow begins with the **AWS EC2 Windows Server 2025 instance**, which represents the cloud-based endpoint within the experimental environment.

Security-relevant activities occurring on the Windows endpoint generate operating-system security events.

---

### Step 2: Endpoint Telemetry Collection

**Elastic Defend** collects relevant endpoint telemetry from the Windows Server environment and makes this information available to Elastic Security.

This provides visibility into security activity occurring on the cloud endpoint.

---

### Step 3: Security Event Detection

A customised detection rule was configured within **Elastic Security** to monitor for **Windows Event ID 4672**.

Event ID 4672 is associated with the assignment of special privileges to a newly created logon session. In the experimental scenario, this event was used to represent privileged account activity.

When the relevant event satisfies the detection rule, Elastic generates a security alert.

---

### Step 4: Webhook Integration

Instead of requiring an analyst to manually open and inspect the alert, the Elastic detection rule is connected to a **webhook connector**.

When the detection rule is triggered, relevant alert information is automatically sent to the configured **Tines webhook**.

The webhook therefore acts as the bridge between Elastic Security and the automated Tines workflow.

---

### Step 5: Tines Webhook

The Tines webhook receives the alert information from Elastic.

The received information is made available to subsequent Tines actions through the **webhook action data tag**.

This allows the original security alert information to be passed into the AI processing stage without requiring the analyst to manually copy and interpret the alert.

---

### Step 6: AI-Based Alert Analysis

The webhook information is passed to the **Tines AI Agent**.

The AI Agent was configured with a prompt instructing it to create a concise and professional SOC security alert notification.

The prompt directs the AI Agent to:

* identify that a new security alert has occurred;
* summarise the detected activity;
* identify relevant alert information;
* include details such as the detection rule, event ID or type, source information and timestamp when available;
* provide appropriate investigation or validation steps for the SOC analyst.

The AI Agent therefore transforms the incoming alert information into a structured security-oriented summary.

---

### Step 7: Automated Email Notification

The output generated by the AI Agent is then passed to the **Send Email** action.

A predefined email structure provides the initial alert context, while the AI Agent data tag inserts the dynamically generated analysis and recommended investigation steps.

The resulting email provides the security team with an automatically generated notification containing both the original alert context and the AI-generated interpretation.

---

### Step 8: SOC Analyst Review

The final output is delivered to the SOC/security analyst.

Instead of receiving only raw technical event information, the analyst receives a structured notification containing:

**Detected activity → Relevant alert information → AI-generated summary → Recommended investigation steps**

This reduces the amount of manual interpretation required at the initial stage of alert handling and demonstrates how AI-driven workflow automation can support SOC operations.

---

# 5. Zero Trust Security Layer

The AI workflow is complemented by an independent **Zero Trust endpoint security layer implemented through ThreatLocker**.

This component does not depend on the AI Agent to determine whether applications or activities should be allowed. Instead, security policies are applied directly to the endpoint.

The Zero Trust layer incorporates principles including:

* continuous verification;
* least-privilege access;
* application allowlisting;
* default-deny enforcement;
* policy-based restrictions.

The purpose of this layer is to reduce the ability of unauthorised applications or activities to execute on the cloud endpoint.

Therefore, the project separates two important security functions.

**AI-driven automation improves the SOC's ability to process and understand security alerts, while Zero Trust enforcement strengthens the endpoint by controlling what is permitted to operate.**

---

# 6. Overall Security Model

The proposed model can therefore be understood as a combination of **detection, intelligence, automation, and enforcement**.

**Elastic Security** provides detection and monitoring.

**Tines** provides workflow orchestration.

**Tines AI Agent** provides automated alert interpretation and investigation guidance.

**Send Email** provides automated analyst notification.

**ThreatLocker** provides Zero Trust enforcement at the endpoint.

Together, these components create the following security workflow:

```text
                    DETECTION
                       │
                       ▼
              Elastic Security
                       │
                       ▼
                    WEBHOOK
                       │
                       ▼
                 Tines Workflow
                       │
                       ▼
                  AI ANALYSIS
                       │
                       ▼
              Structured Alert
                       │
                       ▼
              Analyst Notification


             INDEPENDENT SECURITY LAYER
                       │
                       ▼
                  ThreatLocker
                       │
                       ▼
              Zero Trust Enforcement
                       │
                       ▼
              Protected Endpoint
```

The architecture was evaluated by generating privileged access activity on the cloud-based Windows endpoint and observing the resulting end-to-end process. The experimental workflow demonstrated how an endpoint event could be collected by Elastic Defend, detected by the Elastic Security rule, forwarded automatically to Tines, analysed by the AI Agent, and subsequently delivered to the analyst through an automated notification.

At the same time, the ThreatLocker component provided an additional endpoint enforcement layer based on Zero Trust principles.

The resulting architecture demonstrates that AI-driven workflow automation can reduce several manual stages involved in initial alert handling, while Zero Trust controls provide a complementary mechanism for strengthening endpoint security. Rather than treating AI and Zero Trust as a single security mechanism, the project demonstrates their complementary roles: **AI improves the operational handling and interpretation of security alerts, while Zero Trust strengthens prevention and enforcement at the endpoint.**
