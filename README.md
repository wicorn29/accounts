# Wicorn29 Accounts Public API Documentation

Base URL: https://served.wicorn29.net

---

## Overview

This API allows 3rd party app developers to integrate user authentication, app data storage, and storage quota checking. It uses token-based authentication.

All requests that require authentication expect an Authorization header with the format:

Authorization: Bearer <token>

---

## Endpoints

### 1. Generate Token

**POST** /gentoken

Authenticate user credentials and receive a token.

Request Body:

{
  "email": "user@example.com",
  "password": "userpassword"
}

Response:

- Success (200):

{
  "token": "<generated_token>"
}

- Errors: 400 missing fields, 403 invalid password, 404 user not found.

---

### 2. Get Remaining Storage Quota

**GET** /storageleft

Returns the remaining available storage quota for the authenticated user.

Headers:

- Authorization: Bearer <token>

Response:

- Success (200):

{
  "storageLeftBytes": 12345678
}

- Errors: 401 missing token, 403 invalid token, 404 user not found.

---

### 3. Save or Update App Data

**PUT** /appdata

Store or update app-specific data for the authenticated user.

- The calling app's origin must be included in the request headers as Origin or Referer.
- Data is saved scoped to the root domain extracted from the calling app's origin (e.g., "sub.example.com" → "example.com").

Headers:

- Authorization: Bearer <token>
- Origin: https://yourapp.example.com (or Referer)

Request Body:

{
  "data": { "any": "json-serializable data" }
}

Response:

- Success (200):

{
  "success": true
}

- Errors:
  - 400 invalid data or missing origin header
  - 403 origin/domain mismatch or unauthorized origin
  - 404 user not found
  - 507 user storage quota exceeded

---

### 4. Retrieve App Data

**GET** /appdata

Fetch stored app data for the authenticated user based on the calling app's origin.

Headers:

- Authorization: Bearer <token>
- Origin: https://yourapp.example.com (or Referer)

Response:

- Success (200):

{
  "data": { "your": "stored app data" }
}

- Errors:
  - 400 invalid origin header
  - 403 origin/domain mismatch
  - 404 no app data found or user not found

---

### 5. Delete App Data

**DELETE** /appdata

Delete stored app data for the calling app's root domain.

Headers:

- Authorization: Bearer <token>
- Origin: https://yourapp.example.com (or Referer)

Response:

- Success (200):

{
  "success": true
}

- Errors:
  - 400 invalid origin header
  - 404 no app data found or user not found

---

## Notes for Developers

- All authenticated endpoints require a valid bearer token obtained from /gentoken.
- The root domain is used to scope app data, so subdomains share the same data space.
- User storage quota is currently 128MB total across all apps.
- Requests must include an Origin or Referer header to identify the calling domain.
- Storage quota should be checked regularly using /storageleft to warn users before hitting limits.

---

## Example Workflow

1. User logs in through your app backend, sends credentials to /gentoken to get a token.
2. Your app stores the token securely and uses it for subsequent calls.
3. When saving user data, call /appdata with PUT and pass data JSON.
4. When loading data, call /appdata with GET.
5. Check /storageleft periodically to monitor quota.

---

## Contact & Support

For questions or issues, please contact the Wicorn29 developer support team.

---

End of Public API Documentation
