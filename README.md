# AI Based Digital Advertising

## Overall App Flow : 
                                      ┌──────────────────────────────┐
                                      │    User's Flutter App        │
                                      │ (Initiates login & actions)  │
                                      └─────────────┬────────────────┘
                                                    │ Secure HTTPS Request (login, actions)
                                                    │
                                                    v
                                      ┌──────────────────────────────┐
                                      │    Your Backend Server       │
                                      │  (Stores secrets, handles    │
                                      │   OAuth flows, proxies API   │
                                      │         requests)            │
                                      └─────────────┬────────────────┘
                                                    │
                                                    │ Initiates OAuth2 flow for each API
                                                    │   (Google Ads & Meta)
                                                    v
                                      ┌──────────────────────────────┐
                                      │    OAuth2 Consent Flow       │
                                      │ (User logs in, grants access)│
                                      └─────────────┬────────────────┘
                                                    │
                                                    │ Access tokens obtained
                                                    v
                             ┌────────────────────────────────────────┐
                             │  Manager Account / API Credentials     │
                             │   (Google Ads Manager / Meta Business   │
                             │          Manager Credentials)          │
                             └─────────────┬──────────────────────────┘
                                           │
               ┌───────────────────────────┼─────────────────────────────┐
               │                           │                             │
               v                           v                             v
     ┌─────────────────┐         ┌─────────────────┐          ┌─────────────────┐
     │ Google Ads API  │         │ Meta Ads API    │          │ Billing & Metrics│
     │ (Create & Manage│         │ (Create & Manage│          │ (Client reviews, │
     │   Ad Accounts,  │         │   Ad Accounts,  │          │  monitors ads)   │
     │  Post Ads, etc.)│         │  Post Ads, etc.)│          └─────────────────┘
     └─────────────────┘         └─────────────────┘
               │                           │
               └─────────────┬─────────────┘
                             │
                             v
                  ┌──────────────────────────────┐
                  │   API Responses & Data       │
                  │   (Ad posting results,       │
                  │    metrics, etc.)            │
                  └──────────────────────────────┘

## Account Setup Workflow : 
```bash
──────────────────────────────────────────────────────────────────────────────
                           Google Ads Workflow
──────────────────────────────────────────────────────────────────────────────
 1. Manager Account Setup
    ┌─────────────────────────────────────────────────────────────┐
    │ • Create a Google Ads Manager (MCC) account via the website   │
    │ • Use the MCC to centrally manage multiple client accounts    │
    └─────────────────────────────────────────────────────────────┘
            │
            ▼
 2. API Access & Credentials
    ┌─────────────────────────────────────────────────────────────┐
    │ • Request a developer token (via API Center or support)       │
    │ • Set up a Google Cloud Project & enable the Google Ads API   │
    │ • Create OAuth2 credentials in the GCP project                │
    │ • Complete the OAuth2 flow to obtain a refresh token          │
    └─────────────────────────────────────────────────────────────┘
            │
            ▼
 3. User Authentication for Authorization
    ┌─────────────────────────────────────────────────────────────┐
    │ • Redirect client to the OAuth2 consent screen                │
    │ • Client logs in and grants permission for manager actions    │
    │ • Secure tokens/permissions are returned to the backend       │
    └─────────────────────────────────────────────────────────────┘
            │
            ▼
 4. Client Account Management
    ┌─────────────────────────────────────────────────────────────┐
    │ • Use the API (CustomerService) to automatically create new   │
    │   client accounts under your Manager account                  │
    │ • Or link existing client accounts via invitation acceptance   │
    │ • New client accounts are created with basic settings only     │
    └─────────────────────────────────────────────────────────────┘
            │
            ▼
 5. Billing Setup
    ┌─────────────────────────────────────────────────────────────┐
    │ • Each client account must set up its own billing information  │
    │   manually in the Billing & Payments section                  │
    │ • Automated billing configuration via the API is not available │
    └─────────────────────────────────────────────────────────────┘
            │
            ▼
 6. Automated Ad Management with Python
    ┌─────────────────────────────────────────────────────────────┐
    │ • Write Python code using the Google Ads API client library    │
    │   to create ads (via AdGroupAdService) and retrieve campaign     │
    │   metrics (using GoogleAdsService & GAQL queries)              │
    │ • Use the centralized GCP project and OAuth2 tokens to manage   │
    │   ads across all linked client accounts                        │
    └─────────────────────────────────────────────────────────────┘
──────────────────────────────────────────────────────────────────────────────
──────────────────────────────────────────────────────────────────────────────
                      Meta (Facebook/Instagram) Workflow
──────────────────────────────────────────────────────────────────────────────
 1. Manager Account Setup
    ┌─────────────────────────────────────────────────────────────┐
    │ • Create a Facebook Business Manager account                │
    │ • Use Business Manager to centrally manage client ad accounts │
    └─────────────────────────────────────────────────────────────┘
            │
            ▼
 2. API Access & Credentials
    ┌─────────────────────────────────────────────────────────────┐
    │ • Create a Facebook Developer App in the Developer Dashboard   │
    │ • Obtain App ID, App Secret, and generate an access token        │
    │ • Complete the OAuth2 flow to get necessary permissions          │
    │   (e.g., ads_management, business_management)                   │
    └─────────────────────────────────────────────────────────────┘
            │
            ▼
 3. User Authentication for Authorization
    ┌─────────────────────────────────────────────────────────────┐
    │ • Redirect client to Facebook’s OAuth consent dialog            │
    │ • Client logs in and explicitly grants permission for your app    │
    │   to manage their ad account                                    │
    │ • Secure access tokens are provided to your backend             │
    └─────────────────────────────────────────────────────────────┘
            │
            ▼
 4. Client Account Management
    ┌─────────────────────────────────────────────────────────────┐
    │ • Use the API (via Business Manager endpoints) to create new    │
    │   client ad accounts under your Business Manager                │
    │ • Or link existing ad accounts via invitation acceptance         │
    │ • New ad accounts are created with basic settings only            │
    └─────────────────────────────────────────────────────────────┘
            │
            ▼
 5. Billing Setup
    ┌─────────────────────────────────────────────────────────────┐
    │ • Each ad account must set up its own billing manually          │
    │   in Facebook Ads Manager (Billing & Payments section)          │
    │ • Automated billing configuration via the API is not available   │
    └─────────────────────────────────────────────────────────────┘
            │
            ▼
 6. Automated Ad Management with Python
    ┌─────────────────────────────────────────────────────────────┐
    │ • Write Python code using the Facebook Business SDK or Graph API │
    │   to create and manage campaigns and ads                        │
    │ • Retrieve campaign insights via API calls                       │
    │ • Use centralized app credentials and OAuth tokens to manage     │
    │   ads across all linked client ad accounts                         │
    └─────────────────────────────────────────────────────────────┘
──────────────────────────────────────────────────────────────────────────────

```
