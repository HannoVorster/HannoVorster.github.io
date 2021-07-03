---
layout: post
title: ESP32 Firebase - Store Data In the Cloud
date: 2021-02-03 19:46:28 +0200
# intro: We will use an ESP32 to measure the temperature and store all reading in a Firebase Realtime Database.
categories: [ESP32, Firebase, IoT, C++]
---

Sometimes we would like to save data in a cloud databse to access data outside of your house.

The problem is, is that cloud SQL databases cost money to use and we as hobbyists sometimes cannot affors a cloud SQL database instance or would rather use it temporarily for developer/testing reasons.

Firebase offers a free [Real Time Database](https://en.wikipedia.org/wiki/Real-time_database#:~:text=A%20real%2Dtime%20database%20is,very%20rapidly%20and%20is%20dynamic.) service and we will use it to save data into the database.
![Firebase Home](https://user-images.githubusercontent.com/17809351/106787652-a6693500-6658-11eb-94d0-e31d157ef444.png)

In this post we will use a DHT sensor to read the humidity and temperature and save the data with a date and time into Firebase.

# Requirements
1. Firebase
2. ESP32
3. PlatformIO

# Let's Begin!
First we need to create a project and Real Time Database. It is very simple and I won't go into details on how to create one. Simply follow the instructions on screen and creat a project. When you are done, create a new real time database.

## Setup

I use PlatformIO for my ESP projects. It's a simple framwork that create Arduino and ESP projects, it also handles libraries and when you upload it it does the whole process for you. You can install it from Extensions in VSCode.
![PlatformIO Home](https://user-images.githubusercontent.com/17809351/106788125-39a26a80-6659-11eb-8110-d49b27d6e659.png)

Creat a new project and follow the prompts on screen.

We will use the [Firebase ESP32 Client](https://github.com/mobizt/Firebase-ESP32) library to access our Real Time Database.

## Coding Time
After installing all our required libraries and our project is ready, we will start with our code setup.

We connect our DHT sensor to pin 23 on our ESP32 and define our pin types and import our library. I am using a DHT11 sensor.
```cpp
//DHT...
#include "DHT.h"

#define DHTPIN 23     
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);
```

We will now declare our needed Firebase variables and setup. From our console we get our database host URL and secret key:
```cpp
#define FIREBASE_HOST "https://***********-default-rtdb.firebaseio.com"
#define FIREBASE_AUTH "***************************X8n"
```
**Please Note:** I am using a deprecated value to authenticate my database, I will change this in the future.

In our `setup()` function we will initialize our Firebase instance. This function is only run once.
```cpp
Firebase.begin(FIREBASE_HOST, FIREBASE_AUTH);
Firebase.reconnectWiFi(true);

Firebase.setReadTimeout(fbdo, 1000 * 60);
Firebase.setwriteSizeLimit(fbdo, "tiny");
                                                                
Firebase.setFloatDigits(2);
Firebase.setDoubleDigits(6);
```

### Read Humidity & Temperature
Firstly we need to read the humidity and temperature and store it in variables. If the reading failed then we return the function. In other words we terminate the current loop cycle.
```cpp
float h = dht.readHumidity();
float t = dht.readTemperature();

if (isnan(t) || isnan(h)) {    
    Serial.println(F("Failed to read from DHT sensor!"));
    return;
}
```

### Get Current Date & Time
Next we need to read the date and time and store it in variables.
```cpp
char timeWeekDay[25];
strftime(timeWeekDay,25, "%d %B %Y %H:%M:%S", &timeinfo);
String timeWeekDayStr = timeWeekDay;
```

### Add Variables in JSON
Now that we have all of our variables, we can store it in a Firebase JSON object. We clean the JSON first with `json.clear()`. Individually we add each of the variables in the JSON object with a descriptive key.
```cpp
json.clear();
json.add("Temp", t);
json.add("Humidity", h);
json.add("Date", timeWeekDayStr);
```

### Write To Firebase
Our JSON object is ready and now we can save it in Firebase. The Firebase ESP32 Client library offers many function for storing data, we will use the `.pushJSON()` to write our JSON object into Firebase.

The function returns a `boolean`, thus I wrap it around an `If` statment. If the function passes, then I simply output `PASSED` in the serial monitor. If the function fails, I output the error message in the serial monitor.
```cpp
if (Firebase.pushJSON(fbdo, path + "/DHT/Temperature", json))
{
    Serial.println("PASSED");
}
else
{
    Serial.println("FAILED");
    Serial.println("REASON: " + fbdo.errorReason());
    Serial.println("------------------------------------");
    Serial.println();
}
```

Now all our data are safely stored in a database that costs us no additional money!
![Firebase Data](https://user-images.githubusercontent.com/17809351/106792569-f77c2780-665e-11eb-8a14-a8fd8cdb1d69.jpg)

In a later post I will write an app service that will retrieve the data and output in a chart that we will be able to view in our browser and even on our mobile phone to check the temperature.

The full source code is available on [GitHub](https://github.com/HannoVorster/ESPFirebase).