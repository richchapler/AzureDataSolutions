## Extracting Data from Purview via REST API

Requirement statements might include:

* "We tried using the Power BI, Cost Management connector but lack sufficient permissions at the billing account (and that is not going to change because of corporate policy)"

### Step 1: Prepare Resources

This solution requires the following resources:

* [Application Registration](PrepareResources_ApplicationRegistration.md)
* [Data Lake](PrepareResources_DataLake.md) (with [container](PrepareResources_DataLake_Container.md))
* [Synapse](PrepareResources_Synapse.md) (with [linked services](PrepareResources_Synapse_LinkedService.md) and [datasets](PrepareResources_Synapse_Dataset.md) for your source Azure API and target Data Lake, delimited output)
*	Purview (with collection role assignments **Collection admins**, **Data source admins**, and **Data curators** for your Application Registration)

# RESUME HERE!

### Step 2: Create Pipeline

Complete the following steps:
* Navigate to **Synapse Studio**
* Click the **Integrate** navigation icon
* Click **+** and select **Pipeline** from the resulting dropdown menu

#### Activity 1: Get Token

This activity will make a REST API call to http://login.microsoftonline.com and get a bearer token.

Complete the following steps:
* Expand **General** in the **Activities** bar
* Drag-and-drop a **Web** component into the main window
* Complete the form on the **Settings** tab

  <img src="https://user-images.githubusercontent.com/44923999/179229885-810ac78b-b59c-4ce6-a2c5-6e12047011b7.png" width="800" title="Snipped: July 15, 2022" />

  Prompt | Entry
  ------ | ------
  **URL** | Modify and enter:`https://login.microsoftonline.com/{TenantId}/oauth2/token`  
  **Method** | Select **POST**  
  **Headers** | Click **+ Add** and enter key-value pair: `content-type` :: `application/x-www-form-urlencoded`
  **Body** | ...for Cost Management, modify and enter:<br>`grant_type=client_credentials&client_id={ClientId}&client_secret={ClientSecret}&resource=https://management.azure.com/`<br><br>...for Log Analytics, modify and enter:<br>`grant_type=client_credentials&client_id={ClientId}&client_secret={ClientSecret}&resource=https://api.loganalytics.io/`<br><br>...for Purview, modify and enter:<br>`grant_type=client_credentials&client_id={Client Identifier}&client_secret={Client Secret}& resource=https://purview.azure.net`

* Click **Debug** and monitor to confirm success

#### Activity 2: Get Data

This activity will make a REST API call and capture the response as a delimited file in the Data Lake.

Complete the following steps:
* Expand **Move & Transform** in the **Activities** bar
*	Drag-and-drop a **Copy data** component into the activity window
*	Create a dependency from the **Get Token** component to the **Copy data** component
*	Complete the form on the **Source** tab

    <img src="https://user-images.githubusercontent.com/44923999/179236666-66456de7-73f3-4867-967e-c04289bff466.png" width="800" title="Snipped: July 15, 2022" />

    Prompt | Entry
    ------ | ------
    **Source Dataset** | Select your REST dataset
    **Request Method** | ...for Cost Management, select **POST**<br>...for Log Analytics,  select **POST**<br>...for Purview, select **GET**
    **Request Body** | ...for Cost Management, enter:<br>
`{
  "type": "Usage",
  "timeframe": "TheLastMonth",
  "dataset": {
    "granularity": "None",
    "aggregation": {
      "totalCost": {
        "name": "PreTaxCost",
        "function": "Sum"
      }
    },
    "grouping": [
      {
        "type": "Dimension",
        "name": "ResourceGroup"
      }
    ]
  }
}`
<br><br>...for Log Analytics, enter:<br>`{ "query": "Sample_CL | take 10" }`
  
  
  and enter:`https://login.microsoftonline.com/{TenantId}/oauth2/token`  