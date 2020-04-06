# Shakespeare Sonnet++ Postmortem (incident #465)

## Meeting Scheduled For

_2015-10-21_ 10:00 AM UTC

## Authors

* David Bell
* tbc

## Call recording
_Link to the incident call recording._

## Attendees
_List the attendees of the postmortem meeting_

## Status

Complete, action items in progress

## Overview

_Include a short sentence or two summarizing the contributing factors, timeline summary, and the impact. E.g. "On the morning of August 99th, we suffered a 1 minute P1 due to a runaway process on our primary database machine. This slowness caused roughly 0.024% of alerts that had begun during this time to be delivered out of SLA."_

_Confirm with all involved the channels for communcations._

## What Happened
_Include a short description of what happened._

## Contributing Factors / Triggers
_Include a description of any conditions that contributed to the issue. If there were any actions taken that exacerbated the issue, also include them here with the intention of learning from any mistakes made during the resolution process._

## Resolution 
_Include a description what solved the problem. If there was a temporary fix in place, describe that along with the long-term solution._

## Impact
_Be very specific here, include exact numbers._

| Impact | Quantify |
| ----- | ----------- |
| Estimated 1.21B queries lost, no revenue impact. |  |
| Time in P1 | ?mins |
| Time in P2 | ?mins |
| Notifications Delivered out of SLA	??%|  (?? of ??) |
| Events Dropped / Not Accepted	??% | (?? of ??) Should usually be 0, but always check |
| Customers Affected  | ?? |
| Colleagues Affected | ?? |
| Stores Affected	| |
| Support Requests Raised ?? | Include any relevant links to tickets |

## Root Causes

Cascading failure due to combination of exceptionally high load and a resource leak when searches failed due to terms not being in the Shakespeare corpus. The newly discovered sonnet used a word that had never before appeared in one of Shakespeare's works, which happened to be the term users searched for. Under normal circumstances, the rate of task failures due to resource leaks is low enough to be unnoticed.

## Detection

Borgmon detected high level of HTTP 500s and paged on-call.

## Responders
    Who was the Incident Commander / Major Incident Manager?
    Who was the scribe?
    Who else was involved?

## Timeline
_Some important times to include: (1) time the contributing factor began, (2) time of the notifcations, (3) time that the incident was escalated to MIMs, (4) time of any significant actions, (5) time the P-2/1 ended, (6) links to tools/logs that show how the timestamp was arrived at._

2015-10-21 (*all times UTC*)

| Time  | Description |
| ----- | ----------- |
| 14:51 | News reports that a new Shakespearean sonnet has been discovered in a Delorean's glove compartment |
| 14:53 | Traffic to Shakespeare search increases by 88x after post to */r/shakespeare* points to Shakespeare search engine as place to find new sonnet (except we don't have the sonnet yet) |
| 14:54 | **OUTAGE BEGINS** -- Search backends start melting down under load |

## How Did We Do

### What went well

* Monitoring quickly alerted us to high rate (reaching ~100%) of HTTP 500s
* Rapidly distributed updated Shakespeare corpus to all clusters

### What Didn't Go So Well?

* We're out of practice in responding to cascading failure
* We exceeded our availability error budget (by several orders of magnitude) due to the exceptional surge of traffic that essentially all resulted in failures

### Where we got lucky

* Mailing list of Shakespeare aficionados had a copy of new sonnet available
* Server logs had stack traces pointing to file descriptor exhaustion as cause for crash
* Query-of-death was resolved by pushing new index containing popular search term

## Action Items
_Each action item should be in the form of a ServiceNow task or JIRA ticket, and each ticket should have a consistent set of tags, e.g., “P1_YYYYMMDD” (such as P1_20150911)._
_Include action items such as:_
_
    any fixes required to prevent the contributing factor in the future,
    any preparedness tasks that could help mitigate the problem if it came up again,
    remaining post-mortem steps, such as the internal email, as well as the status-page public post,
    any improvements to our incident response process._
    
| Action Item | Type | Owner | Bug |
| ----------- | ---- | ----- | --- |
| Update playbook with instructions for responding to cascading failure | mitigate | jennifer | n/a **DONE** |
| Use flux capacitor to balance load between clusters | prevent | martym | Bug 5554823 **TODO** |
| Schedule cascading failure test during next DiRT | process | docbrown | n/a **TODO** |
| Investigate running index MR/fusion continuously | prevent | jennifer | Bug 5554824 **TODO** |
| Plug file descriptor leak in search ranking subsystem | prevent | agoogler | Bug 5554825 **DONE** |
| Add load shedding capabilities to Shakespeare search | prevent | agoogler | Bug 5554826 **TODO** |
| Build regression tests to ensure servers respond sanely to queries of death | prevent | clarac | Bug 5554827 **TODO** |
| Deploy updated search ranking subsystem to prod | prevent | jennifer | n/a **DONE** |
| Freeze production until 2015-11-20 due to error budget exhaustion, or seek exception due to grotesque, unbelievable, bizarre, and unprecedented circumstances | other | docbrown | n/a **TODO** |

## Supporting Information

* Monitoring dashboard, <http://monitor/shakespeare?end_time=20151021T160000&duration=7200>

## Messaging
### Internal Email
_This is a follow-up for employees. It should be sent out right after the postmortem meeting is over. It only needs a short paragraph summarizing the incident and a link to this wiki page._

### External Comms Record
_Slack / Teams chat / Skype conversations records_
