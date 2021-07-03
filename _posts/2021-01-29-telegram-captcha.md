---
layout: post
title: Telegram Captcha
date: 2021-01-29 20:12:00 +0200
# intro: Use a Telegram bot captcha authentication.
categories: [Telegram, Bot, IoT, Node.JS, JS, Security]
---

Been a while since I have posted here!

Happy New Year!!!

![Happy New Year!!](https://user-images.githubusercontent.com/17809351/106794160-17ace600-6661-11eb-99b3-fbc3ca9bc65a.jpg)

This post will be short and sweet where I quickly made a Telegram Captcha bot. I will work on this and make it more effecient in the future.

# Requirements
1. Node.js (Express.js)
2. Telegram Bot (Created in Botfater)

# Let's Begin!
Firstly I used the Express Skeleton generator to generate a skeleton structure of a Node.js server (you can also just create a normal Node.js server, I use the generator daily and this is a force of habit).

Create a blank folder and enter `npx express-generator` in your command terminal and immediatly `npm install` to install the dependecies.

For this project we will use 2 NPM packages:
1. [Node Telegram Bot API](https://www.npmjs.com/package/node-telegram-bot-api)
2. [Captcha Generator](https://www.npmjs.com/package/@haileybot/captcha-generator) (This will generate our captcha text and images.)

Install both npm packages with your terminal:
```zsh
npm i node-telegram-bot-api
npm i @haileybot/captcha-generator
```

## Setup Telegram in Node
We will include the package and create a Telegram Bot client:
```javascript
const TelegramBot = require('node-telegram-bot-api');
const token = config.telegram.token;
const bot = new TelegramBot(token, {
  polling: true
});
```

## Generate our Captcha
We will now generate our captcha by creating a new captcha object (this is handle by the captcha generator package):
```javascript
let captcha = new Captcha();
```

From our captcha object we want the generated value and image. We can retrieve it by calling the `dataURL` and `value` functions.
```javascript
let image = captcha.dataURL;
let correctValue = captcha.value;
```

## Telegram Answer Buttons
We would like to send the correct answer with 4 different incorrect answers for the user.
We will send the answers with inline Telegram buttons where the user can simply press on one instead of having to type out the answer.
To achieve this we need to generate 4 different answers and push all of them with the correct answer into an array:
```javascript
let arrValues = [];
arrValues.push(correctValue);

for (i = 0; i < 4; i++) {
    arrValues.push(makeid(6));
}
```

You might have notice the `makeid(6)` function. This is not a JS function and we need to write our own function to generate our random string.
We loop 4 time to generate 4 string and all of them need to be 6 characters long.
The `makeid` function will look as follow:
```javascript
function makeid(length) {
  var result = '';
  var characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
  var charactersLength = characters.length;
  for (var i = 0; i < length; i++) {
    result += characters.charAt(Math.floor(Math.random() * charactersLength));
  }
  return result;
}
```

After we have our correct answer and 4 strings, we need to shuffle the array to make sure that the correct answer will appear randomly when it sends to the user.
To achieve this we will write our own `shuffle()` function:
```javascript
function shuffle(array) {
  var currentIndex = array.length,
    temporaryValue, randomIndex;

  while (0 !== currentIndex) {
    randomIndex = Math.floor(Math.random() * currentIndex);
    currentIndex -= 1;

    temporaryValue = array[currentIndex];
    array[currentIndex] = array[randomIndex];
    array[randomIndex] = temporaryValue;
  }

  return array;
}
```

Our shuffle function is based on the [Fisher-Yates (aka Knuth) Shuffle algorithm](https://bost.ocks.org/mike/shuffle/).

Now we would like to add our button to an array of objects that we will attach to our inline keyboard.
We simply loop through our array and create an object for each button and push it into the array.
```javascript
let arrButtons = [];
let obj;
for (i = 0; i < 5; i++) {
    obj = {};
    obj['text'] = arrValues[i];
    obj['callback_data'] = arrValues[i];
    arrButtons.push(obj);
}
```

We simply add the aaray to the Telegram keyboard and send it with a custom message:
```javascript
const opts = {
    reply_markup: {
    inline_keyboard: [
        arrButtons
    ]}
}
bot.sendMessage(msg.chat.id, "Please choose the correct text on the photo:", opts);
```

## Captcha Image
We won't get far without out captcha image because then the user won't have anything to compare the buttons to.

We simply take the image data url and convert it to a buffer:
```javascript
var regex = /^data:.+\/(.+);base64,(.*)$/;

var matches = string.match(regex);
var ext = matches[1];
var data = matches[2];
var buffer = Buffer.from(data, 'base64');
```

Lastly we send the photo to the user:
```javascript
bot.sendPhoto(msg.chat.id, buffer);
```

Below are screenshots of how the program runs:
![Send Picture and 5 Different Answers](https://user-images.githubusercontent.com/17809351/106352371-328efb80-62eb-11eb-9cd0-23653ab856fd.png)

Below is an example of a successful captcha selection:
![Passed Captcha](https://user-images.githubusercontent.com/17809351/107887720-c75f3f00-6f10-11eb-8b88-ef05c1d930cb.gif) 

Below is an example of a failed captcha selection:
![Failed Captcha](https://user-images.githubusercontent.com/17809351/107887753-f675b080-6f10-11eb-847f-9fa3cd4c15b9.gif)

If you would like to see the [whole set of code](https://github.com/HannoVorster/Telegram-Captcha), it is available on my GitHub page.