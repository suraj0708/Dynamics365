# CRM Notes — Dynamics 365

**Contributors:** Suraj · Deepak · Annu

---

## Security Structure in D365

D365 security is built on six layers that work together:

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

---

## Business Unit Hierarchy Example

```
Org
├── Sales BU
│   ├── Santosh (User)
│   └── Rudra  (Same level as Santosh)
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
| **Choice / OptionSet** | `.getValue()` → returns numeric value; `.getText()` → returns label |
| **Lookup** | `.getValue()` → returns array with `id`, `name`, `entityType` |

**Lookup result example:**
```js
[{ id: "12341234-...", name: "John", entityType: "contact" }]
```

**Customer field note:** A Customer field can point to both `contact` and `account`. Always check `entityType` before making a retrieval (Web API) call.

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
> **Workaround:** Use paging cookies to iterate through pages and retrieve all records.

---

### Async vs Sync in JavaScript

| | Sync | Async |
|---|---|---|
| **Execution** | Blocks until done | Non-blocking |
| **Use case** | Simple, sequential logic | API calls, Web API |
| **Risk** | Freezes UI if slow | Needs proper `await`/`.then()` handling |

**Example — 5 functions of 5 sec each:**
- **Sync:** 5 + 5 + 5 + 5 + 5 = **25 seconds total**
- **Async (parallel):** All run together → **~5 seconds total**

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

### Relationship Behavior (Parent → Child on Delete)

| Behavior | What happens to child record |
|---|---|
| **Cascade Delete** | Child is also deleted |
| **Remove Link** | Child stays, but reference (lookup) is cleared |
| **Restrict** | Parent cannot be deleted while child exists |

> 📝 **Homework:** In the Technician entity, find whichever lookup is available and update the Contact with the email from the current entity.

---

### Duplicate Detection Rules

- Configure rules to prevent duplicate records on Create/Update
- Triggered automatically when a matching rule is active

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

---

## Notifications in D365

| Type | Scope |
|---|---|
| **Field Level Notification** | Shows message/error on a specific field |
| **Form Level Notification** | Shows banner at the top of the form |
| **Global Level Notification** | Shown across the app |
| **In-App Notification** | Bell icon notification — persists in the notification center |
