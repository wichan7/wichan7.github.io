---
title: "[Nodejs] Nodejs, MongoDB, TwitterAPI 연동"
categories: 
  - nodejs
  - mongoDB
  - API
tags:
  - nodejs
  - mongoDB
  - API
---

# 1. server.js
~~~ javascript
const http = require('http');
const url = require('url');
const fs = require('fs');
//const db = require('./db.js');
const getTweet = require('./getTweet.js');

http.createServer((request, response) => {
  const path = url.parse(request.url, true).pathname; // url에서 path 추출
  if (request.method === 'GET') { // GET 요청이면
    if (path === '/test') {
      response.writeHead(200,{'Content-Type':'text/html'});
      fs.readFile(__dirname + '/main.html', (err, data) => {
        if (err) {
          return console.error(err);
        }
        response.end(data, 'utf-8');
      });
      //
    } else if (path === '/') { // 주소가 /이면
      response.setHeader('Content-Type', 'text/html'); // header 설정
      getTweet.getTweets(44196397, (tweets) => {
        response.write(JSON.stringify(tweets));
        //response.write(JSON.stringify(getTweet.getTweets(44196397).body));
        response.end('the end!'); // 정보 탑재 후 브라우저로 전송
      });
    } else { // 매칭되는 주소가 없으면
      response.statusCode = 404; // 404 상태 코드
      response.end('주소가 없습니다');
    }
  }
}).listen(8080);
~~~


# 2. db.js
~~~ javascript
/* db.js */
var mongoose = require('mongoose');
mongoose.connect('mongodb://localhost:27017/test2'); // 기본 설정에 따라 포트가 상이 할 수 있습니다.
var db = mongoose.connection;
db.on('error', console.error.bind(console, 'connection error:'));
db.once('open', function callback () {
	console.log("mongo db connection OK.");
});

var testSchema = mongoose.Schema({
	name: String
});

testSchema.methods.speak = function () {
	var greeting = this.name
	? "Meow name is " + this.name
	: "I don't have a name"
	console.log("speak() - " + greeting);
}

var TestModel = mongoose.model("TestModel", testSchema);

var testIns = new TestModel({ name: "testIns"});

testIns.save(function(err, testIns){
	if(err) return console.error(err);
	testIns.speak();
});

TestModel.find(function(err, models){
	if(err) return console.error(err);
	console.log("find() - "+models);
});

TestModel.find({name:/^testIns/});
~~~

# 3. getTweet.js
~~~ javascript
var request = require('request');

function getTweets(id, callback) {
    let userId = id;
    const url = `https://api.twitter.com/2/users/${userId}/tweets`;
    const bearerToken = "트위터에서 받은 토큰";
    const options = {
        uri: url,
        qs: {
            max_results:5
        },
        headers: {
            "User-Agent": "v2UserTweetsJS",
            "authorization": `Bearer ${bearerToken}`
        }
    };

    request(options, function(error, response, body){
        if (!error && response.statusCode == 200) {
            callback(body);
        }
        else {
            callback(error);
            console.log(error);
        }
    });

}

exports.getTweets = getTweets;
~~~
