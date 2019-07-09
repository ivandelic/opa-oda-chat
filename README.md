# OPA Bots Custom Component
andrew.e.higginbottom@oracle.com - 5 April 2019

some additional work 

This custom component connects to the Oracle Policy Automation Chat API to give your Digital Assistants the ability to conduct OPA interviews with customers.

These instructions assume deployment to the Oracle Digital Assistant Embedded Container but you can deploy to any node.js host.

PREREQUISITES: You need to have node.js installed.

## Expand the OPA Custom Component for ODA
- Expand the opa-oda-chat_XYZ.zip folder.

## Install the Oracle Bots Node SDK
- Open a command prompt in the expanded zip folder.
- `npm install @oracle/bots-node-sdk`

## Configure encryption for sensitive parameters
Some parameters passed from ODA to the OPA Custom Component must be encrypted. See Parameter Reference below for details.
For example, your Chat API user's credentials ('chatClientId' and 'chatClientSecret') MUST be encrypted.

Setup a Chat API user: http://documentation.custhelp.com/euf/assets/devdocs/cloud19a/PolicyAutomation/en/Default.htm#Guides/Project_Administrator_Guide/Permissions/Create_account_for_application_integration.htm

- Go to https://appzaza.com/encrypt-text. Make sure Algorithm is 'AES-256-CBC'.
- Leave `Password1` and `Password2` blank.
- Encrypt your chatClientId
- Copy the generated `Password1` and `Password2` values from the Results section and paste them into the Password1 and Password2 boxes at the top of the page
- Encrypt your chatClientSecret

## Setup config.js
Config.js is used to configure the encryption keys for decrypting the parameter values you encrypted above (see parameter list below for details).

Edit the components/opa-oda/config.js file to reflect the values you got back when encrypting above.
-   CRYPTO_SECRET: '6cccdab95e0530786e24b152b47e58c4',
-    CRYPTO_IV: '14409ce40c598250'

## Deploy the custom component to the ODA Embedded Container
- At the root of the expanded zip folder, run `npm pack`
- Add a component service to your skill in ODA
 - Select Embedded Container for the type and upload the tgz file generated by `npm pack`

## Add your own OPA Interview
- Deploy your interview and make sure the Chat API is enabled, see http://documentation.custhelp.com/euf/assets/devdocs/cloud19a/PolicyAutomation/en/Default.htm#Guides/Developer_Guide/Chat_API/Integrate_OPA_into_VA_experience.htm#Start

- Copy+Paste a new stage in the ODA flow and modify to suit:
~~~~
  mynewInterview:
    component: "OpaInterview"
    properties:
      botName: "<my new bot name>"
      chatURL: "https://<your hub url>/web-determinations/JSONChatStart/latest/<youropaproject>"
      authURL: "https://<your hub url>/opa-hub/api/auth"
      chatClientId: "<your API user>"
      chatClientSecret: <your API client secret>"
    transitions:
      return: "done"
~~~~

## Parameter Reference
The following properties are supported for configuration of the OPA states within your bot flow:

NOTE: All parameters marked as encrypted below should be encrypted using https://appzaza.com/encrypt-text as above.


- `botName`: an internal id used in logs
- `chatURL`: https://<your hub url>/web-determinations/JSONChatStart/latest/<youropaproject>
- `authURL`: https://<your hub url>/opa-hub/api/auth
- `chatClientId`: Your OPA Chat API ClientId (encrypted)
- `chatClientSecret`: Your OPA Chat API client secret (encrypted)
- `greeting`: (optional) Message to greet users before the first screen is rendered
- `instructions`: (optional) Message with instructions for use
- `stopWords`: (optional) Comma delimited string of words that will immediately stop the interview and transition with the action 'stopped'. Default 'stop' if not provided.
- `backWords`: (optional) Comma delimited string of words that will cause the interview to go back one screen. Default 'back' if not provided.
- `trueWord`: (optional) Word to represent Boolean TRUE values. Default 'Yes' if not provided.
- `falseWord`: (optional) Word to represent Boolean FALSE values. Default 'No' if not provided.
- `backFail`: (optional) Message to display if it's not possible to go back. Eg. on the first screen! Default 'You can\'t go back from here.' if not provided
- `offerRestart`: (optional) Message to offer a restart when reaching the end of the interview
- `offerContinue`: (optional) Message to offer to continue the interview from the previously captured state (if the user leaves without completing the interview either by session timeout OR by hitting the 'cancel' transition caused by exceeding maxPrompts, see below). Default is 'Would you like to continue from where you left off last time?' if not provided.
- `sorryContinue`: (optional) Message to say if the interview session was not able to be restored. Eg. if the data model has changed significantly. Default 'Sorry, something went wrong. We need to restart.' if not provided.
- `removeSnapshot`: (optional) Remove interview snapshots when exiting an interview via the 'cancel' transition. Default FALSE if not provided.
- `farewell`: (optional) Message to say after the interview and after any offer to restart is declined.
- `confirmInput`: (optional) Message to display when using similarity-based input matching (and not using maxPrompts). Default is 'Did you mean \'{input}\'?' if not provided.
- `didntUnderstand`: (optional) Message to display if the input is not recognised. Default is 'Sorry, I didn\'t understand that.' if not provided.
- `maxPrompts`: (optional) Same as the standard Bots components. The OPA custom component goes into strict input mode (requires exact input - NOT similarity-based) and will re-prompt up to maxPrompts times (including the first time it asks the question) then transition with the 'cancel' action. The interview session will be preserved, allowing your flow to re-enter the state and continue the OPA interview.
- `seedData`: (optional) Attribute data to seed into the interview. Eg. `"{\"app_name\":\"${profile.firstName}\"}"`
# opa-oda-chat
