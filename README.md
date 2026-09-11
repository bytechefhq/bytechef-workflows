# ByteChef Templates

A collection of example workflows and projects for use with the
[ByteChef](https://github.com/bytechefhq/bytechef) platform. Every folder under
[`workflows/`](workflows) is published as a pre-built workflow template that users can preview and
import into their own projects, and every zip under [`projects/`](projects) is published as a
pre-built project template that users can import as a whole project with all of its workflows.

This README describes how to submit one of each:

- [Workflow templates](#workflow-templates)
- [Project templates](#project-templates)

# Workflow templates

## What a template folder contains

```
workflows/
  ai_email_classifier/
    meta.json                  <- template metadata (required)
    workflow_definition.json   <- the workflow itself (required)
    screenshot.png             <- image of the workflow canvas
```

- One folder per template, directly under `workflows/`. Nothing is read from a subfolder.
- The folder **must** contain both `meta.json` and `workflow_definition.json`. If either is missing,
  the template is skipped and never appears.
- Name the folder in `lower_snake_case`: no spaces, no dashes. Examples: `cv_scanning`,
  `out_of_working_hours`, `monthly_report_generation_for_nifty_project`.
- The folder name becomes the template's permanent identifier and part of its URL. Renaming it later
  breaks every existing link, so choose it carefully the first time.

## 1. Build and test the workflow in ByteChef

Get it working end to end in a real project first. A template that has never run is not a template,
it is a draft. Note anything environment-specific you configure along the way (spreadsheet IDs, form
IDs, label IDs, channel names, app IDs) — you will have to declare those in `prerequisites`.

## 2. Export the workflow definition

Export the workflow as JSON from the ByteChef workflow editor and save it as
`workflows/<your_folder>/workflow_definition.json`. Keep the exported formatting as-is.

Then clean it up:

- **Remove anything private.** Personal email addresses, internal channel names, credentials,
  customer data, real record IDs from a production account. Replace them with neutral placeholders
  (`support@example.com`, `your-channel`). Connections are not part of the export — the importing
  user picks their own — but resource IDs typed into parameters are. Leaving an ID that points at a
  throwaway demo resource is fine, as long as `prerequisites` says what to swap out.
- **Give it a meaningful `label` and `description`.** Both are shown to users on the template's
  detail page, so they are not internal notes. Keep them consistent with `name` and `description` in
  `meta.json`.
- **Label every task and trigger.** The default `googleSheets_1`-style names are fine as `name`
  values (other steps reference them as `${googleSheets_1}`), but the `label` is what a user reads
  in the canvas.

A minimal, valid definition looks like this:

```json
{
    "label" : "Delete Sheet",
    "description" : "Deletes a sheet and logs it.",
    "inputs" : [ ],
    "triggers" : [ ],
    "tasks" : [ {
        "label" : "Google Sheets",
        "name" : "googleSheets_1",
        "type" : "googleSheets/v1/deleteSheet",
        "parameters" : {
            "spreadsheetId" : "1wnYhkyBVNfys2XCxZgxjYFGOB3_pLSxB9ISMUhMvXDs",
            "sheetId" : 769775461
        },
        "metadata" : {
            "ui" : {
                "dynamicPropertyTypes" : { }
            }
        }
    } ]
}
```

`label`, `description`, `inputs`, `triggers` and `tasks` are always present, even when `inputs` and
`triggers` are empty arrays (an on-demand workflow has no trigger).

## 3. Write `meta.json`

Create `workflows/<your_folder>/meta.json` using two-space indentation:

```json
{
  "name": "AI Email Classifier",
  "category": "ai",
  "description": "Polls Gmail for new emails and uses OpenAI to classify each one into a single department label (Sales, Support, Finance, Operations, HR, or URGENT). The matching Gmail label is applied, the department's recipient address is stored, and the original email is forwarded to the correct team.",
  "shortDescription": "Classify incoming Gmail messages with AI and route them to the right team.",
  "author": {
    "name": "",
    "email": "",
    "role": "",
    "socialLinks": ""
  },
  "keyFeatures": [
    "AI classification constrained to a fixed set of labels via a JSON response schema",
    "Multi-way branch routing, one case per department label",
    "Automatic Gmail label tagging of the incoming message"
  ],
  "prerequisites": [
    "Google Mail (Gmail) connection",
    "OpenAI API key",
    "Gmail labels pre-created, matching the label IDs referenced in the workflow"
  ],
  "idealFor": "Support and operations teams that want incoming email triaged and routed to the correct department automatically.",
  "steps": [
    "Trigger: poll Gmail for a new email.",
    "Classify the email into a single label with OpenAI.",
    "Branch on the predicted label: add the matching Gmail label and store the team's recipient address.",
    "Forward the original email to the resolved team address."
  ],
  "screenshot": "screenshot.png"
}
```

| Field | Type | Notes |
| --- | --- | --- |
| `name` | string | Title of the template card. **Must not be blank** — a template without a name is skipped entirely. Keep it consistent with the definition's `label`. |
| `category` | string | Exactly one ID from the [category list](#categories). |
| `description` | string | Description on the template card, and one of the two fields users can search on. Two to four sentences saying what the workflow does, concretely. The card shows about two lines, so front-load the meaning. |
| `shortDescription` | string | One-sentence tagline. |
| `author` | object | `name`, `email`, `role`, `socialLinks`, all strings. The block must be present; leave the fields as empty strings if you would rather not be credited. |
| `keyFeatures` | string[] | Capability bullets ("AI urgency classification of the incoming email"). |
| `prerequisites` | string[] | Everything a user must have before importing: each connection, each API key, and each resource that must already exist (a spreadsheet, a Gmail label, a database table). Be specific — this is the field that decides whether an imported template actually runs. |
| `idealFor` | string | One sentence naming the audience: "Recruiters and hiring teams that want to pre-screen incoming job applications automatically." |
| `steps` | string[] | The workflow narrated step by step, one sentence per step, in execution order. Start with `"Trigger: ..."` when the workflow has a trigger. |
| `screenshot` | string | Always `"screenshot.png"`. |

Every field is expected in a submission. Do not add component names or the trigger — those are taken
from the workflow definition automatically.

### Categories

| ID | Label |
| --- | --- |
| `ai` | AI |
| `sales` | Sales |
| `support` | Support |
| `finance` | Finance |
| `hr` | HR |
| `social` | Social |
| `marketing` | Marketing |
| `it` | IT Ops |
| `documentOps` | Document Ops |
| `other` | Other |

Note the exact spelling of `documentOps` (camelCase). An ID outside this list never matches a filter,
which leaves the template unreachable by category.

## 4. Add `screenshot.png`

Add a `screenshot.png` image of the workflow canvas to the folder, and keep `"screenshot"` in
`meta.json` set to `"screenshot.png"`. Some of the older templates still carry an empty placeholder
file; new submissions should include a real image.

## 5. Open a pull request

One template per pull request. In the description, say what the workflow automates and confirm you
ran it successfully. Before opening it, check:

- [ ] The folder is `workflows/<lower_snake_case_name>/`, directly under `workflows/`.
- [ ] It contains `meta.json`, `workflow_definition.json` and `screenshot.png`.
- [ ] Both JSON files parse (`python3 -m json.tool <file>` or an editor that validates JSON).
- [ ] `meta.json` has a non-blank `name`, and `category` is one of the IDs listed above.
- [ ] `name` matches the definition's `label`; the two descriptions agree.
- [ ] Every field in the table is filled in, and `author` is present (empty strings are fine).
- [ ] `prerequisites` lists every connection, API key and pre-existing resource the workflow needs.
- [ ] `steps` narrates the workflow in execution order.
- [ ] No credentials, personal email addresses, internal channel names or customer data anywhere in the definition.
- [ ] The workflow was actually run end to end in ByteChef.

# Project templates

A project template is a single zip file under `projects/`: the zip that ByteChef produces when you
export a project, cleaned up. It carries the project and every workflow in it. Unlike workflow
templates, nothing is unpacked into folders in this repository: the zip is stored and served as-is.

## What a project template contains

```
projects/
  customer_onboarding.zip
    project.json               <- project name and description
    workflow-<uuid>.json       <- one per workflow
```

- One zip per template, directly under `projects/`. Every file under `projects/` is read as a
  template zip, so put nothing else there.
- All files sit at the root of the zip, exactly as ByteChef exported them. A zip that wraps them in a
  folder is skipped, because the files are matched by exact name.
- The zip **must** contain `project.json` and at least one `workflow-*.json`. If either is missing,
  the template is skipped and never appears.
- Name the zip in `lower_snake_case`: no spaces, no dashes. Examples: `customer_onboarding.zip`,
  `invoice_processing.zip`. The file name becomes the template's permanent identifier and part of its
  URL. Renaming it later breaks every existing link, so choose it carefully the first time.

## 1. Build and test the project in ByteChef

Get every workflow in the project working end to end first. Note anything environment-specific you
configure along the way (spreadsheet IDs, form IDs, label IDs, channel names, app IDs) so you can
replace it with a placeholder before submitting.

## 2. Export the project

Open the project's settings menu in ByteChef and choose **Export**. You get a zip containing
`project.json` and one `workflow-<uuid>.json` per workflow.

`project.json` holds the project's `name` and `description`. Both are shown to users on the
template card, so make them meaningful before you export: the description should say in two to four
sentences what the project does, concretely.

Unzip the export and clean the workflow files up the same way as for a workflow template:

- **Remove anything private.** Personal email addresses, internal channel names, credentials,
  customer data, real record IDs from a production account. Replace them with neutral placeholders.
- **Keep the exported formatting.** Each `workflow-<uuid>.json` is the workflow definition exactly as
  ByteChef exported it. Edit values inside it, but do not reformat or rename the file.
- **Check `label` and `description`** of every workflow. They are shown on the template's detail
  page.

Do not add component names or triggers anywhere. Those are taken from the workflow definitions
automatically.

## 3. Zip it

Zip `project.json` and the `workflow-*.json` files so that they are at the root of the archive, and
save it as `projects/<lower_snake_case_name>.zip`:

```
cd <unzipped export>
zip ../customer_onboarding.zip project.json workflow-*.json
```

Screenshots are not part of a project template. Attach them to the pull request instead.

## 4. Open a pull request

One template per pull request. In the description, say what the project automates, which
[categories](#categories) it belongs to, how you want to be credited (name, role, a GitHub, LinkedIn
or X link), and confirm you ran every workflow successfully. Before opening it, check:

- [ ] The file is `projects/<lower_snake_case_name>.zip`, directly under `projects/`.
- [ ] The zip contains `project.json` and at least one `workflow-*.json`, all at the root of the archive.
- [ ] `project.json` parses (`python3 -m json.tool project.json` or an editor that validates JSON) and has a non-blank `name` and a real `description`.
- [ ] Every workflow has a meaningful `label` and `description`.
- [ ] No credentials, personal email addresses, internal channel names or customer data anywhere in the workflow definitions.
- [ ] Every workflow in the project was actually run end to end in ByteChef.
