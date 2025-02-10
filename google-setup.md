# Summary of the Google Ads API Process

## Manager Account Setup
- Create a **Google Ads Manager (MCC)** account via the [Google Ads website](https://ads.google.com/).
- Use this account to centrally manage multiple client accounts.

## API Access & Credentials
- Request a **developer token** from your Google Ads account (via API Center or support).
- Set up a **Google Cloud Project**, enable the Google Ads API, and create **OAuth2 credentials**.
- Complete the **OAuth2 flow** to obtain a refresh token for authentication.

## Client Account Management
- Use the API (via `CustomerService`) to create new client accounts under your Manager account or link existing ones. This will create a google ads account for that individual account and the user can login through the google ads interface.
  - **Note:** New client accounts will be created with basic settings only.

## Billing Setup
- Each client account must have its own **billing information** set up manually in the **Billing & Payments** section. Once the user ads account is created by the manager, the user will have to login and setup billing.  
- Automated billing configuration via the API is not available.

## Automated Ad Management with Python
- Write Python code (using the **Google Ads API client library**) to create ads (e.g., using `AdGroupAdService`).
- Use Python to retrieve campaign metrics (e.g., with `GoogleAdsService` and **GAQL** queries) for a particular client account.
- Your centralized **GCP project** and OAuth2 credentials allow you to manage ads across all linked client accounts.

# Detailed Process : 
# Step 1. Creating a Google Ads Manager Account (MCC)

## Sign Up for a Manager Account:
- Visit the [Google Ads Manager Account signup page](https://ads.google.com/intl/en/home/tools/manager-accounts/).
- Follow the instructions to create a new Manager account.
- Fill in your business information and complete the verification steps.

## Access Your Manager Dashboard:
- Once created, you’ll have a central dashboard where you can view and manage multiple client (child) accounts.

# Step 2. Setting Up Individual Client Accounts

## A. Manual vs. API-Based Account Creation

### Manual Process
- Log into your Manager account and click the “+” button to create or invite client accounts.
- This method lets you configure billing details, users, and notifications directly in the interface.

### API-Based Account Creation
- To automate account creation, use the Google Ads API’s `CustomerService` to create a new client account under your Manager account.
- The API call requires specifying details such as the descriptive name, currency, time zone, etc.

## B. Example: Creating a New Client Account via API

> **Note:** You must use OAuth2 credentials and a developer token (from your GCP project) associated with your Manager account. The API allows creating a new client account (often called a “customer client”) under a Manager account, but not all billing details can be set via the API.

### Python Example: Creating a New Client Account
```python
from google.ads.googleads.client import GoogleAdsClient
from google.ads.googleads.errors import GoogleAdsException

def create_client_account(manager_customer_id):
    # Load your configuration (google-ads.yaml should contain your OAuth2 details and developer token)
    client = GoogleAdsClient.load_from_storage()

    # Get the CustomerService
    customer_service = client.get_service("CustomerService")

    # Create a new Customer object (this is the new client account)
    new_customer = client.get_type("Customer")
    new_customer.descriptive_name = "New Client Account - Example"
    new_customer.currency_code = "USD"
    new_customer.time_zone = "America/New_York"
    new_customer.tracking_url_template = "{lpurl}?device={device}"
    new_customer.has_partners_badge = False

    # Optional: Specify the email address for the client account
    email_address = "client@example.com"

    try:
        # Create the client account under the manager account
        response = customer_service.create_customer_client(
            customer_id=manager_customer_id,
            email_address=email_address,
            customer_client=new_customer,
        )
        print(f"Created new client account with resource name: {response.resource_name}")
        return response.resource_name  # Uniquely identifies the client account
    except GoogleAdsException as ex:
        print("Request had errors:")
        for error in ex.failure.errors:
            print(f"\tError: {error.message}")
        return None

if __name__ == '__main__':
    # Replace with your manager account customer ID (without dashes, e.g., "1234567890")
    manager_customer_id = "YOUR_MANAGER_CUSTOMER_ID"
    create_client_account(manager_customer_id)
```

# Step 3. Setting Up Individual Billing for Each Client Account

## A. How Billing Is Normally Configured

### Billing Settings in Google Ads
- Each client account has its own **Billing & Payments** section within the Google Ads interface.
- Here, you can add or update payment methods (e.g., credit cards, bank accounts, invoicing details) so that ad spend is charged directly to the client’s account.

### Manual Setup Required
- Currently, Google does not offer a public API endpoint to programmatically create or modify billing setups.
- You must:
  - Log in to the specific client’s Google Ads account (or use the Manager interface to navigate to the client’s billing settings).
  - Manually enter the client’s billing details.

## B. What You Can Automate

### Retrieving Billing Information
- Although setting up billing cannot be automated, you can retrieve existing billing information or check the status of a client’s billing using available Google Ads API resources.

### Separate Billing Responsibility
- Ensure that each client account is configured with its own payment method.
- This prevents your agency (or manager account) from being responsible for processing payments on behalf of clients.

# Step 4. Python Code for Creating Ads and Retrieving Campaign Metrics

## A. Creating an Ad for a Client Account

```python
import sys
import uuid
from google.ads.googleads.client import GoogleAdsClient
from google.ads.googleads.errors import GoogleAdsException

def create_text_ad(client, client_customer_id, ad_group_id):
    # Get the AdGroupAdService
    ad_group_ad_service = client.get_service("AdGroupAdService")

    # Create an ad group ad operation
    ad_group_ad_operation = client.get_type("AdGroupAdOperation")
    ad_group_ad = ad_group_ad_operation.create

    # Set the ad group resource name
    ad_group_service = client.get_service("AdGroupService")
    ad_group_ad.ad_group = ad_group_service.ad_group_path(client_customer_id, ad_group_id)
    
    # Set the initial status (e.g., PAUSED, ENABLED)
    ad_group_ad.status = client.enums.AdGroupAdStatusEnum.PAUSED

    # Configure the ad
    ad = ad_group_ad.ad
    ad.final_urls.append("http://www.example.com")
    
    # Use a UUID to create a unique headline
    unique_suffix = str(uuid.uuid4())[:8]
    ad.expanded_text_ad.headline_part1 = f"Cruise to Mars #{unique_suffix}"
    ad.expanded_text_ad.headline_part2 = "Best Space Cruise Line"
    ad.expanded_text_ad.description = "Buy your tickets now!"

    try:
        # Submit the ad creation request
        response = ad_group_ad_service.mutate_ad_group_ads(
            customer_id=client_customer_id, operations=[ad_group_ad_operation]
        )
        print(f"Created ad group ad with resource name: {response.results[0].resource_name}")
    except GoogleAdsException as ex:
        print("Request had errors:")
        for error in ex.failure.errors:
            print(f"\tError: {error.message}")
        sys.exit(1)

if __name__ == '__main__':
    # Load configuration (ensure your google-ads.yaml contains the OAuth2 and developer token details)
    google_ads_client = GoogleAdsClient.load_from_storage()
    
    # Replace with the client (child) account's customer ID (e.g., "1234567890")
    client_customer_id = "TARGET_CLIENT_CUSTOMER_ID"
    # Replace with the ad group ID in which you want to create the ad
    ad_group_id = "YOUR_AD_GROUP_ID"
    
    create_text_ad(google_ads_client, client_customer_id, ad_group_id)
```

## B. Retrieving Campaign Metrics for a Client Account

```python
from google.ads.googleads.client import GoogleAdsClient
from google.ads.googleads.errors import GoogleAdsException

def get_campaign_metrics(client, client_customer_id):
    ga_service = client.get_service("GoogleAdsService")
    
    # Define a GAQL query to retrieve campaign metrics
    query = """
        SELECT
            campaign.id,
            campaign.name,
            metrics.impressions,
            metrics.clicks,
            metrics.cost_micros
        FROM campaign
        WHERE campaign.status = 'ENABLED'
    """
    
    try:
        # Issues a search request using streaming
        response = ga_service.search_stream(customer_id=client_customer_id, query=query)
        for batch in response:
            for row in batch.results:
                campaign = row.campaign
                metrics = row.metrics
                print(
                    f"Campaign ID {campaign.id} ('{campaign.name}'): "
                    f"{metrics.impressions} impressions, "
                    f"{metrics.clicks} clicks, "
                    f"{metrics.cost_micros / 1_000_000:.2f} cost"
                )
    except GoogleAdsException as ex:
        print("Request had errors:")
        for error in ex.failure.errors:
            print(f"\tError: {error.message}")

if __name__ == '__main__':
    # Load your configuration
    google_ads_client = GoogleAdsClient.load_from_storage()
    
    # Replace with the target client account's customer ID
    client_customer_id = "TARGET_CLIENT_CUSTOMER_ID"
    
    get_campaign_metrics(google_ads_client, client_customer_id)

```
### **Note:**
- Replace placeholder values (like `"YOUR_MANAGER_CUSTOMER_ID"`, `"TARGET_CLIENT_CUSTOMER_ID"`, and `"YOUR_AD_GROUP_ID"`) with actual values.
- Ensure that your **google-ads.yaml** is correctly configured with the OAuth2 credentials, developer token, and (if needed) the manager account’s ID using the `login-customer-id` parameter.
- The code above assumes that the client account and the necessary campaign/ad group structures already exist.

### Notes to self : 
1. **Create a Google Ads Manager Account**  
   - Visit the [Google Ads Manager Account signup page](https://ads.google.com/).  
   - Follow the instructions to create and set up the Manager account.

2. **Get Developer Token**  
   - Navigate to **Tools & Settings > API Center** in your Google Ads account.  
   - Request a **developer token** (this may initially be in test mode and require approval for production use).

3. **Create GCP Project**  
   - Go to the [Google Cloud Console](https://console.cloud.google.com/) and create a new project.  
   - Enable the **Google Ads API** in the **APIs & Services > Library** section.

4. **Create OAuth Client and Get Client ID and Client Secret**  
   - Navigate to **APIs & Services > Credentials** in the GCP Console.  
   - Create **OAuth client credentials** (do not use service accounts).  
   - Select the appropriate application type (e.g., Desktop App or Web Application).  
   - After creating the OAuth client, note down the **Client ID** and **Client Secret**.

   **Get the Refresh Token:**  
   - Run the OAuth flow for the first time to generate a **refresh token**:  
     - Use the **OAuth client ID and secret** to run the flow using an installed app or command-line application.
     - This refresh token will be used for authenticating subsequent API requests.
