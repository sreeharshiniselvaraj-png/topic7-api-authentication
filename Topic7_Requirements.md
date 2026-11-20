# Topic 7 -- API Integration Patterns & Authentication Methods

## Requirements Document

**Document version:** 1.0\
**Status:** As-built coursework requirements\
**Platform:** n8n (v1.0+)\
**Prepared for:** Hands-on Intermediate Assessment

## 1. Objective

Build an n8n API authentication testbed that demonstrates API Key header
authentication, Bearer Token authentication, OAuth2-style token refresh
handling, authentication failure handling, and secure credential
storage.

## 2. Scope

### In scope

-   API Key authentication through a custom HTTP header.
-   Bearer Token authentication through the standard `Authorization`
    header.
-   Access-token expiry evaluation and refresh-token request handling.
-   Successful refresh and failed refresh branches using a mock
    authentication endpoint.
-   Credential references in n8n and workflow-export security review.
-   Execution evidence and audit outputs.

### Out of scope / not demonstrated as production integration

-   Live OAuth2 provider authorization-code consent and callback.
-   Real provider refresh-token rotation.
-   Production identity-provider validation.
-   A measured sub-one-second refresh guarantee.
-   Pagination implementation. Although the broader course title
    mentions pagination, the supplied Topic 7 authentication requirement
    and executed workflow focus on authentication.

## 3. Business Requirements

  -----------------------------------------------------------------------
  ID                      Requirement             Priority
  ----------------------- ----------------------- -----------------------
  BRD-001                 Secrets and             High
                          authorization tokens    
                          must not be hardcoded   
                          in workflow node        
                          parameters.             

  BRD-002                 Services requiring user High
                          authorization should    
                          use OAuth2.             

  BRD-003                 Authorization tokens    High
                          should be refreshed     
                          automatically when      
                          expired.                

  BRD-004                 Custom headers must be  Medium
                          supported for legacy    
                          API Key validation.     
  -----------------------------------------------------------------------

## 4. Product Requirements

  -----------------------------------------------------------------------
  ID                      Capability              Priority
  ----------------------- ----------------------- -----------------------
  PRD-001                 Encrypted credential    High
                          storage that separates  
                          secrets from workflow   
                          logic.                  

  PRD-002                 Automated OAuth2 token  High
                          renewal capability.     

  PRD-003                 Reusable authentication High
                          patterns for API Key,   
                          Bearer Token, and       
                          OAuth2.                 

  PRD-004                 Clear handling and      Medium
                          reporting of            
                          authentication failures 
                          such as HTTP 401.       
  -----------------------------------------------------------------------

## 5. Functional Requirements

  -----------------------------------------------------------------------
  ID                                  Requirement
  ----------------------------------- -----------------------------------
  FR-001                              Support API Key, Bearer Token, and
                                      OAuth2 authentication patterns.

  FR-002                              API Key authentication must place
                                      the key in the configured HTTP
                                      header or query parameter.

  FR-003                              Bearer authentication must send
                                      `Authorization: Bearer <token>`.

  FR-004                              OAuth2 handling must obtain and
                                      retain access-token and
                                      refresh-token information.

  FR-005                              When the access token is missing or
                                      expired, the workflow must request
                                      a fresh token using the
                                      refresh-token flow and continue
                                      with the protected request when
                                      successful.

  FR-006                              Credentials must be stored securely
                                      by the n8n credential mechanism and
                                      not exposed in workflow exports.
  -----------------------------------------------------------------------

## 6. Non-Functional Requirements

-   **NFR-001 -- Security:** Credential secrets must not be serialized
    into exported workflow JSON.
-   **NFR-002 -- Reliability:** Token refresh must happen before the
    protected API call needs the token; the reference requirement
    specifies a target below one second, which was not benchmarked in
    this mock implementation.

## 7. User Stories and Acceptance Criteria

### US-001: API Key Header Injection

As a workflow builder, I want to use an API key stored in n8n
credentials so that I can authenticate requests without exposing the key
in workflow exports.

**Acceptance:** An HTTP GET request uses the configured credential,
sends the expected `X-API-Key` header, and the workflow export contains
only a credential reference.

### US-002: Automated OAuth2 Refresh

As an operations manager, I want an expired access token to be refreshed
so that protected API calls can continue without manual reauthorization.

**Acceptance:** When the token is expired or missing, the workflow
requests a fresh token, stores it in workflow static data, and the
protected request succeeds. A failed refresh must produce a clear
authentication failure result.

## 8. Test Case Requirements

  ---------------------------------------------------------------------------------------
  ID                Test              Expected result                   Linked
                                                                        requirement
  ----------------- ----------------- --------------------------------- -----------------
  TC-001            API Key in HTTP   Request succeeds; server echo     FR-001, FR-002
                    Header            confirms the configured API key   
                                      header.                           

  TC-002            Bearer Token Auth `Authorization: Bearer <token>`   FR-001, FR-003
                                      is present and request succeeds.  

  TC-003            Expired OAuth2    Fresh token is returned and       FR-004, FR-005
                    Token Refresh     saved; protected request          
                                      succeeds.                         

  TC-004            Failed Token      Failed refresh produces a         FR-005
                    Refresh Alert     401-style failure and useful      
                                      diagnostic output.                

  TC-005            Security Export   Exported workflow JSON contains   NFR-001
                    Validation        no plaintext credential secrets.  
  ---------------------------------------------------------------------------------------

## 9. Assumptions, Dependencies, and Risks

### Assumptions

-   External services follow standard HTTP authentication conventions.
-   The n8n instance can make outbound HTTPS requests.
-   The mock endpoint represents the token service for repeatable
    testing.

### Dependencies

-   n8n version 1.0.0 or higher.
-   n8n credential storage and encryption configured.
-   Postman Echo for header inspection and an n8n mock webhook for token
    responses.

### Risks and mitigations

-   **Refresh-token expiry or rotation:** use provider-specific
    credential refresh and monitor token health.
-   **Token revoked before expiry:** invalidate cached token and retry
    once after a 401 in a production implementation.
-   **Clock drift:** retain a conservative expiry buffer (60 seconds is
    used in this workflow).
-   **Credential leakage:** use n8n credentials rather than hardcoded
    secrets and inspect exports before sharing.

## 10. Glossary

-   **API Key:** Secret value used to identify or authorize an API
    client.
-   **Bearer Token:** Token sent in the HTTP Authorization header.
-   **Access Token:** Short-lived token used to access a protected
    resource.
-   **Refresh Token:** Credential used to request a replacement access
    token.
-   **JWT:** JSON Web Token.
-   **Credential Store:** n8n-managed storage for authentication
    secrets.
