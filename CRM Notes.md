# CRM Notes — Dynamics 365

**Contributors:** Suraj · Deepak · Annu

---

## Security Structure in D365

D365 security is built on layers that work together:

| Layer | Purpose |
|---|---|
| **User** | Individual identity in the system |
| **Teams** | Group of users sharing access |
| **Security Roles** | Define what actions users/teams can perform |
| **Business Unit** | Organizational boundary for data scoping |
| **Position Hierarchy** | Manager-subordinate access via reporting chain |
| **Column Security Profile** | Field-level access control |
| **Access Teams** | Record-level sharing without full role assignment |

---

## Org Structure Example

- **3 Locations → 3 Managers → 100 People each = 303 Total Users**
- Assign **System Administrator** role to all 3 managers
- Create **3 Teams** (one per location)

### Setup per Location (e.g. India)

1. India Manager is assigned to the India team
2. Manager manually adds all 100 people to that team
3. Create a **Security Role** → assign to the team
   - Role name example: `Driver & Vehicle Standard Agency`

---

## Security Roles

A security role is a combination of **Access Level** and **Privileges** for each entity.

### Access Levels

| Level | Scope |
|---|---|
| **User** | Only records owned by the user |
| **Business Unit** | Records within the same BU |
| **Parent: Child Business Unit** | Records in BU + child BUs |
| **Organization** | All records across the entire org |

### Privileges

| Privilege | Description |
|---|---|
| **Read** | View a record |
| **Write** | Update/edit a record |
| **Create** | Create a new record |
| **Delete** | Delete a record |
| **Append** | Attach this record to another record |
| **Append To** | Allow other records to attach to this one |
| **Share** | Perform activities on a record without changing ownership |
| **Assign** | Perform activities on a record and transfer ownership |

> **Note:** `Append` and `Append To` are **complementary permissions** — both must be set for lookups to work across entities.

### Append / Append To — Practical Example

> *"I have a lookup of Contact in the Technician entity, but I cannot access the Contact lookup."*

**Fix:**
- Give **Append To** privilege on the `Contact` table
- Give **Append** privilege on the `Technician` table

### Example Role: Technician Read-Only

Assigned to users like Suraj and Santosh — read-level access without write/create privileges on the Technician entity.

---

## Business Unit Hierarchy Example

```
Org
├── Sales BU
│   ├── Santosh (User)
│   ├── Rudra   (Same level as Santosh)
│   └── Manager
└── Marketing BU
    └── Suraj (User)
```

> *Manager sits above the BU level and oversees both.*

---

## JavaScript in D365 — Key Concepts

### Getting Field Values

| Field Type | How to Get Value |
|---|---|
| **Simple Text** | `formContext.getAttribute("fieldname").getValue()` |
| **Choice / OptionSet** | `.getValue()` → numeric value; `.getText()` → label |
| **Lookup** | `.getValue()` → array with `id`, `name`, `entityType` |

**Lookup result example:**
```js
[{ id: "12341234-...", name: "John", entityType: "contact" }]
```

**Customer field note:** A Customer field can point to both `contact` and `account`. Always check `entityType` before making a retrieval (Web API) call so you target the right table.

---

### Async vs Sync in JavaScript

| | Sync | Async |
|---|---|---|
| **Execution** | Blocks until done | Non-blocking |
| **Use case** | Simple, sequential logic | API calls, Web API |
| **Risk** | Freezes UI if slow | Needs proper `await` / `.then()` handling |

**Example — 5 functions of 5 sec each:**
- **Sync:** 5 + 5 + 5 + 5 + 5 = **25 seconds total**
- **Async (parallel):** All run together → **~5 seconds total**

---

### Web API vs FetchXML

| Use **Web API (`$filter`)** when… | Use **FetchXML** when… |
|---|---|
| Simple filtering | Count / Sum / Aggregate |
| Small dataset | Joins between tables |
| Quick form scripts | Large datasets |
| | Complex queries |

#### Why FetchXML is Faster (in some scenarios)

- **Server-side execution** — processed directly by Dataverse, less translation overhead
- **Better for joins & aggregates** — complex queries with linked entities, group by, count, sum are optimized
- **Smaller payloads** — you control exactly which fields come back
- **Built-in paging & filters** — efficient for large datasets

> **Limitation:** Dataverse FetchXML has a **5,000 record limit** per page.
> **Workaround:** Use **paging cookies** (`page`, `count`, `paging-cookie`) in FetchXML to iterate through pages. For Web API, use `@odata.nextLink` / `<MaxPageSize>` headers to follow continuation tokens.

---

### Web API Operations

```
✅ Create a single record
✅ Retrieve a single record
✅ Retrieve multiple records
✅ Update a record
✅ Delete a record
```

#### Delete a Record via Web API
```js
Xrm.WebApi.deleteRecord("entityname", recordId).then(
  function success(result) { console.log("Deleted: " + result.id); },
  function error(error) { console.log(error.message); }
);
```

---

### Client API Cheat Sheet (already covered)

**WebAPI calls:**
- `Xrm.WebApi.createRecord`
- `Xrm.WebApi.retrieveRecord`
- `Xrm.WebApi.retrieveMultipleRecords`
- `Xrm.WebApi.updateRecord`
- `Xrm.WebApi.deleteRecord`

**Navigation / dialogs:**
- `Xrm.Navigation.openAlertDialog`
- `Xrm.Navigation.openConfirmDialog`
- `Xrm.Navigation.openForm`

**Form lifecycle / context:**
- `getFormType` — 1 = Create, 2 = Update, etc.
- `getFormSelector` — `true` = main form, `false` = quick create form
- `save` — force save
- `preventDefault` — block form save

**Attribute / control:**
- `getValue` / `setValue`
- `getAttribute` / `getControl`
- `fireOnChange`
- `setDisabled`
- `setVisible`
- `setRequiredLevel`
- `getId()`
- `addOnChange` / `removeOnChange`

**Notifications (JS):**
- `addGlobalNotification`
- `setFormNotification` / `clearFormNotification`
- `setNotification` / `clearNotification`

---

### Snippets

```javascript
// Alert dialog
var alertStrings = {
    text: "Please fill all required fields",
    title: "Validation",
    confirmButtonLabel: "OK"
};
Xrm.Navigation.openAlertDialog(alertStrings);
```

```javascript
// Confirm dialog
var confirmStrings = {
    text: "Do you want to continue?",
    title: "Confirmation",
    confirmButtonLabel: "Yes",
    cancelButtonLabel: "No"
};
Xrm.Navigation.openConfirmDialog(confirmStrings);
```

```javascript
// Open a form
var entityFormOptions = { entityName: "account" };
Xrm.Navigation.openForm(entityFormOptions);
```

```javascript
// Environment / user info
Xrm.Utility.getGlobalContext().getClientUrl();
Xrm.Utility.getGlobalContext().userSettings.userName;
Xrm.Utility.getGlobalContext().userSettings.userId;
```

```javascript
// Block form save
executionContext.getEventArgs().preventDefault();
```

```javascript
// Wire an onChange
formContext.getAttribute("category").addOnChange(santhoshUpdate);
```

---

### Operators worth remembering

```javascript
// Ternary
condition ? trueValue : falseValue

// Nullish coalescing — falls back only on null/undefined (not 0 or "")
const name = input ?? "Default";
```

---

### JavaScript Topics Covered

- `GET` / `SET` field value
- `SHOW` / `HIDE` field
- Make field **mandatory** or **read-only**
- Get a particular record's information
- Get multiple results
- Sync vs Async call behavior and code impact
- Get the **User ID** of the current logged-in user
- Get value from a **Customer lookup** + its Web API call
- Difference between **Async and Sync** functions
- **Create** a record using Web API
- Use **Duplicate Detection Rules**
- Use of **FetchXML** in JS code
- Overcome **5,000 record limit** in Dataverse
- **Delete** a record via Web API
- **Relationship Behavior** (what happens to child when parent is deleted)
- Show/hide of tabs and sections
- Get current username — `Xrm.Utility.getGlobalContext().userSettings.userName`
- Logged-in user ID — `Xrm.Utility.getGlobalContext().userSettings.userId`
- Security roles — get/set and show/hide

---

## Form / UX Topics

- **Quick create form**
- **Show/hide tabs and sections**
- **Enable/disable autosave:** Advanced Settings → Administration → System Settings
- **Get/set related to BPF**
- `addPreSearch` / custom filter on lookups
- `getValue` of header process fields
- What happens if the same column is added twice on a form (both controls reference the same attribute; setting one syncs both, but each has its own `getControl` instance for visibility/disabled state).
- `addCustomView`
- **Get environment variable values** (using `$expand` in WebAPI)

---

## Relationship Behavior (Parent → Child on Delete)

| Behavior | What happens to child record |
|---|---|
| **Cascade All / Parental** | Child is also deleted with the parent |
| **Remove Link / Referential** | Child stays, but reference (lookup) is cleared |
| **Restrict** | Parent cannot be deleted while child exists |

Append / Append To privileges control whether the relationship can be formed in the first place.

> 📝 **Homework (Done):** In the Technician entity, find whichever lookup is available and update the Contact with the email from the current entity.

---

## Duplicate Detection Rules

- Configure rules to prevent duplicate records on Create/Update
- Triggered automatically when a matching rule is active

---

## Notifications in D365

| Type | Scope |
|---|---|
| **Field Level Notification** | Shows message/error on a specific field — levels: `Required`, `Recommended`, `None` |
| **Form Level Notification** | Shows banner at the top of the form |
| **Global Level Notification** | Shown across the app |
| **In-App Notification** | Bell icon notification — persists in the notification center |

---

## Plugins

### What is a plugin?

A .NET assembly that runs server-side in response to Dataverse events (Create, Update, Delete, etc.).

### Pipeline Stages

| Stage | Number | Rollback supported? |
|---|---|---|
| **Pre-Validation** | 10 | ❌ No |
| **Pre-Operation** | 20 | ✅ Yes |
| **Core Operation** | 30 | (platform) |
| **Post Operation** | 40 | ✅ Yes |

- **Pre-Validation** runs **outside** the database transaction — useful for validation that should fail fast, but rollback is not supported.
- **Pre-Operation** runs **inside** the transaction, before the record is committed. Modifications to the Target entity here are persisted with the original operation.
- For both Pre-Validation and Pre-Operation, the record has not yet been created — it's "on the way."

### What's Been Covered

- How to create a plugin
- How to register a plugin
- Get and set values (string, lookup, etc.)
- Making an update request (and which stage is appropriate)
- How to get the current stage name from inside the plugin

### Open Question (Deepak)

> On create of Department table, "Primary Technician" field should be hidden and not required.
> On update, the field should be visible and required.

This is a form-side behavior, not a plugin behavior — handle via JS using `getFormType()` (1 = Create, 2 = Update) plus `setVisible` and `setRequiredLevel`.

### BPF Questions

- Get the current stage
- Set the stage via JS
- Can we hide a stage? (BPF stages can't be hidden directly, but the entire BPF can be hidden, or stages can be skipped via branching conditions.)

Example setup mentioned: a Technician record with two BPFs, both having a lookup back to Technician.

---

## Homework / TO DO

### Completed ✅

- Show/hide of tabs and sections
- Get current username — `Xrm.Utility.getGlobalContext().userSettings.userName`
- Get logged-in user ID — `Xrm.Utility.getGlobalContext().userSettings.userId`
- Security role get/set and show/hide
- Original Technician homework — update Contact with email from current entity via lookup
- Get environment variable values via `$expand` in WebAPI
- Get/set BPF
- `addPreSearch` / custom filter on lookup

### Outstanding 📝

1. **Service Ticket notification function**
   Write a function in the Service Ticket JS that gathers the relevant info (GUID, Technician, etc.) and creates an in-app notification record sent to the user in the "Send Notification To" field.

2. **Auto-create in-app notification on Service Ticket create/assign**
   When a Service Ticket is created or assigned, send an in-app notification to the user listed in the Technician's "Send Notification To" field, and update the Contact lookup with the email available in the current entity.

3. **Show team members on Technician form**
   When the Owner field on a Technician record is set to a Team, show a custom field (to be created) and populate it with the names of all team members.

4. **Team membership check**
   Check via JS whether the current user is part of the "Suraj Team."

5. **Config separation**
   Move all config items into a separate JS library and reference it from the main library. Goal: demonstrate the importance of library load order.

6. **Conditional optionset filtering**
   Add/remove optionset values dynamically based on another field's selection. E.g., if Option A is selected, show A/B/C in the dependent field; if Option 1 is selected, show 1/2/3.

### Topics Still to Cover in JS Training

- `getValue` of header process fields
- Same column added twice on a form — value retrieval behavior
- `addCustomView`

---

## Misc Notes

- Hyderabad Nawabs
- Khichdi
