# Python Lambda Style Guide

Coding conventions for AWS Lambda functions in Python at Magnitude Instruments.

## Runtime

- **Python 3.12** (match across all SAM projects)
- Only `boto3` is available in the Lambda runtime without a `requirements.txt`
- For additional dependencies, add a `requirements.txt` or use a Lambda layer

## Handler Structure

Every Lambda handler follows this layout:

```python
"""One-line module docstring describing what this Lambda does."""

import json
import logging
import os

import boto3

logger = logging.getLogger()
logger.setLevel(logging.INFO)

# Environment variables — read at module level
S3_BUCKET = os.environ["S3_BUCKET"]
SENDER_EMAIL = os.environ.get("SENDER_EMAIL", "")

# Lazy-initialized clients — reused across warm invocations
_s3 = None


def _get_s3():
    global _s3
    if _s3 is None:
        _s3 = boto3.client("s3", region_name="us-east-1")
    return _s3


def lambda_handler(event, context):
    # Handler logic here
    ...
```

### Why Lazy Init?

Lambda reuses the execution environment across invocations ("warm starts"). Initializing clients in a `_get_*()` helper means:
- The client is created once on the first invocation
- Subsequent invocations reuse the same client (faster)
- The handler function stays focused on business logic

### Handler Naming

Always `lambda_handler`. Configure in the SAM template as:

```yaml
Handler: app.lambda_handler
```

## Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Files | snake_case | `app.py`, `asana_client.py` |
| Functions | snake_case | `lambda_handler`, `_get_ses`, `_build_email_html` |
| Constants | UPPER_SNAKE_CASE | `S3_BUCKET`, `SENDER_EMAIL`, `CORS_HEADERS` |
| Variables | snake_case | `task_gid`, `ticket_number`, `all_contents` |
| Classes | PascalCase | `AsanaClient`, `AsanaApiError` |
| Private helpers | `_` prefix | `_get_s3()`, `_build_email_html()`, `_log_ticket_event()` |

## API Gateway Response Format

All Lambda functions behind API Gateway must return a dict with `statusCode`, `headers`, and `body`:

```python
CORS_HEADERS = {
    "Access-Control-Allow-Origin": "*",
    "Access-Control-Allow-Headers": "*",
}


def lambda_handler(event, context):
    # ... logic ...

    return {
        "statusCode": 200,
        "headers": CORS_HEADERS,
        "body": json.dumps(data),
    }
```

- Define `CORS_HEADERS` as a module-level constant
- Always include CORS headers — API Gateway only handles CORS for OPTIONS preflight; the Lambda response needs them too
- Use `json.dumps()` for JSON responses
- For plain text responses (e.g., presigned URLs), set `body` to the raw string without `json.dumps()`

## Path Parameters

API Gateway `{proxy+}` parameters arrive in `event["pathParameters"]["proxy"]`:

```python
key = event["pathParameters"]["proxy"]
```

URL-decode special characters as needed:

```python
key = key.replace("|", "/").replace("%7C", "/")
```

## Logging

Use the standard `logging` module, not `print()`:

```python
logger = logging.getLogger()
logger.setLevel(logging.INFO)

# In handler:
logger.info("Processing task %s", task_gid)
logger.warning("No email address — skipping")
logger.exception("Failed to send email for %s", ticket_number)  # includes traceback
```

- Use `logger.info()` for normal operations
- Use `logger.warning()` for recoverable issues (missing optional config, skipped steps)
- Use `logger.exception()` inside `except` blocks — it automatically includes the traceback
- Use `%s` formatting (not f-strings) in log calls for lazy evaluation

## Error Handling

- **Never let a Cognito trigger fail silently** — log the error and notify admin if possible, but always return the event object
- **API handlers** should return appropriate HTTP status codes
- **Catch specific exceptions** when you need to handle them; let unexpected errors propagate to CloudWatch

```python
# Cognito trigger pattern — must always return event
def lambda_handler(event, context):
    try:
        _do_work(event)
    except Exception:
        logger.exception("Failed to process trigger")
    return event


# API handler pattern — return error status codes
def lambda_handler(event, context):
    try:
        data = _fetch_data()
        return {"statusCode": 200, "headers": CORS_HEADERS, "body": json.dumps(data)}
    except Exception:
        logger.exception("Unexpected error")
        return {"statusCode": 500, "headers": CORS_HEADERS, "body": json.dumps({"error": "Internal server error"})}
```

## SES Email Pattern

### HTML Emails

Use inline CSS (email clients don't support `<style>` blocks reliably). Follow the branded template pattern:

```python
def _build_email_html(greeting: str, body_content: str) -> str:
    return f"""\
<html>
<body style="margin:0;padding:0;background:#f4f4f4;">
<table width="100%" cellpadding="0" cellspacing="0" style="background:#f4f4f4;padding:20px 0;">
  <tr><td align="center">
    <table width="600" cellpadding="0" cellspacing="0" style="background:#ffffff;border-radius:8px;overflow:hidden;">
      <tr>
        <td style="padding:24px 32px;background:#3D7EDB;text-align:center;">
          <!-- Logo or company name -->
        </td>
      </tr>
      <tr>
        <td style="padding:32px;font-family:Arial,sans-serif;font-size:15px;color:#333333;line-height:1.6;">
          <p style="margin:0 0 16px;">{greeting}</p>
          {body_content}
          <p style="margin:0;">Thank you,<br/>Magnitude Instruments</p>
        </td>
      </tr>
      <tr>
        <td style="padding:16px 32px;background:#f8f8f8;text-align:center;font-family:Arial,sans-serif;font-size:12px;color:#999999;">
          Magnitude Instruments &bull; <a href="https://www.magnitudeinstruments.com" style="color:#999999;">magnitudeinstruments.com</a>
        </td>
      </tr>
    </table>
  </td></tr>
</table>
</body>
</html>"""
```

### Sending

```python
_get_ses().send_email(
    Source=SENDER_EMAIL,
    Destination={"ToAddresses": [recipient]},
    Message={
        "Subject": {"Data": subject, "Charset": "UTF-8"},
        "Body": {"Html": {"Data": html, "Charset": "UTF-8"}},
    },
)
```

- Always set `Charset: "UTF-8"` on Subject and Body
- Use the `SENDER_EMAIL` environment variable (format: `"Display Name <address@domain.com>"`)
- Wrap in try/except and log failures — don't let a failed email crash the handler

## S3 Pagination

Always paginate when listing bucket contents — don't assume the response fits in one page:

```python
all_contents = []
continuation_token = None

while True:
    params = {"Bucket": S3_BUCKET}
    if continuation_token:
        params["ContinuationToken"] = continuation_token
    resp = _get_s3().list_objects_v2(**params)
    all_contents.extend(resp.get("Contents", []))
    if resp.get("IsTruncated"):
        continuation_token = resp["NextContinuationToken"]
    else:
        break
```

## Security

- **Never hardcode AWS credentials** — Lambda execution roles provide credentials automatically via the environment
- **Never hardcode secrets** — Use environment variables or AWS Secrets Manager
- **Scope IAM permissions** to specific resources (see SAM_CONVENTIONS.md)
- **Never log sensitive data** (passwords, tokens, PII beyond what's needed for debugging)

## Type Hints

Use type hints on function signatures for clarity:

```python
def _build_email_html(ticket_number: str, greeting: str, task_name: str = "") -> str:
    ...

def _next_sequence_number(year_month: str) -> int:
    ...
```

Not required on `lambda_handler` (the signature is always `(event, context)`).

## Dependencies

- **boto3** — always available in the Lambda runtime, no need to add to `requirements.txt`
- **requests** or other third-party packages — add to `requirements.txt` or a shared Lambda layer
- For shared code across multiple functions in the same project, use a Lambda layer defined in the SAM template
