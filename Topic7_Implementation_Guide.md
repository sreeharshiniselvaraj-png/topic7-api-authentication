# Topic 7 -- API Integration Patterns & Authentication Methods

## Implementation Guide

**Platform:** n8n\
**Workflow:** Topic 7 - API Authentication Main Workflow\
**Implementation status:** Built and tested with mock authentication
endpoints

## 1. Objective

Implement and demonstrate API Key authentication, Bearer Token
authentication, token-expiry detection, mock token refresh, refresh
failure handling, and credential-export security.

## 2. Environment and Components

-   n8n instance: `https://n8n.tayanasolutions.com`
-   Header inspection endpoint: `https://postman-echo.com/headers`
-   Mock token endpoint:
    `https://n8n.tayanasolutions.com/webhook/topic7/mock/auth/token`
-   Credential types: Header Auth and Bearer Auth.
-   No live third-party OAuth2 provider is connected; token lifecycle
    tests use a mock endpoint.

## 3. Main Workflow Nodes

  -----------------------------------------------------------------------
  Node                                Purpose
  ----------------------------------- -----------------------------------
  `Manual_Trigger`                    Starts the workflow manually.

  `Code_SetAuthMode`                  Sets the authentication test mode.
                                      Current exported value is
                                      `testMode = 1` (Bearer branch).

  `Switch_AuthPattern`                Routes execution to the selected
                                      authentication branch.

  `HTTP_CallWithApiKey`               Executes API Key-authenticated
                                      request.

  `Code_CheckTokenExpiry`             Checks cached token and expiry
                                      time.

  `IF_TokenIsExpired`                 Routes to refresh or reuse-token
                                      path.

  `HTTP_RefreshAuthToken`             Sends POST request to the mock
                                      token endpoint.

  `Code_SaveFreshToken`               Stores returned token and expiry in
                                      workflow static data.

  `HTTP_CallWithBearerToken`          Calls protected endpoint with
                                      Bearer header.

  `Code_AuditAuthSuccess`             Produces successful authentication
                                      audit output.

  `Code_AuditApiKeySuccess`           Produces API Key branch audit
                                      output.

  `Code_HandleRefreshFailure`         Formats failed refresh information.

  `Webhook`                           Mock endpoint entry point used by
                                      the workflow.

  `HTTP_APIKeyRequest`                Separate API Key header-echo test
                                      node.

  `HTTP_BearerRequest`                Separate Bearer header-echo test
                                      node.
  -----------------------------------------------------------------------

## 4. Authentication Branches

### 4.1 API Key header verification

1.  Create/use an n8n Header Auth credential.
2.  Configure the HTTP Request node to call
    `https://postman-echo.com/headers`.
3.  Select Generic Credential Type → Header Auth.
4.  Execute the node and inspect the returned `headers` object.
5.  Confirm the exact header name and value. The latest supplied output
    showed the key `header_apikey_test` with the test value, while an
    earlier test output was reported as `x-api-key`. The latest output
    therefore does not conclusively match the trainer's exact
    `X-API-Key` header-name requirement.

### 4.2 Bearer Token header verification

1.  Create/use an n8n Bearer Auth credential.
2.  Configure `HTTP_BearerRequest` as GET
    `https://postman-echo.com/headers`.
3.  Select Generic Credential Type → Bearer Auth.
4.  Execute and inspect `headers.authorization`.
5.  Verified output contained
    `Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.token123`.

### 4.3 Token expiry evaluation

The token evaluator checks whether a cached token is missing or is
within 60 seconds of expiry. The logic used in the workflow is:

``` javascript
const staticData = $getWorkflowStaticData('global');

const cachedToken = staticData.bearerToken || null;
const tokenExpiresAt = staticData.tokenExpiresAt || 0;

const needsRefresh =
  !cachedToken || Date.now() >= tokenExpiresAt - 60000;
```

The IF node routes `needsRefresh = true` to the refresh request. The
false branch reuses the cached token.

### 4.4 Mock token refresh success

The mock endpoint returns HTTP 200 with a response shaped like:

``` json
{
  "access_token": "jwt_mock_access_token_2026",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

`HTTP_RefreshAuthToken` sends a POST request to the mock endpoint.
`Code_SaveFreshToken` stores the returned access token and computes the
expiry timestamp. `HTTP_CallWithBearerToken` then uses the token in the
Authorization header. The successful run produced `FRESH_TOKEN_SAVED`,
followed by an authorized response and `authenticationStatus: SUCCESS`.

### 4.5 Mock token refresh failure

For the negative test, the mock endpoint was configured to return HTTP
401 with an `invalid_grant` response. `HTTP_RefreshAuthToken` uses the
error output (`continueErrorOutput`) to reach
`Code_HandleRefreshFailure`.

Observed failure output included: - `authenticationStatus: FAILED` -
`authenticationMethod: BEARER_TOKEN` -
`alertType: TOKEN_REFRESH_FAILURE` - `httpStatus: 401` - diagnostic
reason: `Authentication credentials rejected`

The mock endpoint success response was restored after the failure test
and a subsequent successful refresh was verified.

## 5. Credential Security Configuration

-   API Key and Bearer secrets are configured in n8n credentials, not
    pasted into the workflow node's request parameters.
-   The workflow JSON contains credential references/IDs.
-   The latest export inspected did not contain the tested API key value
    or Bearer JWT value in plaintext.
-   Credential labels in the export still appeared as
    `Header Auth account 13` and `Bearer Auth account 14`; the requested
    custom labels were not confirmed in the exported JSON.
-   A credential reference in exported JSON is expected; the secret
    value should not be present.

## 6. Execution and Verification Procedure

1.  Open the Topic 7 workflow in n8n.
2.  Confirm the mock authentication workflow is active.
3.  For API Key verification, execute `HTTP_APIKeyRequest` and inspect
    the echo output.
4.  For Bearer verification, execute `HTTP_BearerRequest` and inspect
    `headers.authorization`.
5.  Set the mock token endpoint to the success response and execute the
    refresh path.
6.  Confirm `Code_SaveFreshToken` reports the fresh token saved and the
    protected call is authorized.
7.  Change the mock endpoint to HTTP 401 with an invalid-grant response
    and execute the failure path.
8.  Confirm `Code_HandleRefreshFailure` reports the failure and HTTP
    status.
9.  Restore the success response.
10. Download the workflow JSON and search for literal secret values
    before submission.

## 7. Known Limitations and Deviations

-   OAuth2 success/failure was demonstrated using an in-n8n mock token
    endpoint. It is not proof of a live provider's OAuth2
    authorization-code or refresh-token rotation lifecycle.
-   The current exported workflow has `testMode = 1`, selecting the
    Bearer branch.
-   The latest API Key echo output showed `header_apikey_test`;
    verify/correct the header name to exactly `X-API-Key` if strict
    trainer matching is required.
-   The exported credential names remain default labels, rather than the
    custom names mentioned during setup.
-   The under-one-second refresh target was not measured.
-   Pagination is not part of the implemented authentication workflow
    described in this guide.
