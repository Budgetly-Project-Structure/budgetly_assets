# Postman Demo: One User Can't Act as Another (SCRUM-115)

This demo answers the question "could someone use Postman to read or change another user's data?" with real requests. User A signs in, then calls the Firestore REST API directly (no web app in between) and tries to read, create, change and delete user B's transactions. Every attempt is refused with **403**, while the same requests on A's own data succeed with **200**.

Why it works is explained in [data-framework.md → Trust model](data-framework.md#trust-model): the uid always comes from the signed login token, and the Firestore rules (SCRUM-113) compare it with each transaction's `userId`.

Files:

* Collection: [`postman/budgetly-impersonation.postman_collection.json`](postman/budgetly-impersonation.postman_collection.json)
* This guide

The collection contains **no tokens, passwords or keys**. You put those in a Postman environment on your own machine, and the sign-in requests fill in the tokens while it runs. Don't export or commit that environment.

## What it sends

| # | Request (as user A unless noted) | Expected |
| --- | --- | --- |
| 0 | Sign in as A, sign in as B, B creates one of B's own transactions | 200 |
| 1 | A lists B's transactions (`runQuery` where `userId == B`) | **403** |
| 1 | A reads B's transaction by its document ID | **403** |
| 1 | A creates a transaction with `userId = B` (impersonation) | **403** |
| 1 | A changes B's transaction (`PATCH category`) | **403** |
| 1 | A deletes B's transaction | **403** |
| 2 | A lists, creates and re-categorizes A's own transactions | 200 |
| 3 | No login at all: list B's transactions | **403** |
| 4 | `deleteImport` with no login and `userId = B` in the body | `UNAUTHENTICATED` |
| 4 | `deleteImport` as A with `userId = B` in the body | `NOT_FOUND` (only A's own statements are searched) |

Folder 4 uses a made-up import ID on purpose, so nothing can be deleted.

## Step by step

### 1. Make two throwaway test accounts

Use test accounts only, never real ones.

1. Firebase console → **budgetly-backend-nodejs** → **Authentication** → **Users** → **Add user**.
2. Add `postman-a@example.com` and `postman-b@example.com`, each with its own throwaway password.

### 2. Find the Web API key

Firebase console → **Project settings** (gear icon) → **General** → **Web API key**. (It's the same value as `VITE_FIREBASE_API_KEY` in budgetly_web_customer's `.env`.) This key only identifies the project. It is not a password, but keep it out of the repo anyway.

### 3. Set up Postman

1. **Import** → choose `docs/postman/budgetly-impersonation.postman_collection.json`.
2. **Environments** → **+** → name it `Budgetly demo (local only)` and add:

   | Variable | Value |
   | --- | --- |
   | `projectId` | `budgetly-backend-nodejs` |
   | `apiKey` | the Web API key from step 2 |
   | `emailA` / `passwordA` | test account A |
   | `emailB` / `passwordB` | test account B |

   Leave `tokenA`, `uidA`, `tokenB`, `uidB`, `txnA` and `txnB` out: the sign-in requests set them.
3. Select that environment (top right).

### 4. Run it

Right-click the collection → **Run collection** → **Run**. Run the folders in order (0 first), because later requests use the tokens and IDs that earlier ones save.

Every test should be green: 403 for everything in folders 1 and 3, 200 in folders 0 and 2, and the expected errors in folder 4.

To show it live, open **A lists B's transactions** and click **Send**. The response is:

```json
{ "error": { "code": 403, "message": "Missing or insufficient permissions.", "status": "PERMISSION_DENIED" } }
```

Then open **A lists A's own transactions** and send it: 200, with A's data.

### 5. Clean up

* Each run leaves two documents named "Postman demo (A)" and "Postman demo (B)" in `transactions`. The app can't hard delete them (by design), so delete them in the Firebase console under **Firestore → Data → transactions**, or delete the two test accounts' data there.
* ID tokens expire after an hour. If requests suddenly fail with 401, run the two sign-in requests again.
* Delete the environment (or at least its passwords) when you're done.

## Running it against the emulators

The collection also works against the local emulators with no real accounts. Set these in the environment:

| Variable | Value |
| --- | --- |
| `projectId` | `demo-budgetly` |
| `apiKey` | anything, e.g. `fake-api-key` |
| `firestoreBase` | `http://127.0.0.1:8080/v1` |
| `authBase` | `http://127.0.0.1:9099/identitytoolkit.googleapis.com/v1` |

Start `firebase emulators:start --only firestore,auth --project demo-budgetly` in budgetly_backend_nodejs, add the two users in the Emulator UI (Authentication tab), and run folders 0 to 3. Folder 4 needs the deployed functions.

Result of that run (Nov 1, 2026), with the rules from budgetly_backend_nodejs: 12 of 12 checks passed. Folders 1 and 3 all returned 403, and folders 0 and 2 all returned 200.
