---
layout: post
status: publish
published: true
title: JaveScript的奇葩写法规则
description: “JaveScript的奇葩写法规则 js语法 js奇怪语法”
excerpt_separator: ===
categories:
- 前端技术
- 其他技术
tags:
- js
- 语法
---

#JaveScript的奇葩写法规则

1. ###=> 箭头函数

		两个箭头同时使用 arg1 => arg2 =>{return xxx;}
		可以理解为
		function( arg1 )
		{
			return function(arg2)
			{
				return xxx;
			}
		}
		也就是可以是 (arg1,arg2)=>{return xxx;}的简写
		也就是function( arg1, arg2 ){ return xxx; }的意思

	===

2. ###... 剩余数据

	######数组中的剩余数据

		[a, b, ...rest] = [10, 20, 30, 40, 50];
		console.log(a); // 10
		console.log(b); // 20
		console.log(rest); // [30, 40, 50]

	######对象中的剩余数据

		({a, b, ...rest} = {a: 10, b: 20, c: 30, d: 40});
		console.log(a); // 10
		console.log(b); // 20
		console.log(rest); // {c: 30, d: 40}


3. ###[] 数组分配赋值用法

	######数组分配1

		var x = [1, 2, 3, 4, 5];
		var [y, z] = x;
		console.log(y); // 1
		console.log(z); // 2

	######数组分配2

		var foo = ['one', 'two', 'three'];
		var [one, two, three] = foo;
		console.log(one); // "one"
		console.log(two); // "two"
		console.log(three); // "three"

	######数组赋值默认值用法

		var a, b;
		[a=5, b=7] = [1];
		console.log(a); // 1
		console.log(b); // 7

	######用数组做变量置换

		var a = 1;
		var b = 3;
		[a, b] = [b, a];
		console.log(a); // 3
		console.log(b); // 1

	######省略赋值用法

		var [a, ...b] = [1, 2, 3];
		console.log(a); // 1
		console.log(b); // [2, 3]

4. ###{} 对象分配用法
	
	######对象分配赋值

		var o = {p: 42, q: true};
		var {p, q} = o;
		console.log(p); // 42
		console.log(q); // true

	######默认值用法

		var {a = 10, b = 5} = {a: 3};
		console.log(a); // 3
		console.log(b); // 5

	######指定key赋值+默认值

		var {a:aa = 10, b:bb = 5} = {a: 3};
		console.log(aa); // 3
		console.log(bb); // 5

	######指定属性key解析赋值

		let key = 'z';
		let {[key]: foo} = {z: 'bar'};
		console.log(foo); // "bar"

	######对象中剩余赋值法

		let {a, b, ...rest} = {a: 10, b: 20, c: 30, d: 40}
		a; // 10 
		b; // 20 
		rest; // { c: 30, d: 40 }

	######弱变量定义与赋值

		const foo = { 'fizz-buzz': true };
		const { 'fizz-buzz': fizzBuzz } = foo;
		console.log(fizzBuzz); // "true"

5. ### Object.assign() 合并对象

	###### 将源对象（ source ）的所有可枚举属性，复制到目标对象（ target ）

		var target = { a: 1 };
		var source1 = { b: 2 };
		var source2 = { c: 3 };
		Object.assign(target, source1, source2);
		target // {a:1, b:2, c:3}

	###### 如果目标对象与源对象有同名属性，或多个源对象有同名属性，则后面的属性会覆盖前面的属性

		var target = { a: 1, b: 1 };
		var source1 = { b: 2, c: 2 };
		var source2 = { c: 3 };
		Object.assign(target, source1, source2);
		target // {a:1, b:2, c:3}

	###### Object.assign方法实行的是浅拷贝，而不是深拷贝。也就是说，如果源对象某个属性的值是对象，那么目标对象拷贝得到的是这个对象的引用。

		var obj1 = {a: {b: 1}};
		var obj2 = Object.assign({}, obj1);
		obj1.a.b = 2;
		obj2.a.b // 2

	###### 对于这种嵌套的对象，一旦遇到同名属性，Object.assign的处理方法是替换，而不是添加。

		var target = { a: { b: 'c', d: 'e' } }
		var source = { a: { b: 'hello' } }
		Object.assign(target, source)
		// { a: { b: 'hello' } }

	###### Object.assign 常见用途

		为对象添加属性
		class Point {
			constructor(x, y) {
				Object.assign(this, {x, y});
			}
		}

		为对象添加方法
		Object.assign(SomeClass.prototype, {
			someMethod(arg1, arg2) {
			···
			},
			anotherMethod() {
			···
			}
		});
		//  等同于下面的写法
		SomeClass.prototype.someMethod = function (arg1, arg2) {
		···
		};
		SomeClass.prototype.anotherMethod = function () {
		···
		};

		克隆对象
		function clone(origin) {
			return Object.assign({}, origin);
		}

		合并多个对象
		const merge =(target, ...sources) => Object.assign(target, ...sources);

		为属性指定默认值
		const DEFAULTS = {
			logLevel: 0,
			outputFormat: 'html'
		};
		function processContent(options) {
			let options = Object.assign({}, DEFAULTS, options);
		}


