#9-5

##1.form表单提交数据

###1-1.form表单作用：提交数据

###1-2.form表单中的input的name很关键，是前后端约定的字段

###1-3.不好的地方：提交数据，会跳转页面,万一写错了要回退，填好的信息可能丢失需要重写

###1-4.action（要提交的地址）

###1-5.method（提交的方式）

* get
* post

**get和post区别**

真正的http协议里，二者没有很明显的区分

讨论二者的区别是在浏览器的范围内讨论

**get 方式**

* http://localhost:8088/backend/php/get.php?user=leo123&password=222
* 发送数据的方式
	* 在地址栏中?的后面，就是查询信息
		* key=value&lkey2=value2&key3=value3 成为为queryString
* 浏览器对地址有长度有限制，所以get的数据会有限制
* 浏览器会缓存地址
* 参数会保存在历史记录里
* 不安全，一般用来发送一些无关紧要的数据


post

* http://localhost:8088/backend/php/post.php
* 发送数据的方式
	* 放在HTTP的请求body（主体）发送的
* 理论上没有大小限制
	* 服务端会有限制
* 理论上是安全的

###1-6.form表单提交数据的改进：
分别验证每条信息，（打开新窗口）

		btn.onclick = function (){
			window.open('http://localhost:8088/backend/php/get.php?user='+username.value,"_blank")	
		}


##2.Ajax


###2-1.什么是 AJAX ？

> AJAX即“Asynchronous Javascript And XML”（异步JavaScript和XML），是指一种创建交互式网页应用的网页开发技术。

###2-2.Ajax的作用

> * 1. 发送数据和服务器进行交互
> * 2. 实现异步更新，不需要刷新整个页面，只做局部更新


###2-3.怎样使用Ajax请求数据？

####2-3-1.创建一个ajax对象，该对象拥有很多方法供我们调用
		let xhr = new XMLHttpRequest;
####2-3-2.连接地址，准备好数据（启动一个请求以备发送）
		
		xhr.open(method,url,async)

> * open方法的作用：规定请求的类型、URL 以及是否异步处理请求。
> * 参数
>   * method：请求的方式；get|post 不区分大小写
> 
>   * url：发送的地址（文件在服务器上的位置）
> 
>   * async：true（异步）或 false（同步）

####2-3-3.发送请求
		xhr.send();
####2-3-4.等到响应回来

> * 怎么知道什么时候响应回来了？
> 	* 只要浏览器接收到服务器的响应，不管其状态如何，都会触发load事件
> * 怎么拿到响应的内容？
> 	* 带回来的内容存放在xhr.responseText里


		xhr.onload = function(){
            //只要浏览器接收到服务器的响应，不管其状态如何(成功不成功)，都会触发load事件
            console.log(xhr.status);
            if(xhr.status >= 200 || xhr.status <= 307){
                console.log('ajax回来了');
                console.log(xhr.responseText);
                tip.innerHTML = xhr.responseText;
            }else if(xhr.status == 404){
                console.log(xhr.status);
                console.log(xhr.statusText);
            }else{
                console.log('服务器有问题');
            }
            
        }

###2-4.使用get方式发送请求的例子

注意：发送的信息要放在地址栏

编码问题


		send.onclick = function(){
			// 向后端发送数据，查询名字是否存在
			// 和服务器进行交互

            let xhr = new XMLHttpRequest;  //创建一个ajax对象，该对象拥有很多方法供我们调用

            //连接地址，准备好数据（启动一个请求以备发送）
            xhr.open(
                'GET',
                'http://192.168.2.68/miaov/2017-09-05/backend/php/get.php?user='+username.value,
                true
            )
            xhr.send();
            //怎么知道什么时候响应回来了？只要浏览器接收到服务器的响应，不管其状态如何，都会触发load事件
            //怎么拿到响应的内容？带回来的内容存放在xhr.responseText里
            xhr.onload = function(){

                //只要浏览器接收到服务器的响应，不管其状态如何(成功不成功)，都会触发load事件
                console.log(xhr.status);
                if(xhr.status >= 200 || xhr.status <= 307){
                    console.log('ajax回来了');
                    console.log(xhr.responseText);
                    tip.innerHTML = xhr.responseText;
                }else if(xhr.status == 404){
                    console.log(xhr.status);
                    console.log(xhr.statusText);
                }else{
                    console.log('服务器有问题');
                }
                
            }
            
        }


###2-4.使用post方式发送请求

		username.onblur = function(){
            let xhr = new XMLHttpRequest;  //创建一个ajax对象，该对象拥有很多方法供我们调用

            //连接地址，准备好数据（启动一个请求以备发送）
            xhr.open(
                'POST',
                'http://192.168.2.68/miaov/2017-09-05/backend/php/post.php',
                true
            );

            xhr.onload = function(){
                tip.innerHTML = xhr.responseText;
            }
			
			//在post里需要手动加上请求头部，设置内容的类型
            xhr.setRequestHeader('Content-Type','application/x-www-form-urlencoded');

            xhr.send('user='+this.value);

            //post可以发送多种数据结构，比如key value 也可以发送二进制形式
        }


###2-5.enctype

enctype属性规定在发送到服务器之前应该如何对表单数据进行编码。
默认是：application/x-www-form-urlencoded
text/plain 文本

multipart/form-data 编码为二进制的，上传文件

**form里**

post请求

		<form 
			action="http://localhost:8088/backend/php/post.php"
			method="post"
			enctype="multipart/form-data" 
		>
			用户名：<input type="text" name="user" id="username" /><span id="tip"></span>
			<br/>
			密码：<input type="password" name="password" /><br/>
			<input type="submit" id="send" value="提交">
		</form>

上传

		<form 
			action="http://localhost:8088/backend/post_file.php"
			method="post"
			enctype="multipart/form-data" 
		>
			<input type="file" name="file">
			<input type="submit" id="send" value="提交">
		</form>


**在post里需要手动加上请求头部，设置内容的类型:**
**post可以发送多种数据，所以需要告知服务器发送过去的是哪种数据，所以要加上请求头部**

	
	 xhr.setRequestHeader('Content-Type','application/x-www-form-urlencoded');

###2-3.Ajax的工作流程

* 1.初始化，未发送
	* 0	UNSENT
* 2.准备数据，连接地址
	* 1	OPENED
* 3.返回响应头
	* 2	HEADERS_RECEIVED   未返回内容，只返回了响应头
* 4.接收数据中
	* 3	LOADING            返回内容，数据量大，分批返回
* 5.接收数据完毕
	* 4	DONE               数据完全接受完成，可以拿数据做事情了

**怎么知道在哪一步？**

xhr.readyState
值有0,1,2,3,4

**怎么监控走到哪一步？**

IE6、7没有onload事件

用**onreadystatechange事件**

ajax每进行一步都会触发这个事件，给这个事件绑定一个事件处理函数（监听器），就可以监视到走到哪一步了

		xhr.onreadystatechange = function (){
			console.log(xhr.readyState);	
			console.log(xhr.getAllResponseHeaders());
			console.log(xhr.responseText);
		}


如果 xhr.readyState===4，就相当于触发了onload事件

只要触发onload，ajax的步骤已经进行到了第五步，也就是xhr.readyState = 4；

		用户名：<input type="text" name="user" id="username" /><span id="tip"></span>
			<br/>
		密码：<input type="password" name="password" /><br/>
		<input type="button" id="send" value="提交">


		username.onblur = function (){
			// 向后端发送数据，查询名字是否存在
			// 和服务器进行交互
			let xhr = new XMLHttpRequest();
			console.log(xhr.readyState);	
			xhr.open(
				'get',
				'http://localhost:8088/backend/php/get.php?user='+this.value,
				true
			);
			// 4. 响应触发的事件

			// 监控ajax每一个步骤，ajax每进行一步都会触发这个事件
			console.log(xhr.readyState);
			xhr.onreadystatechange = function (){
				if(xhr.readyState === 4){  // 相当于触发了onload

					console.log(xhr.readyState);	
					console.log(xhr.getAllResponseHeaders());
					//获取头部信息
					console.log(xhr.responseText);
				}
			}

			// 只要触发onload，ajax的步骤已经进行到了第五步，也就是xhr.readyState = 4；
			// 数据完全接完成，拿数据做事情了
			xhr.onload = function (){
				console.log("ajax回来了");	
				console.log(xhr.readyState,"c");
			};
			// 3.发送
			xhr.send();	
		};


####status ：状态码
无论成功不成ajax都会返回的，会触发load事件，所以要用状态码来判断一下
#####statusText：状态短语




---------------------------------------
##-------------9-6--------------------

##1.JSON数据结构

###1-1.JSON可以表示以下三种类型的值

###1-1-1.简单值（字符串、数值、布尔值、null）

字符串

		"hello"
**JSON字符串必须使用双引号，使用单引号会报语法错误**

数值

		5

布尔值

null

###1-1-2.对象

		{
			"username":"leo",
			"age":30	
		}

* 与js的对象字面量的区别：

	* JSON对象没有声明变量，（JSON中没有变量的概念）
	* 没有末尾的分号
	* 对象的属性（键值）必须加**双引号**！

###1-1-3.数组

JSON数组采用的是js中的数组字面量形式

		['mary',30,false]

JSON数组也没有变量和分号

数组可以和对象相结合，构成更加复杂的数据结构


##1-2.解析与序列化

**问：怎么拿到JSON数据结构里的数据？**

**答：把JSON转成对象**

**问：怎么转？**

**答：JSON对象有方法**
###1-2-1JSON对象

低版本浏览器没有JSON这个全局对象

JSON对象有两个方法：**stringify()**和**parse()**

####1.stringify()

作用：把js对象序列化为JSON字符串

向后端发送数据，不能传对象，必须转成文本，把对象转成json字符串

        let obj2 = {'miaov':1,'ketang':2};

        let str = JSON.stringify(obj2);

        console.log(str);//{"miaov":1,"ketang":2}
        console.log(typeof str);//string


**参数**

> * 第一个参数：是一个过滤器，（要转的对象，可以是数组）
> * 第二个参数： 
> * 第三个参数：缩进的空格数

从后端拿到的如果是对象（比如数组） 如何使用？

		let obj3 = [
            {
                user:'leo',
                age:30
            },
            {
                user:'leo',
                age:30
            },
            {
                user:'leo',
                age:30
            }
        ]
        console.log(obj3);//[Object, Object, Object]
        //如果从后端拿到的是[Object, Object, Object]这种形式，怎么转成上面的竖着排列的缩进的形式？
        console.log(JSON.stringify(obj3,null,2));

        let fn = function(){
            console.log('123')
        }
        console.log(JSON.stringify(fn,null,2));


####2.parse()

后端响应的数据，一般不会是字符串格式，而是类似对象的数据格式（json）
字符串格式JSON，想要把字符串转成可用的对象

		let json = '{"username":"mary","age":30}'
        let obj = JSON.parse(json);
        console.log(obj);
		//{username: "mary", age: 30}

        console.log(JSON);
		//JSON {Symbol(Symbol.toStringTag): "JSON", parse: function, stringify: function}



##2.下载


		<a href="./2017-09-06.zip">2017-09-06.zip</a>

href里是想要下载的文件的路径（绝对路径相对路径均可）

##3.上传

js是不能读取文件的，只有服务器的语言才能读取文件，如果js能读取就乱了，因为js是浏览器的脚本语言，如果js可以读取，发给别人一个html，别人一打开我就可以读取他电脑上的信息

所有的上传都是post方法

上传时都是把读取的文件转成二进制形式传给服务器，服务器再解码

上传到服务器。服务器有一个地方专门存放

###3-1.上传方法

* 可以用form表单上传
    * form表单设置enctype = ""
* 可以用ajax上传（不刷新或跳转页面）

    * 找到要上窜的文件。转成二进制
    * 找到文件？存在
* 两种方式根据需求选择使用


####3-1-1.上传之form表单形式

action里是请求的接口地址

method是请求方法

enctype是数据的形式：
因为上传时都是把读取的文件转成二进制形式传给服务器
所以用multipart/form-data，编码为二进制的，上传文件

		<form
	        action='http://192.168.2.68/miaov/2017-09-05/backend/post_file.php'
	        method="post"
	        enctype="multipart/form-data"
	    >
	        <input type="file" name="file" value="选择文件"/>
	        <input type="submit" value="上传"/>
	    </form> 


####3-1-2.上传之ajax方法



		<input type="file" name="file" value="选择文件" id="inputFile"/>
    	<input type="submit" value="上传" id="submit"/>

		submit.onclick = function(){

            let xhr = new XMLHttpRequest();
            xhr.open(
                'post',
                'http://192.168.2.68/miaov/2017-09-05/backend/post_file.php',
                true
            );

            console.log(inputFile.value);//存的是文件的地址：C:\fakepath\2-form表单提交数据.html
            console.log(inputFile.files);
			//是一个对象:FileList {0: File, length: 1}，File是真正的文件
            console.dir(inputFile.files[0]);
			//拿到文件，文件的名字默认是上传的文件的名字

            let f = new FormData();
            f.append('file',inputFile.files[0])
            
			//xhr.send(f);  ////写到这里会上传失败？？？？
            xhr.onload = function(){
                console.log(xhr.responseText);
            }
            xhr.send(f);
            
        }

##4.FormData对象

通过FormData对象可以组装一组用 XMLHttpRequest发送请求的键/值对。

如果你把表单的编码类型设置为multipart/form-data ，则通过FormData传输的数据格式和表单通过submit() 方法传输的数据格式相同


###4-1.创建一个FormData对象

		let f = new FormData();


###4-2.FormData的append方法

###4-2-1.语法

这个方法有两个版本：一个有两个参数的版本和一个有三个参数的版本。

		formData.append(name, value);
		formData.append(name, value, filename);

###4-2-2.参数

> * name
>   * value中包含的数据对应的**表单名称**（input标签的name，和后台约定好的name）。
>   
> * value
>   * 表单的值。可以是USVString 或 Blob (包括子类型，如 File)。
> * filename 可选
>    * 传给服务器的文件名称 (一个 USVString), 当一个 Blob 或 File 被作为第二个参数的时候， Blob 对象的默认文件名是 "blob"。 File 对象的默认文件名是该文件的名称。

BLOB常常是数据库中用来存储二进制文件的字段类型。BLOB是一个大文件，典型的BLOB是一张图片或一个声音文件


###4-2-3.返回值

空


###4-2-3.使用FormData上传文件

详见3-1-2.上传之ajax方法


##5.监控上传进度

####xhr.upload.onprogress
**注意和hr.onprogress区别**

> * xhr.upload.onprogress在上传阶段(即xhr.send()之后，xhr.readystate=2之前)触发，每50ms触发一次；
> * xhr.onprogress在下载阶段（即xhr.readystate=3时）触发，每50ms触发一次。


####xhr.upload.onprogress的事件处理函数的事件对象ev

> ev.loaded:上传大小
> ev.total:总大小
> Math.round(ev.loaded/ev.total*100)+'%';

##6.上传文件显示进度条


		<style>
			*{
				padding: 0;
				margin: 0;
			}
			#box {
				width: 500px;
				height: 30px;
				border: 1px solid #000;
				position: relative;
			}
			#box p {
				width: 100%;
				text-align: center;
				line-height: 30px;
				position: relative;
				z-index: 10;
			}
			#bar {
				width: 0px;
				height: 100%;
				background: red;
				position: absolute;
				left: 0;
				top: 0;
			}
		</style>


		<body>
		<input type="file" id="inputFile" name="file">

		<input type="button" id="btn" value="上传" />
		<div id="box">
			<p id="text">0%</p>
			<div id="bar"></div>
		</div>
		<script>
			btn.onclick  =function (){
				// 上传
				let xhr = new XMLHttpRequest();

				xhr.open(
					'post',
					'http://localhost/miaov/2017-09-05/backend/post_file.php',
					true
				);

				let f = new FormData();
				f.append('file',inputFile.files[0]); 

				xhr.send(f);

				// 监控上传进度
				xhr.upload.onprogress = function (ev){
					console.log('uoonp触发')
					console.log(ev.loaded); // 上传大小	
					console.log(ev.total); //  总大小	
					console.log(Math.round(ev.loaded/ev.total*100)+'%');

					let scale = ev.loaded/ev.total;

					text.innerHTML = Math.round(ev.loaded/ev.total*100)+'%';
					bar.style.width = scale * 500 + 'px';

				}

				

				xhr.onload = function (){
					console.log(xhr.responseText);	
				};

				// 找到要上传的文件，把文件转成二进制的

				console.log(inputFile.value);  // 文件所在的地址
				// 真正上传的资源，放在元素的files属性中
				console.dir(inputFile.files[0]);

				// 高版本浏览器 FormData

				/* let f = new FormData();
				f.append('file',inputFile.files[0]);

				xhr.send(f);  */

			};
		</script>
		
	</body>


##7.跨域请求

###7-1.同源策略

同源策略（Same origin policy）是一种约定，它是浏览器最核心也最基本的安全功能，如果缺少了同源策略，则浏览器的正常功能可能都会受到影响。可以说Web是构建在同源策略基础之上的，浏览器只是针对同源策略的一种实现。

####同源

域名，协议，端口相同。

有一个不同，就形成跨域

####域名
**域名  是ip的别名（小名）**

* 一级域名：
	* baidu.com  taoba.com  gome.com.cn  
* 二级域名
	* news.baidu.com/              
	* git.oschina.net/
* 三级域名
	* abc.news.baidu.com/          

####协议

**协议 服务器（客户端）进行通信的一种约定**

https
http
ftp
file


####端口 

https  443
http 80
ftp  22
file 不知道


###7-2.解决跨域


####方法1：设置允许跨域的头部
> 
> 在请求的这个域上设置一个header
> 
> 比如http://localhost:3000/访问http://localhost:8888/test  会产生跨域
> 
> 在http://localhost:8888/这个域下设置header，允许3000来访问
> 
> 设置允许3000端口访问

> res.header("Access-Control-Allow-Origin", "http://localhost:3000");
> 
> 设置允许所有端口访问
> 
> res.header("Access-Control-Allow-Origin", "*");


不同的语言有不同的设置方法，这里是node.js的设置方法

####方法2：代理

**请求自己域下的后端，自己域下的后端请求目标域的接口**

> 比如http://localhost:3000/访问http://localhost:8888/test  会产生跨域
> 
> 比如http://localhost:3000/访问http://localhost:3000/abc.php
> 
> 让http://localhost:3000/abc.php 去访问http://localhost:8888/test

**原理**


**例子**

		<input type="button" value="获取数据" id="btn" />


		btn.onclick = function (){
			let xhr = new XMLHttpRequest();	
			xhr.open(
				'get',
				'http://localhost:3000/backend/get_sae.php/?'+Math.random(),
				true
			)
			
			//http://localhost:3000/访问http://localhost:3000/get_sae.php


			xhr.onload = function (){
				console.log(xhr.responseText);	
			};

			xhr.send();
		};

		
		//让http://localhost:3000/get_sae.php 去访问http://localhost:8888/4-test.txt

		<?php
		$content = file_get_contents('http://localhost/miaov/2017-09-06/js/4-test.txt');
		
		echo $content;



#####方法3：jsonp

**允许跨域的标签：**

a标签
img标签
script标签
link标签

**这些标签不在乎给的后缀是什么，在乎的是能不能解析里面的内容**
> 
> * img需要的是src能被解析为图片，并不关心src里是不是图片的url地址
> * script  关心的是内容能否被js解析器解析，并不关心src里是不是js文件，链接一个txt文件，txt里的js语法依然能被解析，从而在浏览器里展示交互效果
> * **php比较特殊**
> * php如果在本地打开，php不能被解析；
> 在服务器环境打开，php解析器解析，输出alert('hello'),这条代码是js语法，被js解析器解析，弹出'hello'
> link  内容能否被css解析器解析


**jsonp = json + padding**

**用json填充数据**

> 1. 先创建一个script标签，src赋值地址
> 2. 访问的地址返回数据，数据中会有一个函数的执行
> 3. 在全局放一个函数，返回了数据，数据中会有函数执行，就会执行这个函数
> 4. 可以通过函数的参数拿到所需要的数据


图片转为base64，不用发请求，
小的图片文本比较少，可以用base64


**小例子**

		<p>姓名为：<span id="name">momo</span></p>
		<script>
			// 在全局放一个函数
			function abc(data){
				console.log(data);
			}



			// 点击document，拿到4-text中的数据
			document.onclick = function (){
				let script = document.createElement("script");
				script.src = './js/4-test.txt';
				document.body.appendChild(script);

			};
		</script>

**分析**

4-test.txt里的内容

		let obj = {
			username:'leo'
		}
		
		// 执行函数
		abc(obj)

* 什么时候加载了4-test.txt文件？
	* 4-test.txt里调用一个函数abc（函数abc在全局定义好，时刻准备着）
* 怎么拿到4-test.txt文件里面的数据？
	* 当加载了4-test.txt文件时，txt里的js语法被js解析器解析，abc函数被调用，传入实参obj，在全局，函数abc打印出传进来的



###8.百度搜索例子，用跨域请求百度的接口

		<style>
			*{
				padding: 0;
				margin: 0;
			}
			ul {
				list-style: none;
			}
			#box {
				width: 500px;
				margin: 50px auto;
			}
	
			#box input {
				width: 100%;
				height: 30px;
			}
			#box ul {
				width: 100%;
				border: 1px solid #333;
			}
	
			#box ul li {
				width: 100%;
				height: 20px;
				padding: 5px 0;
			}
			#box ul li:hover {
				background: #ccc;
			}
	
			a {
				text-decoration: none;
			}
		</style>


		<div id="box">
			<input id="searchInput" type="text" />
			<ul id="list">
				<!-- <li>
					<a href="">123</a>
				</li> -->
			</ul>
		</div>


		<script>
			//准备好一个回调函数abc。当请求到数据之后，数据里会调用函数abc，abc拿数据做一些事情
			function callBackFn(data){
				let sData = data.s;
				console.log(data.g);
				console.log(data.s);
				//用数据拼出ul里的li结构
				let html = sData.map(function(item){
					return `<li>
								<a href="https://www.baidu.com/s?wd=${item}" target="_blank">${item}</a>
							</li>`;
				});
				list.innerHTML = html.join('');
				//使展示出来的众多提示信息被点击时可以跳转，a标签的href里写上
			}
	
			searchInput.addEventListener('input',function(){
				let script = document.createElement('script');//创建一个script标签
				//script的src链接到百度的接口，把百度的回调函数的名字（jQuery110203752589068485399_1504708118439）改成自己定义的回调函数的名字（callBackFn）
				//search值的wd的value值改为输入框的value值
				script.src = `https://sp0.baidu.com/5a1Fazu8AA54nxGko9WTAnF6hhy/su?wd=${this.value}&cb=callBackFn&_=1504708118445`;
				document.body.appendChild(script);
			})
		</script>

相关说明


//当搜索“搜狗时”

	jQuery110203752589068485399_1504708118439({"q":"sou\'gou","p":false,"bs":"","csor":"0","status":0,"g":[ { "q": "搜狗输入法", "t": "n", "st": { "q": "搜狗输入法", "new": 0 } }, { "q": "搜狗输入法下载", "t": "n", "st": { "q": "搜狗输入法下载", "new": 0 } }, { "q": "搜狗", "t": "n", "st": { "q": "搜狗", "new": 0 } }, { "q": "搜狗浏览器", "t": "n", "st": { "q": "搜狗浏览器", "new": 0 } }, { "q": "搜狗地图", "t": "n", "st": { "q": "搜狗地图", "new": 0 } }, { "q": "搜狗壁纸", "t": "n", "st": { "q": "搜狗壁纸", "new": 0 } }, { "q": "搜狗拼音输入法", "t": "n", "st": { "q": "搜狗拼音输入法", "new": 0 } }, { "q": "搜狗五笔输入法", "t": "n", "st": { "q": "搜狗五笔输入法", "new": 0 } }, { "q": "搜狗搜索", "t": "n", "st": { "q": "搜狗搜索", "new": 0 } }, { "q": "搜狗拼音输入法2014官方下载", "t": "n", "st": { "q": "搜狗拼音输入法2014官方下载", "new": 0 } } ],"s":["搜狗输入法","搜狗输入法下载","搜狗","搜狗浏览器","搜狗地图","搜狗壁纸","搜狗拼音输入法","搜狗五笔输入法","搜狗搜索","搜狗拼音输入法2014官方下载"]});

jQuery110203752589068485399_1504708118439是回调函数的名字

使展示出来的众多提示信息被点击时可以跳转

“搜狗输入法”的url：

	https://www.baidu.com/s?wd=%E6%90%9C%E7%8B%97%E8%BE%93%E5%85%A5%E6%B3%95&rsv_spt=1&rsv_iqid=0xd6755e3e00014eba&issp=1&f=8&rsv_bp=0&rsv_idx=2&ie=utf-8&tn=baiduhome_pg&rsv_enter=1&rsv_sug3=9&rsv_sug1=9&rsv_sug7=100

“搜狗浏览器”的url：

	https://www.baidu.com/s?wd=%E6%90%9C%E7%8B%97%E6%B5%8F%E8%A7%88%E5%99%A8&rsv_spt=1&rsv_iqid=0xd6755e3e00014eba&issp=1&f=8&rsv_bp=1&rsv_idx=2&ie=utf-8&rqlang=cn&tn=baiduhome_pg&rsv_enter=1&oq=%25E6%2590%259C%25E7%258B%2597%25E8%25BE%2593%25E5%2585%25A5%25E6%25B3%2595&rsv_t=4567ynIdAsZSF1GeI1vqMo7S5E5nwlqN%2BESpdOTCYmzFrtc1g%2FZQkkN9DMG8df8Xmr3B&inputT=3503&rsv_sug3=21&rsv_sug1=19&rsv_sug7=100&rsv_pq=891390f30001484a&bs=%E6%90%9C%E7%8B%97%E8%BE%93%E5%85%A5%E6%B3%95

发现由于是中文，search值的wd对应的value值出现乱码，在a标签的href里写时把乱码全改成拿到的数据



###9.QQ音乐例子，用跨域请求QQ音乐的部分接口


		<script>
			//准备好一个回调函数abc。当请求到数据之后，数据里会调用函数abc，abc拿数据做一些事情
			function callBackFn(data){
				console.log(data);//打出来看看数据是什么样子,发现data.data.list这个数组里是想要的数据
				let sData = data.data.list;//数组存到sData变量里，便于后面使用
				console.log(JSON.stringify(sData[0],null,1));
				console.log(JSON.stringify(sData[1],null,1));
				//为了方便看，用JSON.stringify方法把sData这个数组转成带缩进格式的字符串
				/*
					{
					"Farea": "1",
					"Fattribute_3": "3",
					"Fattribute_4": "0",
					"Fgenre": "0",
					"Findex": "X",
					"Fother_name": "Joker",
					"Fsinger_id": "5062",
					"Fsinger_mid": "002J4UUk29y8BY",
					"Fsinger_name": "薛之谦",
					"Fsinger_tag": "541,555",
					"Fsort": "1",
					"Ftrend": "0",
					"Ftype": "0",
					"voc": "0"
					}
				*/
		
				let html = sData.map(function(item){
					return `
							<li>
								<a href="https://y.qq.com/n/yqq/singer/${item.Fsinger_mid}.html#">
									<img src="https://y.gtimg.cn/music/photo_new/T001R150x150M000${item.Fsinger_mid}.jpg?max_age=2592000">
								</a>
								<h3>${item.Fsinger_name}</h3>
							</li>
					`;
					//对比发现Fsinger_mid不同，Fsinger_name是歌手的名字
					//点击每个歌手可以跳转
				});
				list.innerHTML = html.join('');
			} 
		
		
			//当在输入框输入的时候，展示歌手列表（按热度的）
			searchInput.addEventListener('input',function(){
				let script = document.createElement('script');
				script.src=`https://c.y.qq.com/v8/fcg-bin/v8.fcg?channel=singer&page=list&key=all_all_all&pagesize=100&pagenum=1&g_tk=5381&jsonpCallback=callBackFn&loginUin=0&hostUin=0&format=jsonp&inCharset=utf8&outCharset=utf-8&notice=0&platform=yqq&needNewCode=0`
				document.body.appendChild(script);
			});
		
		</script>

相关说明

**歌手列表（按热度的）**

		https://c.y.qq.com/v8/fcg-bin/v8.fcg?channel=singer&page=list&key=all_all_all&pagesize=100&pagenum=1&g_tk=5381&jsonpCallback=GetSingerListCallback&loginUin=0&hostUin=0&format=jsonp&inCharset=utf8&outCharset=utf-8&notice=0&platform=yqq&needNewCode=0

**歌手列表（字母C开头的）**
		
		https://c.y.qq.com/v8/fcg-bin/v8.fcg?channel=singer&page=list&key=all_all_C&pagesize=100&pagenum=1&g_tk=5381&jsonpCallback=GetSingerListCallback&loginUin=0&hostUin=0&format=jsonp&inCharset=utf8&outCharset=utf-8&notice=0&platform=yqq&needNewCode=0

对比找规律


**歌手的照片**

薛之谦的：

	https://y.gtimg.cn/music/photo_new/T001R150x150M000002J4UUk29y8BY.jpg?max_age=2592000
周杰伦的

	https://y.gtimg.cn/music/photo_new/T001R150x150M0000025NhlN2yWrP4.jpg?max_age=2592000

在callBackFn里console.log(data)，看一下数据是什么样

GetSingerListCallback是回调函数的名字


**点击每个歌手跳转的地址**
薛之谦页面的url：

		https://y.qq.com/n/yqq/singer/002J4UUk29y8BY.html#
周杰伦页面的url：

		https://y.qq.com/n/yqq/singer/0025NhlN2yWrP4.html#

对比找规律