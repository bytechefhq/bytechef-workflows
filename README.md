# ByteChef Workflow Templates

A collection of example workflows for use with the [ByteChef](https://github.com/bytechefhq/bytechef)
platform. Every folder under [`workflows/`](workflows) is published as a **pre-built workflow
template**: it shows up in the ByteChef UI under *Explore Workflow Templates*, where a user can
preview it and import it into one of their projects with a single click.

This README describes what a contribution must contain to be accepted and to render correctly.

> **Project templates** are not supported by this repository yet. For now, only workflow templates
> can be submitted. Instructions for project templates will be added here once the format is
> settled.

## How a template reaches the ByteChef UI

```
workflows/<folder>/                    this repository (default branch: master)
        |
        |  raw.githubusercontent.com
        v
github-proxy.bytechef.io               reads meta.json + workflow_definition.json,
GET /workflow-templates                derives slug, components and trigger,
GET /workflow-templates/{slug}         serves the template
        |
        v
ByteChef server -> GraphQL             preBuiltWorkflowTemplates
        |
        v
Explore Workflow Templates             card list + detail page + "import into project"
```

Two consequences worth knowing before you start:

- **Merging is publishing.** There is no build step, no registry and no review queue after the
  merge. As soon as your folder is on the default branch, the proxy picks it up on its next cache
  miss and the template is live.
- **The folder name is the public identifier.** The proxy derives the template slug from it
  (`_` becomes `-`, lowercased), and that slug is the URL of the template detail page. Renaming a
  folder later breaks every existing link to it, so choose the name carefully the first time.

## Repository layout

```
workflows/
  ai_email_classifier/
    meta.json                  <- template metadata (required)
    workflow_definition.json   <- the workflow itself (required)
    screenshot.png             <- placeholder for the workflow canvas image
  cv_scanning/
    ...
```

The first two rules are enforced by the loader, the third is convention:

- Exactly **one level** of nesting. `workflows/<folder>/<file>` is read; anything deeper is ignored.
- The folder **must** contain both `meta.json` and `workflow_definition.json`. A folder missing
  either one is skipped without any error being reported.
- Folder names are `lower_snake_case`: no spaces, and no dashes — a dash is what `_` becomes when the
  slug is built. Examples: `cv_scanning`, `out_of_working_hours`,
  `monthly_report_generation_for_nifty_project`.

## Submitting a workflow template

### 1. Build and test the workflow in ByteChef

Get it working end to end in a real project first. A template that has never run is not a template,
it is a draft. Pay attention to anything environment-specific you configure along the way
(spreadsheet IDs, form IDs, label IDs, channel names, app IDs) — you will have to declare those in
`prerequisites`.

### 2. Export the workflow definition

Export the workflow as JSON from the ByteChef workflow editor and save it as
`workflows/<your_folder>/workflow_definition.json`. Keep the exported formatting as-is; there is no
need to reformat it.

Then clean it up:

- **Remove anything private.** Personal email addresses, internal channel names, API keys, tokens,
  customer data, real record IDs from a production account. Replace them with neutral placeholders
  (`support@example.com`, `your-channel`). Note that connections are *not* part of the export — the
  importing user picks their own — but resource IDs typed into parameters are. Leaving an ID that
  points at a throwaway demo resource is fine, as long as `prerequisites` tells the user what they
  have to swap out.
- **Give it a meaningful `label` and `description`.** These two fields are shown on the template's
  detail page in the UI, so they are not internal notes.
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

### 3. Write `meta.json`

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

See the [`meta.json` field reference](#metajson-field-reference) below for what each field means and
where it is displayed.

### 4. Add `screenshot.png`

Add a `screenshot.png` file to the folder. Every existing template carries one as a zero-byte
placeholder: the image is not fetched by the template API yet, but the file and the `"screenshot"`
key in `meta.json` are part of the folder contract, so include both. If you have a real screenshot
of the workflow canvas, commit it — it will be used once image support lands.

### 5. Open a pull request

One template per pull request. In the description, say what the workflow automates and confirm you
ran it successfully. Work through the [submission checklist](#submission-checklist) first.

## `meta.json` field reference

| Field | Required | Type | Where it is used |
| --- | --- | --- | --- |
| `name` | **Yes** | string | Title on the template card in the UI. **Must be non-blank** — a blank or missing `name` causes the whole folder to be skipped. Keep it consistent with the definition's `label`. |
| `category` | Yes (by review) | string | Category badge on the card, and the category filter. Must be one of the [supported category IDs](#categories). |
| `description` | Yes (by review) | string | Description on the template card, and one of the two fields template search matches against. Two to four sentences saying what the workflow does, concretely. The card clamps it to two lines, so front-load the meaning. |
| `shortDescription` | Yes (by review) | string | One-sentence tagline. Served by the template API; not rendered in the app UI yet. |
| `author` | Yes (by review) | object | `name`, `email`, `role`, `socialLinks`, all strings. The block must be present; leave the fields as empty strings if you would rather not be credited. Attribution is served on the template detail page — the list endpoint returns the author fields empty. |
| `keyFeatures` | Yes (by review) | string[] | Capability bullets ("AI urgency classification of the incoming email"). Served by the template API. |
| `prerequisites` | Yes (by review) | string[] | Everything a user must have before importing: each connection, each API key, and each resource that must already exist (a spreadsheet, a Gmail label, a database table). Be specific — this is the field that decides whether an imported template actually runs. |
| `idealFor` | Yes (by review) | string | One sentence naming the audience: "Recruiters and hiring teams that want to pre-screen incoming job applications automatically." |
| `steps` | Yes (by review) | string[] | The workflow narrated step by step, one sentence per step, in execution order. Start with `"Trigger: ..."` when the workflow has a trigger. |
| `screenshot` | Yes (by review) | string | Always `"screenshot.png"`. |

"Required by review" means the loader tolerates the field being absent (it defaults to an empty
string or an empty array), but a pull request without it will not be merged. Only `name` is
technically enforced.

### Categories

`category` takes exactly one ID from the vocabulary shared with the ByteChef UI:

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

Note the exact spelling of `documentOps` (camelCase). An ID outside this list is not rejected — it is
passed straight through, so the badge renders the raw string and no category filter will ever match
your template. A blank category falls back to `other`.

## What is derived automatically

Do **not** hand-write these into `meta.json`; they are computed and any copy you add is ignored.

| Derived value | How |
| --- | --- |
| `slug` | The folder name, `_` replaced with `-`, lowercased. `out_of_working_hours` becomes `out-of-working-hours`. |
| `components` | The component of every task and trigger whose `type` matches `component/vN/operation`, in order of first appearance, deduplicated. This drives the component icon row on the card. Two-segment task-dispatcher types (`loop/v1`, `condition/v1`, `branch/v1`) and property types (`STRING`, `OBJECT`, `ARRAY`) are correctly excluded. |
| `trigger` / `triggerLabel` | The first entry of `triggers`: the component part of its `type`, and its `label`. |

Because component icons are resolved from these names against the components registered in
ByteChef, a typo in a `type` value means a missing icon — another reason to export the definition
from the editor rather than writing it by hand.

## Where each piece of text shows up

Two files each supply a title and a description, and they surface in different places:

| UI surface | Title comes from | Description comes from |
| --- | --- | --- |
| Template card (list) | `meta.json` → `name` | `meta.json` → `description` |
| Template detail page | `workflow_definition.json` → `label` | `workflow_definition.json` → `description` |

Keep `meta.json`'s `name` and the definition's `label` in agreement, and make sure both descriptions
say the same thing — otherwise a user sees the template renamed or re-described when they click into
it. Template search only matches `meta.json`'s `name` and `description`, so the words a user would
search for have to appear there — putting them solely in the definition will not make the template
findable.

## Submission checklist

- [ ] The folder is `workflows/<lower_snake_case_name>/`, one level deep.
- [ ] It contains `meta.json`, `workflow_definition.json` and `screenshot.png`.
- [ ] Both JSON files parse (`python3 -m json.tool <file>` or an editor that validates JSON).
- [ ] `meta.json` has a non-blank `name`, and `category` is one of the IDs listed above.
- [ ] `name` matches the definition's `label`; the two descriptions agree.
- [ ] Every field in the reference table is filled in, and `author` is present (empty strings are fine).
- [ ] `prerequisites` lists every connection, API key and pre-existing resource the workflow needs.
- [ ] `steps` narrates the workflow in execution order.
- [ ] No credentials, personal email addresses, internal channel names or customer data anywhere in the definition.
- [ ] The workflow was actually run end to end in ByteChef.

## Troubleshooting

**My template does not appear in the UI.** Both ways a folder gets dropped are silent, so check them
in this order:

1. Are both `meta.json` and `workflow_definition.json` present, directly inside the folder and not in
   a subdirectory? A folder missing either is excluded during folder discovery.
2. Is `meta.json` valid JSON with a non-blank `name`? A parse failure or a blank name is caught and
   logged as `Skipping workflow folder '<name>'`, and the rest of the catalog loads without it.
3. Is the change on the default branch? The proxy reads `master`, not your PR branch.
4. Give it a moment — the proxy caches the folder list and each template (about a minute by
   default), so a freshly merged template does not appear instantly.

You can verify what the platform actually sees for your template:

```bash
curl -s https://github-proxy.bytechef.io/workflow-templates/<your-slug> | python3 -m json.tool
```

**My template appears but has no category filter.** The `category` value is not one of the supported
IDs — check the spelling, especially `documentOps`.

**My template appears but shows no component icons.** A `type` value in the definition does not match
a registered component. Re-export the definition from the ByteChef editor instead of editing types by
hand.
