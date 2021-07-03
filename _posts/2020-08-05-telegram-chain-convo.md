---
layout: post
title: Telegram Chain Convo
date: 2020-08-05 18:11:28 +0200
# intro: Let's have a converstaion with a Telegra bot!
tags: [Telegram, Bot, IoT, Node.JS, JS]
---

Telegram is a great platform to create bots to interact with users.

One difficult aspect of a Telegram bot is a chain conversation.
This is where the bot sends a message, the user replies, then the bot replies again.

> For example:

> Bot: Enter your name.

> User: Hanno

> Bot: Thank you, now please enter your surname.

> User: …

This is a conversation where the bot asks a question and waits for the user reply. Once the user reply, then the bot asks a follow up question.

The following guide will demonstrate a chain converstion Telegram Bot.

# Requirements:
1. Node.js
2. Basic js knowledge

# Let’s Begin!
Note: Remember to `init` your project

Firstly we need to install the required npm packages.

We will be using node-telegram-bot-api for our bot handling.

```zsh
npm install node-telegram-bot-api --save
```

For our database handling we will be using sqlite3.

```zsh
npm install sqlite3 --save
```

Create a new file called `index.js`. This will be the main file where the bot will handle and send all messages.

## Import all the necessary libraries.

Copy the following code at the top of the file:

```javascript
// sqlite3 setup...
const database = require('./database.js');
var sqlite3 = require('sqlite3').verbose();
var db = new sqlite3.Database('./db/Chat.db');

// telegram bot setup...
const TelegramBot = require('node-telegram-bot-api');
const token = 'YOUR BOT TOKEN';
const bot = new TelegramBot(token, { polling: true });
```

## Register User Command

```javascript
//registeruser command...
bot.onText(/\/registeruser/, (msg) => {
    db.run('INSERT INTO eventslog (code, currentdate, currentime, chatid) VALUES($code, date("now"), time("now"), $chatid)', {
        $code: 'I',
        $chatid: msg.chat.id
    });
    bot.sendMessage(msg.chat.id, 'Enter your name...');
});
```

## Handle Input From User
The following piece of code is where all the magic happens. This handles the input from the user. Database tables are updated according to the event.

```javascript
// Handle replies from user...
bot.on('message', async (msg) => {
    db.get('SELECT code FROM eventslog WHERE chatid = ? ORDER BY id DESC LIMIT 1', msg.chat.id, (err, row) => {
        console.log(row);

        if (row != null) {
            switch (row.code) {
                case 'I':
                    db.run('INSERT INTO eventslog (code, currentdate, currentime, chatid) VALUES ($code, date("now"), time("now"), $chatid)', {
                        $code: 'N',
                        $chatid: msg.chat.id
                    });
                    db.run('INSERT INTO users (name, currentdate, currentime, chatid) VALUES ($name, date("now"), time("now"), $chatid)', {
                        $name: msg.text,
                        $chatid: msg.chat.id
                    });
                    bot.sendMessage(msg.chat.id, 'Enter your surname...');
                    break;

                case 'N':
                    db.run('INSERT INTO eventslog (code, currentdate, currentime, chatid) VALUES ($code, date("now"), time("now"), $chatid)', {
                        $code: 'S',
                        $chatid: msg.chat.id
                    });
                    db.run('UPDATE users SET surname = $surname, currentdate = date("now"), currentime = time("now") WHERE chatid = $chatid', {
                        $surname: msg.text,
                        $chatid: msg.chat.id
                    });
                    bot.sendMessage(msg.chat.id, 'Enter your age...');
                    break;

                case 'S':
                    db.run('INSERT INTO eventslog (code, currentdate, currentime, chatid) VALUES ($code, date("now"), time("now"), $chatid)', {
                        $code: 'A',
                        $chatid: msg.chat.id
                    });
                    db.run('UPDATE users SET age = $age, currentdate = date("now"), currentime = date("now") WHERE chatid = $chatid', {
                        $age: msg.text,
                        $chatid: msg.chat.id
                    });
                    //bot.sendMessage(msg.chat.id, 'Enter your gender...');

                    const opts = {
                        reply_markup: {
                            inline_keyboard: [
                                [
                                    {
                                        text: 'Male',
                                        callback_data: 'M'
                                    },
                                    {
                                        text: 'Female',
                                        callback_data: 'F'
                                    }
                                ]
                            ]
                        }
                    };

                    bot.sendMessage(msg.chat.id, 'Select your gender...', opts);

                    break;

                /*case 'A':
                    db.run('INSERT INTO eventslog (code, currentdate, currentime, chatid) VALUES ($code, date("now"), time("now"), $chatid)', {
                        $code: 'G',
                        $chatid: msg.chat.id
                    });
                    db.run('UPDATE users SET gender = $gender, currentdate = date("now"), currentime = time("now") WHERE chatid = $chatid', {
                        $gender: msg.text,
                        $chatid: msg.chat.id
                    });
                    bot.sendMessage(msg.chat.id, 'Enter your nationality...');
                    break;*/

                case 'G':
                    db.run('INSERT INTO eventslog (code, currentdate, currentime, chatid) VALUES ($code, date("now"), time("now"), $chatid)', {
                        $code: 'N',
                        $chatid: msg.chat.id
                    });
                    db.run('UPDATE users SET nationality = $nat, currentdate = date("now"), currentime = time("now") WHERE chatid = $chatid', {
                        $nat: msg.text,
                        $chatid: msg.chat.id
                    });

                    db.get('SELECT * FROM users WHERE chatid = ? ORDER BY ID DESC LIMIT 1', msg.chat.id, (err, row) => {
                        let res = `You have entered the following details:\n`;
                        res += `Name: ${row.name}\n`;
                        res += `Surname: ${row.surname}\n`;
                        res += `Age: ${row.age}\n`;
                        res += `Gender: ${row.gender}\n`;
                        res += `Nationality: ${row.nationality}\n`;
                        bot.sendMessage(msg.chat.id, res);

                        db.run('INSERT INTO eventslog (code, currentdate, currentime, chatid) VALUES ($code, date("now"), time("now"), $chatid)', {
                            $code: 'C',
                            $chatid: msg.chat.id
                        });
                    });
                    break;
            }
        }
    });
});
```
## Handle Inline Keyboard Callback

```javascript
bot.on('callback_query', function onCallbackQuery(callbackQuery) {
    let action = callbackQuery.data;

    console.log(callbackQuery);

    db.run('INSERT INTO eventslog (code, currentdate, currentime, chatid) VALUES ($code, date("now"), time("now"), $chatid)', {
        $code: 'G',
        $chatid: callbackQuery.message.chat.id
    });
    db.run('UPDATE users SET gender = $gender, currentdate = date("now"), currentime = time("now") WHERE chatid = $chatid', {
        $gender: action,
        $chatid: callbackQuery.message.chat.id
    });
    bot.sendMessage(callbackQuery.message.chat.id, 'Enter your nationality...');