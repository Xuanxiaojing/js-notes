#9-26-express

##1.概念
是基于nodejs的web开发框架，是一个函数

##2.使用

创建一个入口文件app.js
1. 加载express模块
2. 函数执行后返回一个对象，对象有很多方法
3. 做一些事情（发送请求）
4. 启动服务

	let express = require('express'); // 加载express模块
	let app = express();
	// 函数执行后返回一个对象，对象有很多方法
	
	let host = 'localhost';
	let post = 8080;
	app.listen(post,host, () => {
	    console.log('http://'+ host + ':' + post)
	})// 第二个参数如果不写 那么默认是localhost，如果想让别人访问，那么第二个参数写自己的ip

####app.listen(port, [hostname], [backlog], [callback])

参数：
1.端口
2.域名
3.回调函数

##2.路由

**概念**

路由是指如何定义应用的端点（URIs）以及如何响应客户端的请求。

**结构**

app.METHOD(path, [callback...], callback)

- METHOD ：是一个 HTTP 请求方法（get或post）
- path：是服务器上的路径，
- callback： 是当路由匹配时要执行的函数。


**------1.发送简单数据----------**

####res.send([body])
发送HTTP响应（Sends the HTTP response）
> 
> The body parameter can be a Buffer object, a String, an object, or an Array. 

例如：

	res.send(new Buffer('whoop'));
	res.send({ some: 'json' });
	res.send('<p>some html</p>');
	res.status(404).send('Sorry, we cannot find that!');
	res.status(500).send({ error: 'something blew up' });

引用自express官网

**----------2.发送页面-----------**

####res.sendFile(path [, options] [, fn])

path必须是绝对路径

	app.get('/',(req,res) => {
	    console.log(__dirname);  // 得到当前运行的js文件所在的绝对路径：D:\前端学习\miaov2\miaov2\9-26-nodejs\node0926
	    res.sendFile(__dirname + '/views/index.html');
	})
	app.get('/collection',(req,res) => {
	    console.log(__dirname);  // 得到当前运行的js文件所在的绝对路径
	    res.sendFile(__dirname + '/views/collection.html');
	})
	app.get('/myFile',(req,res) => {
	    console.log(__dirname);  // 得到当前运行的js文件所在的绝对路径
	    res.sendFile(__dirname + '/views/myFile.html');
	})

动态路径

	// 动态路径
	app.get('/myFile/:id',(req,res) => {
	    console.log(req.params); // { id: '123' }
	    console.log(req.params.id); // 输出id
	    res.sendFile(__dirname + '/views/myFile.html');
	}) 

###req.params

> An object containing properties mapped to the named route “parameters”. For example, if you have the route /user/:name, then the “name” property is available as req.params.name. This object defaults to {}.

req.params是一个对象，它包含映射到指定的路由“参数”的属性的对象。比如，如果 路径指定为/user/:name ，name属性就可以通过req.params.name访问到


##3.中间件（Middleware）

###3-1.引入

**场景：collection和myFile页面一开始不能访问，需要登录才能访问**

分析：

- 如果没有登录
	- 重定向到login页面，不能在miaov.html地址里发送login页面
- 如果登陆了
	- 发送对应的页面
	
代码
	
	app.get('/',(req,res) => {
	    res.sendFile(__dirname + '/views/index.html');
	})
	// 重定向
	app.get('/collection',(req,res) => {
	    if(true){ // 如果没有登录
	        res.redirect('/login');
	    }else{
	        res.sendFile(__dirname+'/views/collection.html')
	    }  
	})
	app.get('/login',(req,res) => {
	    res.sendFile(__dirname+'/views/login.html')
	})
	app.get('/myFile',(req,res) => {
	    if(true){ // 如果没有登录
	        res.redirect('/login');
	    }else{
	        res.sendFile(__dirname + '/views/myFile.html');
	    }
	}) 


如果写很多判断会很麻烦--所以改用**中间件**

代码

	app.use(function(req,res,next){
	    console.log('现在访问了:' + req.url);
	    let mustLogin = ['/collection','/myFile']
	    if(mustLogin.includes(req.url)){ // 需要限制的
	        if(true){ // 没有登录
	            res.redirect('/login')
	        }else{
	            next();
	        }
	    }else{ // 不需要限制的
	        next();
	    }
	})
	app.get('/',(req,res) => {
	    res.sendFile(__dirname + '/views/index.html');
	})
	// 重定向
	app.get('/collection',(req,res) => {
	    res.sendFile(__dirname+'/views/collection.html')
	})
	app.get('/login',(req,res) => {
	    res.sendFile(__dirname+'/views/login.html')
	})
	app.get('/myFile',(req,res) => {
	    res.sendFile(__dirname + '/views/myFile.html');
	}) 

###3-2.对中间件的理解

引用express官网：
> 中间件（Middleware） 是一个函数，它可以访问请求对象（request object (req)）, 响应对象（response object (res)）, 和 web 应用中处于请求-响应循环流程中的中间件，一般被命名为 next 的变量

功能：

- 执行任何代码。
- 修改请求和响应对象。
- 终结请求-响应循环。
- 调用堆栈中的下一个中间件。

比如：前后端通信，请求和响应；现在在请求和响应中间拦截一层，（可以做一些判断和处理），过滤一下是否响应，以及响应什么内容

如果当前中间件没有终结请求-响应循环，则必须调用 next() 方法将控制权交给下一个中间件，否则请求就会挂起。

分类：

* 没有挂载路径的中间件，应用的每个请求都会执行该中间件

	app.use(function (req, res, next) {
	  console.log('Time:', Date.now());
	  next();
	});
* 挂载至某个路径的中间件，任何指向该路径的请求都会执行它

	app.use('/user/:id', function (req, res, next) {
	  console.log('Request Type:', req.method);
	  next();
	});


##4.响应静态文件
如果在index.html里引入css和jQuery

比如	`<script src="../js/jquery-3.2.1.js"></script>`在服务器运行时会报错，找不到文件

**原因：**本地访问和在服务端访问不同，
**服务端以路由为准，要设置根路径才知道这个相对路径是相对谁的**

设置静态文件访问的根目录：（把css和js放在public文件夹里）

	app.use(express.static('public'));

设置完就可以这样访问，不用写public了

	<script src="/js/jquery-3.2.1.js"></script>

##5.响应ajax请求

###5-1.get请求

html

	<button id="getBtn">get请求</button>
    

前端js: 前端发送ajax请求（get方式）
get方式发送的数据写在url里

	$('#getBtn').click(function(){
        $.ajax({
            url: 'users?random=' + Math.random() + "&userName=xiaobai",
            method: 'get',
            success(data){
                // 后端发送的是json的格式，jq会帮你转成对象的形式
                console.log(typeof data);// object
            }
        })
    })

后端js

	app.get('/users',(req,res) => {
	    // 前端发送过来ajax请求（get方式）
	    // 怎么获取get请求的数据，再去数据库里查找
	    // req.query  拿到一个对象，对象里有get请求的数据
	    console.log(req.query); // { random: '0.28113908475178895', userName: 'xiaobai' }
	    let {random,userName} = req.query;
	    // 是解构赋值
	    console.log(random); // 0.08591504956212481
	    console.log(userName); //xiaobai
	    res.send({
	        code: 0,
	        message:'请求数据成功'
	    })
	})

直接在地址栏访问http://localhost:8080/users可以看到{  code: 0,  message:'请求数据成功'   }

###5-2.post请求
后端拿到前端的post请求，怎么获取post请求的数据？

**req.body：**
> 包含请求主体中提交的数据的键值对。-----------必须配合body-parser使用
> 
> 安装body-parser模块  npm i body-parser --save

**body-parser**

> body-parser是一个HTTP请求体解析中间件，使用这个模块可以解析JSON、Raw、文本、URL-encoded格式的请求体，Express框架中就是使用这个模块做为请求体解析中间件

html

	<button id="postBtn">post请求</button>

前端js

	$('#postBtn').click(function(){
        $.ajax({
            url: 'api/users-post',
            method:'post',
            data: {
                userName: '小黑'
            },
            success(data){
                // data是后端响应回来的，是json的格式，jq会帮咱转成对象的形式
                console.log(data); //Object {code: 0, message: "post请求成功"}
            }
        })
    })

后端js

	const bodyParser = require('body-parser'); // 加载body-parser模块
	app.use(bodyParser.json()); // 解析客户端发送过来的json格式
	app.use(bodyParser.urlencoded({extended: true})); // 解析客户端发送的key=value&key=value
	app.post('/users-post',(req,res) =>{
	    console.log(req.body); // { userName: '小黑' }
	    let {userName} = req.body;
	    res.send({
	        code: 0,
	        message: 'post请求成功'
	    })
	})

注意：直接在地址栏访问http://localhost:8080/users-post显示Cannot GET /users-post  并报错

###5-3.路由拆分

一个app.js文件里写很多路由，有请求页面的，有请求接口的，可以把请求接口的拆出来

创建api文件夹，创建users.js文件，专门存放请求数据，全挂载在router上

users.js文件里

	const express = require('express');
	const router = express.Router();
	
	router.get('/users',(req,res) => {
	    // 前端发送ajax请求（get方式）
	    // 怎么获取get请求的数据，再去数据库里查找
	    // req.query  拿到一个对象，对象里有get请求的数据
	    console.log(req.query); // { random: '0.28113908475178895', userName: 'xiaobai' }
	    let {random,userName} = req.query;
	    // 是解构赋值
	    console.log(random); // 0.08591504956212481
	    console.log(userName); //xiaobai
	    res.send({
	        code: 0,
	        message:'get请求数据成功'
	    })
	})
	router.post('/users-post',(req,res) =>{
	    console.log(req.body); // { userName: '小黑' }
	    let {userName} = req.body;
	    res.send({
	        code: 0,
	        message: 'post请求成功'
	    })
	})
	
	module.exports = router;

在app.js里引入users.js文件，作为中间件使用，

给他们一个明显的标识'/api'，代表众多接口路由的父路由

代码

	const userApi = require('./api/users.js');
	app.use('/api',userApi)

##6.上传文件

* 安装模块：**npm i multer --save**
* 
在app.js里引入

		const multer = require('multer');

*配置：指定上传的目录和存储文件的名字
post的第二个参数


前端代码

	<form method="post" action="/upload" enctype='multipart/form-data'>
        <input type="file" name="xiaobai">
        <input type="submit"  value="上传" />
    </form>
后端代码

	const multer = require('multer');
	// app.use(multer()); // for parsing multipart/form-data会报错？
	var storage = multer.diskStorage({
	    destination: function(req,file,cb){
	        cb(null,__dirname+'/uploads/'); // 设置存储的位置
	    },
	    filename: function(req,file,cb){
	        cb(null,file.originalname); // 存储文件的名字
	    }
	})
	var upload = multer({storage}); // 指定上传的信息
	app.post('/upload',upload.single('xiaobai'),(req,res) => {
	    //xiaobai是前后端约定好的名字
	    res.send('上传成功')
	})

##7.模板引擎

- 套页面
	-   1.前端做
		- 向后端请求数据
	-	2.后端做
		- 一般首页由后端渲染，后端直接向前端返回完整的页面，速度快，因为由js计算渲染需要时间

后端的模板引擎推荐：**swig**

使用：比如用模板引擎，把index.html作为模板

1. 安装 
		
		npm i swig --save
2. 引入模板引擎（告诉express，用模板引擎）

		let swig = require('swig')
3. 设置模板存放的位置 在abc这个文件夹下

		app.set('views','./abc');   // 第一个固定是views，代表模板，后面的路径里不一定叫views
4. 定义模板引擎使用swig.renderFile这个函数渲染，同时设置后缀名为html

		app.engine('html', swig.renderFile)

5. 注册模板引擎，abc目录下的以html为后缀的模板都用swig.renderFile渲染

		app.set('view engine', 'html')

6. 发送给客户端之前，需要使用模板引擎去处理

		let user = {
		    name: 'xiaohei',
		    age:30,
		    sex: '男'
		}
		app.get('/',(req,res) => {
		    // 发送给客户端之前，需要使用模板引擎去处理
		    res.render('index',{
		        user:user,
		        list: [1,2,3,4]
		    })
		})


7. 前端在index.html里：用双大括号渲染结构

		<ul>
	        <li>{{user.name}}</li>
	        <li>{{user.age}}</li>
	        <li>{{user.sex}}</li>
	    </ul>

8. 拓展：用 jQuery语法 在index.html里渲染几个a标签：

index.html里

		{% for item,index in list %}
		    <a href="">{{item}}</a>
		     {% if index === 1 %}
		     	<p>hello {{index}}</p>
		     {% endif %}
		{% endfor %}

其中list是在后端用模板引擎处理过的


##8.其他知识点

###packge.json

	{
	  "name": "node0926",
	  "version": "1.0.0",
	  "description": "",
	  "main": "index.js",
	  "scripts": {
	    "test": "echo \"Error: no test specified\" && exit 1"
	  },
	  "author": "",
	  "license": "ISC",
	  "dependencies": {
	    "body-parser": "^1.18.2",
	    "express": "^4.15.5",
	    "multer": "^1.3.0",
	    "swig": "^1.4.2"
	  }
	}

**字段dependencies** ：表示依赖模块

装模块写--save时，（npm i xx --save） 会自动把依赖的模块名称和版本添加在packge.json中

在最后上线打包的时候，会安装dependencies这里面依赖的模块

**npm i xx --save--dev**

安装在devDependencies的依赖中，只在开发环境使用，一般放的是工具模块

dev是开发环境的意思，开发环境就是正在写代码的环境;
项目上线变为生产环境，不能再写代码了

带^符号的代表会自动更新版本，不带^符号的代表固定为当前这个版本

**scripts字段**

是npm的命令脚本

* 定义格式： "key": "运行的js文件"

用node app.js运行时，如果文件的名字过长会很麻烦
可以用npm的命令脚本

		"scripts": { // 是npm的命令脚本
			"test": "echo \"Error: no test specified\" && exit 1",
			"index": "node aaaaaaaaaa.js"   // 这样就可以用npm run index  来执行，相当于node aaaaaaaaaa.js
			// 比如 npm run dev 跑的是dev对应的js文件
		},

* 启动npm的命令脚本
	* npm run [key]
		* 比如上面的npm run index  相当于node aaaaaaaaaa.js
	
