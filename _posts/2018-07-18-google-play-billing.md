---
layout: post
title: "Adding the Google Play Billing Library to your Application"
excerpt: ""
tags: 
  - android
  - java
  - medium
  - googleplay
  - billing
---

[#Original Medium Article](http://github.com)

Adding the Google Play Billing Library to your Application
While developing my most recent application, I wanted to find a way to monetize my application. I ultimately decided against making my app a paid app, and instead opted to go down the free application with advertisements route. The problem with this is, not all users will want to see advertisements, so I decided to offer the opportunity to remove ads with an in-app purchase. As someone who has never before added in-app purchases to my app, I decided to do some research and what I discovered is — there are very few tutorials out there that explain how to use the new Google Play Billing library with Java which was released towards the end of 2017. Most of the tutorials out there focus on using the old “In-App Purchases” method of using a bunch of util classes taken from Android’s TrivialDrive project. With a couple quick google searches I think you’ll find this old method is far more complex than the newer Play Billing Library. Let’s take a look at how much easier the new Google Play Library is to implement.

Using the Google Play Billing Library to Remove Ads with In-App Purchase:
In this example, I’ll discuss how to remove ads from your application with an in-app purchase (and track purchases using SharedPreferences). I won’t dive into too much detail regarding subscriptions or consumable products (products that can be consumed/purchased more than once), but I think you’ll find the implementation of those methods to be relatively straight forward after understanding the basics of the Play Billing Library. For more info about the Play Billing Library, please reference the Android developer website: (https://developer.android.com/google/play/billing/billing_overview)

Prior to reading this example, you will want to access your application from your developer console and add an “In-App Product” for advertisement removal (Under Application Settings > Store Presence > In-App Products > Create Managed Product).

1. Add the billing dependency to your Gradle File
implementation 'com.android.billingclient:billing:1.1'
2. Determine which activity you will use to manage In-App Purchases
It can be helpful to have a particular activity that is designated for in-app purchases. In my application, I use the InAppBillingActivity for in-app purchases. This activity simply contains one button which the user can click to purchase “Ad Removal”. Of course, this activity can contain all of the buttons/tools you need to manage all of your in-app purchases.

3. Implement the PurchasesUpdatedListener and set up your class

Your billing activity should implement a PurchasesUpdatedListener and implement the required methods associated with it (e.g. OnPurchaseUpdated). This listener will notify you when user purchases have changed.

Define your global variables for your class. You should have a string that exactly matches your product ID from the “In-app products” section of the developer console. Your product must also be Active in your console. I created a “Remove Advertisements” product for my app. I created the string variable ITEM_SKU_ADREMOVAL for this product ID.


You should also have a global BillingClient variable that you will use for your billing client.


InAppBillingActivity
4. Create a Billing Client

You will need to create a new billing client and set a PurchasesUpdatedListener on it (which will be the one you implemented in your activity in step 3).

After creating the Billing Client, you can use the startConnection method to start your connection. You will need to pass in a new BillingClientStateListener in your startConnection method. You can use this listener to specify what you want to do after you are connected (or disconnected) from the Billing Service.

In the onBillingSetupFinished method, you can specify what you want to occur after you are connected (this is a good place to query purchase info — see step 5 below).

In the onBillingServiceDisconnected method you can implement your own retry policy to restart the connection if you wish.


Adding BillingClient
5. Query Purchases

To query purchases, you will create a new List and add your particular Product ID strings to it. Then you will use a SkuDetailsParams.Builder to set your list as a query parameter. You will also set the type of product you are querying as a parameter (in this case we use SkuType.INAPP as the parameter since we’re using a one-time product. For subscriptions, use SkuType.SUBS).
You can run this code in the onBillingSetupFinished method above if you want to query purchase data immediately after you are connected to the BillingClient. Querying purchase data can be helpful if you need any particular information about the price of product, description, type, etc…


Query Purchases
6. Make a Purchase when User Clicks the “Purchase” button:
When a user clicks on your purchase button for your product, you will want to build new BillingFlowParams for your particular product (product ID and product type). Then you will want to launch the billing flow using your billing client and you will get a response code depending on the outcome of the purchase:


Make a Purchase
7. Handle the Purchase:
The onPurchasesUpdated method that you overrode (see Step 3) when you implemented the PurchasesUpdatedListener will be used to handle your response codes. If your response code is BillingResponse.OK, then the purchase went through and you can handle it (I use the handlePurchase method for this). If the response code is USER_CANCELED then the user canceled their purchase. If the response code is ITEM_ALREADY_OWNED, then the user has already purchased the item so you can handle the purchase accordingly. There are plenty of other codes that you can handle if you have a particular use case for it.

In my app, I am using a SharedPreference boolean value to keep track of whether or not the adFree product was purchased by the user. If the response code is BillingResponse.OK or ITEM_ALREADY_OWNED, I am setting this boolean value in my app and referencing it wherever I show ads to determine whether or not to display the ads.


Handle a Purchase Response Code
In my handlePurchase method, if the purchase SKU is equal to my particular product ID then I disable the purchase button and I set the text on the button to indicate that the product has already been purchased (along with storing the adFree boolean value in my SharedPreferences).


Handle a Successful Purchase
8. Removing Ads in App Using SharedPreferences
As mentioned earlier, I use a SharedPreference Boolean value to store whether or not my adFree product was purchased. If you decide to go this route as well, then you can get this adFree preference boolean from your SharedPreferences whenever you need to reference it. Throughout your app, you can set your adViews to INVISIBLE or prevent other ads (i.e. Interstitial ads) from launching based on the value of this Boolean.


Use Shared Preferences Boolean to Track if adFree Product was Purchased

Example of Removing an AdView Based on adFree Preference Boolean
Examples of Play Billing Library:
All of this code is used in my open source Streakr app if you want to check out how it works. The full InAppBillingActivity can be seen there:
https://github.com/patpatchpatrick/Streakr

Android/Google also has a V2 of their Trivial Drive app where they show how use the play billing library in a sample application that is worth checking out:

https://github.com/googlesamples/android-play-billing/tree/master/TrivialDrive_v2