no_breadcrumb:true

# Webhook Notifications JavaScript Quick Start

Welcome to the RingCentral Platform. RingCentral is the leading unified communications platform. From one system developers can integrate with, or build products around all the ways people communicate today: SMS, voice, fax, chat and meetings.

In this Quick Start, we are going to help you create a Webhook push notifications app using our Push Notifications API, which allows your application receiving notifications on instant SMS message events. Let's get started.

## Create an App

The first thing we need to do is create an app in the RingCentral Developer Portal. This can be done quickly by clicking the "Create Notifications App" button below. Just click the button, enter a name and description if you choose, and click the "Create" button. If you do not yet have a RingCentral account, you will be prompted to create one.

<a target="_new" href="https://developer.ringcentral.com/new-app?name=Webhook+Notifications+Quick+Start+App&desc=A+simple+app+to+demo+creating+an+SMS+Notification+RingCentral&public=false&type=ServerOther&carriers=7710,7310,3420&permissions=SubscriptionWebhook,SMS&redirectUri=&utm_source=devguide&utm_medium=button&utm_campaign=quickstart" class="btn btn-primary">Create Notifications App</a>
<a class="btn-link btn-collapse" data-toggle="collapse" href="#create-app-instructions" role="button" aria-expanded="false" aria-controls="create-app-instructions">Show detailed instructions</a>

<div class="collapse" id="create-app-instructions">
<ol>
<li><a href="https://developer.ringcentral.com/login.html#/">Login or create an account</a> if you have not done so already.</li>
<li>Go to Console/Apps and click 'Create App' button.</li>
<li>Select "API App for RingCentral Office" under "What type of app are you creating?"</li>
<li>Select "Other Non-UI" under "Where will you be calling the API from?"
<li>Select "Only members of my organization/company" under "Who will be authorized to access your app?"
<li>On the second page of the create app wizard, enter your app's name and description. Then select the following permissions:
  <ul>
    <li>WebhookSubscriptions, SMS</li>
  </ul>
  </li>
<li>We are using Password Flow authentication, so leave "OAuth Redirect URI" blank.</li>
</ol>
</div>

When you are done, you will be taken to the app's dashboard. Make note of the Client ID and Client Secret. We will be using those momentarily.

## Subscribe for push notification

### Install RingCentral JavaScript SDK

```bash
$ npm install ringcentral --save
```

### Run ngrok to create a localhost tunnel

```bash
$ ngrok http 5000
```

Copy the forwarding address e.g. https://54a0541a.ngrok.io and append the path "/webhook" to the address then paste it into the DELIVERY_ADDRESS variable in the code below.

### Create and Edit webhook-notification.js

Create a file called <tt>webhook-notification.js</tt>. Be sure to edit the variables in ALL CAPS with your app and user credentials. Be sure to also set the recipient's phone number.

```javascript
var http = require('http');
var SDK = require('ringcentral')

RINGCENTRAL_CLIENTID = '<ENTER CLIENT ID>'
RINGCENTRAL_CLIENTSECRET = '<ENTER CLIENT SECRET>'
RINGCENTRAL_SERVER = 'https://platform.devtest.ringcentral.com'

RINGCENTRAL_USERNAME = '<YOUR ACCOUNT PHONE NUMBER>'
RINGCENTRAL_PASSWORD = '<YOUR ACCOUNT PASSWORD>'
RINGCENTRAL_EXTENSION = '<YOUR EXTENSION, PROBABLY "101">'

DELIVERY_ADDRESS= '<https://xxxxxxxx.ngrok.io/webhook>'

PORT=5000

var server = http.createServer(function(req, res) {
    if (req.method == 'POST') {
        if (req.url == "/webhook"){
            if(req.headers.hasOwnProperty("validation-token")) {
                res.setHeader('Validation-Token', req.headers['validation-token']);
                res.statusCode = 200;
                res.end();
            } else {
                var body = []
                req.on('data', function(chunk) {
                    body.push(chunk);
                }).on('end', function() {
                    body = Buffer.concat(body).toString();
                    var jsonObj = JSON.parse(body)
                    console.log(jsonObj.body);
                });
            }
        }
    }else{
        console.log("IGNORE OTHER METHODS")
    }
});
server.listen(PORT);

var rcsdk = new SDK({
      server: RINGCENTRAL_SERVER,
      appKey: RINGCENTRAL_CLIENTID,
      appSecret: RINGCENTRAL_CLIENTSECRET
  });

var platform = rcsdk.platform();
platform.login({
            username: RINGCENTRAL_USERNAME,
            password: RINGCENTRAL_PASSWORD,
            extension: RINGCENTRAL_EXTENSION
        })
        .then(function(resp) {
            var params = {
              eventFilters: ['/restapi/v1.0/account/~/extension/~/message-store/instant?type=SMS'],
              deliveryMode: {
                  transportType: "WebHook",
                  address: DELIVERY_ADDRESS
                  }
              }
            platform.post('/subscription', params)
              .then(function(subscriptionResponse) {
                  console.log("Ready to receive incoming SMS via WebHook.")
              })
              .catch(function(e) {
                  console.error(e);
                  throw e;
              });
      });
```

### Run Your Code

You are almost done. Now run your script.

```bash
$ node webhook-notification.js
```

## Graduate Your App

Congratulations on creating your first RingCentral application. The last step is to graduate your application. We recommend [going through this process](../../../../basics/production) for your first application so you can understand the steps to take in the future, but also to come to appreciate the care taken by RingCentral to ensure that only high-quality apps are allowed into our production environment.
