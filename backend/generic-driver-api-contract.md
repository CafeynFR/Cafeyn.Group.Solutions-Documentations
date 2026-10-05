# CGS Authentication Client — External API Contract

This document describes the HTTP endpoints that an external system must expose to integrate with the **Cafeyn CGS** SDK using the `Generic` driver type.

The CGS SDK communicates with the external system using a `BaseUrl` configured per point-of-sale in the CGS database (`PointOfSaleDriver.BaseUrl`). All paths below are relative to that base URL.

---

## Authentication model

Every request made by CGS SDK carries the user's **external JWT token** in the `Authorization` header:

```
Authorization: Bearer <external-jwt-token>
Accept: application/json
```

The external API is responsible for validating this token on every call.

---

## Endpoints

### 1. `GET /cgs/authenticate`

**Purpose:** Validates an external token and returns both the canonical user identifier and the user's rights/entitlements in a single call.

**When it is called:** During the CGS authentication flow, when a user presents their external token to the CGS SDK. This is the entry point of the integration — it combines token introspection and rights resolution in one round trip.

**Prerequisites:**
- The `Authorization` header contains a valid, non-expired JWT issued by the external system.

**Expected response — `200 OK`:**

```json
{
  "identifier": "string",
  "rights": {}
}
```

| Field        | Type     | Description                                                                                  |
|--------------|----------|----------------------------------------------------------------------------------------------|
| `identifier` | string   | Stable, unique identifier for the user in the external system. Used by CGS to create or match the user account. |
| `rights`     | any JSON | The user's rights/entitlements. Any valid JSON value — object, array, or primitive. CGS stores and forwards this as-is. |

**Notes:**
- The token expiry is read from the JWT `exp` claim by CGS; the external API does not need to return it.

**Error handling:**
- Any non-2xx response causes CGS to reject the authentication attempt with an `Unauthorized` error.

---

### 2. `GET /cgs/profiles`

**Purpose:** Returns the list of reading profiles associated with the user's account.

**When it is called:** After successful authentication, when CGS builds account information or fetches profiles explicitly.

**Prerequisites:**
- The user has previously been authenticated via `GET /cgs/authenticate`.
- The `Authorization` header contains a valid external JWT for that user.

**Expected response — `200 OK`:**

A JSON array of profile objects:

```json
[
  {
    "externalId": "string",
    "ageGroup": "string",
    "concepts": ["string"],
    "locales": ["string"]
  }
]
```

| Field        | Type             | Description                                                        |
|--------------|------------------|--------------------------------------------------------------------|
| `externalId` | string           | Stable unique identifier for this profile in the external system.  |
| `ageGroup`   | string           | Age group for this profile (e.g. `"adult"`, `"child"`).           |
| `concepts`   | array of strings | Content concepts/topics this profile is interested in.             |
| `locales`    | array of strings | Preferred locales/languages (e.g. `["fr-FR", "en-US"]`).         |

An empty array `[]` is valid if the user has no profiles.

**Error handling:**
- Any non-2xx response causes CGS to return an `Unauthorized` error to the client.

---

## Call sequence

```
Client                     CGS                        External API
  |                          |                              |
  |-- POST /auth/token ------>|                              |
  |   { externalToken }      |-- GET /cgs/authenticate ---->|
  |                          |   Authorization: Bearer ...  |
  |                          |<-- 200 { identifier,         |
  |                          |         rights } ------------|
  |<-- CGS session token ----|                              |
  |                          |                              |
  |-- GET /account/info ----->|                              |
  |   Authorization: Bearer  |-- GET /cgs/profiles -------->|
  |   <CGS-token>            |<-- 200 [ ... ] -------------|
  |<-- account data +--------|                              |
  |    profiles              |                              |
```

---

## Configuration

The external API base URL is configured per **External Partner** record in the CGS database (`PointOfSaleDriver.BaseUrl` field).

```
BaseUrl: https://api.your-system.example.com
→ calls https://api.your-system.example.com/cgs/authenticate
→ calls https://api.your-system.example.com/cgs/profiles
```