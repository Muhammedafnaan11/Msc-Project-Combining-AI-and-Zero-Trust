TINES LINEAR WORKFLOW
=====================

The overall Tines workflow consists of three primary actions:

1. Webhook Action
2. AI Agent Action
3. Send Email Action

WORKFLOW:

[Elastic Security]
       |
       | Security Alert
       | Event ID 4672
       v
[Tines Webhook]
       |
       | Webhook Action Data
       v
[Tines AI Agent]
       |
       | AI-generated summary
       | Investigation recommendations
       v
[Send Email]
       |
       v
[SOC Security Alert Notification]


ACTION 1 - WEBHOOK
------------------

The webhook receives alert information forwarded from
Elastic Security following activation of the detection rule.

The webhook data is made available to subsequent actions
through the Tines data-tag mechanism.


ACTION 2 - AI AGENT
-------------------

The AI Agent processes the information supplied by the
webhook.

The configured prompt instructs the AI Agent to:

- identify that a security alert has occurred;
- summarise the detected activity;
- identify important alert information;
- provide appropriate investigation steps.

The AI Agent output is then passed to the Send Email action
using the corresponding Tines data tag.


ACTION 3 - SEND EMAIL
---------------------

The Send Email action delivers the security notification.

A predefined message provides the initial context for the
security team, while the AI Agent data tag inserts the
generated alert analysis and recommended investigation
steps.


AUTOMATION MODEL
----------------

Detection → Webhook → AI Analysis → Notification