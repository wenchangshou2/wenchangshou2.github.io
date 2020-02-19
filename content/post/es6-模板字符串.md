---
title: "Es6 模板字符串"
date: 2020-02-11T14:11:28+08:00
---


es6 新的特殊支持嵌入表达式，自动会处于``内所有的字符串。
<!--more-->

### 多行字符串

```js
\\传统的多行字符串
console.log('string line1 \n\
string line2')
\\es6多行字符串
console.log(`string line
string lin2`);

```

### 表达式插入

传统的方式采用的是字符串拼接的方式进行，通过+号进行拼接

```js
// 传统方式
 let x=1;
 let y=2;
 console.log('x='+(a+b)+';y='+(a*b))
 //es6 语法
 console.log('x='+(a+b)+';y='+(a*b));
 console.log(`x=${x+y},y=${x*y}`);
```

### 带标签的模板字符串

```js
//strings 存放所有的字符串字面的数组
//values 存放第一个数组紧随的参数
 function tag3(strings,...values){
 					  let a = 5;
					  let b = 10;
                console.log(strings[0]);//hello
                console.log(strings[1]);//world
                console.log(strings[2]);//
                console.log(values[0]);//15
                console.log(values[1]);//50
                return "hello world"
 }
  tag3 `Hello ${a+b} world ${ a*b }`;
  //out:hello world
```

通过传入的参数来返回一个函数

```js
function template(strings,...keys){
return (function(...values){
		var dict=values[values.length-1]||{};  //A                 
		var result=[strings[0]];
		keys.forEach(function(key,i){
		var value=Number.isInteger(key)?values[key]:dict[key];              result.push(value,strings[i+1]);
});
return result.join(' ');
});
}
var t1Closure=template `${0}${1}${0}!`;//返回的是一个函数
console.log(t1Closure('Y','A'));//Y A Y
```

### 不转义字符串

在String.raw()输出不转义的字符串,输出原始的字符串

```js
function tag(strings, ...values) {
/*
	倘若不采用raw输出的会直接进行换行 
*/
  console.log(strings.raw[0]); 
  // "string text line 1 \n string text line 2"
}

tag`string text line 1 \n string text line 2`;
```

#### 实例

```js
//判断是否具有权限，倘若没有抛出一个异常
//${user.name}和${action}是两个点占位号，会自动将值带入到该字符串
function authorize(user, action) {
  if (!user.hasPrivilege(action)) {
    throw new Error(
      `用户 ${user.name} 未被授权执行 ${action} 操作。`);
  }
}
```