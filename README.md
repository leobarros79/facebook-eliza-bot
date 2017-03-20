# Facebook Messenger Bot with Eliza AI

This is a project that allows the conversation between humans and robots through Facebook Messenger. The conversational algorithm is ELIZA.

## About ELIZA

ELIZA is an early natural language processing computer program created from 1964 to 1966 at the MIT Artificial Intelligence Laboratory by Joseph Weizenbaum. Created to demonstrate the superficiality of communication between man and machine, Eliza simulated conversation by using a 'pattern matching' and substitution methodology that gave users an illusion of understanding on the part of the program, but had no built in framework for contextualizing events. Directives on how to interact were provided by 'scripts', written originally in MAD-Slip, which allowed ELIZA to process user inputs and engage in discourse following the rules and directions of the script. The most famous script, DOCTOR, simulated a Rogerian psychotherapist and used rules, dictated in the script, to respond with non-directional questions to user inputs. As such, ELIZA was one of the first chatterbots, but was also regarded as one of the first programs capable of passing the Turing Test.

ELIZA's creator, Weizenbaum regarded the program as a method to show the superficiality of communication between man and machine, but was surprised by the number of individuals who attributed human-like feelings to the computer program, including Weizenbaum’s secretary.[2] Many academics believed that the program would be able to positively influence the lives of many people, particularly those suffering from psychological issues and that it could aid doctors working on such patients’ treatment.[2][4] While ELIZA was capable of engaging in discourse, ELIZA could not converse with true understanding.[5] However, many early users were convinced of ELIZA’s intelligence and understanding, despite Weizenbaum’s insistence to the contrary.

## About Facebook Messenger Platform

Facebook opened up their Messenger platform to enable bots to converse with users through Facebook Apps and on Facebook Pages. 

You can read the  [documentation](https://developers.facebook.com/docs/messenger-platform/quickstart) the Messenger team prepared but it's not very clear for beginners and intermediate hackers.

### Get set

Messenger bots uses a web server to process messages it receives or to figure out what messages to send. You also need to have the bot be authenticated to speak with the web server and the bot approved by Facebook to speak with the public.

You can also skip the whole thing by git cloning this repository, running npm install, and run a server somewhere.

#### *Build the server*

1. Install the Heroku toolbelt from here https://toolbelt.heroku.com to launch, stop and monitor instances. Sign up for free at https://www.heroku.com if you don't have an account yet.

2. Install Node from here https://nodejs.org, this will be the server environment. Then open up Terminal or Command Line Prompt and make sure you've got the very most recent version of npm by installing it again:

    ```
    sudo npm install npm -g
    ```

3. Create a new folder somewhere and let's create a new Node project. Hit Enter to accept the defaults.

    ```
    npm init
    ```

4. Install the additional Node dependencies. Express is for the server, request is for sending out messages and body-parser is to process messages.

    ```
    npm install express request body-parser --save
    ```

5. Create an index.js file in the folder and copy this into it. We will start by authenticating the bot.

    ```javascript
    'use strict'
    
    const express = require('express'),
          bodyParser = require('body-parser'),
          request = require('request'),
          app = express();

    app.set('port', (process.env.PORT || 8080))

    // Process application/x-www-form-urlencoded
    app.use(bodyParser.urlencoded({extended: false}))

    // Process application/json
    app.use(bodyParser.json())

    // Index route
    app.get('/', function (req, res) {
    	res.send('Hello world, I am a chat bot')
    })

    // for Facebook verification
    app.get('/webhook/', function (req, res) {
    	if (req.query['hub.verify_token'] === 'your_verify_token') {
    		res.send(req.query['hub.challenge'])
    	}
    	res.send('Error, wrong token')
    })

    // Spin up the server
    app.listen(app.get('port'), function() {
    	console.log('running on port', app.get('port'))
    })
    ```

6. Make a file called Procfile and copy this. This is so Heroku can know what file to run.

    ```
    web: node index.js
    ```

7. Commit all the code with Git then create a new Heroku instance and push the code to the cloud.

    ```
    git init
    git add .
    git commit --message "hello world"
    heroku create
    git push heroku master
    ```

#### *Setup the Facebook App*

1. Create or configure a Facebook App or Page here https://developers.facebook.com/apps/
    
2. In the app go to Messenger tab then click Setup Webhook. Here you will put in the URL of your Heroku server and a token. Make sure to check all the subscription fields. 

3. Get a Page Access Token and save this somewhere. 

4. Go back to Terminal and type in this command to trigger the Facebook app to send messages. Remember to use the token you requested earlier.

    ```bash
    curl -X POST "https://graph.facebook.com/v2.6/me/subscribed_apps?access_token=<PAGE_ACCESS_TOKEN>"
    ```

#### *Setup the bot*

Now that Facebook and Heroku can talk to each other we can code out the bot.

1. Add an API endpoint to index.js to process messages. Remember to also include the token we got earlier. 

    ```javascript
    app.post('/webhook/', function (req, res) {
	    let messaging_events = req.body.entry[0].messaging
	    for (let i = 0; i < messaging_events.length; i++) {
		    let event = req.body.entry[0].messaging[i]
		    let sender = event.sender.id
		    if (event.message && event.message.text) {
			    let text = event.message.text
			    sendTextMessage(sender, "Text received, echo: " + text.substring(0, 200))
		    }
	    }
	    res.sendStatus(200)
    })

    const token = "<PAGE_ACCESS_TOKEN>"
    ```
    
    **Optional, but recommended**: keep your app secrets out of version control!
    - On Heroku, its easy to create dynamic runtime variables (known as [config vars](https://devcenter.heroku.com/articles/config-vars)). This can be done in the Heroku dashboard UI for your app **or** from the command line:

    ```bash
    heroku config:set FB_PAGE_ACCESS_TOKEN=access-token-xpto
    
    # view
    heroku config
    ```

    - For local development: create an [environmental variable](https://en.wikipedia.org/wiki/Environment_variable) in your current session or add to your shell config file.
    ```bash
    # create env variable for current shell session
    export FB_PAGE_ACCESS_TOKEN=access-token-xpto
    
    # alternatively, you can add this line to your shell config
    # export FB_PAGE_ACCESS_TOKEN=access-token-xpto
    
    echo $FB_PAGE_ACCESS_TOKEN
    ```
    
    - `config var` access at runtime
    ``` javascript
    const token = process.env.FB_PAGE_ACCESS_TOKEN
    ```
    
    
3. Add a function to echo back messages

    ```javascript
    function sendTextMessage(sender, text) {
	    let messageData = { text:text }
	    request({
		    url: 'https://graph.facebook.com/v2.6/me/messages',
		    qs: {access_token:token},
		    method: 'POST',
    		json: {
			    recipient: {id:sender},
    			message: messageData,
    		}
    	}, function(error, response, body) {
    		if (error) {
			    console.log('Error sending messages: ', error)
    		} else if (response.body.error) {
			    console.log('Error: ', response.body.error)
		    }
	    })
    }
    ```

4. Commit the code again and push to Heroku

    ```
    git add .
    git commit -m 'updated the bot to speak'
    git push heroku master
    ```
    
5. Go to the Facebook Page and click on Message to start chatting!

### How to share your bot

#### *Add a chat button to your webpage*

Go [here](https://developers.facebook.com/docs/messenger-platform/plugin-reference) to learn how to add a chat button your page.

#### *Create a shortlink*

You can use https://m.me/<PAGE_USERNAME> to have someone start a chat.
