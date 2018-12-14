---
title: Using a Rule to Verify a Contact Email Address
author: v-thopra
description: Tutorial that describes how to create a rule that sends an verification email when a player changes their contact email address.
ms.author: v-thopra
ms.date: 26/10/2018
ms.topic: article
ms.prod: playfab
keywords: playfab, engagement, email, rules
ms.localizationpriority: medium
---

# Using a Rule to Verify a Contact Email Address

This tutorial walks you through the steps for creating a **Rule** that sends an **Verification email** when a **Player** changes their contact email address.

## Requirements

> [!IMPORTANT]
> This is an advanced tutorial. Please *make sure* that all of the **Requirements** have been met, or you will *not* be able to complete this tutorial.

- To send custom emails with email templates, you will need to have your own **SMTP** server with a **Username** and **Password**. Make sure that you have your own **SMTP** server before following our tutorial [Setting up an SMTP server with add-ons](../../engagement/emails/setting-up-an-smtp-server-with-add-ons.md).
> [!NOTE]
> You can use **Gmail** for testing, but with **Gmail** you are limited to 2,000 emails per day.

- Basic knowledge of how to create a **Player** will be necessary, since there will need to be **Players** with a **Username** and **Password** before calling **Account Recovery** logic. Refer to our [Getting Started with PlayFab](../../config/dev-test-live/getting-started-with-playfab.md) tutorial which will run you through the process of creating a **Player** for the **Title**.
- Read the [Game Manager quickstart](../../config/gamemanager/quickstart.md) if you are unfamiliar with the **Game Manager** as it is the place where email templates are created.
- Knowledge of how to work with **Player profiles** is required to confirm that emails will be necessary for checking that a contact email has been added to a **Player's** profile.

> [!NOTE]
> Please read up on how to get a **Player profile** in the [Getting Player Profiles](../../data/playerdata/getting-player-profiles.md) tutorial, and make sure that under the **Client Profile Options** on your **Title**, you allow **Contact email addresses**.
- Creating a **Rule** will be necessary in this tutorial.  It is a good idea to read up on how [Rules](../../automation/actions-rules/playstream-hooks-rules-conditions-and-actions.md) work.

## Step 1 - Create an Email Template

The first thing we will do is create an account recovery email template.

- Select **Content** in the left-hand menu.
- Select the **Email Templates** tab.
- Then select the **NEW EMAIL TEMPLATE** button.

![Game Manager - Content - Email Templates](media/tutorials/game-manager-content-email-template.png)  

Now add a new email template by filling in the fields as follows, leaving the **Error callback URL** empty:

- **Template name**: MyFirstEmailVerificationTemplate
- **Template type**: Email Verification
- **Email subject**: Verify your email
- **Email body**: (enter as below)

```html
<head></head>
<body><p> You recently registered a new email with us.</p>
<p>Please click <a href="$ConfirmationUrl$">here</a> confirm your email. Thanks!</p>
```

- **From Name**: The **Name** you want to show in the **From** field in the email.
- **From Email Address**:  The **Email Address** you want to show in the **From** field in the email. This must be an email domain that the **SMTP** server enables you to send emails from.

  > [!NOTE]
  > Some email servers, like **Gmail**, will ignore this field and will send from the account set up with the **SMTP** server.
  
- **Callback URL**:  https://www.example.com

### A few things to remember

- The `$ConfirmationUrl$` in the **Email body** generates a customized **URL** that when selected, tracks that a **User** has selected the **URL**, and then issues a redirect to the **Callback URL**. In this case, it is injected into an **anchor tag**.
- The **Callback URL** is the **URL** that **PlayFab** will redirect to after the **Player** selects the **Confirmation URL** link. It can be a static page that tells the **User** they were successful in confirming their email. In this case, we will redirect to https://www.example.com.

![Game Manager - Content - Email Templates - New Email Template](media/tutorials/game-manager-content-new-email-template-email-verification.png)  

After filling the form out, select the **SAVE EMAIL TEMPLATE** button, and you will be redirected back to the page containing the list of your email templates. Make a note of the **ID** of the email template as it will be used in Step 4.

![Game Manager - Content - Email Template ID](media/tutorials/game-manager-content-emailverification-template-id.png)  

## Step 2 - Create a rule to send an email when a contact email is updated

Next, we will create a **Rule** to send a **Verification email** every time a **Player** updates their contact email. In **Game Manager**:

- Select **Automation** from the left-hand menu.
- Then select the **Rules** tab.
- Select **NEW RULE**.
- Fill out the **Rule Name** of your **Rule** **VerifyUpdatedEmail**.
- From the **Event type** drop-down menu, pick **com.playfab.player_updated_contact_email**.
- Under **Actions**, select **+ADD ACTION**.

![Game Manager - Automation - New Rule](media/tutorials/game-manager-automation-new-rule-add-action.png)  

- Choose **Send email** from the **Type** drop-down menu.
- The **Email template**  should be populated by the template created in Step 1 **MyFirstEmailVerificationTemplate** (if it isn't, pick **MyFirstEmailVerificationTemplate** from the drop-down menu provided.

![Game Manager - Automation - New Rule](media/tutorials/game-manager-automation-new-rule-save-action.png)  

## Step 3 - Add a contact email to a Player

For this next step, you will need an existing **Player** account. If you don't already have a **Player** account, follow the instructions in [Getting Started with PlayFab](../../config/dev-test-live/getting-started-with-playfab.md).

We will add a **Contact email** to the **Player** using [AddOrUpdateContactEmail](xref:titleid.playfabapi.com.client.accountmanagement.addorupdatecontactemail).

> [!NOTE]
> A **Contact email** field on a **Player profile** is different from the **Login email** field on a **Player profile**, even though they may *both* contain the same email address. Any time you send email to the **Player**, it will *only* go to the **Contact email** address.

### C# Code Example

In the following example, we log in a **Player**, then add a **Contact email** using [AddOrUpdateContactEmail](xref:titleid.playfabapi.com.client.accountmanagement.addorupdatecontactemail). Make sure that the email address associated with the **Player** is one that you have access to.

```csharp
void AddContactEmailToPlayer()
{
    var loginReq = new LoginWithCustomIDRequest
    {
        CustomId = "SomeCustomID", // replace with your own Custom ID 
        CreateAccount = true // otherwise this will create an account with that ID
    }; 
    
    var emailAddress = "testaddress@example.com"; // Set this to your own email

    PlayFabClientAPI.LoginWithCustomID(loginReq, loginRes =>
    {
        Debug.Log("Successfully logged in player with PlayFabId: " + loginRes.PlayFabId);
        AddOrUpdateContactEmail(loginRes.PlayFabId, emailAddress);
    }, FailureCallback);
}

void AddOrUpdateContactEmail(string playFabId, string emailAddress)
{
    var request = new AddOrUpdateContactEmailRequest
    {
        PlayFabId = playFabId,
        EmailAddress = emailAddress
    };
    PlayFabClientAPI.AddOrUpdateContactEmail(request, result =>
    {
        Debug.Log("The player's account has been updated with a contact email");
    }, FailureCallback);
}

void FailureCallback(PlayFabError error)
{
    Debug.LogWarning("Something went wrong with your API call. Here's some debug information:");
    Debug.LogError(error.GenerateErrorReport());
}
```

## Step 4 - Confirm that the contact email was added to the Player's Profile

Next, confirm that the contact email was added to the **Player's profile**.

- Log into the **Game Manager**.
- Go to the **Players Profile** page.
- You should see a **Contact email** listed for that **Player**, with **Verification Status**: **Pending**.

> [!NOTE]
> The **Verification Status** could be **Unverified** if the **Verification email** was not yet sent out, but will be in the **Pending** state as soon as the email is sent.

![Game Manager - Player Profile - Contact email](media/tutorials/game-manager-player-profile-contact-email-verification-pending.png)  

You can also make a call to [GetPlayerProfile](xref:titleid.playfabapi.com.client.accountmanagement.getplayerprofile) with **ShowContactEmailAddresses** in the [PlayerProfileViewConstraints](xref:titleid.playfabapi.com.server.accountmanagement.getplayerprofile#playerprofileviewconstraints) set as **True** to show that the **Player** now has the contact email that we just added.


## Step 5 - Check that the email was sent

Finally, we can check that the **Account Recovery** email was sent.

- Go to the the **Player's PlayStream**.
- In **Game Manager** go to **Players** tab.
- **PlayStream** should show a **Sent email** event.

![Game Manager - Players - PlayStream - Sent email event](media/tutorials/game-manager-players-playstream-sent-email-event.png)  

Selecting the **Info** icon on the **Event** should show **JSON** similar to the example provided below.

```json
{
    "EventName": "sent_email",
    "EventNamespace": "com.playfab",
    "Source": "PlayFab",
    "EntityType": "player",
    "TitleId": "YourTitleId",
    "EventId": "a05625e48b1f4194bd08d1ff6a889cf8",
    "EntityId": "64647AA368D6448E",
    "SourceType": "BackEnd",
    "Timestamp": "2017-10-27T09:35:16.2946918Z",
    "History": null,
    "CustomTags": null,
    "Reserved": null,
    "emailTemplateId": "7D6438687903D4DC",
    "emailTemplateName": "MyFirstEmailVerificationTemplate",
    "emailTemplateType": "EmailVerification",
    "success": true,
    "emailName": "Primary"
}
```

- To verify that you actually received the email, go to the email of the **Player** you created in Step 3. There should be an email that looks similar to the one provided below.

![Verify your email - email](media/tutorials/verify-your-email-email.png)  

If you inspect the **URL** in that email, you will see that it looks something like the one shown below.

```html
https://a5f3.playfabapi.com/EmailConfirmation/Confirm/?token=2346241B7C277796&titleId=A5F3&templateId=38017AAE7F494AB3
```

When the **Player** selects that **URL**, three things happen:

1. **PlayFab** generates a new **PlayStream Event** called **auth_token_validated**. This is how you know that the **Player** selected that **URL** in the email.
   - You can use that **Event** to trigger actions, like granting coins or items to the **Player**.
2. Because this email template was the special **Email Verification** template, **PlayFab** will then mark the **Player** email as Verified.
3. **PlayFab** will return a redirect **URL** sending the **Player** to the callback **URL** website.
    - On this website you can show a static **Thanks for verifying your email** message, or something more elaborate. The redirect **URL** will look something like this: **https://www.example.com/?token=2346241B7C277796**.

    - Go ahead and select the **URL** found in the email.
    - You will be taken to the **example.com** website.
    - View your **Player Profile** using the **Game Manager**.
    - You will see that the **Verification status** has changed.

![Game Manager - Player Profile - Contact email](media/tutorials/game-manager-player-profile-contact-email-verification-confirmed.png)  

## Conclusion

So that's it for this tutorial. You've seen how to setup your **SMTP** server, create an email template, and create a **Rule** that sends an email to a **Player** verifying their email address.

If you have any questions or feedback on this tutorial, please email us at [devrel@playfab.com](mailto:devrel@playfab.com).
