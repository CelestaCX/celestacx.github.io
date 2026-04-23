# Forms & Form Builder

Here is the **Forms & Form Builder** page:

---

## Forms & Form Builder

CelestaCX includes a built-in form builder that allows administrators to create structured data-collection forms without writing any code. Forms appear to agents on the Agent Desk during or after interactions, guiding them through consistent data capture — for customer verification, case intake, after-call work, escalation handling, and more.

---

### What Forms Are Used For

Forms serve several distinct purposes within CelestaCX:

| Form Type | When It Appears | Typical Use |
| --- | --- | --- |
| Customer Verification Form | When an interaction is accepted by an agent | Collecting or confirming identity information before the conversation begins |
| Case Intake Form | During an active interaction | Capturing the reason for contact, relevant details, and customer data |
| After-Contact Work (ACW) Form | After an interaction ends, during wrap-up | Categorising the interaction, logging outcomes, and completing required fields |
| Escalation Form | When an agent initiates a transfer or escalation | Capturing context to pass to the receiving agent or team |
| Custom Form | Triggered manually or by a routing rule | Flexible data collection for any operational workflow |

A single form can be linked to multiple queues or interaction types. The same form can also be reused across channels — the agent sees the same structure whether they are handling a voice call or a WhatsApp message.

---

### Form Builder Overview

The form builder is accessed at **Administration → Forms → Form Builder**. It provides a drag-and-drop interface for assembling forms from a library of field types, with no technical skills required.

#### Supported Field Types

| Field Type | Description |
| --- | --- |
| Text (single line) | Free-text input for short values such as names, reference numbers, or notes. |
| Text (multi-line) | Larger text area for longer notes or descriptions. |
| Dropdown | A predefined list of options. The agent selects one value. |
| Multi-select | A predefined list where the agent can select multiple values. |
| Checkbox | A single true/false toggle. Useful for confirmations or binary flags. |
| Date | A date picker. Useful for scheduling, date of birth, or event dates. |
| Date & Time | A combined date and time picker. |
| Number | Numeric input only. Supports validation for minimum and maximum values. |
| Phone Number | A formatted phone number field with optional country code selector. |
| Email | An email address field with format validation. |
| URL | A web address field with format validation. |
| Section Header | A non-input element used to visually divide the form into labelled sections. |
| Instructional Text | Static text displayed to the agent to provide guidance or context within the form. |

---

### Creating a Form

1. Navigate to **Administration → Forms → Form Builder → New Form**.
2. Enter a form name and optional description. The name is shown to agents at the top of the form panel.
3. Select the **form type** (Verification, Case Intake, ACW, Escalation, or Custom). This determines where the form appears in the agent workflow.
4. Build the form by dragging fields from the field library on the left into the form canvas.
5. For each field, configure its properties:

| Property | Description |
| --- | --- |
| Label | The text displayed above the field. Keep it short and clear. |
| Placeholder | Optional hint text shown inside the field before the agent types. |
| Required | If enabled, the agent cannot submit the form without completing this field. |
| Default Value | A pre-populated value. Can be static or dynamically populated from interaction attributes or CRM data (see Dynamic Population below). |
| Validation | Field-specific rules — minimum/maximum length, numeric ranges, format checks. |
| Help Text | Optional tooltip or inline guidance shown to the agent when they hover or focus on the field. |

1. Use **Section Headers** and **Instructional Text** fields to structure the form logically, especially for longer forms.
2. Set the form **visibility conditions** if certain fields should only appear based on other field values (see Conditional Logic below).
3. Preview the form using the **Preview** button to see how it will appear to agents.
4. Save the form as a **Draft** or publish it immediately by setting the status to **Active**.

---

### Conditional Logic

Fields can be shown or hidden based on the values entered in other fields. This keeps forms concise — agents only see fields that are relevant to their current situation.

#### Setting Up a Condition

1. Select the field you want to show or hide conditionally.
2. In the field properties panel, open the **Visibility** section.
3. Click **Add Condition**.
4. Select the **source field** whose value will control visibility.
5. Select the **operator** (equals, does not equal, contains, is empty, is not empty).
6. Enter the **value** to match against.
7. Set whether the field is **shown** or **hidden** when the condition is met.

Multiple conditions can be combined using **AND** (all must be true) or **OR** (any one must be true) logic.

#### Example

A form has a dropdown field called "Contact Reason" with options including "Billing Query" and "Technical Issue". A second field, "Invoice Reference Number", should only appear when the agent selects "Billing Query". Setting a visibility condition on "Invoice Reference Number" where Contact Reason equals "Billing Query" achieves this cleanly.

---

### Dynamic Field Population

Form fields can be pre-populated automatically using data that CelestaCX already holds about the interaction or the customer. This reduces manual data entry and speeds up the agent workflow.

#### Sources for Dynamic Population

| Source | Description |
| --- | --- |
| Interaction Attributes | Data collected before the interaction reached the agent — from a bot, IVR, or routing rule. For example, a bot may have captured the customer's account number. |
| CRM Connector Data | If a CRM connector is active and a customer record has been matched, CRM fields can be mapped directly into form fields. |
| Agent Desk Context | Certain platform values — such as queue name, channel type, and interaction ID — are always available for population. |

Dynamic population is configured per field under **Administration → Forms → [Form] → [Field] → Default Value → Dynamic Source**.

---

### Linking Forms to Queues

Forms are activated by linking them to queues. When an interaction arrives from a linked queue, the appropriate form is automatically presented to the agent.

1. Navigate to **Administration → Queues → [Queue] → Edit**.
2. In the **Forms** section, assign forms by type:

  - Verification Form
  - Case Intake Form
  - After-Contact Work Form
  - Escalation Form
3. Save. The assigned forms will appear in the agent's interaction panel for all interactions from this queue.

A queue can have one form of each type assigned at a time. If no form is assigned for a given type, that form panel does not appear to agents handling interactions from that queue.

---

### Form Submission & Data Storage

When an agent submits a form, the data is:

- **Stored against the interaction record** and accessible in interaction history and reporting.
- **Available in historical reports** — form field values can be included as dimensions or filters in custom reports. See [Historical Reports](#).

Submitted form data is retained according to your platform data retention policy, configured under **Platform Administration → Data Retention**.

---

### Managing Forms

#### Editing a Published Form

Forms can be edited at any time. Changes take effect immediately for new interactions. Interactions already in progress at the time of an edit will continue to display the previous version of the form until they are completed.

#### Versioning

CelestaCX maintains a version history for each form. Previous versions can be viewed but not restored directly through the UI. If you need to revert to an earlier version, contact your platform administrator.

#### Deactivating a Form

Setting a form's status to **Inactive** removes it from all queues it is linked to. The form and all its historical submission data are retained. To reactivate, set the status back to **Active** and relink it to the relevant queues.

#### Deleting a Form

Forms with submission history cannot be permanently deleted, as doing so would remove associated interaction records. Instead, deactivate forms that are no longer needed.

---

### Form Builder Best Practices

- **Keep forms short.** Agents are completing forms while managing live interactions. Aim for the minimum fields needed to capture required information accurately.
- **Use required fields sparingly.** Only mark fields as required if the data is genuinely necessary for every interaction. Over-use of required fields slows agents down and creates friction.
- **Use section headers** for forms with more than six fields to break content into logical groups.
- **Test conditional logic** thoroughly before publishing. Use the Preview function and walk through every condition path to confirm fields show and hide correctly.
- **Use descriptive form names** that indicate the queue or workflow they belong to — for example, "Billing Support — ACW Form" rather than just "ACW Form".
- **Review ACW forms regularly** against your reporting needs. If analysts need a new category in a dropdown, update the form and communicate the change to agents.

---

*What's next: See the*[*Admin Setup Guide*](#)*for a step-by-step walkthrough that ties all configuration areas together into a structured onboarding and setup workflow.*
