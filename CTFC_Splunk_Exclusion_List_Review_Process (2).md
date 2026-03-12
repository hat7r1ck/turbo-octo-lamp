# CTFC Splunk Exclusion List Review Process

**Space:** CTFC Documentation  
**Scope:** Splunk Cloud (WF02958.KV.IOC_Whitelist)  
**Last Updated:** March 2026

---

## Purpose

This document provides step-by-step instructions for CTFC Tier II security engineers and Strategists to perform a weekly review of expiring Splunk Indicators of Compromise (IOC) whitelist entries in WF02958.KV.IOC_Whitelist.

---

## Overview

The WF02958.KV.IOC_Whitelist is a list of IOCs maintained by the CTFC. Each entry suppresses alert creation for predefined non-malicious activity, preventing XSOAR case generation for the CTFC. Suppression may be temporary or permanent depending on the IOC and its justification. Periodic review is required to confirm each entry remains valid.

The Splunk Cloud review process is managed through the **CTFC Exclusion List Review Dashboard**. No manual spreadsheet export is required. The dashboard surfaces all entries expiring within 30 days, displays 30-day and 90-day traffic data for each entry, and routes engineer determinations through Strat approval before the KV store is updated.

---

## Requirements

Each reviewed entry in WF02958.KV.IOC_Whitelist must have the following fields populated upon completion:

- Submitted date
- Expiry date within one year of the submitted date
- Notes containing an XSOAR case number, a Jira ticket reference (CPSS/SCR/CTFCSE), or detailed justification if no ticket reference exists
- Submitter AD-ENT username

Additional guidelines:

- One year of inactivity is the threshold for recommending an entry be disabled
- Expiry dates for all entries must not be set to the same date, as this will cause a flood of reviews in a future week
- VAT scanning IPs are excluded from the review scope and are reviewed through a separate process via the CTFC Service Desk

> **Rotation responsibility:** CTFC Tier II engineers are responsible for the weekly exclusion list review on a rotating basis. During the final full work week of their rotation, the outgoing engineer must schedule a handover meeting with the incoming engineer to transfer duties and answer any open questions.

> **Notable ownership:** Not all entries in the exclusion list are CTFC-owned. Always review the ticket referenced in the entry to identify the owning team before making a determination.

---

## Setting Up a Confluence Review Calendar

Use a Confluence Team Calendar to track which engineer is responsible for which review week. This gives the team a shared view organized by ISO week number.

**Create the calendar:**

1. Open the CTFC Documentation space in Confluence
2. Click **Calendars** in the space sidebar
3. Click the **+** button at the top right and select **Add new calendar**
4. Name it something clear (e.g. "CTFC Exclusion Review Rotation")
5. Click **Create**

**Enable week numbers:**

Week numbers need to be turned on by a Confluence admin. If you do not see them:

1. Go to **General Configuration > Team Calendars > Global Settings**
2. Check the **Display week numbers** option
3. Click **Save**

This displays the ISO week number (e.g. W11, W12) alongside each week in the calendar view.

**Add rotation events:**

1. Click a date on the calendar to open the event dialog
2. Set the event name to the responsible engineer (e.g. "Jeffrey Morris - Exclusion Review")
3. Set the date range to cover the full work week (Monday through Friday)
4. If the rotation repeats on a fixed cycle, toggle the recurring event option
5. Click **Save**

**Add the calendar to a Confluence page:**

1. Edit the page where you want the calendar displayed
2. Type `/team calendar` and select the Team Calendars macro from the dropdown
3. In the sidebar, select the calendar you created and set the view to **Month** or **Timeline**
4. Publish the page

The team can now see at a glance who owns which week. Engineers can reference this before starting their review to confirm it is their rotation.

---

## How the Dashboard Works

A saved search called `exclusion_bulk_risk_check` runs every Monday at 7:00 AM. It pulls all active entries expiring within 30 days, checks `index=risk` for hit counts (30-day and 90-day), finds the last time each IOC appeared in risk events, and caches the results.

When you open the dashboard, it loads from that cache. No live queries run against the KV store or risk index on load.

**If the dashboard is empty**, the saved search may not have completed yet:

1. Go to **Activity > Search Jobs**
2. Set the filter to **All Users**
3. Search for `exclusion_bulk_risk_check`
4. Confirm the status shows **Done** and the result count is not zero

### Traffic Flag Values

| Flag | What it tells you | What to do |
|---|---|---|
| **ACTIVE** | The IOC hit risk events within the last 90 days | Recommend extend |
| **Validate in Notables / Jira** | Zero matching risk events in 90 days | Check Mission Control notables and the Jira ticket before deciding |

"Validate in Notables / Jira" does not mean the entry is inactive. It means the risk index had no hits for that specific rule and IOC combination. The entry could still be doing work elsewhere. Verify before you disable anything.

---

## Known Behavior: Risky Command Prompt

Splunk Cloud shows a security warning every time a search writes to a CSV lookup using `outputlookup`. This is a platform-level policy and cannot be turned off from the dashboard side.

**What happens on every write:**

1. You select **Confirm** from the dropdown
2. Splunk shows a dialog about the search containing a risky command
3. Click **Run Anyway**
4. Select **Confirm** from the dropdown a second time (the first selection was consumed by the popup)

Every write action in this dashboard follows that pattern: Confirm, Run Anyway, Confirm again. Nothing is lost or duplicated by confirming twice.

A request to exempt this dashboard from the prompt has been submitted to the platform team. Until that goes through, this is the expected behavior.

---

## Splunk Cloud: Engineer Workflow

### Step 1: Open the Dashboard

Go to **Splunk Cloud > wf-ctfc-content > CTFC Exclusion List Review**.

Set the **Role** dropdown to **Engineer**. The dashboard loads on its own. There is no Submit button on the main page.

### Step 2: Pick a Review Method

Use the **Action** dropdown to choose how you want to work:

| Action | When to use it |
|---|---|
| **Bulk Review** | Multiple entries under the same rule all get the same determination |
| **Single Entry Review** | An entry needs its own determination or different notes |
| **Submit Week to Strat** | You are done reviewing and ready to hand off for approval |

### Step 3: Review Entries

The dashboard shows all entries expiring within 30 days. Each row contains:

| Column | Description |
|---|---|
| IOC Value | The suppressed value |
| Content | The correlation rule associated with this entry |
| Expires | Date the entry expires |
| Traffic | ACTIVE or Validate in Notables / Jira |
| 30d Hits | Times the IOC appeared in risk events in the last 30 days |
| 90d Hits | Times the IOC appeared in risk events in the last 90 days |
| Last Seen | Most recent date the IOC was seen in risk events |
| Owner | Team that owns the correlation rule |

**How to decide:**

1. **Traffic = ACTIVE:** It is suppressing events right now. Extend it.
2. **Traffic = Validate in Notables / Jira with 0 hits across both windows:** Go to Mission Control. Look at the notables for this rule. Open the Jira ticket on the entry.
   - Notable still active and relevant: extend
   - Notable closed and no activity in 90 days: disable
3. **Owner is not CTFC:** Contact the owning team. Get their determination. Write their response in your notes before submitting.

CTFC does not own notables. The team that created the rule owns them. If SCD is listed as the owner, check the SCD ticket on the entry to find the actual owning team.

### Step 4: Submit Determinations

#### Bulk Review

1. Type part of the rule name into the **Content filter** (or leave `*` for everything)
2. Select **Apply Filter** to show the write fields
3. Pick a determination from the dropdown
4. Add your notes (ticket number, reason, or N/A)
5. Select **Confirm** from the **Confirm Bulk Submit** dropdown
6. Risky command prompt appears. Click **Run Anyway**, then select **Confirm** again

#### Single Entry Review

1. Click any row in the review table
2. The entry details populate in the fields above the table
3. Pick a determination and add notes
4. Select **Confirm** from the **Confirm Entry Submit** dropdown
5. Risky command prompt appears. Click **Run Anyway**, then select **Confirm** again

#### Determination Options

| Option | When to use it |
|---|---|
| Extend - 1 year | Active or still needed. No reason to re-review soon |
| Extend - 6 months | Active but worth checking again sooner |
| Extend - 3 months | Borderline. Short extension while you gather more information |
| Disable - no activity | No hits, no active notables, no reason to keep it |
| Disable - no longer needed | The original issue was resolved or the entry was temporary |
| Modify - scope change needed | The rule association, IOC value, or scope needs updating before extending |

Notes are required for everything except Extend - 1 year. Strat will see your notes.

### Step 5: Submit the Week to Strat

Once every entry for the week has a determination, switch the **Action** dropdown to **Submit Week to Strat**.

Before submitting, create a Jira ticket through the Engineer Intake / Submit Ideas process. Include:

- Every rule that is up for expiry
- Your recommendation for each (extend or disable)
- Your rationale (traffic data, Jira history, owner follow-up, case count)

Then:

1. Review the table showing your determinations for the week
2. Enter the Jira ticket number (e.g. CTFCSE-1234)
3. Select **Confirm** from the **Confirm Submit to Strat** dropdown
4. Risky command prompt appears. Click **Run Anyway**, then select **Confirm** again

The week moves to pending Strat approval. The Jira ticket number is written to every entry for that week.

---

## Splunk Cloud: Strat Workflow

Open the same dashboard and set the **Role** dropdown to **Strategist (Strat)**.

1. Click the row for the week you need to review in the approval queue
2. All entries load below with the engineer's determinations, notes, traffic data, and Jira ticket
3. Pick **Approved** or **Denied** from the **Decision** dropdown
4. Add notes (required if denying)
5. Select **Confirm** from the **Confirm Decision** dropdown
6. Risky command prompt appears. Click **Run Anyway**, then select **Confirm** again

**Approved:** The engineer proceeds to update the KV store.

**Denied:** The week goes back to the engineer with your notes. The engineer addresses the issues, updates determinations if needed, and resubmits.

### Decision History

The Decision History panel at the bottom of the dashboard shows every past Strat decision. Approved and denied weeks are both shown with the entry count, who decided, when, and any notes. This panel is visible to all roles and updates automatically.

---

## Updating WF02958.KV.IOC_Whitelist

> **Do not use the Lookup Editor app to edit or delete entries.** The Lookup Editor has caused most of the expiry issues that already exist. Entries must never be deleted. They serve as the historical record.

Once Strat approves the week, update each entry through the **Exclusion List Input** dashboard:

1. Open the **SCD Exclusion List Input** dashboard
2. Set Action to **Create/Update**
3. Pick the correlation rule from the **Content** dropdown
4. Click the entry row to populate the form
5. Set the **Expiry** date based on the approved determination
6. Make sure **Notes** retains the original notes. Append the current Jira ticket number if it is missing
7. Make sure **Ticket #** has a valid reference (INC#, JIRA#, or N/A)
8. Set **Bit** to ENABLED or DISABLED based on the determination
9. Click **Submit**. The confirmation panel shows UPDATE if it worked

### Verify the Write

Run this in Splunk Cloud after updating an entry:

```spl
| inputlookup WF02958.KV.IOC_Whitelist where value="REPLACE_WITH_IOC_VALUE"
| search content=*
| eval expiry=expiry/1000
| eval submitted_date=submitted_date/1000
| fieldformat expiry = strftime(expiry, "%Y-%m-%d %H:%M")
| fieldformat submitted_date = strftime(submitted_date, "%Y-%m-%d %H:%M")
| sort -content
```

Swap `REPLACE_WITH_IOC_VALUE` for the actual IOC value.

---

## Backup: Review History Query

To check the status of past weeks outside the dashboard:

```spl
| inputlookup exclusion_review_tracker.csv
| stats
    count as total_entries,
    count(eval(determination!="" AND determination!="Needs follow-up")) as reviewed,
    count(eval(strat_status="approved")) as approved,
    count(eval(strat_status="denied")) as denied,
    count(eval(strat_status="pending_strat")) as pending_strat,
    values(jira_ticket) as jira_tickets,
    values(strat_updated_by) as strat_reviewer
    by iso_week
| sort - iso_week
| rename iso_week AS "Week",
         total_entries AS "Total Entries",
         reviewed AS "Reviewed",
         approved AS "Approved",
         denied AS "Denied",
         pending_strat AS "Pending Strat",
         jira_tickets AS "Jira Tickets",
         strat_reviewer AS "Strat Reviewer"
```

One row per week. Use it to confirm a week was fully processed or to audit prior weeks.

---

## Links

- [CTFC Exclusion List Review Dashboard](https://es-wf-sp.splunkcloud.com/en-US/app/wf-ctfc-content/exclusion_review)
- [SCD Exclusion List Input](https://es-wf-sp.splunkcloud.com/en-US/app/wf-ctfc-content/scd_exclusion_list_input)
- [Content Management](https://es-wf-sp.splunkcloud.com/en-US/app/wf-ctfc-content/content_management) (check if a rule is enabled or disabled)
- Team share: `\\NCCCNSF701Z1.wellsfargo.net\c_cis_groups\CTFC\CTFC_Security_Wiki\Attachments\Splunk_Exclusion_List_Review_Process`

---

## Splunk On-Premises

This process covers Splunk Cloud only. The on-premises review process has not changed. Refer to the existing Confluence documentation for the WF02958.KV.IOC_Whitelist Validation Spreadsheet process using the CTFC Incident Management Metrics dashboard.
