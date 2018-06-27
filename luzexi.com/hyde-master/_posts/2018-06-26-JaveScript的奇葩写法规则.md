---
layout: post
status: publish
published: true
title: JaveScript的奇葩写法规则
excerpt_separator: ===
categories:
- 前端技术
tags:
- js
- 语法
---

#JaveScript的奇葩写法规则

1. => 箭头函数
	两个箭头同时使用 arg1 => arg2 =>{return xxx;}
	可以理解为 function( arg1 ){ return function(arg2){ return xxx; } }
	也就是可以是 (arg1,arg2)=>{return xxx;}的简写
	也就是function( arg1, arg2 ){ return xxx; }的意思

===

2. ... 剩余数据
	数组中的剩余数据
	[a, b, ...rest] = [10, 20, 30, 40, 50];
	console.log(a); // 10
	console.log(b); // 20
	console.log(rest); // [30, 40, 50]

	对象中的剩余数据
	({a, b, ...rest} = {a: 10, b: 20, c: 30, d: 40});
	console.log(a); // 10
	console.log(b); // 20
	console.log(rest); // {c: 30, d: 40}

3. [] 数组分配赋值用法
	数组分配1
	var x = [1, 2, 3, 4, 5];
	var [y, z] = x;
	console.log(y); // 1
	console.log(z); // 2

	数组分配2
	var foo = ['one', 'two', 'three'];
	var [one, two, three] = foo;
	console.log(one); // "one"
	console.log(two); // "two"
	console.log(three); // "three"

	数组赋值默认值用法
	var a, b;
	[a=5, b=7] = [1];
	console.log(a); // 1
	console.log(b); // 7

	用数组做变量置换
	var a = 1;
	var b = 3;
	[a, b] = [b, a];
	console.log(a); // 3
	console.log(b); // 1

	省略赋值用法
	var [a, ...b] = [1, 2, 3];
	console.log(a); // 1
	console.log(b); // [2, 3]

4. {} 对象分配用法
	对象分配赋值
	var o = {p: 42, q: true};
	var {p, q} = o;
	console.log(p); // 42
	console.log(q); // true

	默认值用法
	var {a = 10, b = 5} = {a: 3};
	console.log(a); // 3
	console.log(b); // 5

	指定key赋值+默认值
	var {a:aa = 10, b:bb = 5} = {a: 3};
	console.log(aa); // 3
	console.log(bb); // 5

	指定属性key解析赋值
	let key = 'z';
	let {[key]: foo} = {z: 'bar'};
	console.log(foo); // "bar"

	对象中剩余赋值法
	let {a, b, ...rest} = {a: 10, b: 20, c: 30, d: 40}
	a; // 10 
	b; // 20 
	rest; // { c: 30, d: 40 }

	弱变量定义与赋值
	const foo = { 'fizz-buzz': true };
	const { 'fizz-buzz': fizzBuzz } = foo;
	console.log(fizzBuzz); // "true"

Thanks for reading.