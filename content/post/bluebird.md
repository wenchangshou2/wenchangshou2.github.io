---
title: "Bluebird"
date: 2020-02-11T14:34:15+08:00
tags: [ "前端","js"]
---

# bluebird 笔记

## 将传统方法封闭成promise方式

 Promise.promisify能够将原有的函数封装成promise的调用方式。
 下面的是传统的异步调用方式。

```js
import {readFile} from 'fs'
readFile('file1.txt',function(err,data){
    if(err){
        console.log('read file error');
    }else{
        console.log(`read file data:${data.toString()}`);
    }
})
```

利用Promise.promisify转换函数后的调用方式

```js
let Promise = require("bluebird")
var readFile = Promise.promisify(require("fs").readFile);
readFile("file1.txt","utf8").then(function(contents){
    console.log(`read file data:${contents}`);
}).catch(function(e){
    console.log('error read file',e);
})
```

除了用Promise.prosisify之后bluebird还提供了一种能够转换整个调用对象的方法 Promise.promisifyAll。

```js
let Promise = require("bluebird")
var fs = Promise.promisifyAll(require("fs"));
fs.readFile("file1.txt","utf8").then(function(contents){
    console.log(`read file data:${contents}`);
}).catch(function(e){
    console.log('error read file',e);
})
```

## 处理多个promise返回值

```javascript
var Promise = require('bluebird');
var fs=Promise.promisifyAll(require("fs"))
Promise.all([
    fs.readFileAsync("file1.txt"),
    fs.readFileAsync("file2.txt")
]).spread(function(file1text, file2text) {
    if (file1text === file2text) {
        console.log("files are equal");
    }
    else {
        console.log("files are not equal");
    }
});
```

上面的代码的代码是异步读取file1.txt和file2.txt文件，并且将文件的内容存入到file1text和file2text中，一旦两个文件的内容相同输出files are equal,不相同输出files are not equal。

## 多个promise 处理
``` 
 Promise.join(
    Promise<any>|any values...,
    function handler
) -> Promise
```

```js
var Promise = require("bluebird");
var join = Promise.join;

join(getPictures(), getComments(), getTweets(),
    function(pictures, comments, tweets) {
    console.log("in total: " + pictures.length + comments.length + tweets.length);
});
```

上面的的代码调用 getPictures(), getComments(), getTweets()三个异步的操作，同时输出操作的结果
下面的代码完成赞同于上面的代码，只不过将Promise.join换成了Promise.all，并且传一个数组

```js
var Promise = require("bluebird");
var join = Promise.join;

Promise.all([getPictures(), getComments(), getTweets()],
    function(pictures, comments, tweets) {
    console.log("in total: " + pictures.length + comments.length + tweets.length);
});
```

## Promise.props

```js
Promise.props({
    file1: fs.readFileAsync("file1.txt"),
    file2: fs.readFileAsync("file2.txt")
}).then(function(result) {
    console.log(result.file1,result.file2);
});
```

props能将将返回的结果放置在一个Object对象当中，可以通过result.file1来访问fs.readFileAsync("file1.txt")的文件内容

## Promise.some

```js
Promise.some([
    ping("ns1.example.com"),
    ping("ns2.example.com"),
    ping("ns3.example.com"),
    ping("ns4.example.com")
], 2).spread(function(first, second) {
    console.log(first, second);
});
```

上面通过Promise.some方法来调用四次ping请求，输出其中最快的二次请求的信息。

### Promise.finally

finally方法有点类似于c++中的try catch..finaly，不过promise过程是否成功，都将调用finally的方法，可以在这个方法里面写一些资源的释放之类的语句。

```javascript
var Promise=require('bluebird')
function _setTimeout(time){
  return new Promise((resolve, reject) => {
    setTimeout(resolve('resolve'),time);
  }).finally(()=>{
    console.log('success');
  })
}
_setTimeout(2000).then((content)=>{
  console.log(content);
})
```

### Promises.map

这个方法可以遍历Object,再通过遍历的值来直接调用 Promise方法



```js
var Promise=require('bluebird')
var fs=Promise.promisifyAll(require("fs"))
var promise=[]
var filesNames=[]
for(var i=1;i<=2;i++){
  promise.push(fs.readFileAsync(`file${i}.txt`))
  filesNames.push(`file${i}.txt`)
}
//使用传统的Promise.all方法
Promise.all(promise).then(function(){
  console.log('all');
})
//通过map来批量调用
Promise.map(filesNames, (fileName) => {
  return fs.readFileAsync(fileName)
}).then(() => {
  console.log('done');
})
```

### Promise.reduct

这个方法可以顺序的执行promise

```js
Promise.reduce(["file1.txt", "file2.txt", "file3.txt"], function(total, fileName) {
    return fs.readFileAsync(fileName, "utf8").then(function(contents) {
        return total + parseInt(contents, 10);
    });
}, 0).then(function(total) {
  console.log(total);
});
```

上面的代码段是，顺序的读取file1,file2,file3三个文件的内容，并且将内容转换成整形，最终输出累加后的数据。