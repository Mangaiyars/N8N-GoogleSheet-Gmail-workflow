n8n Automation: Google Sheets to Gmail
CAIE Course Program - Assignment 9

About
An n8n workflow that reads demand records from a Google Sheet, turns them into a
structured email with a status summary, and sends it through Gmail. If the sheet
is empty, no email is sent.

Workflow
Manual Trigger -> Google Sheets (Get Rows) -> Code (Build Email) -> IF (Has Data?) -> Gmail (Send Message)

Nodes
- Manual Trigger: starts the workflow
- Google Sheets (Get Rows): reads all rows from the "demands" tab of the "Demand-Status" sheet
- Code (JavaScript): builds the subject, a per-status summary and a table of demands
- IF (Has Data?): sends the email only when the sheet has rows
- Gmail (Send Message): delivers the formatted email
- No Operation: ends the run cleanly when the sheet is empty

Sample email
Subject: Demand Status Update - 6 demands - 17 Sep 2026
Summary: TP1 WIP: 3 | CI Pending: 1 | Profiles Awaited: 2
Followed by a table listing Date-created, DemandID, Primary skill, Region,
Location-type, Location and Status for each demand.

Highlights
- One consolidated email for all rows, not one email per row
- Status summary at the top for a quick overview
- Empty sheet handled gracefully, so no blank email is sent
- Cell values cleaned for a tidy layout
- Credentials kept inside n8n; none are stored in this repository

Tools
n8n, Google Sheets, Gmail
