---
id: node
title: Node.js
---
## Getting started

### 1. Install *ConfigCat SDK*
*via NPM*
```bash
npm i configcat-node
```
### 2. Import package
```js
const configcat = require("configcat-node");
```

### 3. Create the *ConfigCat* client with your *API Key*
```js
let configCatClient = configcat.createClient("#YOUR-API-KEY#");
```
### 4. Get your setting value
```js
configCatClient.getValue("isMyAwesomeFeatureEnabled", false, (value) => {
    if(value) {
        do_the_new_thing();
    } else {
        do_the_old_thing();
    }
});
```
## Creating the *ConfigCat Client*
*ConfigCat Client* is responsible for:
- managing the communication between your application and ConfigCat servers.
- caching your setting values and feature flags.
- serving values quickly in a failsafe way.

`createClient()` returns a client with default options.

| Properties | Description                                                                                                        |
| ---------- | ------------------------------------------------------------------------------------------------------------------ |
| `apiKey`   | **REQUIRED.** API Key to access your feature flags and configurations. Get it from *ConfigCat Management Console*. |

`createClientWithAutoPoll()`, `createClientWithLazyLoad()`, `createClientWithManualPoll()`  
Creating the client is different for each polling mode.
[See below](#polling-modes) for details.

> We strongly recommend using the *ConfigCat Client* as a Singleton object in your application.

## Anatomy of `getValue()`

| Parameters     | Description                                                                                                  |
| -------------- | ------------------------------------------------------------------------------------------------------------ |
| `key`          | **REQUIRED.** Setting-specific key. Set in *ConfigCat Management Console* for each setting.                  |
| `defaultValue` | **REQUIRED.** This value will be returned in case of an error.                                               |
| `callback`     | **REQUIRED.** Called with the actual setting value.                                                          |
| `user`         | Optional, *User Object*. Essential when using Targeting. [Read more about Targeting.](../../advanced/targeting) |
```js
configCatClient.getValue(
    "keyOfMySetting", // Setting Key
    false, // Default value
    (value) => { console.log(value) }, // Callback function
    { identifier : "435170f4-8a8b-4b67-a723-505ac7cdea92" } // Optional User Object
);
```
### User Object 
``` javascript
let userObject = {
    identifier : "435170f4-8a8b-4b67-a723-505ac7cdea92"
};   
```
``` javascript
let userObject = {
    identifier : "john@example.com"
};   
```

| Parameters   | Description                                                                                                                     |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------- |
| `identifier` | **REQUIRED.** Unique identifier of a user in your application. Can be any `string` value, even an email address.                |
| `email`      | Optional parameter for easier targeting rule definitions.                                                                       |
| `country`    | Optional parameter for easier targeting rule definitions.                                                                       |
| `custom`     | Optional dictionary for custom attributes of a user for advanced targeting rule definitions. e.g. User role, Subscription type. |

``` javascript
let userObject = {
    identifier : "435170f4-8a8b-4b67-a723-505ac7cdea92",
    email : "john@example.com",
    country : "United Kingdom",
    custom : {
        "SubscriptionType": "Pro",
        "UserRole": "Admin"
    }
};
```

## Polling Modes
The *ConfigCat SDK* supports 3 different polling mechanisms to acquire the setting values from *ConfigCat*. After latest setting values are downloaded, they are stored in the internal cache then all requests are served from there. With the following polling modes, you can customize the SDK to best fit to your application's lifecycle.

### Auto polling (default)
The *ConfigCat SDK* downloads the latest values and stores them automatically every 60 seconds.

Use the `pollIntervalSeconds` option parameter to change the polling interval.
```js
let configCatClient = configcat.createClientWithAutoPoll("#YOUR-API-KEY#", { pollIntervalSeconds: 95 });
```
Adding a callback to `configChanged` option parameter will get you notified about changes.
```js
let configCatClient = configcat.createClientWithAutoPoll("#YOUR-API-KEY#", { configChanged: () => {
    console.log("Your config has been changed!");
}});
```
Available options:

| Option Parameter      | Description                                            | Default        |
| --------------------- | ------------------------------------------------------ | -------------- |
| `pollIntervalSeconds` | Polling interval. Range: `1 - Number.MAX_SAFE_INTEGER` | 60             |
| `configChanged`       | Callback to get notified about changes.                | -              |
| `logger`              | Custom logger. See below for details.                  | Console logger |

### Lazy loading
When calling `getValue()` the *ConfigCat SDK* downloads the latest setting values if they are not present or expired in the cache. In this case the `callback` will be called after the cache is updated.

Use `cacheTimeToLiveSeconds` option parameter to set cache lifetime.
```js
let configCatClient = configcat.createClientWithLazyLoad("#YOUR-API-KEY#", { cacheTimeToLiveSeconds: 600 });
```
Available options:

| Option Parameter         | Description                                     | Default        |
| ------------------------ | ----------------------------------------------- | -------------- |
| `cacheTimeToLiveSeconds` | Cache TTL. Range: `1 - Number.MAX_SAFE_INTEGER` | 60             |
| `logger`                 | Custom logger. See below for details.           | Console logger |

### Manual polling
Manual polling gives you full control over when the setting values are downloaded. *ConfigCat SDK* will not update them automatically. Calling `forceRefresh()` is your application's responsibility.

```js
let configCatClient = configcat.createClientWithManualPoll("#YOUR-API-KEY#");

configCatClient.forceRefresh(() =>{

    configCatClient.getValue("key", "my default value", (value)=>{
        
        console.log(value);
    });
});
```
Available options:

| Option Parameter | Description                           | Default        |
| ---------------- | ------------------------------------- | -------------- |
| `logger`         | Custom logger. See below for details. | Console logger |

> `getValue()` returns `defaultValue` if the cache is empty. Call `forceRefresh()` to update the cache.

```js
let configCatClient = configcat.createClientWithManualPoll("#YOUR-API-KEY#");

configCatClient.getValue("key", "my default value", (value)=>{

    console.log(value) // console: "my default value"
});

configCatClient.forceRefresh(() =>{

    configCatClient.getValue("key", "my default value", (value)=>{

        console.log(value); // console: "value from server"
    });
});
```

## Logging
To customize logging create a logger instance and add it to Options object when creating the ConfigCat client. 2 log levels are supported: `log` and `error`.
```js
let customLogger = {
    log: (message) => {
        console.log('ConfigCat log: ' + message);
    },
    error: (message) => {
        console.error('ConfigCat log: ' + message);
    }
};

let configCatClient = configcat.createClientWithManualPoll("#YOUR-API-KEY#", { logger: customLogger });
```

## CDN base url (forward proxy, dedicated subscription)
You can customize your CDN path in the SDK with `baseUrl` property in the `options` paramter.

```js
let configCatClient = configcat.createClientWithManualPoll("#YOUR-API-KEY#", { baseUrl: "https://myCDN.configcat.com" });
```

## Sample Application
<a href="https://github.com/configcat/node-sdk/blob/master/samples/console" target="_blank">Sample Console App</a>

## Look under the hood
* <a href="https://github.com/configcat/node-sdk" target="_blank">ConfigCat's Node.js SDK on GitHub</a>
* <a href="https://www.npmjs.com/package/configcat-node" target="_blank">ConfigCat's Node.js SDK in NPM</a>