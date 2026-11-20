# Topic 7 -- API Integration Patterns & Authentication Methods

An n8n coursework workflow demonstrating API Key authentication, Bearer
Token authentication, mock OAuth2-style token refresh, refresh failure
handling, and credential-export security.

## Project contents

-   `Topic7_Requirements.md` -- business, product, functional,
    non-functional, and acceptance requirements.
-   `Topic7_Implementation_Guide.md` -- workflow architecture, node
    responsibilities, setup, execution, and limitations.
-   `Topic7_Test_Cases.md` -- five trainer-aligned test cases and
    observed results.
-   n8n workflow export JSON -- keep the latest export as a separate
    project artifact.

## Environment

-   n8n instance: `https://n8n.tayanasolutions.com`
-   Header echo endpoint: `https://postman-echo.com/headers`
-   Mock token endpoint:
    `https://n8n.tayanasolutions.com/webhook/topic7/mock/auth/token`

## Workflow overview

The main workflow uses a Manual Trigger, an authentication mode
selector, and a Switch node to route between API Key and Bearer Token
paths. The Bearer path checks token expiry, requests a replacement token
from the mock endpoint when needed, saves it in workflow static data,
and then calls the protected resource. A separate error path formats
refresh failures.

## How to test

1.  Import/open the latest Topic 7 workflow JSON in n8n.
2.  Ensure the mock authentication endpoint workflow is active.
3.  Execute `HTTP_APIKeyRequest` and inspect the returned headers.
4.  Execute `HTTP_BearerRequest` and verify the Authorization header.
5.  Configure the mock token endpoint for HTTP 200 and run the refresh
    success path.
6.  Configure the mock token endpoint for HTTP 401 / invalid grant and
    run the failure path.
7.  Restore the success response after negative testing.
8.  Export the workflow and verify that secret values are not present in
    plaintext.

## Test status at handover

-   TC-001 API Key: PARTIAL; latest output echoed a header named
    `header_apikey_test`, not the exact required `X-API-Key`.
-   TC-002 Bearer: PASS.
-   TC-003 refresh success: PASS using mock endpoint.
-   TC-004 refresh failure: PASS using mock endpoint.
-   TC-005 export security: PASS; credential references remain, but
    tested secrets were absent.

See `Topic7_Test_Cases.md` for the evidence and detailed observations.

## Important implementation notes

-   The token refresh workflow uses a mock endpoint; it is not a live
    OAuth2 provider integration.
-   The exported credential labels were still default labels
    (`Header Auth account 13` and `Bearer Auth account 14`) in the last
    inspected export.
-   Do not commit real credentials, access tokens, refresh tokens, or
    client secrets to source control.
-   The broader course topic name mentions pagination, but these
    deliverables describe the API authentication work that was
    implemented in this workflow.

## Suggested repository structure

``` text
topic7-api-authentication/
├── README.md
├── Topic7_Requirements.md
├── Topic7_Implementation_Guide.md
├── Topic7_Test_Cases.md
└── Topic7_API_Authentication_Workflow.json
```
