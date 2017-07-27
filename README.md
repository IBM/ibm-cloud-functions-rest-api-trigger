# OpenWhisk Building Block - HTTP REST API Trigger
Create REST API mappings with Apache OpenWhisk on IBM Bluemix. This tutorial should take about 5 minutes to complete. After this, move on to more complex serverless applications such as those tagged [_openwhisk-hands-on-demo_](https://github.com/search?q=topic%3Aopenwhisk-hands-on-demo+org%3AIBM&type=Repositories).

![Sample Architecture](https://openwhisk-ui-prod.cdn.us-south.s-bluemix.net/openwhisk/ngow-public/img/getting-started-serverless-api.svg)

If you're not familiar with the OpenWhisk programming model [try the action, trigger, and rule sample first](https://github.com/IBM/openwhisk-action-trigger-rule). [You'll need a Bluemix account and the latest OpenWhisk command line tool](https://github.com/IBM/openwhisk-action-trigger-rule/blob/master/docs/OPENWHISK.md).

This example provides two REST endpoints for HTTP `POST` and `GET` methods that are mapped to corresponding OpenWhisk `create-cat` and `fetch-cat` actions.

1. [Create OpenWhisk actions](#1-create-openwhisk-actions)
2. [Create REST endpoints](#2-create-rest-endpoints)
4. [Clean up](#3-clean-up)

# 1. Create OpenWhisk actions
## Create an action to create a cat entity
Create a file named `create-cat.js`. This file will define an OpenWhisk action written as a JavaScript function. It checks for the required parameters(`name` and `color`) and returns a unique identifier for the cat, or an error if either parameter is missing.
> **Note**: This example is simplified, and does not connect to a backend datastore. For a more sophisticated example, check out this [REST API example](https://github.com/IBM/openwhisk-serverless-apis).

```javascript
function main(params) {

  return new Promise(function(resolve, reject) {

    if (!params.name) {
      reject({
        'error': 'name parameter not set.'
      });
    } else {
      resolve({
        id: 1
      });
    }

  });

}
```

## Create an action to return a cat entity
Create a file named `fetch-cat.js`. This file will define an OpenWhisk action written as a JavaScript function. It checks for the required parameter(`id`) and returns Tahoma, the tabby colored cat.
> **Note**: Again, for the purpose of this simplified demo we always return Tahoma the cat, rather than connecting to a backend datastore.

```javascript
function main(params) {

  return new Promise(function(resolve, reject) {

    if (!params.id) {
      reject({
        'error': 'id parameter not set.'
      });
    } else {
      resolve({
        id: params.id,
        name: 'Tahoma',
        color: 'Tabby'
      });
    }

  });

}
```

## Upload the actions
The next step will be to create OpenWhisk actions from the JavaScript functions that we just created. We also add the `--web true` flag, to annotate these actions as "Web Actions". This will be necessary later when we add REST endpoints.
```bash
wsk action create create-cat create-cat.js --web true
wsk action create fetch-cat fetch-cat.js --web true
```

## Unit test the actions
OpenWhisk actions are stateless code snippets that can be invoked explicitly or in response to an event. For right now, we will test our actions by explicitly invoking them. Later, we will trigger our actions in response to an HTTP request. Invoke the actions using the code below and pass the parameters using the `--param` command line argument.

```bash
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
Now that we have our OpenWhisk actions created, we will expose our OpenWhisk actions through the Bluemix API Gateway. To do this we will use: `wsk api create $BASE_PATH $API_PATH $API_VERB $ACTION `

This feature is part of the [Bluemix Native API Management](https://console.ng.bluemix.net/docs/openwhisk/openwhisk_apigateway.html#openwhisk_apigateway) service and currently supports very powerful API management features like security, rate limiting, and more. For now though we're just using the CLI to expose our action with a public REST endpoint.

```bash
# Send along credentials with the command or provide them interactively
wsk bluemix login --user $YOUR_BLUEMIX_USERNAME --password $YOUR_BLUEMIX_PASSWORD

# Exposes POST /v1/cat {"name": "Tahoma", "color": "Tabby"}
wsk api create -n "Cats API" /v1 /cat post create-cat

# Exposes /v1/cat?id=1
wsk api create /v1 /cat get fetch-cat
```
In both cases, the CLI will output the URL required to use the API. Make note of it for the next section.

## Test with `curl` HTTP requests
Take note of the API URL that is generated from the previous command. Send an http POST and GET request using `curl` to test the actions. Remember to send the required parameters in the body of the request for POST, or as path parameters for GET. OpenWhisk automatically forwards these parameters to the actions we created.

```bash
# POST /v1/cat {"name": "Tahoma", "color": "Tabby"}
curl -X POST -H 'Content-Type: application/json' -d '{"name":"Tahoma","color":"Tabby"}' $THE_URL_FROM_ABOVE

# GET /v1/cat?id=1
curl $THE_URL_FROM_ABOVE?id=1
```

# 3. Clean up
## Remove the API mappings and delete the actions

```bash
# Remove API base which removes all the mappings
wsk api delete /v1

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
