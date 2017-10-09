#1.jquery写选项卡

**手册使用说明：静态方法和原型上的方法**

- 静态方法：挂载在函数上的方法
- 原型上的方法：函数（构造函数）的prototype上的方法

手册里如果前面没有$，说明是原型上的方法，必须通过实例$().方法()来调用
	
	console.log($().click);
	console.log($().merge); // 原型上没有merge方法，所以是 undefined
如果前面有$，说明是静态方法，需要通过$.方法()调用

	console.log( $.merge );
	console.log($.click); // jq没有click静态属性，所以是undefined

- 调用原型上的方法
	- 	$().方法()
- 调用静态方法
	- 	$.方法()


###选项卡思维

1. 选择元素
	tagname classname id css3选择器
2. 绑定事件
3. 控制样式 class 和 style
4. 点击按钮对应的div出现

**选项卡代码**

html

	<body>
	    <input type="button" value="按钮" class="yellow">
	    <input type="button" value="按钮">
	    <input type="button" value="按钮">
	    <div style="display:block;">1</div>
	    <div>2</div>
	    <div>3</div>
	</body>

js

	$(function(){ // 等页面中的元素加载完成之后执行这个函数
        $('input').click(function(){
            $('input').removeClass('yellow'); // 所有的按钮清空高亮
            console.log(this); // 首先，this指向触发这个函数的元素，其次，这里的this是原生的元素
			// 所以如果要调用jQuery的addClass()方法，就用$(this)把this转成jq的元素

            $(this).addClass('yellow'); // 当前点击的按钮添加高亮
            // $('div').css('display','none'); // 所有的div隐藏
            $('div').hide(); // 所有的div隐藏
            // 怎么找到对应的下标？  index
            // let i = $(this).index(); // i是 调用index方法的这个元素在同级的元素中所处的下标
            // 想要的是这个元素在指定的元素集合中所处的下标
            let i = $('input').index(this);  // this这个元素在所有input集合里所处的下标
            // 根据下标获取当前所点击的按钮对应的div
            // $('div')[i].css('display','block'); // 报错，因为 直接用下标去取的话 获取的是原生的元素，原生的元素没有css()方法
            
            // $('div').eq(i).css('display','block');
            $('div').eq(i).show();  // 当前点击的按钮对应的div显示
        })
    })

精简写法：

	$(function(){
		$('input').click(function (){
			/*$('input').removeClass('yellow');
			$(this).addClass('yellow');
			$('div').hide();
			$('div').eq($('input').index(this)).show();*/

			let i = $('input').index(this);
			$('input').removeClass('yellow').eq(i).addClass('yellow');  // 链式操作
			$('div').hide().eq(i).show();


		})
	});
####1.页面加载后获取元素

jQuery(callback) 也就是 **$(callback)**

$ 是jquery这个库暴露出来的全局的函数
callback是当DOM加载完成后要执行的函数

	$(function(){
	  // 文档就绪
	});

**$(callback)和原生里的window.onload的区别**

- window.onload事件是等页面里所有的**资源**加载完成后触发，文档（即各种标签和文本）先加载完，图片地址src连接的是资源，js和css也是资源（如果是远程的加载需要时间）
- $的callback函数是当页面的文档（标签）加载完成后执行



html

	<body>
		<div></div>
		<div></div>
		<div></div>
		<div></div>
		<img src="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1507525481937&di=53f2aac91da494ae275988e1b606c3a7&imgtype=0&src=http%3A%2F%2Fimgsrc.baidu.com%2Fimgad%2Fpic%2Fitem%2F3801213fb80e7beca9004ec5252eb9389b506b38.jpg">
		<img src="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1507525490489&di=40e7c960e3e8e3b914e8fc30e4074b9b&imgtype=0&src=http%3A%2F%2Fimgsrc.baidu.com%2Fimage%2Fc0%253Dshijue1%252C0%252C0%252C294%252C40%2Fsign%3D0a0eda45c9cec3fd9f33af36bee1be4a%2Ffd039245d688d43f3488b67b771ed21b0ef43b11.jpg">
	</body>

js

	<script>
		/*
			window.onload  页面中所有的资源（css，js，图片）加载完成之后 触发
			$(callback)  页面的文档（标签）加载完成后触发
		*/
		window.onload = function (){
			console.log('window.onload');	
		}

		$(function (){
			console.log("$");		
		})
		$(document).ready(function (){
			console.log('ready');	
		})
	</script>

以上代码先打印$，再打印ready，再打印window.onload

####2. 选择元素

**$("id|class|tagName|css选择器");**

	$('input')   -------页面中所有的input
	$('div input')   --------div下的input

####3.绑定事件

$(选择器).事件名(callback事件处理函数)

一个元素的集合.事件名(callback事件处理函数)，那么该集合里的每个元素都会绑定上该事件处理函数,比如：

html

	<div>1</div>
    <div>2</div>
    <div>3</div>

js

	$('div').click(function(){
		console.log(123)
	})
	

	

####4.原生元素和jq元素

- 用$()获取的元素是jQuery的元素，有jQuery的方法。
- 用getElementById()等一系列方法获取的是原生的元素
- 原生的元素不能使用jQuery的方法，需要包装成jQuery的元素才能使用
- jQuery的元素不能使用原生的方法，需要包装成原生的元素才能使用

- 原生元素->jq元素： 
	- `$(原生的元素)` 就把原生的元素变成了jq的元素

- jq元素-> 原生元素： 
	1. 加下标:
	
			let jqBox = $('#box')jqBox[0]就是原生的元素
	2. get(index):通过检索匹配jQuery对象，得到对应的DOM元素
	
			console.log( jqBox.get(0) );
			console.log(jqBox[0]);

####5.获取一个集合里的其中一个元素：

- 方法1：直接用下标： $('input')[下标]
- 方法2：用get()方法： $('input').get[下标]
		- $(this).get(0)与$(this)[0]等价。
- 方法3：$('input').eq(1)
	- 可以接收负参数





#2.api

##2-1.eq(index|-index)
**作用**：获取指定小标的元素的jq对象形式

**参数：**

- index：一个整数，指示元素基于0的位置,这个元素的位置是从0算起。
- -index：一个整数，指示元素的位置，从集合中的最后一个元素开始倒数。(-1算起)

	$('div').eq(1);  // 获取的是div集合里的下标为1的div(jq元素)
	$('div').eq(-1); // 获取的是div集合里的倒数第一个div(jq元素)
	$('div')[1];  // 获取的是div集合里的下标为1的div(原生元素)

##2-2.hide()和show()

对应display:none和display:block

	$('div').css('display','none'); // 所有的div隐藏
	$('div').hide(); // 所有的div隐藏

	$('div').eq(i).css('display','block');
    $('div').eq(i).show();  // 当前点击的按钮对应的div显示

##2-3.css(name|pro|[,val|fn])

**a.读：**
.css(属性名)
**获取的是计算后的样式**，

1. jq元素.css(属性名)
	
2. jq元素集合.css(属性名)：获取的其实是集合里的第一个元素的这个css属性

		console.log($("div").css("backgroundColor"))

**b.写（设置）**

1. .css(属性名，属性值)

		$('div').eq(1).css('background','red')
2. .css({  属性名：属性值， 属性名：属性值， 属性名：属性值 })

		$('div').eq(1).css({
		 	width: 100,
		 	height: 200,
		 	fontSize: 50
		 })
3. .css( { 属性名：回调函数，属性名：回调函数} )

比如：逐渐增加div的大小

		$("div").click(function() {
		    $(this).css({
		      width: function(index, value) {
		        return parseFloat(value) * 1.2;
		      }, 
		      height: function(index, value) {
		        return parseFloat(value) * 1.2;
		      }
		    });
		  });

##2-4.attr(name|properties|key,value|fn)
**a.作用：**设置或返回被选元素的属性值。(给元素在行间设置属性（也就是说设置的属性会在行间显示出来），也只能获取元素行间的属性)

**b.参数：**

**读**：

name：属性名称

1. jq元素.attr(属性名)
	
2. jq元素集合.attr(属性名)：获取的其实是集合里的第一个元素的这个属性

		$('div').attr('id') // 如果是jq元素集合，只能获取一个，默认获取第一个

**写**：

properties：作为属性的“名/值对”对象

	$("img").attr({ src: "test.jpg", alt: "Test Image" });

key,value：属性名称，属性值

	$('input').attr('type','button')
	// 把所有input的type属性设置为"button"
	$('div').attr('c', 'miaov');
	// 给所有div设置c属性，属性值为miaov
	<div c="miaov">123</div>
key,function(index, attr)
	
	$('div').attr('class',function(){return 'box'})
	// 把所有div的class名设置为“box”

**注意：如果不在元素的行间设置，而是在元素上挂载属性，那么用attr()方法访问不到该属性**

	$('div').eq(0).a = 'hello'; // 在元素上挂载属性
	console.log($('div').eq(0).attr('a')); // undefined

**attr()与原生的setAttribute()和getAttribute()类似**

	let box = document.getElementById('div1');
	box.index = 1;
	console.log(box.getAttribute('index')); // null

	let box = document.getElementById('div1');
	box.index = 1;
	// box.setAttribute('index',1);
	console.log(box.getAttribute('index'));
##2-6.removeAttr(name)

**作用：**从每一个匹配的元素中删除一个属性

**参数：**
name：属性名

	$('div').removeAttr('class', 'box');

##2-7.data([key],[value])

缓存数据的方法:不写在行间，把数据缓存到元素上，类似于`box.index = 1;`直接在元素上加属性

**作用：**在元素上存放或读取数据,返回jQuery对象。

	box.data('miaov', 'ketang');

**参数：**

- 一个参数：读（只能读到通过data存放的数据）
	
	box.data('miaov'); // ketang
	document.getElementById('box').i = 11111;
	console.log(box.data('i')); // undefined
- 两个参数：写

	box.data('a', 'hello');

**和attr()对比：**
attr()方法相当于原生的setAttribute()和getAttribute()，获取和设置的是行间的属性
data()方法是把数据缓存到元素上


##2-8.removeData([name|list])

**作用：**在元素上移除存放的数据（只能移除通过data方法存放的数据）

	document.getElementById('box').i = 11111;
	box.removeData('i')		
	console.log(document.getElementById('box').i); //11111

	
* 不传参数：移除所有通过data方法存放的数据

* 传参数
	* name：	`box.removeData('miaov')`
	* list: `box.removeData(['miaov','a'])`移除miaov和a这两个数据



1:属性名称。

2:返回属性值的函数,第一个参数为当前元素的索引值，第二个参数为原先的属性值。



##2-9.index([selector|element])

##2-10.get([index])方法
**作用：**取得其中一个匹配的元素。返回的是DOM对象，类似的有eq(index),不过eq(index)返回的是jQuery对象。
**参数：**

- 传参数：取得第 index 个位置上的元素
- 不传参数：取得所有匹配的 DOM 元素集合。（是原生的元素）

##2-11.each()
循环数组

	.each(function (index,item){
		// this => 循环的原生的元素	
	})

##2-12..find(选择器)

找到元素的指定选择器的子孙（后代元素）元素

	let active = $('.active');
	console.log(active.find('.abc')); // 找到active元素下的所有类名为.abc的子孙元素

##2-13.on(events,[selector],[data],fn)绑定事件
类似.addEventListener()

**作用**：绑定事件

**参数：**

1. 事件名：可以是多个事件名，用空格隔开 'click keydown mouseover'
2. 选择器：过滤触发事件的元素
3. 往事件处理函数里传的参数
4. 事件处理函数（通过ev.data拿传进来的参数）

在原生js里，这样不能给事件处理函数传递参数，因为这个函数不是开发者调用的

	document.onclick  = function (){
			
	}
多个事件共用一个事件处理函数

	$(document).on('click keydown','.active',{a:1},function (ev){
		console.log(ev.data); //  {a: 1}
	})

通过**ev.type判断事件类型**

**jq的事件对象ev是经过jq包装的对象（jq内部做了兼容），可以通过ev.originalEvent获取原生的事件对象**

	$(document).on('click mouseover mouseout',function (ev){
		console.log(ev.type)
		if(ev.type === 'click'){

		}else if(){

		}
	})

写成对象形式

	$(document).on({
		click(){
			console.log('click');
		},
		mouseover(){
			console.log('over');
		}
	})


##2-14.off(events,[selector],[fn])

**作用：**去掉匹配的元素所有事件的事件处理函数

**参数**

- 第一个参数是指定的事件
- 第二个参数是指定的处理函数
- 不传参：去掉匹配的元素所有事件的处理函数
- 传参：
	- 只传第一个参数：会取消指定事件的所有事件处理函数
	

类似.removeEventListener()

	function fn2(){
		console.log('over2');
	}
	$(document).mouseover(fn2)
	$(document).off('mouseover',fn2);

##2-15.prop(name|properties|key,value|fn)

**作用**：获取在匹配的元素集中的第一个元素的属性值。

Attributes： 标签写在行间的属性， 使用attr()获取

Properties：获取元素后的属性 

例子：

html

	<input type="checkbox" checked='false'  />
	<input type="button" value="获取checkbox的状态" />

js

	$(function (){
		// 属性选择器
		console.log($('input[type="button"]'));
		let checkbox = $('input[type="checkbox"]')
		$('input[type="button"]').click(function 	(){
			 console.log(checkbox.attr('checked'));  // 获取的是行间中的checked属性，打印“checked”
			 console.log(checkbox.prop('checked')); // 随着选框的勾选和取消勾选，打印true或false
		})

	})

##2-16.toArray()

语法：jq元素集合.toArray()

**作用：**集合（类数组）转成数组


#3.回顾es6里数组的find()方法

见ES6的markdown笔记

应用见百度音乐全选的判断是否全选的代码

	$('#list').on('click','input',function (){
		let i = inputs.toArray().find(function (item){
			return !item.checked;
		})
		checkedAll.prop("checked",!i);
	})