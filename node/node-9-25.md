#nodejs
文件夹名字最好不要起和已有的模块名相同的名字，有时也可以同名，但是比如如果在这个文件夹里用express模块，这个文件夹的名字就不能为express

##1.概念
> nodejs不是一个库，是一个基于chromeV8引擎的运行js的环境，或者叫运行时（runtime）

**npm**:包管理器

> npm官网www.npmjs.com/  如果需要哪个node的模块，都可以去这里找

##2.模块

作用：把多个类似的功能封装在一起

之前咱们用匿名函数自执行来模拟模块
	
		let t = (function(){
			function add(){

			}
			function fn(){

			}
			return {
				add,
				fn
			}
		})()

在node里，一个文件就是一个模块

###2-1.定义模块


要暴露本模块的api，要把暴露的函数挂载再module.exports上，module.exports表示模块对外输出的值

		function add(a, b) {
			return a + b
		}
		function isString (str) {
			return typeof str === 'string'
		}
		/* module.exports.add = add;
		module.exports.isString = isString; */
		
		// module.exports默认是一个对象，但是它可以被改写
		module.exports = {
			add,
			isString
		}

###2-2.加载（使用）一个模块：

####2-2-a.加载一个**文件模块**

require("相对路径或者绝对路径")  
一定要加相对路径或者绝对路径

require是一个函数，当使用其加载模块时，其返回值是加载的模块里的module.exports这个对象

比如在index.js里要用到utils.js模块，加载utils.js文件模块

	let u = require('./utils.js')

####2-2-b.加载内置模块

**内置模块**：无需使用npm下载，安装node时候已经安装好了，可以直接加载，比如http模块，fs模块

比如http模块，fs模块
加载http模块

	let http = require('http')   不用写路径

####2-2-c.加载文件夹模块

**文件夹模块**：

- 第三方模块，既不是内置的，也不是文件模块，是第三方模块（比如express,jquery），需要安装才能使用，安装之后会在项目目录下创建node_modules文件夹（存放第三方模块），并且第三方的依赖模块也会安装进去

		let $ = require('jquery');
- 第三方模块不要和内置模块重名
- 文件夹的src文件夹都有一个package.json，如果是自己手动创建的文件夹模块，要自己创建package.json文件
	- 创建方法：进入src目录，打开命令窗口，npm init，一路回车

###2-3.查找模块

require('http')


###2-4.模块的几个规范的历史

在浏览器端没有文件的概念，js文件要通过`<script>`标签的src引入，如果多的话要写很多`<script>`标签 => 能否把一个js文件作为一个模块？=> **模块加载器**

1. seajs国内（依赖就近的原则）,用的是CMD规范，用在浏览器端
	1. 引入模块：require()
	1. 定义模块：defined
2. nodejs用的是Common JS规范。用在服务端
	1. 如果一个既想在浏览器环境下运行，也想在node环境下运行，
就要用Common JS规范
3. SE6的module，用的是自己的规范，用在服务端，即将可以用在浏览器端

##3.http模块

1. 加载http模块
2. 创建服务
3. 监听端口
4. 注意如果修改js文件，要重新起服务

**例子1.请求简单数据**

		let http = require('http'); // 1.加载http模块
		
		// 2.创建服务
		// request, response
		let app = http.createServer((req,res) => {
			console.log("有请求来了");
			/*
				当有服务请求过来了，会触发这个回调函数
				回调函数接收两个参数：
					request:是一个对象，保存的是请求的信息，简写为req
					response: 是一个对象，保存的是响应的功能，简写为res
			*/
		
			res.write('ok'); //向客户端响应内容
			res.end(); // 表示请求结束，否则会一直请求
		})
		
		app.listen(3000, () => {
			console.log("服务启动了");
		})

**例子2：请求文件**

* 后端需要读取并返回html文件里面的内容，而不是返回html文件
用到文件模块：fs模块

* 怎么判断请求的是哪个文件?，从req里可以找到请求的地址的路径：`req.url`

* 读取文件的方法：
	* **fs.readFile(path[, options], callback)**：
**异步地**读取一个文件的全部内容
* 在node里的回调函数，都是错误前置（Error-first），如果没有错误那么error就是null

**场景：请求index.html和miaov.html**

	let http = require('http'); // 加载http模块
	let fs = require('fs'); // 需要读取文件
	// 创建服务
	let app = http.createServer((req,res) => {
		/*
			当有服务请求过来了，会触发这个函数
			回调函数接收两个参数：
				request:是一个对象，保存的是请求的信息，简写为req
				response: 是一个对象，保存的是响应的功能，简写为res
		*/
		console.log('有请求来了')
	
		// 如果后端有一个文件index.html，想要请求index.html，响应的时候怎么做？
		// 怎么判断请求的是index.html?，从req里可以找到请求的地址的路径：req.url
		console.log(req.url);  // 拿到地址的路径
		if(req.url === '/index.html'){
			// 如果返回index.html，需要读取index.html里面的内容发送，而不是发生html文件
			// 在node里的回调函数，都是错误前置（Error-first），如果没有错误那么error就是null
			fs.readFile('./views/index.html',(error,data) => {
				if(error){
					console.log(error)
				}else{
					console.log(data); // buffer类型的里面存的额是二进制的
					console.log(data.toString('utf-8'))
					res.write(data); //向客户端响应内容，是二进制的
					res.end();
				}
			})
		}else if(req.url === '/miaov.html'){
			fs.readFile('./views/miaov.html', (error,data) => {
				if(error){
					console.log(error);
				}else{
					console.log(data);  // buffer类型的里面存的额是二进制的
					console.log( data.toString('utf-8') );
					res.write(data); //向客户端响应内容
					res.end();
				}
			})
		}
		
	})
	
	app.listen(3000,()=>{
		console.log('服务启动了')
	})


**请求index.html和miaov.html之封装函数**

- 封装函数getHtmlData：
	- 传入路径和回调函数，读取该路径对应的文件并且响应给客户端
	- 因为响应的数据可能不同，所以不能在这里写死，要把决定权交给使用者，所以这里设置一个回调函数

			let http = require('http');
			let fs = require('fs');
			
			// 创建服务
			// request, response
			let app = http.createServer((req,res) => {
				// 请求的是那个页面，index.html,响应的时候怎么做？
			
				// 读取index.html这个页面的内容，发送给客户端
			
				console.log(req.url);  // 拿到地址的路径
			
				if(req.url === '/index.html'){
					// 返回index.html，需要读取index.html里面的内容发送
			
					getHtmlData('./views/index.html', function (data){
						console.log(data);
						res.write(data);	
						res.end();
					})
						
				}else if(req.url === '/miaov.html'){
					getHtmlData('./views/miaov.html',function (data){
						console.log(data);
						res.write(data);	
						res.end();
					})
				}	
			})
			
			function getHtmlData(filePath, callback){
				fs.readFile(filePath, (error,data) => {
					if(error){
						console.log(error);
					}else{
						// 向客户端相应数据，因为响应的数据可能不同，所以不能在这里写死，要把决定权交给使用者，所以这里设置一个回调函数
						callback(data)
					}
				})
			}
			
			app.listen(3000, () => {
				console.log("服务启动了");
			})

**读文件是异步操作  和ajax类似，在读取文件时，可以先执行下面的代码，异步需要写回调函数来拿数据**
=>

**引出的问题：如果需求是读完文件a再读文件b再读……**

* 方法1：需要fs.readFile嵌套很多层fs.readFile，但是如果嵌套太多会引发回调地狱；
* 方法2：同步，但是同步必须得文件读取完毕才能执行下面的代码

不想嵌套，又想保证读取文件的顺序

* 方法3：用promise async await实现不嵌套的异步，既保证读取文件的顺序，又不嵌套，也不阻塞下面代码的执行

**用异步还是同步根据需求来决定用哪个**

**方法3：用promise async await实现不嵌套的异步**

		let http = require('http');
		let fs = require('fs');
		
		// 创建服务
		// request, response
		let app = http.createServer((req,res) => {
			// 请求的是那个页面，index.html,响应的时候怎么做？
		
			// 读取index.html这个页面的内容，发送给客户端	
			console.log(req.url);  // 拿到地址的路径:/index.html 	还有小图标的地址/favicon.ico
		
			if(req.url === '/index.html'){
				// 返回index.html，需要读取index.html里面的内容发送
				let d = getHtmlData('./views/index.html');	
				d.then((data) => {
					console.log(0)
					res.write(data);
					res.end();
				})
				console.log(123)
			}else if(req.url === '/miaov.html'){
				let d = getHtmlData('./views/miaov.html');
				d.then((data) => {
					res.write(data);
					res.end();
				})
			}	
		})
		
		// 函数readFilePromise的作用：传进来一个路径，返回一个Promise对象
		function readFilePromise(filePath){
			return new Promise((resolve,reject) => {
				fs.readFile(filePath, (error,data) => {
					if(error){
						reject(error)
					}else{
						resolve(data)
					}
				})
			})	
		}
		// 异步函数getHtmlData的作用：传进去一个路径，
		async function getHtmlData(filePath){
			// 如果await后面是一个Promise对象，那么它会等待 Promise 的解析并返回解析的的值。
			// 如果await后面不是一个 Promise，它将该值转换为已解决的Promise，并等待它。
			return await readFilePromise(filePath)
		}
		
		app.listen(3000, () => {
			console.log("服务启动了");
		})

以上代码的执行结果：
我先执行
服务启动了
/index.html
123
0
/favicon.ico

**方法2：作者给咱们提供了一种同步的方法:**fs.readFileSync(path[, options])** ；但是同步必须得文件读取完毕才能执行下面的代码**



		let http = require('http');
		let fs = require('fs');
		
		// 同步的
		let d1 = fs.readFileSync('./views/index.html');
		let d2 = fs.readFileSync('./views/miaov.html');
		console.log(d1);
		console.log("我先执行");
		
		// 创建服务
		// request, response
		let app = http.createServer((req,res) => {
			// 请求的是那个页面，index.html,响应的时候怎么做？
		
			// 读取index.html这个页面的内容，发送给客户端
		
			console.log(req.url);  // 拿到地址的路径:/index.html 	还有小图标的地址/favicon.ico
		
			if(req.url === '/index.html'){
				// 返回index.html，需要读取index.html里面的内容发送
				res.write(d1);
				res.end();
			}else if(req.url === '/miaov.html'){
				res.write(d2);
				res.end();
			}	
		})
		app.listen(3000, () => {
			console.log("服务启动了");
		})




###------------下午---------------

##4.文件系统
let fs = require('fs');
###4-1.异步形式
非阻塞式
如果一个地方报错，那么下面的代码仍然可以执行

		fs.readFile('./abc.txt', (err,data) => {
			if(err){
				console.log(err);
			}else{
				console.log(data.toString());
			}
		})
		
		console.log('我先执行');
###4-2.同步形式
阻塞式

因为如果有需求：某段代码执行完之后再执行另一段代码，回调函数嵌套，嵌套太多形成回调地狱，所以提供了一种同步的方式

有问题：如果一个地方报错，那么下面的代码都不能执行了

解决：try catch

		try{
			let d = fs.readFileSync('./abcde.txt');
			console.log(d.toString());
		}catch(err){
			console.log(err);
		}
		console.log('我执行了');

如果是动态改变的最好都用try catch捕获一下
###4-3.例子：保存一个数据到一个文件中

####需求：往users.json里写入数据

- users.json存在
	- 	先读取数据，新的数据往后追加，不覆盖原有的数据
- users.json不存在
	- 	创建写入文件，并写入数据
	- 	fs.writeFile(文件，要写入的数据[,option],回调函数)
		- 如果文件已经存在，就创建新文件覆盖原文件（由第三个参数的flag的值决定）

代码

		let fs = require('fs');

		let arr = [
		  {
		    name: '小明',
		    age: 30,
		    sex: '男'
		  }
		]
		
		let isExist = fs.existsSync('./data/users.json'); // 判断文件是否存在
		
		if(isExist){ // 如果存在
		  // 先读取数据
		  let d = fs.readFileSync('./data/users.json'); // 是拿到的是Buffer格式
		  let d2 = d.toString(); // 转成字符串
		  let d3 = JSON.parse(d2); // 把字符串转成json对象
		  console.log(d3)
		  let d4 = [...d3,...arr]; // 把新的数据arr和原来的数据d3合并，得到新的数组
		  // 往users.json里写入新数据，转为json字符串的形式写入,flag: 'w'（如果没有就创建并写入，如果有就覆盖）
		  fs.writeFile('./data/users.json',JSON.stringify(d4), {flag: 'w'},(err,data) => {
		    console.log(data)
		  })
		}else{ // 如果不存在，创建写入文件，并写入数据。flag: 'wx'（如果没有就创建并写入，如果有就写入失败）
		  fs.writeFile('./data/users.json',JSON.stringify(arr), {flag: 'wx'}, (err,data) => {
		    console.log(data)
		  })
		}

###http模块的例子

写一个网站：
静态数据比如css，js都放在static文件夹里
视图（html）放在views文件夹里

根据url访问页面，也就是url和页面一一对应，在vue-router里可以方便地配置，vue在背后帮我们处理了很多事情

用原生js，需要咱们自己if判断

判断一下路径是否有后缀（
因为静态数据都放在static文件夹里，html都放在views文件夹里，路径不同）
	
		let http = require("http");
		let fs = require("fs");
		
		let app = http.createServer((req,res) => {
			// res.write('ok');
			let url = req.url  // 路径
			/*
				路径是否有后缀（因为静态数据都放在static文件夹里，html都放在views文件夹里，路径不同）
					有后缀看是js,css,png,gif jpg
			*/
			let re = /\.(js|css)$/g;
		
			if(re.test(url)){ // 判断一下路径是否有后缀，这里只示范判断.css和.js
				// 静态目录static
				let d = fs.readFileSync('./static'+url);
				res.end(d);
			}else{
				if(url === '/a'){
					// 发送的是index.html文件内容
					let d = fs.readFileSync('./views/index.html');
					res.end(d);
				}else if(url === '/b'){
					// 发送的是miaov.html文件内容
					let d = fs.readFileSync('./views/miaov.html');
					res.end(d);
				}else{
					//// 发送的是404.html文件内容
					let d = fs.readFileSync('./views/404.html');
					res.end(d);
				}
			}
		})
		
		app.listen(8000, () => {
			console.log("服务启动成功");
		})

###用express实现

		let express = require('express');

		let app = express();
		
		app.use(express.static('./static'))
		
		app.get('/a', (req,res) => {
			res.sendFile(__dirname+'/views/index.html')
		})
		
		app.get('/b', (req,res) => {
			res.sendFile(__dirname+'/views/miaov.html')
		})
		app.get('*', (req,res) => {
			res.sendFile(__dirname+'/views/404.html')
		})
		
		app.listen(8080, () => {
			console.log("服务启动成功");
		})