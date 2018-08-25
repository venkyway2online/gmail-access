How to access Gmail using NodeJS and the Gmail api
 April 17, 2018  sameer  nodejs
If you at any time need to programatically retrieve your Gmail account mails than the only proper way to do that is to use the Gmail api. The following post will show you how to access your Gmail account using the Gmail api and NodeJS. The code will handle all OAuth authentication and token processing.

Turning on the Gmail api
You first need to enable Gmail api and get the required OAuth credentials from your Google account. The steps which are shown below.

1. Use this wizard to create or select a project in the Google Developers Console and automatically turn on the API. Click Continue, then Go to credentials.

2. On the Add credentials to your project page, click the Cancel button.

3. At the top of the page, select the OAuth consent screen tab. Select an Email address, enter a Product name if not already set, and click the Save button.

4. Select the Credentials tab, click the Create credentials button and select OAuth client ID.

5. Select the application type Other, enter the name “Gmail API Quickstart”, and click the Create button.

6. Click OK to dismiss the resulting dialog.

7. Click the Download JSON button to the right of the client ID.

8. Move this file to your working directory and rename it client_secret.json.

Once the above steps are done your Gmail access setup is done. Next, install the Gmail api NodeJSlibrary.

npm install googleapis --save
Authentication
Before we proceed further take note that Gmail API uses OAuth 2.0 to handle authentication and authorization. There are various authentication scopes which can be used individually or in combination.

Once the above things are done, create a ‘gmail.js’ file and add the following code. This will login to your gmail account and return all the email category labels for your account. Now execute the below code.

E:\localhost\nodejs>node gmail.js
var fs = require('fs');
var readline = require('readline');
var {google} = require('googleapis');
 
// If modifying these scopes, delete your previously saved credentials
// at TOKEN_DIR/gmail-nodejs.json
var SCOPES = ['https://www.googleapis.com/auth/gmail.readonly'];
 
// Change token directory to your system preference
var TOKEN_DIR = ('E:/localhost/nodejs/credentials/');
var TOKEN_PATH = TOKEN_DIR + 'gmail-nodejs.json';
 
var gmail = google.gmail('v1');
 
// Load client secrets from a local file.
fs.readFile('client_secret.json', function processClientSecrets(err, content) {
  if (err) {
    console.log('Error loading client secret file: ' + err);
    return;
  }
  // Authorize a client with the loaded credentials, then call the
  // Gmail API.
  authorize(JSON.parse(content), listLabels);
});
 
/**
 * Create an OAuth2 client with the given credentials, and then execute the
 * given callback function.
 *
 * @param {Object} credentials The authorization client credentials.
 * @param {function} callback The callback to call with the authorized client.
 */
function authorize(credentials, callback) {
    var clientSecret = credentials.installed.client_secret;
    var clientId = credentials.installed.client_id;
    var redirectUrl = credentials.installed.redirect_uris[0];
 
    var OAuth2 = google.auth.OAuth2;
 
    var oauth2Client = new OAuth2(clientId, clientSecret,  redirectUrl);
 
    // Check if we have previously stored a token.
    fs.readFile(TOKEN_PATH, function(err, token) {
      if (err) {
        getNewToken(oauth2Client, callback);
      } else {
        oauth2Client.credentials = JSON.parse(token);
        callback(oauth2Client);
      }
    });
}
 
/**
 * Get and store new token after prompting for user authorization, and then
 * execute the given callback with the authorized OAuth2 client.
 *
 * @param {google.auth.OAuth2} oauth2Client The OAuth2 client to get token for.
 * @param {getEventsCallback} callback The callback to call with the authorized
 *     client.
 */
function getNewToken(oauth2Client, callback) {
  var authUrl = oauth2Client.generateAuthUrl({access_type: 'offline', scope: SCOPES});
  console.log('Authorize this app by visiting this url: ', authUrl);
  var rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
  });
 
  rl.question('Enter the code from that page here: ', function(code) {
    rl.close();
    oauth2Client.getToken(code, function(err, token) {
      if (err) {
        console.log('Error while trying to retrieve access token', err);
        return;
      }
      oauth2Client.credentials = token;
      storeToken(token);
      callback(oauth2Client);
    });
  });
}
 
/**
 * Store token to disk be used in later program executions.
 *
 * @param {Object} token The token to store to disk.
 */
function storeToken(token) {
  try {
    fs.mkdirSync(TOKEN_DIR);
  } catch (err) {
    if (err.code != 'EEXIST') {
      throw err;
    }
  }
  fs.writeFile(TOKEN_PATH, JSON.stringify(token));
  console.log('Token stored to ' + TOKEN_PATH);
}
 
/**
 * Lists the labels in the user's account.
 *
 * @param {google.auth.OAuth2} auth An authorized OAuth2 client.
 */
function listLabels(auth) {
  gmail.users.labels.list({auth: auth, userId: 'me',}, function(err, response) {
    if (err) {
      console.log('The API returned an error: ' + err);
      return;
    }
 
    var labels = response.data.labels;
 
    if (labels.length == 0) {
      console.log('No labels found.');
    } else {
      console.log('Labels:');
      for (var i = 0; i < labels.length; i++) {
        var label = labels[i];
        console.log('%s', label.name);
      }
    }
  });
}
Once the above code is executed from the command line it will return a OAuth url which you need to copy to your browser for authentication. (Before pasting the url into the browser make sure that it is well formed and does not contain any line breaks. To make sure of that copy the url into a text editor first.) The url will then authenticate and return a OAuth credential token string which you will need to then paste at the given prompt. Once this is correctly done, the code will store the access token into a file called ‘gmail-nodejs.json’. The next time you execute the above code Google will use this file and the access token therein to login and authenticate.

If you have done everything correctly, you will see a list of category labels for your account.

CATEGORY_PERSONAL
CATEGORY_SOCIAL
Notes
CATEGORY_FORUMS
Receipts
Work
IMPORTANT
CATEGORY_UPDATES
CHAT
SENT
INBOX
TRASH
CATEGORY_PROMOTIONS
DRAFT
SPAM
STARRED
UNREAD
Now that we have a proper working Gmail api setting, we will next see how to access your emails. The function for the same is given below. All the authentication code remains the same, we just keep adding various Gmail methods as required for our purpose. In this post I will only show the function to retrieve and read emails.

/**
 * Get the recent email from your Gmail account
 *
 * @param {google.auth.OAuth2} auth An authorized OAuth2 client.
 */
function getRecentEmail(auth) {
    // Only get the recent email - 'maxResults' parameter
    gmail.users.messages.list({auth: auth, userId: 'me', maxResults: 1,}, function(err, response) {
        if (err) {
            console.log('The API returned an error: ' + err);
            return;
        }
 
      // Get the message id which we will need to retreive tha actual message next.
      var message_id = response['data']['messages'][0]['id'];
 
      // Retreive the actual message using the message id
      gmail.users.messages.get({auth: auth, userId: 'me', 'id': message_id}, function(err, response) {
          if (err) {
              console.log('The API returned an error: ' + err);
              return;
          }
 
         console.log(response['data']);
      });
    });
}
Call the function from the main code from where before we called the ‘listLabels’ function.

...
// Authorize a client with the loaded credentials, then call the
// Gmail API.
authorize(JSON.parse(content), getRecentEmail);
This will return something like the following, depending on your email type. You will then need to parse the following user message resource to get to the data.

E:\localhost\test\nodejs>node gmail.js
 
{ id: '162bca5abe3acbf5',
  threadId: '162bca5315t5yc28',
  labelIds: [ 'IMPORTANT', 'SENT', 'INBOX' ],
  snippet: '',
  historyId: '5076456',
  internalDate: '1523876396000',
  payload:
   { partId: '',
     mimeType: 'multipart/mixed',
     filename: '',
     headers:
      [ [Object],
        [Object],
        [Object],
        [Object],
        [Object],
        [Object],
        [Object] ],
     body: { size: 0 },
     parts: [ [Object], [Object] ] },
  sizeEstimate: 4841772 }
The main part we are interested in is the payload section. This contains the primary message along with attachments. We can access that part using the following. Note that the email body content is encoded in base64, so we will need to decode it to display the email content.

// Access the email body content, like this...
message_raw = response['data']['payload']['parts'][0].body.data;
 
// or like this
message_raw = response.data.payload.parts[0].body.data;
Then we will need to decode the base64 encoded message.

data = message_raw;  
buff = new Buffer(data, 'base64');  
text = buff.toString();
console.log(text);
This will than give us the actual human readable text message. Using the other api methods follows the same calling pattern with different parameters. The authentication code remains the same. More usage of the Gmail api will be considered in another post.

API usage limit
The Gmail API is subject to a daily usage limit that applies to all requests made from your application, as well as per-user rate limits. Exceeding a rate limit will cause an HTTP 403 or HTTP 429 Too Many Requests response and your app should respond by retrying with exponential backoff. Exact limit restrictions are given here.

