# n8n Automation: Google Sheets → Gmail (Demand Status Report)

**CAIE Course Program – Assignment 9**

An n8n workflow that reads demand records from a Google Sheet, turns them into a clean, structured HTML email with a status summary, and sends it through Gmail. If the sheet is empty, no email is sent.

---

## Workflow overview

```mermaid
flowchart LR
    A[Manual Trigger] --> B[Google Sheets<br/>Get Row(s)]
    B --> C[Code<br/>Build Email]
    C --> D{Has Data?}
    D -- true --> E[Gmail<br/>Send Message]
    D -- false --> F[No Operation<br/>Skipped – Sheet Empty]
```

| Node | Type | Purpose |
|------|------|---------|
| When clicking 'Execute workflow' | Manual Trigger | Starts the workflow |
| Get row(s) in sheet | Google Sheets (Get Rows) | Reads all rows from the `demands` tab |
| Code in JavaScript | Code | Builds the subject, a status summary and an HTML table |
| Has data? | IF | Routes to Gmail only when rows exist |
| Send a message | Gmail (Send) | Sends the formatted email |
| Skipped – Sheet Empty | No Operation | Ends the run cleanly when the sheet is empty |

---

## Source data

Google Sheet **Demand-Status**, tab **demands**, with these headers in row 1:

| Date-created | DemandID | Primary skill | Region | Location-type | Location | Status |
|---|---|---|---|---|---|---|
| 4-Sep-2026 | DMD39943 | Azure DevOps | EMEA | offshore | Chennai | TP1 WIP |
| 6-Sep-2026 | DMD39945 | BA | EMEA | onshore | Poland | CI Pending |
| 9-Sep-2026 | DMD39947 | AI/ML | EMEA | offshore | Chennai | Profiles Awaited |

---

## Sample output

**Subject:** `Demand Status Update – 6 demands – 17 Sep 2026`

**Body:**

> Hi,
>
> Here is the latest demand status (6 open demands).
>
> **Summary:** TP1 WIP: 3 | CI Pending: 1 | Profiles Awaited: 2
>
> *(bordered table of all demands)*
>
> Regards,
> Your Automation Bot

---

## Key design choices

- **One consolidated email.** The Code node runs once for all items, so all rows go into a single email instead of one email per row.
- **Status summary.** Demands are counted per status so the reader gets the picture before reading the table.
- **Empty-sheet handling.** *Always Output Data* is enabled on the Sheets node, so the workflow continues when there are no rows. The Code node returns `hasData: false`, and the IF node routes to a No Operation node, so no blank email is sent.
- **Data cleaning.** Values are trimmed, so stray spaces or line breaks in cells don't split the status counts, and HTML-escaped, so cell content can't break the email layout.

---

## Setup

### Prerequisites
- An n8n instance (cloud or self-hosted)
- A Google account with Google Sheets and Gmail
- A Google Cloud project with an OAuth client (for self-hosted n8n)

### 1. Enable the Google APIs
In Google Cloud Console → **APIs & Services → Library**, enable **both**:
- Google Sheets API
- Google Drive API

> The Drive API only lets n8n *list* spreadsheets. Without the **Sheets API**, the workbook appears in the list but reading a tab fails with `404 – Requested entity was not found`.

### 2. Create credentials in n8n
- **Google Sheets OAuth2 API**: sign in and grant all requested permissions
- **Gmail OAuth2 API**: sign in with the sending account

### 3. Import the workflow
1. In n8n, go to **Workflows → Import from File** and select `workflow.json`.
2. Open **Get row(s) in sheet**, select your credential, then choose your document and sheet **From list**.
3. Open **Send a message**, select your Gmail credential, and set the **To** address.

### 4. Run
Click **Execute workflow** and check the recipient's inbox.

---

## Testing

| Scenario | Expected result |
|---|---|
| Sheet has data | Email received with the summary line and a table of every row |
| Sheet has only the header row | Run ends at **Skipped – Sheet Empty**, and no email is sent |

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `404 – Requested entity was not found` | Enable the **Google Sheets API** in the same Cloud project as the OAuth client, then sign in to the credential again |
| Sheet dropdown doesn't load | Select the sheet **From list** rather than typing its name, and check that the credential's Google account can open the file |
| One email per row | Set the Code node to **Run Once for All Items** |
| Workflow stops when the sheet is empty | Enable **Settings → Always Output Data** on the Sheets node |

---

## Possible extensions
- Replace the Manual Trigger with a **Schedule Trigger** (for example, weekdays at 9 AM)
- Filter to open demands only, or group the table by region
- Color-code the Status cells in the email

---

## Security
All credentials are stored inside n8n. The exported `workflow.json` contains credential names and IDs only, never secrets. Recipient addresses and sheet IDs have been replaced with placeholders before committing.

---

## Repository contents

```
├── workflow.json     # n8n workflow export
├── README.md         # this file
└── screenshots/      # workflow canvas and received email (optional)
```
