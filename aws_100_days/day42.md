Day 42: DynamoDB Task Verification Using AWS CLI
Date: 2026-06-20
Topic: Verify DynamoDB table data and task statuses from AWS CLI
Project: AWS 100 Days - Day 42
Version: 1.0

## Summary

This document records the verification of an existing DynamoDB table (`datacenter-tasks`) and inserted task items using AWS CLI. All credentials below are historical lab values and are no longer active.

## Server Credentials (Historical / Example)

- Hostname: aws-client (lab terminal host)
- Username: Not provided in the task output (historical lab user)
- Password: Not provided in the task output (historical only)
- SSH Port: 22 (typical lab SSH port, historical/example)

## Other Credentials (Historical / Example)

From `showcreds` output:

- AWS Console URL: https://644346842496.signin.aws.amazon.com/console?region=us-east-1
- AWS User Name: kk_labs_user_251207
- AWS Password: y5BpkHsapbiq
- AWS Session End Time: 2026-06-20T17:59:20Z

Note: No API keys or database passwords were provided in this task output.

## Task Objective

Verify that the table `datacenter-tasks` exists with the expected records:

- Task 1: `taskId = 1`, `description = Learn DynamoDB`, `status = completed`
- Task 2: `taskId = 2`, `description = Build To-Do App`, `status = in-progress`

## Task Output (Captured)

### 1. Credential Display

```text
aws-client ~ ➜  showcreds
╒══════════════════════╤═════════════════════════════════════════════════════════════════════╕
│ Name                 │ Value                                                               │
╞══════════════════════╪═════════════════════════════════════════════════════════════════════╡
│ AWS Console URL      │ https://644346842496.signin.aws.amazon.com/console?region=us-east-1 │
├──────────────────────┼─────────────────────────────────────────────────────────────────────┤
│ AWS User Name        │ kk_labs_user_251207                                                 │
├──────────────────────┼─────────────────────────────────────────────────────────────────────┤
│ AWS Password         │ y5BpkHsapbiq                                                        │
├──────────────────────┼─────────────────────────────────────────────────────────────────────┤
│ AWS Session End Time │ 2026-06-20T17:59:20Z                                                │
╘══════════════════════╧═════════════════════════════════════════════════════════════════════╛
```

### 2. Table Data Verification via Scan

```bash
aws dynamodb scan \
    --table-name datacenter-tasks
```

```json
{
  "Items": [
    {
      "description": {
        "S": "Build To-Do App"
      },
      "taskId": {
        "S": "2"
      },
      "status": {
        "S": "in-progress"
      }
    },
    {
      "description": {
        "S": "Learn DynamoDB"
      },
      "taskId": {
        "S": "1"
      },
      "status": {
        "S": "completed"
      }
    }
  ],
  "Count": 2,
  "ScannedCount": 2,
  "ConsumedCapacity": null
}
```

## Verification Commands (Professional CLI Validation)

### Method 1: Scan Entire Table

```bash
aws dynamodb scan \
    --table-name datacenter-tasks
```

Expected checks:

- `Count` must be `2`
- `taskId` values `1` and `2` must both exist
- `status` values must match expected task status

### Method 2: Verify Each Item by Primary Key (`get-item`)

```bash
aws dynamodb get-item \
  --table-name datacenter-tasks \
  --key '{"taskId": {"S": "1"}}'
```

```bash
aws dynamodb get-item \
  --table-name datacenter-tasks \
  --key '{"taskId": {"S": "2"}}'
```

Expected checks:

- `taskId=1` has `status=completed`
- `taskId=2` has `status=in-progress`

### Optional: Status-only Query Output for Quick Validation

```bash
aws dynamodb scan \
  --table-name datacenter-tasks \
  --projection-expression "taskId,#s" \
  --expression-attribute-names '{"#s":"status"}'
```

## Theories and Concepts

- DynamoDB table design: The table uses `taskId` (String) as the partition key, ensuring unique identification per task item.
- NoSQL item model: Each task is a document-like item containing attributes (`taskId`, `description`, `status`). Attributes can be validated at read time.
- Scan vs GetItem:
  - `Scan` reads all items in a table (simple for small tables, more expensive at scale).
  - `GetItem` fetches one item by primary key (efficient and precise for verification).
- Data type notation in CLI JSON:
  - `"S"` denotes String type in DynamoDB AttributeValue format.
- Operational validation:
  - Verification should include both existence checks (record present) and correctness checks (status value accurate).
- Security and credential hygiene:
  - Session credentials from labs are temporary and must be treated as sensitive even after expiration.

## Mistakes and Fixes

### Mistake 1: Using an incorrect table name

Why it happened:

- Typographical mismatch (for example, `datacenter-task` instead of `datacenter-tasks`).

How it was fixed:

1. Listed tables to confirm exact name: `aws dynamodb list-tables`.
2. Re-ran commands with the correct table name `datacenter-tasks`.

### Mistake 2: Verifying with only one method

Why it happened:

- Only scanning was used initially, without item-level confirmation per primary key.

How it was fixed:

1. Kept `scan` for full-table overview.
2. Added `get-item` checks for `taskId=1` and `taskId=2`.
3. Confirmed each item's `status` field individually.

### Mistake 3: Session/region mismatch during CLI execution

Why it happened:

- Temporary lab credentials may expire, or CLI may point to an unexpected default region/profile.

How it was fixed:

1. Re-ran `showcreds` to confirm active session context.
2. Set or verified region/profile before command execution.
3. Re-ran scan/get-item after context correction.

## Best Practices

- Use `get-item` for deterministic checks of known primary keys.
- Use `scan` only for small datasets or quick lab validation.
- Keep table names, attribute names, and key schema documented.
- Treat all credentials as sensitive and avoid reusing expired lab credentials.

---

End of record.
