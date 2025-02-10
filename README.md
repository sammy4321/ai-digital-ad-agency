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
