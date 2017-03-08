# OpenWhisk Building Block - HTTP REST Trigger
[Create REST API mappings](https://github.com/IBM/openwhisk-serverless-apis/wiki) with Apache OpenWhisk on IBM Bluemix. This tutorial takes less than 5 minutes to complete. After this, move on to more complex serverless applications such as those tagged [_openwhisk-hands-on-demo_](https://github.com/search?q=topic%3Aopenwhisk-hands-on-demo+org%3AIBM&type=Repositories).

If you're not familiar with the OpenWhisk programming model [try the action, trigger, and rule sample first](https://github.com/IBM/openwhisk-action-trigger-rule). [You'll need a Bluemix account and the latest OpenWhisk command line tool](docs/OPENWHISK.md).

This example provides two REST endpoints, HTTP `POST` and `GET` methods that are mapped to corresponding OpenWhisk `create-cat` and `fetch-cat` actions.

1. [Create OpenWhisk actions](#1-create-openwhisk-actions)
2. [Create REST endpoints](#2-create-rest-endpoints)
4. [Clean up](#3-clean-up)

# 1. Create OpenWhisk actions
## Create a file named `create-cat.js`
```javascript
function main(params) {

  return new Promise(function(resolve, reject) {
    console.log(params.name);
    console.log(params.color);

    if (!params.name) {
      console.error('name parameter not set.');
      reject({
        'error': 'name parameter not set.'
      });
      return;
    } else {
      resolve({
        success: true,
        id: '1'
      });
      return;
    }

}
```

## Create a file named `fetch-cat.js`
```javascript
function main(params) {

  return new Promise(function(resolve, reject) {
    console.log(params.id);

    if (!params.id) {
      console.error('id parameter not set.');
      reject({
        'error': 'id parameter not set.'
      });
      return;
    } else {
      resolve({
        success: true,
        id: '1',
        name: 'Tahoma',
        color: 'Tabby'
      });
      return;
    }

}
```

## Upload actions and test

```bash
# Create
wsk action create create-cat create-cat.js
wsk action create fetch-cat fetch-cat.js

# Test
wsk action invoke \
  --blocking \
  --param name Tahoma \
  --param color Tabby \
  create-cat

wsk action invoke \
  --blocking \
  --param id 1 \
  fetch-cat
```

> **Note**: If you see any error messages, refer to the [Troubleshooting](#troubleshooting) section below.

# 2. Create REST endpoints
## Create POST and GET REST mappings for `/v1/cat` endpoint

```bash
# POST /v1/cat {"name": "Tahoma", "color": "Tabby"}
wsk api-experimental create -n "Cats API" /v1 /cat post create-cat

# GET /v1/cat?id=1
wsk api-experimental create /v1 /cat get fetch-cat
```

## Test with `curl` HTTP requests
```bash
# Get the REST URL base
export CAT_API_URL=`wsk api-experimental list | tail -1 | awk '{print $5}'`

# POST /v1/cat {"name": "Tahoma", "color": "Tabby"}
curl -X POST -d "{\"name\":\"Tahoma\",\"color\":\"Tabby\"}" $CAT_API_URL

# GET /v1/cat?id=1
curl ${CAT_API_URL}?id=1
```

# 3. Clean up
## Remove the API mappings and delete the actions

```bash
# Remove API base
wsk api-experimental delete /v1

# Remove actions
wsk action delete create-cat
wsk action delete fetch-cat
```

# Troubleshooting
Check for errors first in the OpenWhisk activation log. Tail the log on the command line with `wsk activation poll` or drill into details visually with the [monitoring console on Bluemix](https://console.ng.bluemix.net/openwhisk/dashboard).

If the error is not immediately obvious, make sure you have the [latest version of the `wsk` CLI installed](https://console.ng.bluemix.net/openwhisk/learn/cli). If it's older than a few weeks, download an update.
```bash
wsk property get --cliversion
```

# License
[Apache 2.0](LICENSE.txt)
