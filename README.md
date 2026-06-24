# 📋 API Response Format Specification

> **Standard Response Format for JDB Card API Gateway**  
> **Version:** 1.0.0  
> **Last Updated:** 2026-06-23

---

## Table of Contents

1. [Overview](#overview)
2. [Success Response Format](#success-response-format)
3. [Error Response Format](#error-response-format)
4. [Response Code & Status Mapping](#response-code--status-mapping)
5. [API Response Examples](#api-response-examples)
6. [Error Code Reference](#error-code-reference)

---

## Overview

All API responses from the JDB Card API Gateway follow a unified response format to simplify partner integration.

### Response Format Principles

* ✅ **Consistent:** All APIs use the same response format.
* ✅ **Traceable:** Every response includes a `transactionId` for tracing and troubleshooting.
* ✅ **Timestamped:** Every response contains a timestamp indicating when it was generated.
* ✅ **Standardized:** Response fields such as `status`, `message`, and `errorCode` follow a standardized structure across all APIs.


---

## Success Response Format

### Structure

```json
{
  "timestamp": "2026-06-23T15:30:45.123+07:00",
  "success": true,
  "message": "SUCCESS",
  "transactionId": "550e8400-e29b-41d4-a716-446655440000",
  "data": {}
}
```

### Field Descriptions

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `timestamp` | string (ISO 8601) | ✅ | Response timestamp with timezone |
| `success` | boolean | ✅ | Always `true` for success responses |
| `message` | string | ✅ | Status message (`SUCCESS`, `CREATED`, etc.) |
| `transactionId` | string (UUID) | ✅ | Unique transaction identifier for tracking |
| `data` | object/array | ✅ | Response payload (varies by endpoint) |
| `status` | number | ❌ | HTTP status code (optional for clarity) |

### Timestamp Format

ISO 8601 format with timezone offset:
- Format: `YYYY-MM-DDTHH:mm:ss.SSSZ` with timezone
- Example: `2026-06-23T15:30:45.123+07:00`
- Python: `datetime.now().isoformat()`
- JavaScript: `new Date().toISOString()`

### Message Values

| Message | Meaning |
|---------|---------|
| `SUCCESS` | Request executed successfully |
| `CREATED` | Resource created successfully |
| `ACCEPTED` | Request accepted and processing |

---

## Error Response Format

### Structure

```json
{
  "timestamp": "2026-06-23T15:30:45.123+07:00",
  "success": false,
  "status": 400,
  "message": "VALIDATION_ERROR",
  "errorCode": "INVALID_CARD_NUMBER",
  "transactionId": "550e8400-e29b-41d4-a716-446655440000",
  "details": "Card number format is invalid"
}
```

### Field Descriptions

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `timestamp` | string (ISO 8601) | ✅ | Response timestamp with timezone |
| `success` | boolean | ✅ | Always `false` for error responses |
| `status` | number | ✅ | HTTP status code |
| `message` | string | ✅ | Error status message |
| `errorCode` | string | ✅ | Error code for programmatic handling |
| `transactionId` | string (UUID) | ❌ | Unique transaction identifier |
| `details` | string | ❌ | Additional error details |

### HTTP Status Codes

| Status | Code | Meaning |
|--------|------|---------|
| 200 | `OK` | Request successful |
| 201 | `CREATED` | Resource created |
| 400 | `BAD_REQUEST` | Invalid request format |
| 401 | `UNAUTHORIZED` | Authentication failed |
| 403 | `FORBIDDEN` | Access denied |
| 404 | `NOT_FOUND` | Resource not found |
| 409 | `CONFLICT` | Rate limit exceeded / Conflict |
| 422 | `UNPROCESSABLE_ENTITY` | Validation error |
| 500 | `INTERNAL_SERVER_ERROR` | Server error |
| 503 | `SERVICE_UNAVAILABLE` | Service down / External API error |

---

## Response Code & Status Mapping

### HTTP Status to Response Status

```
200 OK                      → message: "SUCCESS"
201 CREATED                 → message: "CREATED"
400 BAD_REQUEST             → message: "VALIDATION_ERROR"
401 UNAUTHORIZED            → message: "UNAUTHORIZED"
404 NOT_FOUND               → message: "NOT_FOUND"
409 CONFLICT                → message: "CONFLICT"
422 UNPROCESSABLE_ENTITY    → message: "VALIDATION_ERROR"
500 INTERNAL_SERVER_ERROR   → message: "ERROR"
503 SERVICE_UNAVAILABLE     → message: "SERVICE_UNAVAILABLE"
```

---

## API Response Examples

### 1. Authentication - Success

**Endpoint:** `POST /api/v1/authentication`

**Response (200 OK):**

```json
{
  "timestamp": "2026-06-23T15:30:45.123+07:00",
  "success": true,
  "status": 200,
  "message": "SUCCESS",
  "transactionId": "246543515643123",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c",
    "expiresIn": 86400,
    "tokenType": "Bearer"
  }
}
```

**Response (401 UNAUTHORIZED):**

```json
{
  "timestamp": "2026-06-23T15:30:46.456+07:00",
  "success": false,
  "status": 401,
  "message": "UNAUTHORIZED",
  "errorCode": "INVALID_CREDENTIALS",
  "transactionId": "246543515643124",
  "details": "Invalid userId or secretId"
}
```

---

### 2. Card Balance - Success

**Endpoint:** `POST /api/v1/cardBalance`

**Request:**

```json
{
  "requestId": "5461564515456123",
  "cardNo": "1234567890123456"
}
```

**Response (200 OK):**

```json
{
  "timestamp": "2026-06-23T15:30:47.789+07:00",
  "success": true,
  "message": "SUCCESS",
  "transactionId": "5461564515456123",
  "data": {
    "avl_Balance": 50000.00,
    "current_Balance": 100000.00,
    "cardStatus": "ACTIVE",
    "currency": "THB",
    "lastUpdated": "2026-06-23T15:30:47Z"
  }
}
```

**Response (404 NOT_FOUND):**

```json
{
  "timestamp": "2026-06-23T15:30:48.012+07:00",
  "success": false,
  "status": 404,
  "message": "NOT_FOUND",
  "errorCode": "CARD_NOT_FOUND",
  "transactionId": "5461564515456123",
  "details": "Card number not found in system"
}
```

**Response (503 SERVICE_UNAVAILABLE):**

```json
{
  "timestamp": "2026-06-23T15:30:49.345+07:00",
  "success": false,
  "status": 503,
  "message": "SERVICE_UNAVAILABLE",
  "errorCode": "IT_API_ERROR",
  "transactionId": "5461564515456123",
  "details": "IT API service is temporarily unavailable"
}
```

---

### 3. Card Transaction History - Success

**Endpoint:** `POST /api/v1/cardTransaction`

**Request:**

```json
{
  "requestId": "546515465154655",
  "cardNo": "1234567890123456",
  "fromDate": "2026-06-01",
  "toDate": "2026-06-23"
}
```

**Response (200 OK):**

```json
{
  "timestamp": "2026-06-23T15:30:50.678+07:00",
  "success": true,
  "message": "SUCCESS",
  "transactionId": "546515465154655",
  "data": [
    {
      "trnRefNo": "006YACIUSD000001",
      "txnDate": "2026-06-23 10:30:45",
      "cardNo": "1234567890123456",
      "accountNo": "00620010100001478",
      "srNo": 2742550284,
      "description": "TRANSFER PAYMENT",
      "trnCode": "TRF",
      "authstat": "A",
      "drAmount": 0,
      "crAmount": 5000.00,
      "endBal": 105000.00,
      "userId": "PARTNER001"
    },
    {
      "trnRefNo": "006YACIUSD000002",
      "txnDate": "2026-06-22 14:15:30",
      "cardNo": "1234567890123456",
      "accountNo": "00620010100001478",
      "srNo": 2742550285,
      "description": "WITHDRAWAL",
      "trnCode": "WTH",
      "authstat": "A",
      "drAmount": 2000.00,
      "crAmount": 0,
      "endBal": 100000.00,
      "userId": "PARTNER001"
    }
  ]
}
```

**Response (200 OK - Empty List):**

```json
{
  "timestamp": "2026-06-23T15:30:51.901+07:00",
  "success": true,
  "message": "SUCCESS",
  "transactionId": "546515465154656",
  "data": []
}
```

**Response (422 UNPROCESSABLE_ENTITY):**

```json
{
  "timestamp": "2026-06-23T15:30:52.234+07:00",
  "success": false,
  "status": 422,
  "message": "VALIDATION_ERROR",
  "errorCode": "INVALID_DATE_RANGE",
  "transactionId": "546515465154657",
  "details": "fromDate must be before toDate"
}
```

---

### 4. Rate Limit Exceeded

**Response (409 CONFLICT):**

```json
{
  "timestamp": "2026-06-23T15:30:53.567+07:00",
  "success": false,
  "status": 409,
  "message": "CONFLICT",
  "errorCode": "RATE_LIMIT_EXCEEDED",
  "details": "Too many requests. Maximum 100 requests per 15 minutes"
}
```

---

### 5. Invalid HMAC Signature

**Response (401 UNAUTHORIZED):**

```json
{
  "timestamp": "2026-06-23T15:30:54.890+07:00",
  "success": false,
  "status": 401,
  "message": "UNAUTHORIZED",
  "errorCode": "INVALID_SIGNATURE",
  "transactionId": "5461564515456124",
  "details": "Request signature verification failed"
}
```

---

### 6. Invalid JSON Request

**Response (400 BAD_REQUEST):**

```json
{
  "timestamp": "2026-06-23T15:30:55.123+07:00",
  "success": false,
  "status": 400,
  "message": "VALIDATION_ERROR",
  "errorCode": "INVALID_REQUEST",
  "details": "Invalid JSON format in request body"
}
```

---

### 7. Missing Required Field

**Response (400 BAD_REQUEST):**

```json
{
  "timestamp": "2026-06-23T15:30:56.456+07:00",
  "success": false,
  "status": 400,
  "message": "VALIDATION_ERROR",
  "errorCode": "VALIDATION_ERROR",
  "details": "Missing required field: cardNo"
}
```

---

### 8. Internal Server Error

**Response (500 INTERNAL_SERVER_ERROR):**

```json
{
  "timestamp": "2026-06-23T15:30:57.789+07:00",
  "success": false,
  "status": 500,
  "message": "ERROR",
  "errorCode": "SYSTEM_ERROR",
  "details": "Internal Server Error"
}
```

---

## Error Code Reference

### Authentication Errors

| ErrorCode | HTTP Status | Message | Description |
|-----------|------------|---------|-------------|
| `INVALID_CREDENTIALS` | 401 | UNAUTHORIZED | Invalid userId or secretId |
| `INVALID_SIGNATURE` | 401 | UNAUTHORIZED | HMAC signature verification failed |
| `INVALID_TOKEN` | 401 | UNAUTHORIZED | JWT token invalid or expired |

### Card/Account Errors

| ErrorCode | HTTP Status | Message | Description |
|-----------|------------|---------|-------------|
| `CARD_NOT_FOUND` | 404 | NOT_FOUND | Card not found in system |
| `CLIENT_NOT_FOUND` | 404 | NOT_FOUND | Client/Customer not found |
| `ACCOUNT_NOT_FOUND` | 404 | NOT_FOUND | Bank account not found |
| `PARTNER_NOT_FOUND` | 404 | NOT_FOUND | Partner not found |

### Validation Errors

| ErrorCode | HTTP Status | Message | Description |
|-----------|------------|---------|-------------|
| `VALIDATION_ERROR` | 422 | VALIDATION_ERROR | General validation error |
| `INVALID_REQUEST` | 400 | VALIDATION_ERROR | Invalid request format |
| `INVALID_CARD_NUMBER` | 422 | VALIDATION_ERROR | Invalid card number format |
| `INVALID_DATE_RANGE` | 422 | VALIDATION_ERROR | Invalid date range (fromDate > toDate) |

### Service Errors

| ErrorCode | HTTP Status | Message | Description |
|-----------|------------|---------|-------------|
| `CMS_API_ERROR` | 503 | SERVICE_UNAVAILABLE | CMS API error |
| `IT_API_ERROR` | 503 | SERVICE_UNAVAILABLE | IT API error |
| `ORACLE_ERROR` | 503 | SERVICE_UNAVAILABLE | Oracle database error |
| `DATABASE_ERROR` | 503 | SERVICE_UNAVAILABLE | MySQL database error |
| `SERVICE_UNAVAILABLE` | 503 | SERVICE_UNAVAILABLE | External service down |

### System Errors

| ErrorCode | HTTP Status | Message | Description |
|-----------|------------|---------|-------------|
| `SYSTEM_ERROR` | 500 | ERROR | Internal server error |
| `RATE_LIMIT_EXCEEDED` | 409 | CONFLICT | Rate limit exceeded |

---

## Implementation Guide for Partners

### Parsing Response

```javascript
// Example: Parse authentication response
const response = await fetch('https://api.jdb.com/api/v1/authentication', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'SignedHash': signedHash
  },
  body: JSON.stringify({
    requestId: '...',
    userId: '...',
    secretId: '...'
  })
});

const json = await response.json();

if (json.success) {
  // Handle success
  console.log(`Transaction: ${json.transactionId}`);
  console.log(`Token: ${json.data.token}`);
  console.log(`Expires in: ${json.data.expiresIn} seconds`);
} else {
  // Handle error
  console.error(`Error: ${json.message}`);
  console.error(`Code: ${json.errorCode}`);
  console.error(`Details: ${json.details}`);
  console.log(`Transaction ID for support: ${json.transactionId}`);
}
```

### Logging Response for Debugging

```python
import json
from datetime import datetime

def log_response(response_json):
    """Log API response for debugging"""
    timestamp = response_json.get('timestamp')
    transaction_id = response_json.get('transactionId')
    success = response_json.get('success')
    message = response_json.get('message')
    error_code = response_json.get('errorCode')
    
    status = "✓ SUCCESS" if success else "✗ FAILED"
    print(f"{status} | Transaction: {transaction_id}")
    print(f"  Timestamp: {timestamp}")
    print(f"  Message: {message}")
    if error_code:
        print(f"  Error Code: {error_code}")
    print(f"  Response: {json.dumps(response_json, indent=2)}")
```

---

## Best Practices

### Request Tracking

Always use `transactionId` from response for customer support:

```
"Customer reports issue"
→ Get transactionId from response
→ Search logs with transactionId
→ Trace complete flow from request to response
```

### Error Handling

```javascript
// Categorize errors by status code
const handleApiError = (response) => {
  switch (response.status) {
    case 400:
    case 422:
      // Validation error - check your request format
      return "Please check your request format";
    
    case 401:
      // Authentication error - get new token
      return "Please authenticate again";
    
    case 404:
      // Not found error - verify your data
      return "Card/Account not found";
    
    case 409:
      // Rate limit - wait and retry
      return "Too many requests, please wait before retrying";
    
    case 503:
      // Service unavailable - retry with backoff
      return "Service temporarily unavailable, please retry later";
    
    case 500:
      // Server error - contact support
      return "Server error, please contact support with transaction ID";
    
    default:
      return "Unknown error occurred";
  }
};
```

### Retry Strategy

```javascript
const retryWithBackoff = async (request, maxRetries = 3) => {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const response = await request();
      if (response.success) return response;
      
      // Don't retry validation errors
      if (response.status === 400 || response.status === 422) {
        throw new Error(`Validation error: ${response.message}`);
      }
      
      // Retry on service errors
      if (response.status >= 500) {
        const backoffMs = Math.pow(2, attempt) * 1000; // Exponential backoff
        await sleep(backoffMs);
        continue;
      }
    } catch (error) {
      if (attempt === maxRetries) throw error;
    }
  }
};
```

---

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-06-23 | Initial standardized response format specification |

---

**Document Version:** 1.0.0  
**Last Modified:** 2026-06-23  
**Maintained By:** BOPBY KEODOAUNGDY  
**Tel:** 20295873197

