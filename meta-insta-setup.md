# Overview of the Process for Meta Advertising

## Set Up Your Meta Business Infrastructure

### Business Manager Account
- Create (or use an existing) Facebook Business Manager account. This is analogous to the Google Ads manager (MCC) account.

### Facebook Developer App
- Create an app on the Facebook Developer Dashboard.
- In the app settings, generate your **App ID**, **App Secret**, and obtain an **access token** (with the required scopes for the Marketing API).

## Automate Creation of New Client Ad Accounts (API-Based)
- Using the Business Manager’s API endpoints, you can programmatically create new client ad accounts under your Business Manager.
- You’ll supply details such as the **account name**, **currency**, and **time zone**.

## Ad and Campaign Management via Python
- Once the ad account is created (or an existing account is linked), you can use the **Marketing API** to create campaigns, ad sets, and ads.
- You can also query insights and metrics for each campaign.

## Billing Setup Limitations
- Although you can set spending limits or retrieve billing info, there is no public API to set up or modify payment methods (i.e. link a client’s credit card) due to security and compliance restrictions.
- Each new ad account must have its billing/payment methods configured manually via **Facebook’s Ads Manager interface**.

# Detailed Steps and Python Code Samples

## Step 1: Set Up Your Meta Business Environment

### 1.1 Create or Use a Facebook Business Manager Account
- Visit [Facebook Business Manager](https://business.facebook.com/) and follow the instructions to set up your account.
- If you already have a Business Manager account, you can use the existing one.

### 1.2 Create a Facebook Developer App
- Go to the [Facebook Developer Dashboard](https://developers.facebook.com/) and create a new app.
- In the app settings:
  - Add the **Marketing API** product to the app.
  - Generate an **access token** with the required permissions such as:
    - `ads_management`
    - `business_management`
    - Any other scopes required for your specific operations.

> 💡 **Note:** Keep the following details secure and ready:
- **App ID**
- **App Secret**
- **Access Token**

### 1.3 Install the Facebook Business SDK for Python
You can install the required SDK using pip:

```bash
pip install facebook_business
```

## Step 2: Automate Creating a New Client Ad Account

The following Python sample demonstrates how to create a new client ad account under your Business Manager using the Facebook Business SDK. This "client" ad account is created under your business infrastructure but requires manual billing setup later.

### 🔔 **Note:**
- Not all parameters may be required; check the [Business Ad Account Creation Documentation](https://developers.facebook.com/docs) for further details.
- The billing/payment setup must be configured manually via the Facebook Ads Manager.

---

### Python Code Sample
```python
from facebook_business.api import FacebookAdsApi
from facebook_business.adobjects.business import Business

# Replace with your app details and access token.
access_token = 'YOUR_ACCESS_TOKEN'
app_secret = 'YOUR_APP_SECRET'
app_id = 'YOUR_APP_ID'
FacebookAdsApi.init(access_token=access_token, app_secret=app_secret, app_id=app_id)

# Your Business Manager ID.
business_id = 'YOUR_BUSINESS_ID'  # e.g., '1234567890'
business = Business(business_id)

# Parameters for the new ad account.
params = {
    'name': 'New Client Ad Account - Example',
    'currency': 'USD',
    'timezone_id': 1,  # Use the appropriate timezone ID as defined by Facebook.
    'end_advertiser': 'CLIENT_BUSINESS_NAME',  # Typically the client’s business name.
    # Additional optional fields may be added here as needed.
}

try:
    # Create the new ad account under the business
    new_ad_account = business.create_ad_account(params=params)
    print("Created new ad account:", new_ad_account)
except Exception as e:
    print("Error creating ad account:", e)
```

### 🔍 **What This Does:**

- **Uses your Business Manager’s ID to create a new ad account:**  
  The provided `business_id` allows the SDK to programmatically create a new ad account associated with your Business Manager.

- **The new account appears under your Business Manager:**  
  Once successfully created, the ad account will be listed under the associated Business Manager account and will be ready for campaign setup.

- **Billing:**  
  After creating the ad account, it **requires manual billing setup**.  
  You must configure the payment method (e.g., linking a client’s credit card) using the **Facebook Ads Manager**. Facebook does not provide an API for billing setup due to security and compliance reasons.

## Step 3: Creating Ads and Retrieving Campaign Metrics via Python
After you have an ad account (whether newly created or pre-existing and linked), you can use the Marketing API to manage campaigns and retrieve metrics. Here are sample snippets.

### A. Creating a Campaign, Ad Set, and Ad : 
Below is an example of how you might create a campaign, ad set, and an ad creative using the SDK. (Note that full production code would include error handling and more detailed parameters.)
```python
from facebook_business.adobjects.adaccount import AdAccount
from facebook_business.adobjects.campaign import Campaign
from facebook_business.adobjects.adset import AdSet
from facebook_business.adobjects.adcreative import AdCreative
from facebook_business.adobjects.ad import Ad

# Replace with your client ad account ID. It usually starts with "act_"
ad_account_id = 'act_YOUR_CLIENT_AD_ACCOUNT_ID'
ad_account = AdAccount(ad_account_id)

# Step 1: Create a Campaign
campaign_params = {
    Campaign.Field.name: 'My Automated Campaign',
    Campaign.Field.objective: Campaign.Objective.link_clicks,
    Campaign.Field.status: Campaign.Status.paused,
}
campaign = ad_account.create_campaign(params=campaign_params)
print("Created campaign with ID:", campaign[Campaign.Field.id])

# Step 2: Create an Ad Set (for targeting, budget, etc.)
adset_params = {
    AdSet.Field.name: 'My Ad Set',
    AdSet.Field.campaign_id: campaign[Campaign.Field.id],
    AdSet.Field.daily_budget: 5000,  # Budget in cents (or the currency's smallest unit)
    AdSet.Field.billing_event: AdSet.BillingEvent.impressions,
    AdSet.Field.optimization_goal: AdSet.OptimizationGoal.link_clicks,
    AdSet.Field.bid_amount: 100,  # Bid amount in cents
    AdSet.Field.targeting: {
        'geo_locations': {
            'countries': ['US'],
        },
        'age_min': 18,
        'age_max': 65,
    },
    AdSet.Field.start_time: '2025-03-01T00:00:00-0700',
    AdSet.Field.end_time: '2025-03-10T00:00:00-0700',
}
adset = ad_account.create_ad_set(params=adset_params)
print("Created ad set with ID:", adset[AdSet.Field.id])

# Step 3: Create an Ad Creative (for the ad’s content)
creative_params = {
    AdCreative.Field.name: 'Sample Creative',
    AdCreative.Field.object_story_spec: {
        'page_id': 'YOUR_PAGE_ID',
        'link_data': {
            'message': 'Check out our new product!',
            'link': 'https://www.example.com',
            'caption': 'example.com',
            'picture': 'https://www.example.com/image.jpg'
        }
    }
}
creative = ad_account.create_ad_creative(params=creative_params)
print("Created ad creative with ID:", creative[AdCreative.Field.id])

# Step 4: Create the Ad
ad_params = {
    Ad.Field.name: 'My Automated Ad',
    Ad.Field.adset_id: adset[AdSet.Field.id],
    Ad.Field.creative: {'creative_id': creative[AdCreative.Field.id]},
    Ad.Field.status: Ad.Status.paused,
}
ad = ad_account.create_ad(params=ad_params)
print("Created ad with ID:", ad[Ad.Field.id])
```
### B : Retrieving Campaign Metrics
```python
from facebook_business.adobjects.adsinsights import AdsInsights

campaign_id = campaign[Campaign.Field.id]
# Create a Campaign object using the campaign ID.
from facebook_business.adobjects.campaign import Campaign
campaign_obj = Campaign(campaign_id, api=ad_account.api)

# Define the parameters for the insights query.
params = {
    'date_preset': 'last_7d',
    'fields': [
        AdsInsights.Field.campaign_name,
        AdsInsights.Field.impressions,
        AdsInsights.Field.clicks,
        AdsInsights.Field.spend,
    ],
}

try:
    insights = campaign_obj.get_insights(params=params)
    for insight in insights:
        print(f"Campaign {insight.get('campaign_name')}: "
              f"{insight.get('impressions')} impressions, "
              f"{insight.get('clicks')} clicks, "
              f"${insight.get('spend')} spent")
except Exception as e:
    print("Error retrieving insights:", e)
```

## Step 4. Billing Setup

- Ad Account Billing in Meta : Although you can create and manage ad accounts and campaigns via the API, the billing setup (linking a client’s payment method) cannot be automated through the API.
- Manual Action Required : Once a new ad account is created, you or the client must log into Facebook Ads Manager and configure the billing settings (adding credit card, bank details, or setting up invoicing).

# Final Summary

- Ad Account Creation and Management : You can use the Meta Marketing API (via the Facebook Business SDK for Python) to programmatically create new client ad accounts under your Business Manager, set up campaigns, ad sets, and ads, and pull metrics.
- Billing Automation : While many operational tasks can be automated with Python, billing and payment method configuration must be set up manually in Facebook’s interface due to security and compliance restrictions.
- Integration for Facebook and Instagram : Since Instagram ads are managed through Facebook’s ad platform, the same ad account can serve both networks; your API calls will cover ads on both platforms based on how you configure the creative and placements.
