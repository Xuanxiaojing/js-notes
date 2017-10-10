#1.jQuery源码浅析

##1-1.关于构造函数
$是一个构造函数

一般的函数都可以作为构造函数，除了箭头函数不能作为构造函数使用！

	let obj = {
		func: function(){
			this.a = 10;
		},
		abc: {
			f:function init(){
				this.b = 20;	
			}
		}
	}

	//console.log(obj.func);
	// obj.func可以作为构造函数
	let newObj = new obj.func();

	console.log(newObj); // func {a: 10}
	// 函数f也可以作为构造函数
	let a = new obj.abc.f();

	console.log(a); // init {b: 20}

##1-2.sizzle.js库
有了sizzle.js，可以在IE6可以使用css3选择器

html

	<div id="abc">
		<ul>
			<li>
				<ol>
					<li>123</li>
					<li>123</li>
					<li>123</li>
					<li>123</li>
				</ol>
			</li>
		</ul>
	</div>

js

	console.log(Sizzle('#abc ul li'));


##1-3.匿名函数自执行，在外部访问匿名函数里的函数

**方式1：把匿名函数里的方法return出去，这样就可以在外部访问到了**

	let a = (function (){
		let Foo = function (){
			console.log(123);	
		}

		return {
			Foo
		}	
	})();

	a.Foo()

**方式2：把匿名函数里的方法挂载在window这个全局变量上**

	(function (){
		let Foo = function (){
			console.log(123);	
		}
		window.Foo = window.$ = Foo;
		// window.变量名 = Fn; 变量名可以自己定义，Miaov有点长，可以用$代替
		/* window.Foo = Foo;
		window.$ = Foo;	 */	
	})();

	$();

##1-4.关于new和省略new

	(function (){
				
		let Foo = function (){

		}

		Foo.prototype.css = function (){
			console.log('css');	
		}
		window.Foo = window.$ = Foo;

	})();
	// 要用new创建出来实例，才可以访问到Miaov的原型上的方法
	let s = new $(); 
	s.css()
	let s2 = new $();
	s2.css()

**不想每次都new，能否在调用Foo函数时在内部返回一个new出来的实例，使开发者自己不用手动new**

1. 使用new来调用构造函数 Foo
2. 不让开发者自己手动的new
3. 调用函数在内部已经new好了实例
4. new的时候不能自己调用自己
5. 定义另一个函数作为构造函数 init
6. 方法都是放在Foo这个函数的原型上，所以需要把init的原型改写为Foo的原型
	* 方便去找Foo原型上的方法

js代码

	(function(){
        // 构造函数Foo
        let Foo = function(){
            // 调用函数时在内部返回一个new出来的实例，使开发者自己不用手动new
            // return new Foo(); // 会死循环
            // 用另外一个函数init作为构造函数来new
            return new init();
        }
        /*
        Foo.prototype.css = function (){
            console.log('css');	
        }
        // 如果要添加很多方法，把prototype写成对象的形式比较方便
        */
        // prototype太长了，可以简写为fn
        Foo.fn = Foo.prototype = {
            constructor:Foo, // prototype改写为对象之后要把constructor指向Foo
            css(){
                console.log('css');
            },
            html(){

            },
            prop(){

            }
        }
        /* 
        // 重新定义一个函数，把这个函数作为构造函数
        let init = function(){

        } 
        */
        // 不想让init函数孤立的存在，把init函数挂载再Foo这个构造函数的原型上  Foo.Foo.init = init
        let init = Foo.fn.init = function(){

        }
        // 调用Foo（或者$）直接返回一个实例，这个实例的构造函数是init，要访问Foo这个函数的原型上的方法，把Foo.prototype直接赋值给init.prototype
        init.prototype = Foo.prototype;

        window.Foo = window.$ = Foo;
    })();    

    /* let m = new Foo();
    console.log(m) */
    console.log(Foo()); // Foo.fn.init {} 
    console.log(Foo().css);
    let s3 = $();   // 返回的对象的构造函数是init
    console.log(s3); // Foo.fn.init {}
    s3.css();

**关于顶层对象window**

把window当做函数的参数传进去，可以节约时间，函数里用到window时不用每次都去全局找window

**关于undefined**

1.变量：声明变量未赋值默认是undefined

	var noGlobal;  // 声明一个变量不赋值，就是undefined

	var undefined = 123;  // chrome不能改

	console.log(undefined);  // ie8下改写了undefined

	var fn;
	// 如果undefined被改写，再做判断会误判的
	if(fn === undefined){
		console.log('没赋值');
	}else{
		console.log('赋值了');
	}

	// 所以用一个没有赋值的变量比较稳妥
	if(fn === noGlobal){
		console.log('没赋值');
	}else{
		console.log('赋值了');
	}

2.作为形参：形参未赋值默认是undefined

	(function (noGlobal){  // 形参未赋值默认是undefined
		var abc;

		if(abc === noGlobal){
			console.log("abc未赋值");
		}else{
			console.log('abc赋值了');
		}
	})()

**jQuery代码片段解析**

	( function( global, factory ) { // 这里的global是自执行时传过来的typeof window !== "undefined" ? window : this；factory是传过来的一大坨函数
	// 为什么要传window进来？为了节约时间，用到window时不用每次都去全局找window
	
		"use strict";
	
		if ( typeof module === "object" && typeof module.exports === "object" ) {
	
			// For CommonJS and CommonJS-like environments where a proper `window`
			// is present, execute the factory and get jQuery.
			// For environments that do not have a `window` with a `document`
			// (such as Node.js), expose a factory as module.exports.
			// This accentuates the need for the creation of a real `window`.
			// e.g. var jQuery = require("jquery")(window);
			// See ticket #14549 for more info.
			module.exports = global.document ?
				factory( global, true ) :
				function( w ) {
					if ( !w.document ) {
						throw new Error( "jQuery requires a window with a document" );
					}
					return factory( w );
				};
		} else {
			factory( global ); // 执行传过来的函数，传一个参数global进去，没有传第二个参数noGlobal，所以下面函数的形参noGlobal是undefined
		}
	
	// Pass this if window is not defined yet
	} )( typeof window !== "undefined" ? window : this, function( window, noGlobal ) { 
		// noGlobal是为了防止IE8以下有人修改undefined
		// 这里是jQuery的主体代码
	}

##1-5.jq整体写法

	(function (global,factory){

		// 兼容各种规范的

		if(typeof module.exports === 'object'){
			module.exports = factory;
		}else{
			factory(global)
		}
		
		
	})(window,function (global){
		// 20000行代码 ....
		let jq = function (){
				
		}	

		jq.prototype =  {

		}
	});

#2.留言板讲解

##2-1.api

###2-1-a.val()方法，

**作用**：获取或设置文本框的值（value值）

**参数**：

- 无参数：获取文本框中的值
	
		$("input").val();
- 有参数：设定文本框的值

		$("input").val("hello world!");

###2-1-b.html()方法

**作用**：获取或设置匹配元素的html内容（相当于innerHTML）

**参数**：

- 无参数：返回元素的内容（取得**第一个**匹配元素的html内容）
	
		$('p').html();
- 有参数：设定元素的内容(设置**所有**匹配元素的html内容)

		$("p").html("Hello <b>world</b>!");

###2-1-c.text()方法

**作用**：获取或设置匹配元素的文本（不带结构 **纯文本**）

**参数**：

- 无参数：返回元素的文本内容（返回**所有**匹配元素的文本内容）

		html

			<p>大明王朝</p>
			<p>大明王朝</p>
			<p>大明王朝</p>
			<p>大明王朝</p>
			<p>大明王朝</p>
		js

		$('p').text(); // 大明王朝大明王朝大明王朝大明王朝大明王朝
- 有参数：设定**所有**匹配元素的文本内容

		$("p").text('<h2>权利的游戏</h2>')


###2-1-d.append(content)

**作用**：向每个匹配的元素内部追加内容。

**参数**content:

	- 写结构
	- 写jq对象
	- 原生元素


- 父级.append(子级)
- 子级.appendTo(父级)


###2-1-e.parent()
**作用**：获取元素的父级(获取到所有匹配到元素，放在一个集合中)

###2-1-f.parents()
**作用**：获取元素的祖先节点(获取到所有匹配到元素，放在一个集合中)

**参数**：

- 不传参：获取所有的祖先节点
- 传参：获取筛选后的祖先节点

###2-1-g：siblings()
兄弟元素们  

###2-1-h：prev()
相邻的上个兄弟 

###2-1-i：next()
相邻的下一个个兄弟 

###2-1-j：prevAll()
上面的兄弟们 

###2-1-k：nextAll() 
下面的兄弟们

###2-1-l：end()
回到最近的一个"破坏性"操作之前。即，将匹配的元素列表变为前一次的状态。

	$('.active').css('background','red').text('miaov').attr('miaov','ketang')
	.siblings('.yellow').attr('abc',1).siblings('.a').end().end().attr('custom',123)

###2-2.代码

html

	<section id="wrap">
		<div id="content">
			<h2 class="logo"><img src="img/log.png"></h2>
			<div class="clearfix">
				<textarea class="message"></textarea>
				<input type="button" value="提交" class="btn">
			</div>
			<ul id="mList">
				<!-- <li class="mText">
					<span class="mIcon1"></span>
					<p class="mComtent">你好!</p>
					<input type="button" value="删除" class="mRemove" />
				</li> -->
			</ul>
		</div>
	</section>

js

	$(".btn").click(function (){
		let val = $(".message").val().trim();

		if(val === ''){
			alert('请输入内容')
		}else{
			let html = `<li class="mText">
				<span class="mIcon1"></span>
				<p class="mComtent">${val}</p>
				<input type="button" value="删除" class="mRemove" />
			</li>`;

			// 插入到ul中 appendChild

			$('#mList').prepend(html)

		}

		$(".message").val('')	
	})

	$("#mList").on('click','input',function (){
		$(this).parent().remove();	
	})

**nodeValue属性**（文本节点内容nodeValue）

		let childs = $('.xiaoshuo')[0].childNodes;

		console.log(childs[0].nodeValue.trim());


#3.用jQuery写商品筛选

**分析：**

- 问：怎么使每行li都只有一个当前被点击的a标签高亮，其他的都不高亮？
- 答：用siblings(选择器)找到高亮的兄弟元素，然后取消他们的样式

- 问：怎么使上面生成的mark标签每个类别只有一个？
- 答：把点击的a标签的内容存在对象里，利用对象key值的唯一性

		{
			点击的a标签的父级li的下标：点击的a标签的内容
		}

- 问：怎么使上面生成的mark标签 和列表里li从上到下的顺序相同
- 答：用for循环去循环数据，for循环刚好是有顺序的

html代码

	<div id="wrap">
		<section id="section">
			<nav id="choose">
			你的选择:
			<!-- <mark>小米<a href="javascript:;" index="0">x</a></mark> -->
			</nav>
			<ul id="type">
				<li>
					品牌：
					<a href="javascript:;">苹果</a>
					<a href="javascript:;">小米</a>
					<a href="javascript:;">锤子</a>
					<a href="javascript:;">魅族</a>
					<a href="javascript:;">华为</a>
					<a href="javascript:;">三星</a>
					<a href="javascript:;">OPPO</a>
					<a href="javascript:;">vivo</a>
					<a href="javascript:;">乐视</a>
					<a href="javascript:;">360</a>
					<a href="javascript:;">中兴</a>
					<a href="javascript:;">索尼</a>
				</li>
				<li>
					尺寸：
					<a href="javascript:;">3.0英寸以下</a>
					<a href="javascript:;">3.0-3.9英寸</a>
					<a href="javascript:;">4.0-4.5英寸</a>
					<a href="javascript:;">4.6-4.9英寸</a>
					<a href="javascript:;">5.0-5.5英寸</a>
					<a href="javascript:;">6.0英寸以上</a>
				</li>
				<li>
					系统：
					<a href="javascript:;">安卓 ( Android )</a>
					<a href="javascript:;">苹果 ( IOS )</a>
					<a href="javascript:;">微软 ( WindowsPhone )</a>
					<a href="javascript:;">无</a>
					<a href="javascript:;">其他</a>
				</li>
				<li>
					网络：
					<a href="javascript:;">联通3G</a>
					<a href="javascript:;">双卡单4G</a>
					<a href="javascript:;">双卡双4G</a>
					<a href="javascript:;">联通4G</a>
					<a href="javascript:;">电信4G</a>
					<a href="javascript:;">移动4G</a>
				</li>
			</ul>
		</section>
	</div>

js代码

	let obj = {};

	/*
		{
			0: '锤子',
			1: '4.0'
		}
	*/

	let lis = $('#type li');
	let len = lis.length;
	let choose = $('#choose');

	$('#type').on('click', 'a' ,function (){
		$(this).siblings('.yellow').removeClass('yellow').end().addClass('yellow');// 找到当前点击的a标签的类名为yellow的兄弟元素，并且 去掉它的yellow类名，并且给当前点击的a添加yellow类名
		let p = $(this).parent(); // 找到当前点击的a的父级li
		let index = p.index(); // 找到当前点击的a的父级li的下标
		obj[index] = $(this).text(); // 用obj存一下当前点击的a的内容
		/*
			{
				点击的a标签的父级li的下标：点击的a标签的内容（因为对象的key值是唯一的，所以可以保证每行只有一对key：value，obj里存的一直是当前最新点击的a标签 以及 a标签对应的li的下标）
			}
		*/
		choose.html('你的选择:')
		// 每点击一个li，obj里就增加一对key value			
		// 对象可能是无序的，如果用for in去取数据来生成，可能是是无序的，而for循环刚好是有顺序的，所以用for循环可以按li的下标顺序生成有序的mark标签
		// 有几个li就循环几次（len是li的个数）
		for( var i = 0; i < len; i++ ){
			// obj[i]不一定存在，因为点击某一行的a标签时，如果其他行没有被选中的a标签，obj[i]就是undefined，所以要过滤一下
			if(obj[i]){ // 如果obj[i]存在，生成mark结构，append到choose里
				let h = `<mark>${obj[i]}<a  href="javascript:;" index="${i}">x</a></mark>`;
				choose.append(h);
			}
		}
	})
	/*
		 删除  事件委托 
		 点击删除按钮，要移除其父级，并且把下面列表里对应的a标签取消高亮
		 //问：怎么找到所点击的删除按钮对应的下面列表里的a标签？ 
		 //答：通过下标，使每个mark标签和列表里每个li的下标相对应
		 	// 这时要在生成结构时，在mark里的a标签的行间，用data-自定义属性把obj的key值作为删除按钮的下标，这样删除按钮的下标和obj里的key值（也就是原来所点击的a所在的li的下标）对应起来了
	*/
	
	choose.on('click',"a",function (){ // a是删除按钮
		$(this).parent().remove();	// 移除当前所点击的删除按钮的父级
		let index = $(this).attr('index'); // 获取一下当前点击的a标签的index属性值（事先存的对应的li的下标，也就是obj里的对应的key值）
		lis.eq(index).find('.yellow').removeClass('yellow'); // 找到这个li下黄色的a标签，取消高亮
		delete obj[index]; // 把obj里对应的数据删除
	})

#4.用vue写商品筛选
##4-1.抽离数据

**抽离商品列表的数据**

**contents()**

查找匹配元素内部所有的子节点（包括文本节点）。


html同上

js

	let lis = $('li');

	$("li").contents().filter(function(){ return this.nodeType != 1 && this.nodeValue.trim() !== ''; }).wrap("<b/>");
	/*
		$("li").contents()拿到所有li的所有的子节点（包括文本节点）
		然后用filter筛选：
			筛选条件：如果不是元素节点（标签，即每一个HTML元素） 并且 文本节点内容去除前后空格之后不是空字符串（换行也算一个文本节点，表现为一堆空格）
		经过筛选，返回的是咱们想要的商品内容，把每个内容用<b></b>标签包起来（便于后面识别和操作）
	*/
	let b = $('li b'); // 获取所有li下的b
	// [...lis]把lis转成数组
	let arr = [...lis].map(function (item,index){ 
		// item是lis里的某个li，index是每个li的下标
		let as = $(item).find('a'); // 找到某个li下的所有a标签，as是一个集合
		// 把as转成数组然后进行map循环
		let arr2 = [...as].map(function (item2){
			// item2是某个li下的某个a标签
			return item2.innerHTML; // 把这个a标签的内容放在新数组里返回出去
		})
		// 最终arr2存放着某个li下的a标签的innerHTML
		// 返回
		return {
			title: b.eq(index).text().trim(),
			list: arr2
		}
		/*
		// 比如循环到第一个li，以下内容被放到新数组里赋值给arr，继续循环……
			{
				"title": "品牌：",
				"list": [
				"苹果",
				"小米",
				"锤子",
				"魅族",
				"华为",
				"三星",
				"OPPO",
				"vivo",
				"乐视",
				"360",
				"中兴",
				"索尼"
				]
			}
		*/

	})
	// 等arr的map循环完毕，就获取了所有的数据，并进行了分类
	console.log(JSON.stringify(arr,null,2));
	/*
		[
			{
				"title": "品牌：",
				"list": [
				"苹果",
				"小米",
				"锤子",
				"魅族",
				"华为",
				"三星",
				"OPPO",
				"vivo",
				"乐视",
				"360",
				"中兴",
				"索尼"
				]
			},
			{
				"title": "尺寸：",
				"list": [
				"3.0英寸以下",
				"3.0-3.9英寸",
				"4.0-4.5英寸",
				"4.6-4.9英寸",
				"5.0-5.5英寸",
				"6.0英寸以上"
				]
			},
			{
				"title": "系统：",
				"list": [
				"安卓 ( Android )",
				"苹果 ( IOS )",
				"微软 ( WindowsPhone )",
				"无",
				"其他"
				]
			},
			{
				"title": "网络：",
				"list": [
				"联通3G",
				"双卡单4G",
				"双卡双4G",
				"联通4G",
				"电信4G",
				"移动4G"
				]
			}
			]
	*/

##4-2.代码

html

	<div id="wrap">
		<section id="section">
			<nav id="choose">
			你的选择:
			<!-- 根据selectArr（存放被选中的a标签）生成结构 -->
			<mark v-for="value in selectArr">{{value}}<a href="javascript:;" index="0">x</a></mark>
			</nav>
			<ul id="type">
				<!-- 根据数据生成li -->
				<li v-for="item,index of list">
					{{item.title}}
					<!-- 
						根据每个对象里的list数组生成a标签
						是否添加class名：每条数据里有一个selectIndex属性，存的是当前点击的a标签在同一行的a标签集合里的下标（代表这个li里给哪个a标签添加高亮），初始值是-1，没有一个a标签匹配到
							点击a标签时，改变selectIndex属性值，这样data里的list就改变了，会自动重新渲染结构，重新判断哪个a标签的下标和当前点击的a标签的下标相同，相同就添加类名yellow
					 -->
					<a 
						href="javascript:;" 
						v-for="itemA,i in item.list"
						:class="{yellow: i == item.selectIndex}"
						@click="clickA(item,i,itemA,index)"
					>{{itemA}}</a>
				</li>
			</ul>
		</section>
	</div>

js

	let data = [
	  {
		"title": "品牌：",
		// 'selectIndex':-1,
	    "list": [
	      "苹果",
	      "小米",
	      "锤子",
	      "魅族",
	      "华为",
	      "三星",
	      "OPPO",
	      "vivo",
	      "乐视",
	      "360",
	      "中兴",
	      "索尼"
	    ]
	  },
	  {
	    "title": "尺寸：",
	    "list": [
	      "3.0英寸以下",
	      "3.0-3.9英寸",
	      "4.0-4.5英寸",
	      "4.6-4.9英寸",
	      "5.0-5.5英寸",
	      "6.0英寸以上"
	    ]
	  },
	  {
	    "title": "系统：",
	    "list": [
	      "安卓 ( Android )",
	      "苹果 ( IOS )",
	      "微软 ( WindowsPhone )",
	      "无",
	      "其他"
	    ]
	  },
	  {
	    "title": "网络：",
	    "list": [
	      "联通3G",
	      "双卡单4G",
	      "双卡双4G",
	      "联通4G",
	      "电信4G",
	      "移动4G"
	    ]
	  }
	];


	// 对象的顺序，如果key值为数字，是否是按顺序排的？？？

	data.forEach(function (item){
		item.selectIndex = -1;
	})

	new Vue({
		el:'#wrap',
		data: {
			list: data,
			selectArr: {} // 存放被选中的数据
		},
		methods: {
			// 点击列表里的a标签时，执行这个函数，传入当前a标签对应的数据item（改变该item的selectIndex属性值）
			clickA(item,i,itemA,index){
				item.selectIndex = i;  // 下标，目的就是让对应下标的a添加class
				//this.selectArr.push(itemA);  // 要在筛选条件中要生成结构的
				this.selectArr[index] = itemA; // 把当前点击的a标签存入selectArr对象里，
				// 因为对象的key值是唯一的，所以selectArr里存的一直是当前最新点击的a标签
				/*
					{
						当前点击的a标签对应的li的下标：当前点击的a标签
					}
				*/
			}
		}
	})
	





