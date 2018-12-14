---
title: Getting started with PlayFab, Unity IAP, and Android
author: v-thopra
description: How to set up (In-App Purchasing) IAP using PlayFab, the Unity + IAP Service, and the Android Billing API.
ms.author: v-thopra
ms.date: 06/11/2018
ms.topic: article
ms.prod: playfab
keywords: playfab, commerce, economy, iap, unity, android billing api
ms.localizationpriority: medium
---

# Getting started with PlayFab, Unity IAP, and Android

This tutorial shows you how to set up **In-App Purchasing** (**IAP**) using **PlayFab**, the **Unity** + **IAP** Service, and the **Android Billing API**.

## Before we start

Setting up **IAP** may be tedious if you are not quite sure how different services are supposed to integrate and cooperate.

The following image illustrates how the **Android Billing API** and **PlayFab** work together to provide a solid **IAP** experience for your **Client**.

![Android Billing - PlayFab - integration timeline](media/tutorials/android-billing-playfab-integration-timeline.png)  

Start by setting up your **Product IDs** and **Prices** via **Play Market**. Initially, all the products are *faceless* - they are just digital entities your **Player** is able to purchase - but they have no meaning to **PlayFab Players**.

To make those entities useful, we need to mirror them in the **PlayFab Item Catalogs**. This will turn faceless **Entities** into **Bundles**, **Containers**, and individual **Items**.

Each will have their own unique face, with:

- **Titles**
- **Descriptions**
- **Tags**
- **Types**
- **Images**
- **Behaviors**

All of these are linked to **Market Products** by sharing **IDs**.

The best way to access **Real Money Items** available for purchase is to use [GetCatalogItems](xref:titleid.playfabapi.com.client.title-widedatamanagement.getcatalogitems) and [GetStoreItems](xref:titleid.playfabapi.com.client.title-widedatamanagement.getstoreitems). These are the same **API** methods that are used by free-currency stores, so the process should be familiar.

The **ID** of the **Item** is the link between **PlayFab** and any external **IAP** system. So we pass the **Item ID** to the **IAP** service.

At this point, the purchase process starts. The **Player** interacts with the **IAP** interface and, if purchase is successful, you obtain a receipt.

**PlayFab** is then able to validate the receipt and register the purchase, granting the **PlayFab Player** the **Items** that they purchased.

This is a rough idea of how **IAP** integration works, and the following example shows most of it in action.

## Setting up a client application

This section shows you how to configure a very primitive application to test **IAP** using **PlayFab**, **UnityIAP**, and the **Android Billing API**.

Prerequisites:

- A **Unity** project.
- The **PlayFab SDK** imported and configured to work with your **Title**.

Our first step is setting up **UnityIAP**:

- Navigate to **Services (1)**.
- Make sure the **Services** tab is selected **(2)**.
- Select your **Unity Services** profile or organization **(3)**.
- Select the **Create** button **(4)**.

![Setting up UnityIAP service](media/tutorials/setting-up-unity-iap-service.png)  

- Next, navigate to the **In-App Purchasing (IAP)** service **(1)**:

![Navigate to UnityIAP service](media/tutorials/navigate-to-unity-iap-service.png)  

- Make sure to enable the **Service**, by setting the **Simplify cross-platform IAP** toggle **(1)**.

- Then select the **Continue** button **(2)**.

![Enable UnityIAP service](media/tutorials/enable-unity-iap-service.png)  

A page with a list of plugins will appear.

- Select the **Import** button **(1)**.

![UnityIAP service - Import plugins](media/tutorials/import-plugins-unity-iap.png)  

Continue the **Unity** install and import procedure up to the point where it has imported all the plugins.

- Verify that the plugins are in place **(1)**.
- Then create a new script **(2)** called **AndroidIAPExample.cs**.

![UnityIAP Create new script](media/tutorials/create-new-script-unity-iap.png)  

**AndroidIAPExample.cs** will contain the code shown below (please refer to the code comments for further explanation).

```csharp
using PlayFab;
using PlayFab.ClientModels;
using PlayFab.Json;
using System;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Purchasing;

public class AndroidIAPExample : MonoBehaviour, IStoreListener {
    // Items list, configurable via inspector
    private List<CatalogItem> Catalog;

    // The Unity Purchasing system
    private static IStoreController m_StoreController;

    // Bootstrap the whole thing
    public void Start() {
        // Make PlayFab log in
        Login();
    }

    public void OnGUI() {
        // This line just scales the UI up for high-res devices
        // Comment it out if you find the UI too large.
        GUI.matrix = Matrix4x4.TRS(new Vector3(0, 0, 0), Quaternion.identity, new Vector3(3, 3, 3));

        // if we are not initialized, only draw a message 
        if (!IsInitialized) {
            GUILayout.Label("Initializing IAP and logging in...");
            return;
        }

        // Draw menu to purchase items
        foreach (var item in Catalog) {
            if (GUILayout.Button("Buy " + item.DisplayName)) {
                // On button click buy a product
                BuyProductID(item.ItemId);
            }
        }
    }

    // This is invoked manually on Start to initiate login ops
    private void Login() {
        // Login with Android ID
        PlayFabClientAPI.LoginWithAndroidDeviceID(new LoginWithAndroidDeviceIDRequest() {
            CreateAccount = true,
            AndroidDeviceId = SystemInfo.deviceUniqueIdentifier
        }, result => {
            Debug.Log("Logged in");
            // Refresh available items 
            RefreshIAPItems();
        }, error => Debug.LogError(error.GenerateErrorReport()));
    }

    private void RefreshIAPItems() {
        PlayFabClientAPI.GetCatalogItems(new GetCatalogItemsRequest(), result => {
            Catalog = result.Catalog;

            // Make UnityIAP initialize
            InitializePurchasing();
        }, error => Debug.LogError(error.GenerateErrorReport()));
    }

    // This is invoked manually on Start to initialize UnityIAP
    public void InitializePurchasing() {
        // If IAP is already initialized, return gently
        if (IsInitialized) return;

        // Create a builder for IAP service
        var builder = ConfigurationBuilder.Instance(StandardPurchasingModule.Instance(AppStore.GooglePlay));

        // Register each item from the catalog
        foreach (var item in Catalog) {
            builder.AddProduct(item.ItemId, ProductType.Consumable);
        }

        // Trigger IAP service initialization
        UnityPurchasing.Initialize(this, builder);
    }

    // We are initialized when StoreController and Extensions are set and we are logged in
    public bool IsInitialized {
        get {
            return m_StoreController != null && Catalog != null;
        }
    }

    // This is automatically invoked automatically when IAP service is initialized
    public void OnInitialized(IStoreController controller, IExtensionProvider extensions) {
        m_StoreController = controller;
    }

    // This is automatically invoked automatically when IAP service failed to initialized
    public void OnInitializeFailed(InitializationFailureReason error) {
        Debug.Log("OnInitializeFailed InitializationFailureReason:" + error);
    }

    // This is automatically invoked automatically when purchase failed
    public void OnPurchaseFailed(Product product, PurchaseFailureReason failureReason) {
        Debug.Log(string.Format("OnPurchaseFailed: FAIL. Product: '{0}', PurchaseFailureReason: {1}", product.definition.storeSpecificId, failureReason));
    }

    // This is invoked automatically when succesful purchase is ready to be processed
    public PurchaseProcessingResult ProcessPurchase(PurchaseEventArgs e) {
        // NOTE: this code does not account for purchases that were pending and are
        // delivered on application start.
        // Production code should account for such case:
        // More: https://docs.unity3d.com/ScriptReference/Purchasing.PurchaseProcessingResult.Pending.html

        if (!IsInitialized) {
            return PurchaseProcessingResult.Complete;
        }

        // Test edge case where product is unknown
        if (e.purchasedProduct == null) {
            Debug.LogWarning("Attempted to process purchasewith unknown product. Ignoring");
            return PurchaseProcessingResult.Complete;
        }

        // Test edge case where purchase has no receipt
        if (string.IsNullOrEmpty(e.purchasedProduct.receipt)) {
            Debug.LogWarning("Attempted to process purchase with no receipt: ignoring");
            return PurchaseProcessingResult.Complete;
        }

        Debug.Log("Processing transaction: " + e.purchasedProduct.transactionID);

        // Deserialize receipt
        var googleReceipt = GooglePurchase.FromJson(e.purchasedProduct.receipt);

        // Invoke receipt validation
        // This will not only validate a receipt, but will also grant player corresponding items
        // only if receipt is valid.
        PlayFabClientAPI.ValidateGooglePlayPurchase(new ValidateGooglePlayPurchaseRequest() {
            // Pass in currency code in ISO format
            CurrencyCode = e.purchasedProduct.metadata.isoCurrencyCode,
            // Convert and set Purchase price
            PurchasePrice = (uint)(e.purchasedProduct.metadata.localizedPrice * 100),
            // Pass in the receipt
            ReceiptJson = googleReceipt.PayloadData.json,
            // Pass in the signature
            Signature = googleReceipt.PayloadData.signature
        }, result => Debug.Log("Validation successful!"),
           error => Debug.Log("Validation failed: " + error.GenerateErrorReport())
        );

        return PurchaseProcessingResult.Complete;
    }

    // This is invvoked manually to initiate purchase
    void BuyProductID(string productId) {
        // If IAP service has not been initialized, fail hard
        if (!IsInitialized) throw new Exception("IAP Service is not initialized!");

        // Pass in the product id to initiate purchase
        m_StoreController.InitiatePurchase(productId);
    }
}

// The following classes are used to deserialize JSON results provided by IAP Service
// Please, note that Json fields are case-sensetive and should remain fields to support Unity Deserialization via JsonUtilities
public class JsonData {
    // Json Fields, ! Case-sensetive

    public string orderId;
    public string packageName;
    public string productId;
    public long purchaseTime;
    public int purchaseState;
    public string purchaseToken;
}

public class PayloadData {
    public JsonData JsonData;

    // Json Fields, ! Case-sensetive
    public string signature;
    public string json;

    public static PayloadData FromJson(string json) {
        var payload = JsonUtility.FromJson<PayloadData>(json);
        payload.JsonData = JsonUtility.FromJson<JsonData>(payload.json);
        return payload;
    }
}

public class GooglePurchase {
    public PayloadData PayloadData;

    // Json Fields, ! Case-sensetive
    public string Store;
    public string TransactionID;
    public string Payload;

    public static GooglePurchase FromJson(string json) {
        var purchase = JsonUtility.FromJson<GooglePurchase>(json);
        purchase.PayloadData = PayloadData.FromJson(purchase.Payload);
        return purchase;
    }
}
```

- Create a new game object called **Code (1)**.
- Add the **AndroidIAPExample** component to it **(2)** **(3)**.
- Make sure to save the scene **(4)**.  

![UnityIAP create example game object](media/tutorials/create-example-game-object-unity-iap.png)  

Finally, navigate to **Build Settings**.

- Verify that your scene has been added to the **Scenes In Build** area **(1)**.
- Make sure that the **Android** platform has been selected **(2)**.
- Move to the **Player Settings** area **(3)**.
- Assign your package name.

> [!NOTE]
> Make sure to come up with your *own* package name to avoid any **Play Market** collisions.

![UnityIAP add example game object](media/tutorials/add-example-game-object-unity-iap.png)  

Finally, build the application as usual and ensure you have an **APK** safe and sound.

We have no means to test it just yet.  We need to configure **Play Market** and **PlayFab** first, which is described in the following sections.

## Setting up a Play Market application for IAP

This section describes the specifics of how to enable **IAP** for your **PlayMarket** application.

> [!NOTE]
> Setting up the application itself is beyond the scope of this tutorial.  We assume you already *have* an application, and that is configured to publish at least **Alpha** releases.

![Enable PlayMarket Application](media/tutorials/enable-playmarket-application.png)  

Useful notes:

- Getting to that point will require you to have an **APK** uploaded. Please, use the **APK** we constructed in the previous section.
- When asked to upload the **APK**, you may upload it as an **Alpha** or **Beta** application to enable the **IAP** sandbox.
- Configuring **Content Rating** will include questions about how **IAP** is enabled in the application.
- **Play Market** does *not* allow **Publishers** to use or test **IAP** - so please, pick *another* **Google** account for testing purposes, and add it as a tester for your **Alpha/Beta** build.

Once you have the application build published:

- Select **In-app products** from the menu **(1)**.
- If you are asked for a merchant account, link or create one.
- Select the **Add New Product** button **(2)**.

![PlayMarket add new product](media/tutorials/playmarket-add-new-product.png)  

- On this screen, select **Managed Product (1)**.
- Give it a descriptive **Product ID (2)**.
- Select **Continue (3)**.

![PlayMarket add product ID](media/tutorials/playmarket-add-product-id.png)  

**Play Market** requires you to fill in a **Title (1)** and Description **(2)**. However, these are not much use in our case. We will grab **Data Item** data exclusively from the **PlayFab** service and only require **IDs** to match.

![PlayMarket add product title description](media/tutorials/playmarket-add-product-title-description.png)  

Scroll further and select the **Add a price** button **(1)**.

![PlayMarket add product price](media/tutorials/playmarket-add-product-price.png)  

- Enter a valid **price (1)** (notice how price is converted for each country independently).
- Select the  **Apply** button **(2)**.

![PlayMarket add product apply local prices](media/tutorials/playmarket-add-product-apply-local-prices.png)  

Finally, scroll back to the top of your screen, and change the status of the item to **Active (1)**.

![PlayMarket make product active](media/tutorials/playmarket-make-product-active.png)  

While this concludes configuring the **App**, we need a couple more tweaks:

- First, let us save the **Licensing Key** (this will come in handy for linking **PlayFab** with **Play Market**.
- Navigate to **Services & APIs** in the menu **(1)**.
- Then locate and save the **Base64** version of the **Key (2)**.

![PlayMarket save product licensing key](media/tutorials/playmarket-save-product-licensing-key.png)

The next step is enabling IAP testing. While sandbox is automatically enabled for Alpha and Beta builds, we need to set up accounts that are authorized to test the app:

- Navigate to **Home (1)**.
- Locate and select the **Account details** sub-tab **(2)**.
- Locate the **License Testing** area **(3)**.
- Verify that your test accounts are in the list **(4)** and that the **License Test Response** is set to **RESPOND_NORMALLY (5)**.
- Do *not* forget to apply the settings.

![PlayMarket enable IAP testing](media/tutorials/playmarket-enable-iap-testing.png)  

The **Play Market** side of the integration should be set up at this point.

## Setting up a PlayFab Title

Our last step is configuring a **PlayFab Title** to reflect our products and integrate with the **Google Billing API**.

- Select **Add-ons** from the menu **(1)**.
- Then select the **Google** add-on **(2)**.

![PlayFab open Google Add-on](../../authentication/platform-specific-authentication/media/tutorials/google-html5/open-google-add-on.png)  

- Fill in your **Package ID (1)**.
- Fill in the **Google App License Key (2)** that you acquired in the previous section.
- Commit your changes by selecting the **Install Google** button **(3)**.

![PlayFab install Google Add-on](media/tutorials/playfab-install-google-add-on.png)  

Our next step is reflecting our **Golden Sword Item** in **PlayFab**:

- Select **Economy** from the menu **(1)**.
- Verify that the **Catalogs** sub-tab **(2)** is selected.
- Selet the **New Catalog** button **(3)**.

![PlayFab open new Catalog](media/tutorials/playfab-open-new-catalog.png)  

- Provide a **Version Name** for the **Catalog (1)**.
- Select the **Save Catalog** button **(2)**.

![PlayFab save Catalog](media/tutorials/playfab-save-catalog.png)  

If the **Catalog** has *no items*, it is *automatically* removed. That's why any new **Catalog** comes with a pre-added item called **One**.

We can *always* create a *new* **Item**, but to keep things clean, let's modify the existing **One Item**.

- Select the entry **(1)** in the list.

![PlayFab open Catalog Item](media/tutorials/playfab-open-catalog-item.png)  

- Set the **Item ID (1)** to match exactly with the **ID** in **Play Market**.
- Next, give a **Display name (2)** and Description **(3)** to your item.

  > [!NOTE]
  > Keep in mind that this data has *nothing to do* with the **Play Market Item Title** and **Description** - it is *totally* independent.

- Assign a **Price (4)** to your item.

In this tutorial, IAP mainly refers to purchases for **Real Money**. That's why we use **RM** - special **Real Money** currency. The **PlayFab** amount is defined in US Cents.

- Select the **Save Item** button **(5)** to commit your changes.

![PlayFab Edit and Save Catalog Item](media/tutorials/playfab-edit-save-catalog-item.png)  

Observe your item in the **Item ID** list **(1)**.

![PlayFab Catalog Items list](media/tutorials/playfab-catalog-items-list.png)  

This concludes the setup for your PlayFab Title.

## Testing

For testing purposes, download the **App** using the **Alpha/Beta** release.

- Make sure to use a test account and a real **Android** device.
- Once you start the **App**, you should see **IAP** initialized and *one button* representing your item.
- Select that button.

![Test app - Buy Golden Sword button](media/tutorials/test-app-buy-golden-sword-button.png)  

The **IAP** purchase will be initiated. Follow the **Google Play** instruction up to the point where purchase is successful.

![Test app - Google Play - payment successful](media/tutorials/test-app-google-play-payment-successful.png)  

Finally, navigate to your **Title** in the **PlayFab Game Manager** dashboard and locate new events.

![Game Manager Dashboard new events](media/tutorials/game-manager-dashboard-new-events.png)  

This means that purchase was successfully provided, validated, and piped to the **PlayFab** ecosystem.

At this point you have successfully integrated **UnityIAP** and the **Android Billing API** into your **PlayFab** application.
