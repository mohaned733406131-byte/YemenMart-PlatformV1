# 23 - Templates

**Category:** Templates  
**Purpose:** Document templates and boilerplates for consistent documentation

---

## Contents

This directory contains reusable templates for various document types used throughout the project.

### Document Templates
- `requirement-template.md` - Functional/non-functional requirement format
- `use-case-template.md` - Use case documentation format
- `adr-template.md` - Architecture Decision Record format
- `test-case-template.md` - Test case specification format
- `api-endpoint-template.md` - API endpoint documentation format
- `database-table-template.md` - Database table specification format
- `meeting-notes-template.md` - Meeting notes format
- `incident-report-template.md` - Incident report format

---

## Requirement Template

### File: `requirement-template.md`

```markdown
---
document_id: [FR-NNN | NFR-TYPE-NNN]
title: [Requirement Title]
category: [functional | non-functional | security | data | integration]
status: [DRAFT | APPROVED | IMPLEMENTED | VERIFIED]
version: 1.0.0
created: YYYY-MM-DD
updated: YYYY-MM-DD
author: [Author Name]
source_of_truth: true
related_requirements: [FR-NNN, NFR-NNN]
related_documents: [Document IDs]
---

# [Requirement ID]: [Requirement Title]

## Description
Brief description of what this requirement addresses.

## Acceptance Criteria (EARS Format)
**AC-XXX-001:**
WHEN [precondition]
THE [system] SHALL [requirement]

**AC-XXX-002:**
WHEN [precondition]
THE [system] SHALL [requirement]

## Business Rules
- BR-XXX-01: [Business rule description]
- BR-XXX-02: [Business rule description]

## Dependencies
- Depends on: [Other requirements]
- Blocks: [Other requirements]

## Use Cases
- UC-X01: [Use case name]
- UC-X02: [Use case name]

## Design Components
- Component: [Component name]
- Database: [Table names]
- API: [Endpoint paths]

## Test Cases
- TC-XXX-001: [Test case name]
- TC-XXX-002: [Test case name]

## Notes
Additional notes, constraints, or considerations.
```

---

## Use Case Template

### File: `use-case-template.md`

```markdown
# UC-[ACTOR]-[NN]: [Use Case Name]

**Actor:** [Customer | Vendor | Admin | Delivery Provider | System]
**Goal:** Brief statement of actor's goal
**Frequency:** [Daily | Weekly | Monthly | On-demand]
**Priority:** [Critical | High | Medium | Low]

## Preconditions
- Condition 1
- Condition 2

## Basic Flow
1. Actor performs action A
2. System responds with B
3. Actor performs action C
4. System completes D

## Alternative Flows
### Alt-1: [Alternative scenario]
1. If condition X occurs
2. System performs Y
3. Return to step N of basic flow

## Postconditions
- Condition 1 achieved
- Condition 2 achieved

## Business Rules
- BR-XXX-01: [Rule description]

## Related Requirements
- FR-XXX: [Requirement name]

## UI/UX Considerations
- UI element requirements
- User experience notes

## Test Scenarios
- TC-XXX-001: [Test case name]
```

---

## Architecture Decision Record (ADR) Template

### File: `adr-template.md`

```markdown
# ADR-XXX: [Decision Title]

**Status:** [Proposed | Accepted | Rejected | Deprecated | Superseded]
**Date:** YYYY-MM-DD
**Deciders:** [Name 1, Name 2, Name 3]
**Related ADRs:** [ADR-NNN, ADR-NNN]

## Context
What is the issue we're trying to address? What factors led to this decision?

Describe the forces at play:
- Technical factors
- Business constraints
- Resource limitations
- Team skills
- Market pressures

## Decision
We have decided to [decision statement].

Example: "We will use PostgreSQL as our primary database."

## Rationale
Why did we make this decision?

- Reason 1: [Explanation]
- Reason 2: [Explanation]
- Reason 3: [Explanation]

## Consequences

### Positive Consequences
- ✅ Benefit 1
- ✅ Benefit 2
- ✅ Benefit 3

### Negative Consequences
- ❌ Drawback 1
- ❌ Drawback 2

### Neutral Consequences
- Trade-off 1
- Trade-off 2

## Alternatives Considered

### Alternative 1: [Option name]
- **Pros:** [Benefits]
- **Cons:** [Drawbacks]
- **Reason for rejection:** [Explanation]

### Alternative 2: [Option name]
- **Pros:** [Benefits]
- **Cons:** [Drawbacks]
- **Reason for rejection:** [Explanation]

## References
- [Link to research]
- [Link to documentation]
- [Link to discussion]

## Notes
Additional context or implementation notes.
```

---

## Test Case Template

### File: `test-case-template.md`

```markdown
# TC-[BLOCK]-[NNN]: [Test Case Name]

**Type:** [Unit | Integration | E2E | Performance | Security]
**Priority:** [Critical | High | Medium | Low]
**Status:** [Not Run | Passed | Failed | Blocked]
**Related Requirement:** FR-NNN / NFR-NNN
**Related Use Case:** UC-X-NN

## Objective
What this test is validating.

## Preconditions
- System state before test
- Test data required
- Environment setup

## Test Steps
1. Step 1 action
2. Step 2 action
3. Step 3 action

## Expected Result
What should happen if the test passes.

## Actual Result
[To be filled during test execution]

## Pass/Fail Criteria
- Criterion 1
- Criterion 2

## Test Data
```json
{
  "field1": "value1",
  "field2": "value2"
}
```

## Dependencies
- Depends on: TC-XXX-NNN

## Automation
- [ ] Automated
- [ ] Manual only
- **Automation Framework:** [Jest | Playwright | k6]
- **Test File:** `path/to/test.spec.ts`

## Notes
Additional notes or known issues.
```

---

## API Endpoint Template

### File: `api-endpoint-template.md`

```markdown
# [HTTP METHOD] /path/to/endpoint

**Category:** [Authentication | Customers | Products | Orders | Payments]
**Authentication:** [Required | Optional | None]
**Rate Limit:** [100 req/min | 20 req/min]
**Version:** v1

## Description
Brief description of what this endpoint does.

## Request

### Headers
```http
Authorization: Bearer {jwt_token}
Content-Type: application/json
X-Request-ID: {unique_id}
```

### Path Parameters
- `{param1}` (string, required) - Description
- `{param2}` (integer, optional) - Description

### Query Parameters
- `limit` (integer, optional, default: 20) - Page size
- `cursor` (string, optional) - Pagination cursor
- `sortBy` (string, optional) - Sort field

### Request Body
```json
{
  "field1": "string",
  "field2": 123,
  "field3": {
    "nested": "object"
  }
}
```

### Request Body Schema
- `field1` (string, required) - Description
- `field2` (integer, optional) - Description

## Response

### Success Response (200 OK)
```json
{
  "data": {
    "id": "123",
    "field1": "value1",
    "field2": 456
  },
  "meta": {
    "timestamp": "2026-09-15T10:30:00Z"
  }
}
```

### Error Response (400 Bad Request)
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input parameters",
    "details": {
      "field1": "Field is required"
    },
    "timestamp": "2026-09-15T10:30:00Z"
  }
}
```

### Response Codes
- `200` - Success
- `201` - Created
- `400` - Bad Request
- `401` - Unauthorized
- `403` - Forbidden
- `404` - Not Found
- `429` - Too Many Requests
- `500` - Internal Server Error

## Examples

### cURL Example
```bash
curl -X POST https://api.yemenmart.com/v1/path/to/endpoint \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{"field1": "value1"}'
```

### JavaScript Example
```javascript
const response = await fetch('https://api.yemenmart.com/v1/path/to/endpoint', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({ field1: 'value1' })
});
```

## Business Rules
- BR-XXX-01: [Business rule description]

## Related Endpoints
- GET /path/to/related - [Description]

## Notes
Additional implementation notes or considerations.
```

---

## Database Table Template

### File: `database-table-template.md`

```markdown
# Table: `table_name`

**Schema:** public / specific_schema
**Purpose:** Brief description of table purpose
**Related Tables:** table1, table2, table3

## Columns

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | uuid | NOT NULL | gen_random_uuid() | Primary key |
| created_at | timestamp | NOT NULL | now() | Creation timestamp |
| updated_at | timestamp | NOT NULL | now() | Last update timestamp |
| field1 | varchar(255) | NOT NULL | - | Description |
| field2 | integer | NULL | 0 | Description |

## Indexes

| Name | Type | Columns | Purpose |
|------|------|---------|---------|
| table_name_pkey | PRIMARY KEY | id | Primary key |
| idx_table_name_field1 | BTREE | field1 | Fast lookup by field1 |
| idx_table_name_created_at | BTREE | created_at | Time-based queries |

## Foreign Keys

| Name | Column | References | On Delete | On Update |
|------|--------|------------|-----------|-----------|
| fk_table_name_user_id | user_id | users(id) | CASCADE | CASCADE |

## Constraints

| Name | Type | Definition | Description |
|------|------|------------|-------------|
| ck_table_name_amount | CHECK | amount >= 0 | Amount must be non-negative |

## Sample Data
```sql
INSERT INTO table_name (id, field1, field2) VALUES
('uuid-1', 'value1', 100),
('uuid-2', 'value2', 200);
```

## Partitioning
- **Type:** [Range | List | Hash]
- **Key:** created_at
- **Strategy:** Monthly partitions

## Retention Policy
- **Retention:** 7 years
- **Archival:** Move to cold storage after 1 year

## Related Requirements
- FR-XXX: [Requirement name]

## Notes
Additional notes about data patterns, performance considerations, or migrations.
```

---

## Meeting Notes Template

### File: `meeting-notes-template.md`

```markdown
# Meeting: [Meeting Title]

**Date:** YYYY-MM-DD
**Time:** HH:MM - HH:MM
**Location:** [Virtual / Office / Room]
**Attendees:** Name 1, Name 2, Name 3
**Note Taker:** [Name]

## Agenda
1. Agenda item 1
2. Agenda item 2
3. Agenda item 3

## Discussion

### Topic 1: [Topic Name]
- Discussion point 1
- Discussion point 2
- **Decision:** [Decision made]

### Topic 2: [Topic Name]
- Discussion point 1
- Discussion point 2
- **Decision:** [Decision made]

## Action Items
| Action | Owner | Due Date | Status |
|--------|-------|----------|--------|
| Action 1 description | Name 1 | YYYY-MM-DD | Open |
| Action 2 description | Name 2 | YYYY-MM-DD | Open |

## Decisions Made
1. Decision 1 description
2. Decision 2 description

## Parking Lot
- Topic to revisit later
- Question for follow-up

## Next Meeting
- **Date:** YYYY-MM-DD
- **Focus:** [Focus area]
```

---

## Incident Report Template

### File: `incident-report-template.md`

```markdown
# Incident Report: [INC-YYYYMMDD-NN]

**Status:** [Open | Investigating | Resolved | Closed]
**Severity:** [Critical | High | Medium | Low]
**Reported:** YYYY-MM-DD HH:MM
**Resolved:** YYYY-MM-DD HH:MM
**Duration:** [Duration]

## Summary
Brief description of the incident.

## Impact
- **Users Affected:** [Number/Percentage]
- **Services Affected:** [Service names]
- **Business Impact:** [Revenue loss, customer complaints, etc.]

## Timeline
| Time | Event |
|------|-------|
| 10:00 | Incident detected by monitoring |
| 10:05 | Alert triggered, on-call notified |
| 10:10 | Investigation started |
| 10:30 | Root cause identified |
| 10:45 | Fix deployed |
| 11:00 | Service restored |
| 11:15 | Incident closed |

## Root Cause
Detailed explanation of what caused the incident.

## Resolution
What actions were taken to resolve the incident.

## Preventive Measures
- Action 1 to prevent recurrence
- Action 2 to prevent recurrence
- Action 3 to detect earlier

## Lessons Learned
- Lesson 1
- Lesson 2

## Follow-Up Actions
| Action | Owner | Due Date | Status |
|--------|-------|----------|--------|
| Action 1 | Name 1 | YYYY-MM-DD | Open |
| Action 2 | Name 2 | YYYY-MM-DD | Open |

## Related Incidents
- INC-YYYYMMDD-NN: [Related incident]
```

---

## Using Templates

### How to Use
1. Copy the appropriate template from this directory
2. Rename with specific identifier (e.g., `fr-001-authentication.md`)
3. Fill in all sections marked with `[placeholders]`
4. Add to version control
5. Update related documents with cross-references

### Template Maintenance
- Templates are reviewed quarterly
- Propose template changes via pull request
- All templates follow markdown standards

---

## Related Categories
- `22-glossary` - Terminology for templates
- All other categories - Templates used throughout

---

*Source: Best practices for documentation and standardization*
