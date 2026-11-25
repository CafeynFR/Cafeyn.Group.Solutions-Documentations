# CGS Authentication Client — External API Contract

This document describes the HTTP endpoints that an external system must expose to integrate with the **Cafeyn CGS** using the `Generic` driver type.

The CGS calls your API under a base URL that you provide to Cafeyn. All paths below are relative to that base URL (see [Configuration](#configuration)).

---

## Authentication model

Every request made by CGS carries the user's **external JWT token** in the `Authorization` header:

```
Authorization: Bearer <external-jwt-token>
Accept: application/json
```

The external API is responsible for validating this token on every call.

---

## Endpoints

### 1. `GET /cgs/authenticate`

**Purpose:** Validates an external token and returns both the canonical user identifier and the user's rights/entitlements in a single call.

**When it is called:** When a user signs in to a Cafeyn application with their external token. This is the entry point of the integration — it combines token introspection and rights resolution in one round trip.

**Prerequisites:**
- The `Authorization` header contains a valid, non-expired JWT issued by the external system.

**Expected response — `200 OK`:**

```json
{
  "identifier": "string",
  "rights": {
    "articleIds": [],
    "issueIds": [],
    "publicationIds": [],
    "podcastIds": [],
    "videoIds": [],
    "bundleIds": [3, 5],
    "issueFilters": [
      {
        "publication_id_0001": {
          "dates": [
            "2026-05-21",
            "2026-07-12"
          ],
          "dateRanges": [
            {
              "start": "2025-01-01",
              "end": "2025-06-01"
            },
            {
              "start": "2026-09-01",
              "end": "2027-02-01"
            },
            {
              "start": "2025-01-01"
            }
          ]
        }
      },
      {
        "publication_id_0002": {
          "dateRanges": [
            {
              "start": "2026-10-01"
            }
          ]
        }
      },
      {
        "publication_id_0003":
        {
          "numbers":[1, 2, 8],
          "numberRanges": [
            {
              "start": 10,
              "end": 20
            },
            {
              "start": 30
            }
          ]
        }
      }
    ],
    "capabilities": {
      "Alerts": true
    }
  }
}
```

| Field        | Type        | Required         | Description                                                                                  |
|--------------|-------------|------------------|----------------------------------------------------------------------------------------------|
| `identifier` | string      | **Mandatory**    | Stable, unique identifier for the user in the external system. Used by CGS to create or match the user account. |
| `rights`     | JSON object | **Optional**     | The user's rights/entitlements. Can be omitted when Cafeyn has set default rights for your integration (see the notes below). |

> **`identifier` is mandatory; `rights` is optional.** A response without `identifier` is not a valid authentication. `rights` can be left out, but only safely when Cafeyn has set default rights for your integration: without them, omitting it may result in a `400` error.

**Typical `rights` content:**

| Field            | Type                         | Description                                                          |
|------------------|------------------------------|----------------------------------------------------------------------|
| `articleIds`     | array of strings             | Articles the user can read.                                          |
| `issueIds`       | array of strings             | Issues the user can read.                                            |
| `publicationIds` | array of strings             | Publications the user can read entirely (all their issues).          |
| `podcastIds`     | array of strings             | Podcasts the user can access.                                        |
| `videoIds`       | array of strings             | Videos the user can access.                                          |
| `bundleIds`      | array of integers            | Bundles the user has access to.                                      |
| `issueFilters`   | array of objects             | Access to part of a publication's issues, by date or number (see below). |
| `capabilities`   | object of booleans           | Features enabled for the user, keyed by feature name (e.g. `Alerts`). |

#### `issueFilters`

Each object in `issueFilters` has a single key, the publication id, whose value holds the issues of that publication the user can read. Each of `dates`, `dateRanges`, `numbers` and `numberRanges` can hold several entries, and a publication can use any of them together:

| Field          | Type                                | Description                                                       |
|----------------|-------------------------------------|-------------------------------------------------------------------|
| `dates`        | array of dates                      | Issues whose release date for the point of sale is exactly one of these dates (`yyyy-MM-dd`). |
| `dateRanges`   | array of `{ start, end }` (dates)   | Issues whose release date for the point of sale is within any of these date ranges (`yyyy-MM-dd`). |
| `numbers`      | array of integers                   | Issues whose number is exactly one of these.                      |
| `numberRanges` | array of `{ start, end }` (integers) | Issues whose number is within any of these ranges.                |

In a range, only `start` is required. `end` can be left out or set to `null`: the range then has no end. Both bounds are inclusive: an issue dated on `start` or `end`, or numbered `start` or `end`, is part of the range.

`issueFilters` only refers to publications and issues **not already referenced** in `publicationIds` or `issueIds`: a publication readable entirely belongs in `publicationIds`, a single issue in `issueIds`, and neither is repeated in `issueFilters`.

The filters in the example above:

| Filter                                                                                   | Access                                                               |
|------------------------------------------------------------------------------------------|----------------------------------------------------------------------|
| `publication_id_0001`, `dates: ["2026-05-21", "2026-07-12"]`                             | Issues of the publication published on 2026-05-21 or 2026-07-12.     |
| `publication_id_0001`, `{ "start": "2025-01-01", "end": "2025-06-01" }`                  | Issues of the publication published from 2025-01-01 to 2025-06-01.   |
| `publication_id_0001`, `{ "start": "2026-09-01", "end": "2027-02-01" }`                  | Issues of the publication published from 2026-09-01 to 2027-02-01.   |
| `publication_id_0001`, `{ "start": "2025-01-01" }`                                       | Issues of the publication published from 2025-01-01 on, with no end. |
| `publication_id_0002`, `{ "start": "2026-10-01" }`                                       | Issues of the publication published from 2026-10-01 on, with no end. |
| `publication_id_0003`, `numbers: [1, 2, 8]`                                              | Issues of the publication numbered 1, 2 or 8.                        |
| `publication_id_0003`, `{ "start": 10, "end": 20 }`                                  | Issues of the publication numbered 10 to 20.                         |
| `publication_id_0003`, `{ "start": 30 }`                                               | Issues of the publication from number 30 on, with no upper bound.    |

**Notes:**
- `rights` must be a JSON **object**, written as JSON. A string containing JSON (e.g. `"rights": "{\"issueIds\":[\"1245\"]}"`), an array or a primitive is not accepted, and the user is then treated as having no rights.
- The response must be strict JSON: no comments, no trailing commas. A response that has either cannot be read, and the authentication fails.
- The token expiry is read from the JWT `exp` claim; the external API does not need to return it.
- Cafeyn can set default rights for your integration. When it does, they replace the returned `rights` for every user, so `rights` does not need to be returned; the call is still made, for the identifier. When no default rights are set, `rights` must be returned: omitting it may result in a `400` error.

**Error handling:**
- Any non-2xx response causes CGS to reject the authentication attempt with an `Unauthorized` error.

---

### 2. `GET /cgs/profiles`

**Purpose:** Returns the list of reading profiles associated with the user's account.

> **Implementing this endpoint is not mandatory.** Cafeyn can set a default profile for your integration; when it does, that profile is used instead of calling this endpoint. Implement this endpoint only if no default profile is set.

**When it is called:** Right after `GET /cgs/authenticate`, each time a user signs in or their profiles are requested.

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
| `ageGroup`   | string           | Age group for this profile (e.g. `"adult"`, `"child"`).            |
| `concepts`   | array of strings | Content concepts/topics this profile is interested in.             |
| `locales`    | array of strings | Preferred locales/languages (e.g. `["fr-FR", "en-US"]`).           |

An empty array `[]` is valid if the user has no profiles.

**Error handling:**
- Any non-2xx response causes CGS to return an `Unauthorized` error to the client.

---

## Call sequence

```
Cafeyn app                      CGS                        External API
  |                               |                              |
  |-- sign in with the            |                              |
  |   external token ------------>|                              |
  |                               |-- GET /cgs/authenticate ---->|
  |                               |   Authorization: Bearer ...  |
  |                               |<-- 200 { identifier,         |
  |                               |         rights } ------------|
  |                               |-- GET /cgs/profiles -------->|
  |                               |   Authorization: Bearer ...  |
  |                               |<-- 200 [ ... ] --------------|
  |<-- session + rights           |                              |
  |    + profiles ----------------|                              |
```

When Cafeyn has set a default profile for your integration, the `GET /cgs/profiles` call is skipped.

---

## Configuration

To set up the integration, provide Cafeyn with:

| Setting                      | Description                                                                                  |
|------------------------------|----------------------------------------------------------------------------------------------|
| Base URL                     | Base URL of your API. Required.                                                              |
| Default rights (optional)    | Rights applied to every user instead of those returned by `GET /cgs/authenticate`.           |
| Default profile (optional)   | Profile used for every user instead of calling `GET /cgs/profiles`.                          |

```
Base URL: https://api.your-system.example.com
→ calls https://api.your-system.example.com/cgs/authenticate
→ calls https://api.your-system.example.com/cgs/profiles
```

A base URL that has a path must end with `/`; otherwise its last segment is dropped:

```
Base URL: https://partner.example.com/api/
→ calls https://partner.example.com/api/cgs/authenticate
→ calls https://partner.example.com/api/cgs/profiles

Base URL: https://partner.example.com/api
→ calls https://partner.example.com/cgs/authenticate
→ calls https://partner.example.com/cgs/profiles
```
