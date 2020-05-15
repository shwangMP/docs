---
title: Event
---

Adobe Target lets you optimize every experience every time with A/B testing. Learn more about Adobe Target [here](https://www.adobe.com/marketing/target.html), and see below for how mParticle's integration works.

## Adobe Traget Overview and Prerequisites

Currently mParticle supports Adobe Target v1.8. Note that you must host your instance of Adobe Target's at.js file and load it before you load mParticle's dynamic loading script. This is because Adobe Target's at.js file is customized to each client. Additionally, there are optional global functions such as `targetPageParams`, `targetPageParamsAll`, and `targetGlobalSettings` which can all be edited and included in the at.js script when edited in Adobe Target's Dashboard at `Setup > Implementation > Edit at.js Settings > Code Settings > Library Header`

Once you have set up your global functions and hosted your at.js file, see below for an example of how to load both at.js and mParticle's web sdk in your index.html file:

```
// index.html

// First load at.js
<script src="https://www.examplel.com/at.js"></script> // replace the URL in the script src here with the path to your hosted at.js file

// Second, load mParticle's web SDK
// This is identical to the snippet at https://docs.mparticle.com/developers/sdk/web/getting-started#add-the-sdk-snippet

<script type="text/javascript">
  window.mParticle = {
    config: {
      isDevelopmentMode: true //switch to false (or remove) for production
    }
  };

  (
    function(t){window.mParticle=window.mParticle||{};window.mParticle.EventType={Unknown:0,Navigation:1,Location:2,Search:3,Transaction:4,UserContent:5,UserPreference:6,Social:7,Other:8};window.mParticle.eCommerce={Cart:{}};window.mParticle.Identity={};window.mParticle.config=window.mParticle.config||{};window.mParticle.config.rq=[];window.mParticle.config.snippetVersion=2.2;window.mParticle.ready=function(t){window.mParticle.config.rq.push(t)};var e=["endSession","logError","logBaseEvent","logEvent","logForm","logLink","logPageView","setSessionAttribute","setAppName","setAppVersion","setOptOut","setPosition","startNewSession","startTrackingLocation","stopTrackingLocation"];var o=["setCurrencyCode","logCheckout"];var i=["identify","login","logout","modify"];e.forEach(function(t){window.mParticle[t]=n(t)});o.forEach(function(t){window.mParticle.eCommerce[t]=n(t,"eCommerce")});i.forEach(function(t){window.mParticle.Identity[t]=n(t,"Identity")});function n(e,o){return function(){if(o){e=o+"."+e}var t=Array.prototype.slice.call(arguments);t.unshift(e);window.mParticle.config.rq.push(t)}}var mp=document.createElement("script");mp.type="text/javascript";mp.async=true;mp.src=("https:"==document.location.protocol?"https://jssdkcdns":"http://jssdkcdn")+".mparticle.com/js/v2/"+t+"/mparticle.js";var c=document.getElementsByTagName("script")[0];c.parentNode.insertBefore(mp,c)}
  )("REPLACE WITH API KEY");
</script>



```

## Supported Features

| Adobe Target Feature Name       | mParticle Supported? | Additional Comments                                                                                       |
| ------------------------------- | -------------------- | --------------------------------------------------------------------------------------------------------- |
| getOffer                        | Yes                  | ---                                                                                                       |
| getOffer success/error handlers | Yes                  | ---                                                                                                       |
| applyOffer                      | Yes<super>\*</super> | applyOffer cannot be invoked directly. Instead it is called immediately upon success of a `getOffer` call |
| trackEvent                      | Yes                  | ----                                                                                                      |

<super>\*</super>`applyOffer` is automatically called with the received offer from a `getOffer` call. You cannot call `applyOffer` directly via any mParticle call.

### Event Tracking

All event mappings to Adobe Target requires at a minimum a custom flag of 'ADOBETARGET.MBOX'.
For example, to perform a basic track event call:

```
const customAttrs = {foo: 'bar'};
const customFlags = {
    'ADOBETARGET.MBOX': 'fooBox'
};

mParticle.logEvent('custom event', mParticle.EventType.Other, customAttrs, customFlags)
```

will map to the following:

```
adobe.target.trackEvent({
    mbox: 'fooBox',
    params: {
        foo: 'bar'
    }
})
```

### Calling Adobe Target's getOffer Method

The following custom flags call will map to `adobe.target.getOffer`.

| Custom Flag          | type     | optional/required | Description                                                                                                  |
| -------------------- | -------- | ----------------- | ------------------------------------------------------------------------------------------------------------ |
| ADOBETARGET.MBOX     | string   | required          | mbox name passed to getOffer                                                                                 |
| ADOBETARGET.GETOFFER | boolean  | required          | pass `true` in order to initiate adobe.target.getOffer                                                       |
| ADOBETARGET.SUCCESS  | function | optional          | additional logic to process inside of getOffer's success handler. Has a single argument which is the `offer` |
| ADOBETARGET.ERROR    | function | optional          | additional logic to process inside of getOffer's error handler. Has 2 arguments, the `status`, and `error`   |

```
const customAttrs = {foo: 'bar'};
const customFlags = {
    'ADOBETARGET.MBOX': 'fooBox',
    'ADOBETARGET.GETOFFER': true,
    'ADOBETARGET.SUCCESS': successHandlerFunction,
    'ADOBETARGET.ERROR': errorHandlerFunction,
};

mParticle.logEvent('[Your Event Name]', mParticle.EventType.Other, customAttrs, customFlags)
```

will map to the following:

```
window.adobe.target.getOffer({
    mbox: MBOXNAME,
    params: customAttrs,
    success: function(offer) {
        window.adobe.target.applyOffer(offer);
        successHandler(offer); // optional
    },
    error: function(status, error) {
        errorHandler(status, error); //optional
    },
});
```

<!-- The following methods are used to track commerce events:

| Adobe Mobile Services SDK | mParticle SDK      |
| ------------------------- | ------------------ |
| trackAction:data:         | logEvent:eventData |

Product events are mapped between Adobe and mParticle as follows:

| Adobe Product Events | mParticle Commerce event |
| -------------------- | ------------------------ |
| prodView             | Product.DETAIL           |
| scCart               | N/A                      |
| scOpen               | N/A                      |
| scAdd                | Product.ADD_TO_CART      |
| scRemove             | Product.REMOVE_FROM_CART |
| scCheckout           | Product.CHECKOUT         |
| purchase             | Product.PURCHASE         |

### LTV Tracking

| Adobe Mobile Services SDK  | mParticle SDK  | Additional Comments                                                                                 |
| -------------------------- | -------------- | --------------------------------------------------------------------------------------------------- |
| trackLifetimeValueIncrease | logLTVIncrease | mParticle SDK has a "MPProduct" object to help with logging transactions that lead to LTV increase. |

The mParticle SDK can calculate the lifetime value of customers once the mParticle SDK has been added to an app.

### Opt-in/Opt-out Management

| Adobe Mobile Services SDK                                                                                                   | mParticle SDK                           |
| --------------------------------------------------------------------------------------------------------------------------- | --------------------------------------- |
| Set default value via privacyStatus in a SDK config file (also has setPrivacyStatus method to change the status in the app) | `OptOut` in iOS, `setOptOut` in Android |

mParticle assumes that users have opt-in status by default, whereas Adobe’s SDK supports setting the default status in an SDK config file per app.

### Location Tracking

| Adobe Mobile Services SDK | mParticle SDK                                                       |
| ------------------------- | ------------------------------------------------------------------- |
| trackLocation:data:       | `beginLocationTracking` in iOS, `enableLocationTracking` in Android |

If the _Generate Location Message_ setting is enabled, mParticle will forward the location data (if available) of each event to Adobe.

### Offline Tracking

| Adobe Mobile Services SDK        | mParticle SDK      |
| -------------------------------- | ------------------ |
| offlineEnabled setting in config | enabled by default |

The mParticle SDK always collects offline data and sends that data to the mParticle SDK server.

If the _Offline Tracking Enabled_ setting is enabled, mParticle will includes a "ts" parameter that represents the timestamp (in seconds) of the event.

### User Identification

| Adobe Mobile Services SDK | mParticle SDK                          |
| ------------------------- | -------------------------------------- |
| setUserIdentifier         | setUserIdentity (with CustomerId type) |

If the _Use Customer ID_ setting is enabled, and the User Identity Customer ID has been set, mParticle will forward customerId as a custom user identifier (the "vid" parameter in Adobe). Adobe Mobile Services’ 4.x version SDK also has a tracking identifier (aid) that Adobe uses to identify each unique device per app. mParticle generates a random GUID for each device per app and sends it to Adobe as the "aid". Similar to Adobe's SDK, this ID is preserved between app upgrades, is saved and restored during the standard application backup process, and is removed on uninstall.

**Notes for Existing Adobe Customers**

If an app has already integrated with the Adobe Marketing Cloud before using mParticle, then "vid" and "aid" are likely already stored on consumers' devices. The mParticle SDK checks if there is existing data stored on a device from Adobe's SDK; if there is it will get the IDs from a device and send them to the mParticle SDK server. mParticle will then use those IDs when sending data to ensure a seamless transition.

### App and Device Attributes

mParticle supports forwarding selected App and Device attributes to Adobe as context variables. Add the values you want to forward as a comma-separated list in the Connection Settings panel under **App and Device Attributes**. Accepted values are:

- mp_GoogleAdvertisingIdentifier
- mp_VendorIdentifier
- mp_AdvertisingIdentifier
- mp_LocaleCountry
- mp_LocaleLanguage
- mp_DeviceManufacturer
- mp_DeviceName
- mp_AppName
- mp_PackageName
- mp_AppVersion

These values are case sensitive and must be entered exactly.

## Configuration Settings

| Setting Name             | Data Type | Default Value | Description                                                                                                                                                              |
| ------------------------ | --------- | ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Report Suite IDs         | `string`  |               | The report suite ID from Adobe settings page. Multiple IDs can be entered, separated by commas                                                                           |
| Tracking Server          | `string`  |               | The URL of the Adobe tracking server                                                                                                                                     |
| Media Tracking Server    | `string`  |               | Web Only. The URL of the Adobe media tracking server. When this is filled in, Adobe Heartbeat is loaded. Leave this box blank if you do not want to load Adobe Heartbeat |
| Report Suite IDs         | `string`  | <unset>       | The report suite ID from Adobe settings page. Multiple IDs can be entered, separated by commas                                                                           |
| Tracking Server          | `string`  | <unset>       | The URL of the Adobe tracking server                                                                                                                                     |
| Character Set            | `string`  | UTF-8         | The character set used to display data in the Adobe interface                                                                                                            |
| Timestamp Enabled        | `bool`    | True          | If enabled, the timestamp will be included in messages sent to Adobe                                                                                                     |
| Send Messages Securely   | `bool`    | True          | If enabled, mParticle will forward all data to Adobe using SSL                                                                                                           |
| Offline Tracking Enabled | `bool`    | True          | If enabled, any messages that are received when the device is offline will be forwarded                                                                                  |

## Connection Settings

| Setting Name              | Data Type      | Default Value | Platform | Description                                                                                                       |
| ------------------------- | -------------- | ------------- | -------- | ----------------------------------------------------------------------------------------------------------------- |
| Use Customer ID           | `bool`         | True          | All      | If enabled, Customer ID will be forwarded if it exists                                                            |
| Include User Attributes   | `bool`         | False         | All      | If enabled, all user attributes will be included in the context data for each event                               |
| Generate Location Message | `bool`         | True          | All      | If enabled, location data will be forwarded if available                                                          |
| Context Variables         | `Custom Field` | <unset>       | All      | Mapping of your application's event attributes to Adobe context variables                                         |
| Product Incrementors      | `Custom Field` | <unset>       | All      | Mapping of your application's custom event names to Adobe product incrementor event numbers                       |
| Product Merchandisings    | `Custom Field` | <unset>       | All      | Mapping of your application's event attributes to Adobe product merchandising                                     |
| Events                    | `Custom Field` | <unset>       | All      | Mapping of your application's custom event names to Adobe event numbers                                           |
| Props                     | `Custom Field` | <unset>       | All      | Mapping of your application's custom event attributes to Adobe props                                              |
| eVars                     | `Custom Field` | <unset>       | All      | Mapping of your application's custom event attributes to Adobe eVars                                              |
| Hier Variables            | `Custom Field` | <unset>       | All      | Mapping of your application's screen view attributes to Adobe hier variables                                      |
| App and Device Attributes | `string`       | <unset>       | All      | A [comma separated list of app and device attributes](#app-and-device-attributes) to forward as context variables | -->
