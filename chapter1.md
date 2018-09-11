## package.json



```js
{
  "name": "source",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "axios": "^0.16.2",
    "babel-polyfill": "^6.26.0",
    "bluebird": "^3.5.1",
    "mz": "^2.7.0"
  }
}

```



## Iterator



```js
// 배열.values
require("babel-polyfill");

class Log {
  constructor(){
    this.messages = [];
  }
  add(message){
    this.messages.push({message, timestamp: Date.now()});
  }
  // 로그 기록을 순회 처리하고 싶은 경우 이터레이터 함수를 구현한다.
  [Symbol.iterator](){
    // 배열은 이미 Iterator 객체를 반환하도록 행동합니다.
    // {value, done} 프로퍼티가 있는 객체를 반환하는 next 메소드를 가진 객체를 반환합니다.
    return this.messages.values();
  }
}

const log = new Log();
console.log(log);
log.add("first");
log.add("second");
log.add("third");
console.log(log);

// console.log(log);

for (let item of log) {
  console.log(`${item.message} @ ${item.timestamp}`)
}
```



```js
require("babel-polyfill");

class Log {
  constructor(){
    this.messages = [];
  }
  add(message){
    this.messages.push({message, timestamp: Date.now()});
  }
  [Symbol.iterator](){
    // 직접 이터레이터 프로토콜을 구현해 봅니다.
    let i = 0;
    const messages = this.messages;
    // next 함수가 존재하는 객체가 이터레이터입니다.
    return {
      next() {
        if (i >= messages.length) {
          return { value: undefined, done: true };
        }
        return { value: messages[i++], done: false };
      }
    }
  }
}

const log = new Log();
log.add("first");
log.add("second");
log.add("third");

// console.log(log);

for (let item of log) {
  console.log(`${item.message} @ ${item.timestamp}`)
}

```



## Generator



```js
// 제너레이터란 이터레이터를 사용해 자신의 실행을 제어하는 함수입니다.
// 1. 함수의 실행을 여러 단계로 나누어 함수의 실행을 제어할 수 있도록 합니다.
// 제너레이터는 호출한 즉시 실행되지 않고 대신 이터레이터를 반환합니다.
// 제너레이터는 호출자에게 제어권을 yield 로 넘길 수 있습니다.
// 2. 실행 중인 다른 함수와 대화(값을 리턴하거나 파라미터를 받는다)합니다.

function* rainbow(){
  yield 'red';
  yield 'orange';
  yield 'yellow';
}

const iter = rainbow();

console.log(iter.next());
console.log(iter.next());
console.log(iter.next());
console.log(iter.next());

console.log('---------');

for (let color of rainbow()) {
  console.log(color);
}
```



```js
// 호출자가 제너레이터의 처리 단계마다 정보를 전달하여 처리방식을 변경할 수 있습니다.
function* interrogate(){
  const name = yield 'What is your name?';
  const color = yield 'What is your favorite color?';
  return `${name}'s favorite color is ${color}.`;
}

const iter = interrogate();

var cycle = iter.next();
console.log(cycle.value);

cycle = iter.next('Aaron');
console.log(cycle.value);

cycle = iter.next('blue');
console.log(cycle.value);
```



```js
// 제너레이터는 연산을 지연시켰다가 필요할 때만 수행하게 만들 수 있습니다.
function* abc(){
  yield 'a';
  yield 'b';
  // 특정 조건일 때 제너레이터의 수행을 중단할 수 있다.
  if (true) {
    return;
  }
  yield 'c';
}

const iter = abc();

console.log(iter.next());
console.log(iter.next());
console.log(iter.next());

```



## Async



```js
const fs = require('fs');

function nfcall(f, ...args){
  return new Promise(function(resolve, reject){
    f.call(null, ...args, function(err, ...args){
      if (err) {
        return reject(err);
      }
      resolve(args.length < 2 ? args[0] : args);
    });
  });
}

function ptimeout(delay){
  return new Promise(function(resolve, reject){
    setTimeout(resolve, delay);
  });
}

// 기초적인 제너레이터 재귀 실행기입니다.
// 제너레이터 함수를 파라미터로 전달하여 실행합니다.
// yield 로 값을 넘기고 이터레이터에서 next 를 호출할 때까지 대기합니다.
// 이 함수는 위 과정을 재귀적으로 반복합니다.
// ES7에서 도입한 await 키워드는 grun 함수와 같은 기능을 제공합니다.
function grun(g){
  const it = g();

  (function iterate(val){
    const x = it.next(val);
    if (!x.done) {
      if (x.value instanceof Promise) {
        x.value.then(iterate).catch(err => it.throw(err));
      } else {
        setTimeout(iterate, 0, x.value);
      }
    }
  })();
}

function* theFutureIsNow(){
  // 파일 3개를 읽는다.
  const dataA = yield nfcall(fs.readFile, __dirname+'/a.txt');
  const dataB = yield nfcall(fs.readFile, __dirname+'/b.txt');
  const dataC = yield nfcall(fs.readFile, __dirname+'/c.txt');
  // 잠시 대기한다.
  yield ptimeout(5*1000);
  // 파일에 저장한다.
  yield nfcall(fs.writeFile, __dirname+'/d.txt', dataA+dataB+dataC);
}

grun(theFutureIsNow);

```



```js
const fs = require('fs');

function nfcall(f, ...args){
  return new Promise(function(resolve, reject){
    f.call(null, ...args, function(err, ...args){
      if (err) {
        return reject(err);
      }
      resolve(args.length < 2 ? args[0] : args);
    });
  });
}

function ptimeout(delay){
  return new Promise(function(resolve, reject){
    setTimeout(resolve, delay);
  });
}

function grun(g){
  const it = g();

  (function iterate(val){
    const x = it.next(val);
    if (!x.done) {
      if (x.value instanceof Promise) {
        x.value.then(iterate).catch(err => it.throw(err));
      } else {
        setTimeout(iterate, 0, x.value);
      }
    }
  })();
}

function* theFutureIsNow(){
  // 세 파일을 모두 읽은 다음 잠시 대기한 후 새 파일에 저장한다.
  const data = yield Promise.all([
    nfcall(fs.readFile, __dirname+'/a.txt'),
    nfcall(fs.readFile, __dirname+'/b.txt'),
    nfcall(fs.readFile, __dirname+'/c.txt'),
  ]);
  // const dataA = yield nfcall(fs.readFile, __dirname+'/a.txt');
  // const dataB = yield nfcall(fs.readFile, __dirname+'/b.txt');
  // const dataC = yield nfcall(fs.readFile, __dirname+'/c.txt');
  yield ptimeout(5*1000);
  yield nfcall(fs.writeFile, __dirname+'/d.txt', data[0]+data[1]+data[2]);
}

grun(theFutureIsNow);

```



```js
const fs = require('fs');

// 노드 스타일 콜백은 Q 를 쓰십시오.
function nfcall(f, ...args){
  return new Promise(function(resolve, reject){
    f.call(null, ...args, function(err, ...args){
      if (err) {
        return reject(err);
      }
      resolve(args.length < 2 ? args[0] : args);
    });
  });
}

function ptimeout(delay){
  return new Promise(function(resolve, reject){
    setTimeout(resolve, delay);
  });
}

// 제너레이터 실행기는 co 나 Koa 를 쓰십시오.
// https://github.com/tj/co
// http://koajs.com
function grun(g){
  const it = g();

  (function iterate(val){
    const x = it.next(val);
    if (!x.done) {
      if (x.value instanceof Promise) {
        x.value.then(iterate).catch(err => it.throw(err));
      } else {
        setTimeout(iterate, 0, x.value);
      }
    }
  })();
}

function* theFutureIsNow(){
  let data;
  try {
    // 동시에 실행할 수 있는 부분을 Promise.all 로 처리하십시오.
    data = yield Promise.all([
      nfcall(fs.readFile, __dirname+'/a.txt'),
      nfcall(fs.readFile, __dirname+'/b.txt'),
      nfcall(fs.readFile, __dirname+'/c.txt'),
    ]);
  } catch (e) {
    console.log(e.message);
    throw e;
  }
  // 프라미스는 반드시 결정된다는 보장은 없습니다. 프라미스에 타임아웃을 걸면 이 문제가 해결됩니다.
  yield ptimeout(5*1000);
  try {
    yield nfcall(fs.writeFile, __dirname+'/d.txt', data[0]+data[1]+data[2]);
  } catch (e) {
    console.log(e.message);
    throw e;
  }
}

grun(theFutureIsNow);

```



```js
// const fs = require('fs');

// The mz module provides promisified versions of the core node library.
// https://stackoverflow.com/questions/10058814/get-data-from-fs-readfile/10058879
const fs = require('mz/fs');

function nfcall(f, ...args){
  return new Promise(function(resolve, reject){
    f.call(null, ...args, function(err, ...args){
      if (err) {
        return reject(err);
      }
      resolve(args.length < 2 ? args[0] : args);
    });
  });
}

function ptimeout(delay){
  return new Promise(function(resolve, reject){
    setTimeout(resolve, delay);
  });
}

async function theFutureIsNow() {
  console.log(1); //

  // let data = await Promise.all([
  //   new Promise(function(resolve, reject) {
  //     fs.readFile(__dirname + '/a.txt', 'utf-8', function(err, data) {
  //       if (err) { reject(err); }
  //       resolve(data);
  //     })
  //   }),
  //   new Promise(function(resolve, reject) {
  //     fs.readFile(__dirname + '/b.txt', 'utf-8', function(err, data) {
  //       if (err) { reject(err); }
  //       resolve(data);
  //     })
  //   }),
  //   new Promise(function(resolve, reject) {
  //     fs.readFile(__dirname + '/c.txt', 'utf-8', function(err, data) {
  //       if (err) { reject(err); }
  //       resolve(data);
  //     })
  //   })
  // ]);

  let data = await Promise.all([
    // nfcall(fs.readFile, __dirname+'/a.txt'),
    await fs.readFile(__dirname+'/a.txt'),
    nfcall(fs.readFile, __dirname+'/b.txt'),
    nfcall(fs.readFile, __dirname+'/c.txt'),
  ]);

  console.log(2); //
  await ptimeout(500);
  console.log(3); //

  // await fs.writeFile(__dirname + '/d.txt', data[0] + data[1] + data[2], function(err) {
  //   if (err) { throw err; }
  //   console.log('Done.');
  // });

  await nfcall(fs.writeFile, __dirname+'/d.txt', data.reduce((a, b) => a + b));

  console.log(4); //
}

theFutureIsNow();

```



## Async 10 min



```js
// Async - declares an asynchronous function (async function someName(){...}).
//
// 1. Automatically transforms a regular function into a Promise.
// 2. When called async functions resolve with whatever is returned in their body.
// 3. Async functions enable the use of await.
//
// Await - pauses the execution of async functions. (var result = await someAsyncCall();).
//
// 1. When placed in front of a Promise call, await forces the rest of the code
// to wait until that Promise finishes and returns a result.
// 2. Await works only with Promises, it does not work with callbacks.
// 3. Await can only be used inside async functions.


// Here is a simple example that will hopefully clear things up:
//
// Let's say we want to get some JSON file from our server.
// We will write a function that uses the axios library
// and sends a HTTP GET request to https://tutorialzine.com/misc/files/example.json.
//
// We have to wait for the server to respond, so naturally this HTTP request will be asynchronous.
//
// Below we can see the same function implemented twice.
// First with Promises, then a second time using Async/Await.


const axios = require('axios');

// Promise approach
function getJSON() {
  return new Promise(function(resolve) {
    axios.get('https://tutorialzine.com/misc/files/example.json')
      .then(function(response) {
      // The data from the request is available in a .then block
      // We return the result using resolve.
      resolve(response.data);
    });
  });
}

// let promise = getJSON();
//
// promise.then((data) => {
//   console.log(data[0]);
// });


// Async/Await approach
// The async keyword will automatically create a new Promise and return it.
async function getJSONAsync(){
    // The await keyword saves us from having to write a .then() block.
    let response = await axios.get('https://tutorialzine.com/misc/files/example.json');

    // The result of the GET request is available in the json variable.
    // We return it just like in a regular synchronous function.
    return response.data;
}

let promise = getJSONAsync();

promise.then((data) => {
  console.log(data[0]);
});


// It's pretty clear that the Async/Await version of the code is much shorter
// and easier to read.
// Both functions are completely identical -
// they both return Promises and resolve with the JSON response from axios. 

```



```js
// So, does Async/Await make promises obsolete?
//
// No, not at all.
// When working with Async/Await,
// we are still using Promises under the hood.
//
// A good understanding of Promises will actually help you in the long run and is highly recommended.
//
// There are even use cases where Async/Await doesn't cut it
// and we have to go back to Promises for help.
// One such scenario is when we need to make multiple independent asynchronous calls
// and wait for all of them to finish.


// If we try and do this with async and await, the following will happen:

// async function getABC() {
//   let A = await getValueA(); // getValueA takes 2 second to finish
//   let B = await getValueB(); // getValueB takes 4 second to finish
//   let C = await getValueC(); // getValueC takes 3 second to finish
//   return A * B * C;
// }

// Each await call will wait for the previous one to return a result.
// Since we are doing one call at a time,
// the entire function will take 9 seconds from start to finish (2+4+3).
//
// This is not an optimal solution,
// since the three variables A, B, and C aren't dependent on each other.
// In other words we don't need to know the value of A before we get B.
// We can get them at the same time and shave off a few seconds of waiting.
//
// To send all requests at the same time a Promise.all() is required.
// This will make sure that we still have all the results before continuing,
// but the asynchronous calls will be firing in parallel, not one after another.

// async function getABC() {
//   // Promise.all() allows us to send all requests at the same time.
//   let results = await Promise.all([getValueA, getValueB, getValueC]);
//   return results.reduce((total, value) => total * value, 1);
// }

// This way the function will take much less time.
// The getValueA and getValueC calls will have already finished by the time getValueB ends.
// Instead of a sum of the times,
// we will effectively reduce the execution
// to the time of the slowest request (getValueB - 4 seconds).

```



```js
// Handling Errors in Async/Await
// 
// Another great thing about Async/Await is that
// it allows us to catch any unexpected errors in a good old try/catch block.
// We just need to wrap our await calls like this:

async function doSomethingAsync() {
  try {
    // This async call may fail.
    let result = await someAsyncCall();
  } catch (error) {
    // If it does we will catch the error here.
  }
}

// The catch clause will handle errors provoked by the awaited asynchronous calls
// or any other failing code we may have written inside the try block.
// 
// If the situation requires it, we can also catch errors upon executing the async function.
// Since all async functions return Promises
// we can simply include a .catch() event handler when calling them.

// Async function without a try/catch block.
async function doSomethingAsync() {
  // This async call may fail.
  let result = await someAsyncCall();
  return result;
}

// We catch the error upon calling the function.
doSomethingAsync().then(successHandler).catch(errorHandler);

// It's important to choose which method of error handling you prefer and stick to it.
// Using both try/catch and .catch() at the same time will most probably lead to problems.

```



