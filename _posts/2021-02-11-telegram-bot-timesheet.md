---
layout: post
title: Timesheet Telegram Bot
date: 2021-02-11 20:00:00 +0200
intro: Create new timesheet entries through a Telegram bot.
categories: [Telegram, Bot, Node.JS, JS, Firebase]
---

Everyday I have to submit timesheets when I am done working. Our company timesheets requires the following: Work Package number, hours worked & description.

Our timesheet web page is on premise so we need to connect to it VIA a VPN. This is tedious and sometimes after work there is not enough time to go through the whole process.

Thus I have decided to create a Telegram Bot that I can use to capture my timesheets and at the end of the week I can submit my timesheets onto our website.

# Requirements
1. Node.JS (Express.js)
2. Telegram Bot Token
3. Firebase Account

# NPM Packages
We will be using the followong NPM packages:
* [express](https://www.npmjs.com/package/express)
* [Node.js Telegram Bot API](https://www.npmjs.com/package/node-telegram-bot-api)
* [Firebase - App success made simple](https://www.npmjs.com/package/firebase)
* [Node Schedule](https://www.npmjs.com/package/node-schedule)

Install all of there NPM packages in your project folder.

# Let's Begin!
Fist you should make sure you have an active Firebase Real Time Database, a Telegram Bot Token (generated in [Bot Father](https://t.me/botfather)) and express js installed in your project.

I did not use the skeleton generator in this project, I simply created an `index.js` file and import all of my libraries.

## Setup
Add both your Firebase config object and Telegram Bot. I have also included my user chat ID for sending custom messages to my chat from the bot.

```javascript
//FIREBASE...
var firebase = require('firebase');

var firebaseConfig = {
    apiKey: "********************Wuus",
    authDomain: "*********.firebaseapp.com",
    projectId: "*********-2f0da",
    storageBucket: "**************.appspot.com",
    messagingSenderId: "31*********",
    appId: "*:***********:7e******634e5",
    measurementId: "G-**********"
};

firebase.initializeApp(firebaseConfig);
let database = firebase.database();

// TELEGRAM...
const TOKEN = '*****************0twKN5kE*********bz8';
const TelegramBot = require('node-telegram-bot-api');
const options = {
    polling: true
};
const bot = new TelegramBot(TOKEN, options);
const hannoChatId = '*******73';
```

We will also be using a new NPM package calle Node Cache. We will be using this package to temporarily remember values entered by the Telegram user.
```javascript
// NODE CACHE...
const NodeCache = require("node-cache");
const moment = require('moment');
const myCache = new NodeCache();
```

## Time To Submit Those Timesheets!
In Telegram you can send the bot commands with the `/` character. I have created my own command that triggers the workflow.

Now that everything is set up, we can start implementing our bot. Whenever a user wishes to submit a new timesheet, they should user the `/addtime` command.
```javascript
bot.onText(/\/addtime/, (msg) => {
    addNewTime(msg, 'C');
});
```

The function `addNewTime()` will initiate the workflow. First it will send the user work packages as reply buttons.
```javascript
addNewTime = (msg, type) => {
    let msgId;

    const opts = {
        reply_markup: {
            inline_keyboard: [
                [{
                        text: 'System Dev',
                        callback_data: '7109-00-00'
                    },
                    {
                        text: 'NCDoH',
                        callback_data: '3214-00-00'
                    },
                    {
                        text: 'ELIDZ',
                        callback_data: '2977-00-04'
                    }
                ]
            ]
        }
    }

    if (type == 'C') {
        msgId = msg.from.id;
    } else {
        msgId = msg.chat.id;
    }

    bot.sendMessage(msgId, '📦 Select a work package...', opts);
}
```

The bot will now await the user for reply. In the Telegram API we will use the `callabck_query` function. Whenever the user taps on a button, then this function will be called.
```javascript
// Handle callback queries
bot.on('callback_query', (callbackQuery) => {
    const action = callbackQuery.data;
    const msg = callbackQuery.message;
    const opts = {
        chat_id: msg.chat.id,
        message_id: msg.message_id,
    };
    let text;

    switch (action) {
        case '7109-00-00':
            text = 'You have selected System Development!';
            createEntry(action, msg.chat.id);
            break;

        case '3214-00-00':
            text = 'You have selected NCDoH!';
            creatEntry(action, msg.chat.id);
            break;

        case '2977-00-04':
            text = 'You have selected ELIDZ';
            createEntry(action, msg.chat.id);
            break;
    }
}
```

We will now store the workpackage number in the node cache.
```javascript
function createEntry(wp, chatId) {
    (async function () {
        try {
            obj = {
                WorkPackage: wp,
                TelegramUser: chatId,
                LastState: 'W'
            };

            myCache.set(chatId, obj, 10000);
            console.log(myCache.get(chatId));

            setHours(chatId);
        } catch (err) {
            console.log(err);
        }
    })()
}
```

Next we will request the user to enter the amount of hours they worked.
```javascript
function setHours(chatId) {
    bot.sendMessage(chatId, '⏰ Enter number of hours...');
}
```

The bot will now await the user to reply with a message. The Telegram Bot API has a function called `bot.on('message')`. This function is triggered when the user sends the bot a message. 
```javascript
if (isNaN(msg.text)) {
    bot.sendMessage(msg.chat.id, '❕ Invalid format, please enter a valid number');
} else {

    row['Hours'] = msg.text;
    row['LastState'] = 'H';

    myCache.set(msg.chat.id, row, 10000);
    console.log(myCache.get(msg.chat.id));

    setDescription(msg.chat.id);
}
```

The hours is stored in Node Cache with the work package number and the telegram user ID. next the bot will request the user to enter a description.
```javascript
function setDescription(chatId) {
    bot.sendMessage(chatId, '📔 Enter a description...');
}
```

When the user send the description, the `bot.on('message')` will be triggered again.
```javascript
if (msg.text.length < 20) {
    bot.sendMessage(msg.chat.id, 'Please enter a longer description');
} else {

    row['Description'] = msg.text;
    row['LastState'] = 'D';

    myCache.set(msg.chat.id, row, 10000);
    console.log(myCache.get(msg.chat.id));

    confirmEntry(msg.chat.id);
}
```

The complete `bot.on('message')` looks as follow:
```javascript
bot.on('message', (msg) => {
    //console.log(msg);

    if (msg.text != '/help' || msg.text != '/checkentries' || msg.text != '/testmail' || msg.text != '/cleantable') {
        (async function () {
            try {
                let row = myCache.get(msg.chat.id);

                console.log(row);

                if (row != null) {
                    switch (row.LastState) {
                        case 'W':
                            if (isNaN(msg.text)) {
                                bot.sendMessage(msg.chat.id, '❕ Invalid format, please enter a valid number');
                            } else {

                                row['Hours'] = msg.text;
                                row['LastState'] = 'H';

                                myCache.set(msg.chat.id, row, 10000);
                                console.log(myCache.get(msg.chat.id));

                                setDescription(msg.chat.id);
                            }
                            break;

                        case 'H':
                            if (msg.text.length < 20) {
                                bot.sendMessage(msg.chat.id, 'Please enter a longer description');
                            } else {

                                row['Description'] = msg.text;
                                row['LastState'] = 'D';

                                myCache.set(msg.chat.id, row, 10000);
                                console.log(myCache.get(msg.chat.id));

                                confirmEntry(msg.chat.id);
                            }
                            break;
                    }
                }
            } catch (err) {
                console.log(err);
            }
        })()
    }
})
```

The description is stored in Node Cache with all the other input values. Lastly the bot sends a summaryu back to the user with 2 buttons for confirmation:
```javascript
function confirmEntry(chatId) {
    (async function () {
        try {
            let row = myCache.get(chatId);

            const opts = {
                reply_markup: {
                    inline_keyboard: [
                        [{
                                text: '✅ Yes',
                                callback_data: 'yes_confirm'
                            },
                            {
                                text: '❌ No',
                                callback_data: 'no_confirm'
                            }
                        ]
                    ]
                }
            }

            var res = 'Would you like to submit the following entry?\n';
            res += `📦 Work package: ${row.WorkPackage}\n`;
            res += `⏰ Hours: ${row.Hours}\n`;
            res += `📔 Description: ${row.Description}`;

            bot.sendMessage(chatId, res, opts);
        } catch (err) {
            console.log(err);
            bot.sendMessage(chatId, 'Error in the bot!');
        }
    })()
}
```

Once the user confirms the entry is correct then by using our callback_query functio, we save the entry:
```javascript
case 'yes_confirm':
    saveEntry(msg.chat.id);
    break;

case 'no_confirm':
    cancelEntry(msg.chat.id);
    break;
```

Now we will access the Node Cache and save our timesheet obj in Firebase. Our unique row id is generated from Unix timestamp.
```javascript
function saveEntry(chatId) {

    let row = myCache.get(chatId);

    let obj = {
        'WorkPackage': row.WorkPackage,
        'TelegramUser': chatId,
        'Description': row.Description,
        'Hours': row.Hours,
        'TimeSheetDate': moment().format('yyyy-MM-DD HH:mm:ss')
    };

    database.ref(`TimeSheet/${Math.floor(new Date().getTime() / 1000)}`).set(obj, function (error) {
        if (error) {
            console.log("Failed with error: " + error)
        } else {
            console.log("success")
        }
    });

    bot.sendMessage(chatId, '✅ Entry saved!');
}
```

Below is a quick demo of how the timesheet bot process works:
![Timesheet Bot Demo](https://user-images.githubusercontent.com/17809351/107645623-94ebe280-6c81-11eb-8bdd-fdddf3399aa7.gif)


The data is safely stored in Firebase! I will in a future page build further on this server to display data in a HTML page.
![Firebase Database](https://user-images.githubusercontent.com/17809351/107646187-57d42000-6c82-11eb-87b8-a4bb6272adf7.png)

## BONUS!
Sometimes I completely forget to submit my timesheets. Thus I have created a CRON statement where the bot sends me a remnder to submit mu timesheets. It reminds me every day during the weekday at 19:00.
```javascript
schedule.scheduleJob('0 19 * * 1-5', function () {
    bot.sendMessage(hannoChatId, '❕ Please submit your timesheets');
});
```