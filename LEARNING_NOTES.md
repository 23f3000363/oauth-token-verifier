# Learning Notes — OAuth 2.0 / OIDC Token Verification Service

## 1. What kind of problem is this?

This is a backend API development and authentication problem.

The task is to build a FastAPI endpoint that receives a JSON Web Token (JWT), verifies it using an assigned RSA public key, validates its claims, and returns a structured result.

The endpoint is:

`POST /verify`

The grader generates fresh tokens for each verification call, so the implementation must verify incoming tokens dynamically.

## 2. Requirements from the assignment

### Assigned configuration

- Issuer (`iss`): `https://idp.exam.local`
- Audience (`aud`): `tds-tc44wfb.apps.exam.local`
- Algorithm: RS256
- Public key: supplied by the assignment

### Request format

The request body contains a token:

`{"token": "<JWT string>"}`

### Valid token

Return HTTP 200 with a JSON response containing:

- `valid`: `true`
- `email`: the token's email claim
- `sub`: the token's subject claim
- `aud`: the token's audience claim

The claims must come from the verified token.

### Invalid token

Reject the token with a non-200 status, preferably HTTP 401, and return:

`{"valid": false}`

## 3. What must be verified?

Reject a token if any of these checks fail:

1. **Signature:** Verify the RS256 signature using the assigned public key.
2. **Issuer:** The `iss` claim must equal `https://idp.exam.local`.
3. **Audience:** The `aud` claim must match `tds-tc44wfb.apps.exam.local`.
4. **Expiry:** The `exp` claim must be in the future.

Never trust claims extracted from an unverified token.

## 4. Concepts to learn

### HTTP and FastAPI

- GET versus POST
- JSON request bodies
- HTTP status codes
- Request validation
- Returning JSON responses

### JSON Web Tokens

A JWT commonly contains three Base64URL-encoded sections:

`header.payload.signature`

- Header: contains metadata such as the signing algorithm.
- Payload: contains claims such as `iss`, `aud`, `sub`, `email`, and `exp`.
- Signature: allows the verifier to detect unauthorized changes.

Encoding is not encryption. A JWT payload should not be treated as secret merely because it is encoded.

### RSA and RS256

RS256 means RSA signature verification using SHA-256.

The issuer signs the token with its private key. The API verifies that signature with the corresponding public key.

The API must never use a public key to create or forge valid tokens.

### Important claims

- `iss`: identifies the token issuer.
- `aud`: identifies the intended recipient.
- `exp`: specifies the expiration time.
- `sub`: identifies the subject.
- `email`: an application-specific claim in this assignment.

## 5. Implementation approach

1. Create a FastAPI application.
2. Define a POST `/verify` endpoint.
3. Accept the token from the JSON request body.
4. Verify its signature using the supplied RSA public key.
5. Restrict verification to RS256.
6. Validate issuer, audience, and expiration.
7. Return the required fields for valid tokens.
8. Reject invalid tokens with HTTP 401 and `valid: false`.
9. Test valid, expired, wrong-audience, and tampered tokens.
10. Deploy the application to a public HTTPS host.
11. Submit the full deployed `/verify` URL to the grader.

## 6. Testing checklist

- [ ] Valid token returns HTTP 200.
- [ ] Valid token returns `valid: true`.
- [ ] Returned claims match the verified token.
- [ ] Expired token is rejected.
- [ ] Wrong-audience token is rejected.
- [ ] Tampered token with an invalid signature is rejected.
- [ ] Missing or malformed token is rejected safely.
- [ ] Public endpoint works after deployment.
- [ ] Grader receives the complete `/verify` URL.

## 7. Prerequisites for similar assignments

Learn these in order:

1. Python functions, dictionaries, and exceptions.
2. HTTP methods, JSON, request bodies, and status codes.
3. FastAPI routes and request validation.
4. JWT structure and standard claims.
5. RSA public/private keys and digital signatures.
6. Authentication libraries and secure verification settings.
7. API testing with curl.
8. GitHub and public deployment.

## 8. General lesson

Authentication is not the same as decoding a token.

A secure verifier must authenticate the token's origin through signature verification and check that its claims satisfy the application's requirements. Merely reading the payload is insufficient.

For similar graded backend tasks, separate the problem into configuration, endpoint implementation, security validation, testing, and deployment. Verify each stage before moving to the next.