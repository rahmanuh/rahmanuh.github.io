---
layout: post
title: "First Nodejs Application Ever"
date: 2020-07-15 14:45:00 +0200
---

Hello,

I just created a simple pregnancy logger yesterday. This is my first Nodejs app ever.
Well, I created this because I want to log my wife's weight and abdominal circumference
(I do not know the best idiom for "lingkar perut").
You can read the pregnancy story [here](http://83.226.101.206:7000/2020/05/octa-is-pregnant/).
This app is served by my Raspberry Pi at home. Anyway, let me show you my little app.

```javascript
const http = require('http');
const fs = require('fs');
const url = require('url');

var file = "log.txt";
let date_ob = new Date();

function getFullDate() {
    // current date
    // adjust 0 before single digit date
    let date = ("0" + date_ob.getDate()).slice(-2);

    // current month
    let month = ("0" + (date_ob.getMonth() + 1)).slice(-2);

    // current year
    let year = date_ob.getFullYear();

    var fullDate = year + "-" + month + "-" + date;

    return fullDate;
}

http.createServer(function (req, res) {
    var fullDate = getFullDate();

    if (req.url === '/favicon.ico')  
        return res.end();
    else
        var text = fullDate + req.url + "\n";

    fs.appendFile(file, text, function(err) {
        if (err) console.log(err);
    });

    fs.readFile(file, function(err, data) {
        if (err) {
            return res.end("404 Not Found");
        } 
        res.write(data);
        return res.end();
    });

}).listen(3000);
```
As you can see in the code above, the app will log `req.url` to file log.txt. 
I made a simple filter to filter out favicon.ico,
because it always requested by browser after sending an actual request (data log).
Yes, you may say that this is not the best aproach to create an API. 
But it is enough to me, hehehe :). That's it. I wish her pregnancy is always OK until delivery.

Cheers,\
Rahmanu
