# Topic 7 -- API Integration Patterns & Authentication Methods

## Test Cases and Execution Results

**Test environment:** n8n at `https://n8n.tayanasolutions.com`\
**Execution date:** 20 November 2026\
**Test scope:** API Key, Bearer Token, mock token refresh
success/failure, workflow-export security

## 1. Test Summary

  -----------------------------------------------------------------------------
  Test ID           Test              Result            Evidence /
                                                        qualification
  ----------------- ----------------- ----------------- -----------------------
  TC-001            API Key in HTTP   PARTIAL -- header Latest echo output used
                    Header            value echoed;     `header_apikey_test`,
                                      latest header     not the required
                                      name mismatch     `X-API-Key`.

  TC-002            Bearer Token Auth PASS              Echo output contained
                                                        the exact Authorization
                                                        Bearer value.

  TC-003            Expired OAuth2    PASS -- mock      Fresh mock token saved
                    Token Refresh     simulation        and protected call
                                                        returned authorized.

  TC-004            Failed Token      PASS -- mock      HTTP 401 /
                    Refresh Alert     simulation        invalid-grant path
                                                        produced structured
                                                        failure alert.

  TC-005            Security Export   PASS with naming  No tested API key or
                    Validation        note              Bearer token appeared
                                                        in plaintext;
                                                        credential references
                                                        remained.
  -----------------------------------------------------------------------------

## 2. Detailed Test Cases

### TC-001 -- API Key in HTTP Header

**Requirement:** FR-001, FR-002\
**Precondition:** Header Auth credential configured and HTTP Request
node targets Postman Echo.

**Steps** 1. Execute `HTTP_APIKeyRequest`. 2. Inspect the returned
`headers` object. 3. Compare the actual header name and value with the
expected `X-API-Key: secretKey123`.

**Expected** - Request succeeds. - Echo output includes `x-api-key` with
the configured value.

**Observed** The latest supplied execution output included
`header_apikey_test` with the configured value. An earlier execution was
reported as showing `x-api-key`, but the latest evidence does not
establish that the final exported configuration still uses the required
header name.

**Result: PARTIAL -- correct transmission of a header value
demonstrated; exact header-name compliance requires correction/retest.**

### TC-002 -- Bearer Token Authentication

**Requirement:** FR-001, FR-003\
**Precondition:** Bearer Auth credential configured on
`HTTP_BearerRequest`.

**Steps** 1. Execute `HTTP_BearerRequest`. 2. Inspect
`headers.authorization`.

**Expected** `Authorization: Bearer <token>` is present in the request.

**Observed** The echo output contained:
`Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.token123`

**Result: PASS.**

### TC-003 -- Expired OAuth2 Token Refresh

**Requirement:** FR-004, FR-005\
**Precondition:** Token cache is empty/expired and mock token endpoint
returns HTTP 200.

**Steps** 1. Execute the Bearer token lifecycle branch. 2. Confirm
expiry logic routes to `HTTP_RefreshAuthToken`. 3. Confirm the token
response is passed to `Code_SaveFreshToken`. 4. Confirm the protected
request executes with the fresh Bearer token.

**Expected** A new access token is obtained and saved; protected API
call succeeds.

**Observed** `Code_SaveFreshToken` produced `FRESH_TOKEN_SAVED`; the
subsequent protected request returned `AUTHORIZED`; audit output
reported `authenticationStatus: SUCCESS`.

**Result: PASS -- mock refresh simulation. Not a live OAuth2 provider
test.**

### TC-004 -- Failed Token Refresh Alert

**Requirement:** FR-005\
**Precondition:** Mock token endpoint returns HTTP 401 and an
invalid-grant response.

**Steps** 1. Configure the mock endpoint to return HTTP 401 with
`invalid_grant`. 2. Execute the refresh path. 3. Confirm the HTTP
Request error output reaches `Code_HandleRefreshFailure`. 4. Inspect the
structured failure output.

**Expected** Refresh fails safely and a clear authentication failure is
reported.

**Observed** Output included `authenticationStatus: FAILED`,
`alertType: TOKEN_REFRESH_FAILURE`, `httpStatus: 401`, and a diagnostic
reason indicating rejected credentials.

**Result: PASS -- mock failure handling.**

### TC-005 -- Security Export Validation

**Requirement:** NFR-001\
**Precondition:** Workflow has been configured with n8n credentials.

**Steps** 1. Download/export the workflow JSON. 2. Search for the tested
API key and Bearer token values. 3. Inspect credential references in the
HTTP Request nodes.

**Expected** Plaintext secrets are absent; credential references may
remain.

**Observed** The latest inspected export contained credential
references/IDs and did not contain the tested API key or Bearer token in
plaintext. Exported credential labels still showed default names such as
`Header Auth account 13` and `Bearer Auth account 14`.

**Result: PASS with credential-label note.**

## 3. Overall Result

-   PASS: 4 (TC-002, TC-003, TC-004, TC-005)
-   PARTIAL: 1 (TC-001)
-   FAIL: 0
-   Total: 5

**Overall assessment:** Authentication workflow demonstrations are
substantially complete. TC-001 must be retested after ensuring the
transmitted header name is exactly `X-API-Key`. OAuth2 lifecycle results
are mock-based and should be described as simulations.

## 4. Evidence to Attach

-   TC-001 latest Postman Echo output and corrected retest output.
-   TC-002 Postman Echo output.
-   TC-003 successful refresh execution output.
-   TC-004 failed refresh execution output.
-   TC-005 latest workflow JSON export.
