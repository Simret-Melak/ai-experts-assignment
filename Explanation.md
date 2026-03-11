# Explanation

## What was the bug?

The bug was in `Client.request(..., api=True)`. When `self.oauth2_token` was a plain dictionary like `{"access_token": "stale", "expires_at": 0}`, the client did not refresh it and did not attach an `Authorization` header to the request.

## Why did it happen?

The refresh logic only handled two cases:

1. when the token was missing (`None`)
2. when the token was an expired `OAuth2Token`

A dictionary token matched neither case, so `refresh_oauth2()` was skipped. After that, the code only added the `Authorization` header if the token was an `OAuth2Token`, so the request was sent without authentication.

## Why does your fix actually solve it?

The fix changes the logic so that any token that is not an `OAuth2Token` is treated as invalid and refreshed before the request is prepared. After that, the code always adds the `Authorization` header from the refreshed or already-valid token.

This solves the failing case because:

- `None` tokens are refreshed
- dictionary tokens are refreshed
- expired `OAuth2Token` values are refreshed
- valid `OAuth2Token` values are reused without refresh

So API requests always end up with a valid bearer token in the header.

## What’s one realistic case / edge case your tests still don’t cover?

The tests do not cover the situation where a token is about to expire very soon (for example within a few seconds). In real systems, clients often refresh tokens slightly before expiration to avoid making a request with a token that expires during the request.