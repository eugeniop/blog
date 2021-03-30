---
layout: post
title: "A Broadcaster for SMS MFA Challenges"
date:   2021-03-29 9:33 -0800
categories: sms otp mfa
comments: true
author: Eugenio Pace
---

Many web applications now require MFA which is great to see. Some use time based tokens (TOTP), with application like Google Authenticator to enroll and generate them. Others use SMS messages which are not super secure but better than nothing I guess... Others use a combination: TOTP with SMS as fallback. Very few apps have the sophistication to allow multiple phones to be enrolled on a given account.

> There are many reported issues with SMS based MFA. Please consider other options if you are implementing MFA in your apps.

This little project started with a simple problem: I've got 2 phones (work + personal) and while I carry both most of the time, I sometimes don't have the one I need at hand. Naturally, this seems to happen more often when I need to respond to an SMS MFA challenge.

Also, I am (was) often on an airplane with no access to SMS sometimes (Apple Messages works in misterious ways), but with access to Slack (via de magic of onflight WiFi). Wouldn't it be great to have SMS MFA challenges sent to Slack?


### The SMS Broadcaster

The solution was incredible straight forward. 

1. Get a Twilio Phone
2. Use that phone in all apps
3. Write a simple API (a webhook in Twilio's lingo) that forwards all messages to your phones/Slack/etc.
4. Enjoy! 

The code for the API is very simple (at least as a proof of concept):

```js
const express = require('express');
const async = require('async');
const _ = require('lodash');
const twilio = require('twilio');

const { log } = console;

const server = express();

const subscriber = [
  {
    app: 'An App',
    from: '+12344567890', //The source phone (the application's phone)
    to: ['+11112223333', '+12223334444', '+14445556666']  //Phones to broadcast to
  },
  {
    app: 'Another app',
    from: '+19876541111',
    to: ['+11112223333', '+12223334444']  //Phones to broadcast to
  },
];

// SMS
// The Webhook route
server.post('/sms', twilio.webhook(), smsHandler);
server.get('/sms', twilio.webhook(), smsHandler); 

function smsHandler(req, res, next){

  //Twilio supports both GET and POST. GET is easier to test with a browser
  const from = req.body ? req.body.From : req.query.From;
  const msg = req.body ? req.body.Body : req.query.Body; 

  //Do we recognize the sender?
  const s = _.find(subscriber, (i) => from === i.from);

  if(s){
    log(`Message from ${s.app}`);
    async.series(
      //This creates the array of functions for _async_ to call
      _.map(s.to, (phone) => {
        return function(cb){
          return sendSMSToRecipient(process.env.FROM_PHONE, phone, msg, cb);
        };
      }),
      (error) => {
        if(error){
          log("Error broadcasting Message");
        }
        res.status(200).end();
      }
    );
  } else {
    log(`Application with phone: ${from} not found`);
    log(`Message: ${msg}`);
    res.status(200).end();
  }
}

function sendSMSToRecipient(from, to, msg, done){
  const tw = require('twilio')(process.env.TWILIO_SID, 
                               process.env.TWILIO_AUTH_TOKEN);
  tw.messages
          .create({
                    to: to,
                    from: from,
                    body: msg
                  }, done);
};
```

#### Disclaimers

Usual disclaimers apply. There's plenty here missing. To name a few:

1. Not a lot of error handling.
2. No rate limiting on the API (I guess you can front it with Cloudflare or equivalent).
3. There's no bootstrap code (e.g. no signup). Notice the `subscriber` array is _hardcoded_. 
4. SMS is not that great as an MFA factor. I would strongly advice everyone to consider other alternatives, but alas, we live in an imperfect world.


So...use at your own risk. 

