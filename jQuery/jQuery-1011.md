#1.jQuery写拖拽

##1-1.知识点

##1) jq元素.offset()


**作用**：
获取匹配元素在**当前视口**的相对偏移。

回顾原生里的方法：

元素.offsetLeft：元素到定位父级左边的距离
元素.getClientBoundingRect() ： 返回一个对象

**比如：**
{top: 0, right: 100, bottom: 100, left: 0, width: 100,height:100}

- top：元素的顶部距离浏览器视口顶部的距离
- bottom：元素的底部距离浏览器视口顶部的距离
- left：元素的最左边距离浏览器视口左边的距离
- right：元素的最右边距离浏览器视口左边的距离
- width和height：元素的宽高（包括边框）

##2) jq元素.position()
获取匹配元素相对**定位父级**的相对偏移

##3）ev.clientX和ev.clientY
ev.clientX：鼠标相对视口左边的距离
ev.clientY：鼠标相对视口右边的距离


**非面向对象的写法**

html

	<div id="box1"></div>
	<div id="box2"></div>

js

	$("#box1").mousedown(function (ev){
		let disX = ev.clientX - $(this).offset().left;
		let disY = ev.clientY - $(this).offset().top;

		function move(ev){
			$("#box1").css({
				left: ev.clientX - disX,
				top: ev.clientY - disY
			})	
		}
		function up(ev){
			//$(document).off('mousemove mouseup');	
			$(document).off('mousemove', move);	
			$(document).off('mouseup', up);	
		}

		$(document).mousemove(move)
		$(document).mouseup(up)

	})

**弊端**：如果好多元素都需要拖拽功能，要一个一个重复写
**封装函数**：需要传参，也不是最佳方案

所以用**面向对象**的方法 封装一个**类**

##1-2.面向对象的思想

###1-2-1.**比如，多个函数想要共享数据：只要这些函数（方法）都能访问到该数据，就可以共享数据了**

	function fn1(){
        var a = 1;
    }
    function fn2(){
        var b = 2;
    }

如果fn1和fn2都想访问变量a和b，

####方法1：把a,b都放在全局

	var a = 1;
	var b = 2;
	function fn1(){
        console.log(a)
    }
    function fn2(){
        console.log(b)
    }
要尽量少的把变量暴露在全局

####方法2：把要共享的变量放在全局的一个对象上

	let obj = {
        a:1,
        b:2
    }
	function fn1(){
		obj.a = 10;
        console.log(obj.a); // 10
    }
    function fn2(){
        console.log(obj.b); // 2
    }

但是上面 函数fn1，fn2和obj都在全局暴露

####所以用面向对象，直接把要共享的多个函数和方法封装到一起，函数的功能是独立的或者要配合完成一个功能

	class custom {
		constructor(){
			this.a = 10;
			this.b = 20
		}
		fn1(){
			console.log(this.a)
		}
		fn2(){
			console.log(this.b)
		}
	}
###1-2-2.new 调用一个函数做了什么？

* 1.隐式创建一个对象（yinshiObj）（要往这个对象上添加属性）
* 2.this =>这个隐式对象（通过this往这个对象上添加属性）
* 3.执行函数开始添加属性
* 4.把这个隐式对象返回出去（为了方便在外面操作这个隐式对象）


构造函数里有一个隐式对象

- 共享的方法都统一放在构造函数的原型上（构造函数里的隐式对象可以访问到）；
- 属性都统一挂载在这个隐式对象上（用new调用时会把这个隐式对象返回出去，称为“实例对象”）
- 所以综上所述，可以通过隐式对象访问到这个构造函数里封装的所有属性和方法
- 因而，可以通过new出来的实例访问到这个构造函数里封装的所有属性和方法

**注意this的指向**
new 调用构造函数时，内部令this =>这个隐式对象，所以可以通过this访问到这个构造函数里封装的所有属性和方法

但是，在构造函数里的函数里使用this时，要注意this的指向

###1-2-3.面向对象写拖拽

> - 把两个相似的功能提取出来，做到通用性
> - 功能就是做拖拽的，作用的元素不同
> - 在拖拽的过程中，基础拖拽功能中，不应该做具体的事情，应该交给使用这个功能的开发者自由决定如何去写

**需求：**div1和div2都可以拖拽，div1在鼠标down下时变成蓝色

**订阅发布类的封装**

	class OrderPublish{
        constructor(){
            this.telBook = {}; // 记录订阅者
        }
        // 订阅方法subscribe：往subscribe函数里传一个订阅类型area和一个函数telNum，放在“电话本”上
        //（函数就相当于订阅者的电话号码，订阅者收到发布信息就执行这个函数）
        subscribe(area,telNum){ // area是订阅的类型，telNum是放置的监听器（函数）,类似addEventListener();
            if(!this.telBook[area]){
                this.telBook[area] = []
            }
            this.telBook[area].push(telNum)
        }
        // 发布的方法：某个时刻，要发布area有关信息，就调用publish函数，随之会调用“电话本里的”函数（这些函数在监听publish()什么时候触发）
		//调用函数就相当于订阅者接到了客服打来的电话，订阅者收到电话，就执行函数体
        publish(area){
            let arr = this.telBook[area];
            $(arr).each(function(index,item){
                item();
            })
        }
        // 取消订阅的方法：类似removeEventListener();
        remove(area,telNum){
            //找到传进来的area对应的数组，数组里存的是area对应的监听者（函数）
            let arr = this.telBook[area];
            for(var i=0; i<arr.length; i++){
                if(arr[i] === telNum){//如果某个函数等于传进来的函数
                    arr.splice(i,1);//就把这个函数从数组arr里删除，下次再发布area时，这个被删除的就不会监听到消息了
                }
            }
        }
    }

**拖拽类的封装**

	class Drag extends OrderPublish{
        constructor(box){
            super(box);
            this.dragBox = box;             
            
            // 如果在up时用 $(document).off(this.downFn.bind(this));，会发现无效
            // 调用bind之后，会返回新函数，所以把这个新函数挂载在实例上
            // 目的就是当取消的时候，可以取消这个返回的新函数，因为这个返回的新函数才是事件处理函数
            this.downFn = this.downFn.bind(this);
            this.moveFn = this.moveFn.bind(this);
            this.upFn = this.upFn.bind(this);
        }
        dragInit(){ // 启动函数
            this.dragBox.on('mousedown',this.downFn)                
        }
        downFn(ev){
            this.publish('down-pub')
            ev.preventDefault();
            // 获取鼠标down下时距离box的左边和上边的距离
            // 鼠标相对于文档的左边缘的位置 - 
            // console.log(this.dragBox.offset()); // 是一个对象{top: 0, left: 0}
            this.disX = ev.pageX - this.dragBox.offset().left;
            this.disY = ev.pageY - this.dragBox.offset().top;
            $(document).on('mousemove',this.moveFn);
            $(document).on('mouseup',this.upFn);
        }
        moveFn(ev){
            this.left = ev.pageX - this.disX;
            this.top = ev.pageY - this.disY;
            console.log(this.left,this.top);              
            this.dragBox.css({
                left: this.left,
                top: this.top
            })
        }
        upFn(){
            console.log(123)
            $(document).off(this.moveFn);
            $(document).off(this.upFn);
        }
    }

使用

	let div1 = $('#div1')
    let d1 = new Drag(div1)
    d1.dragInit();
    function down(){
        div1.css('background','skyblue')
    }
	d1.subscribe('down-pub',down); // 订阅down-pub“事件”
	
	let div2 = $('#div2')
    let d2 = new Drag(div2)
    d2.dragInit();

##1-3.自定义事件

**系统的事件：**

- 比如click，mouseover，等等，默认就有，不需要自己定义
- 点击时，默认会发布click事件，不管有没有人监听，都会发布click事件
- 如果想要在点击某个元素时，做一些事情，那么在该元素的click事件上绑定一个监听器即可，当内部发布click事件时，就会自动执行监听器的函数体

###1-3-1.jQuery里怎么自定义事件？
**jq元素.on()**
	
	// 在$('#inputs')元素的click123事件上绑定一个监听器
	$('#inputs').on('click123',function (){
		alert(1)	
	});
	// 这样不会弹出1，因为系统没有提供名字为click123的事件
	
###1-3-2.jQuery里怎么发布事件？
**jq元素.trigger(事件名)**

* 点击按钮时发布click123事件

		// 在$('#inputs')元素的click123事件上绑定一个监听器
		$('#inputs').on('click123',function (){
			alert(1)	
		});
		// 发布click123事件
		$("#inputs").trigger('click123')
		// 这样就会弹出1，只要发布了click123事件，前面绑定的监听器监听到click123事件，就会执行，弹出1

* 可以不一上来就发布事件，可以在某个时刻（比如点击按钮时）发布click123事件

		$("#inputs").click(function (){
			$("#inputs").trigger('click123')	
		})

###1-3-3.命名空间
事件命名空间类似css的类，我们在事件类型的后面通过点加名称的方式来给事件添加命名空间

命名空间可以**有效地管理同一事件的不同监听器**

比如

1. 场景1 

		// 给inputs的click.a事件有绑定监听器
		$('#inputs').on('click.a',function (){
			alert(1)	
		});
		$('#inputs').on('click.b',function (){
			alert(2)	
		});

	这样，在触发inputs的click事件时（点击inputs时），会同时触发click.a和click.b事件，点击按钮时，会先弹出1，再弹出2

2. 场景2：
当点击inputs2时发布click.a事件

		// 给inputs的click.a事件有绑定监听器
		$('#inputs').on('click.a',function (){
			alert(1)	
		});
		// 当点击inputs2时发布click.a事件，上面的监听器会执行，弹出1
		$("#inputs2").click(function (){
			$('#inputs').trigger("click.a")	
		});

###1-3-4。解耦的思想

	let obj = {
		fn1(){},
		fn2(){},
		func1(){},
		o:{
			func1(){},
			func2(){}
		}
	}

调用时

	$('#div1').on('click',function (){
		obj.fn1()	
	});
	('#div2').on('click',function (){
		obj.fn2()	
	});
这样，如果某天要删除fn1，直接在obj里删除即可，下面调用的额
	
**js的发展**
： 原生js -> prototype.js -> jquery.js(10) -> YUI(KISSY) -> backbone(MVC) -> ng(MVVM) -> Vue.js(MVVM) + reactjs(V) ???

##4.api

##4-1.off([type,listener])

- type 事件类型
- listener 监听器

- 不写参数 会取消掉元素所有事件的监听器
- 写了type  就会取消这个type对应的所有的监听器
- 写了type和listener 就会取消这个type对应的listener这个监听器
- 隔行运动以及获取正在运动的元素

	$(document).mouseover(function (){
		console.log('mouseover');		
	})
	$(document).click(function (){
		console.log('click');		
	})
	$(document).click(function (){
		console.log('click');		
	})

	function fn(){
		console.log('click');		
	}
	$(document).click(fn)
	$(document).off('click');
	$(document).off('click',fn);



##4-2.动画相关的

- hide(持续时间)  -------同时改变宽高和透明度
- show(持续时间)  -------同时改变宽高和透明度
- toggle(持续时间)
- slideUp(持续时间)   ------收起，改变高，最终display:none
- slideDown(持续时间) ------放下，改变高
- slideToggle(持续时间) ------改变高
- fadeOut(持续时间)
- fadeIn(持续时间)
- fadeToggle(持续时间)
- .animate(运动的属性对象，持续时间，运动结束回调函数)

	

先运动left值，再运送width值，再运动opacity值

	$("#box1").animate({
		left: 1000
	},2000,function (){
		$("#box1").animate({
			width: 500
		},2000,function (){
			$("#box1").animate({
				opacity: 0
			},2000,function (){
				console.log('结束');
			})
		})
	})
这样嵌套太多不好，写成扁平化的

	$("#box1")
		.animate({
			left: 1000
		},2000)
		.animate({
			width: 500
		},2000)
		.animate({
			opacity: 0
		},2000)



	// 一开始使隔行运动
    $('div:even').animate({
        width:400
    },2000)
    
    console.log($('div:animated')); // 正在运动的元素们
    console.log($('div:not(:animated)')); // 没有在运动的元素们

例子：

html

	<input type="button" value="test" id="test">
    <input type="button" value="stop" id="stop">

js

	$('#test').click(function(){
        console.log($('#box').has())
        $('#box')
            // .delay(1000)
            .animate({
                left:600
            },1000)
            .animate({
                width:400
            })
            .animate({
                opacity:0
            })
    })
    $('#stop').click(function(){
        // sotp()  停止所有在指定元素上**正在运行的**动画。
        // $('#box').stop(); // 不传参数：停止第一个动画，队列中其他动画继续执行
        // $('#box').stop(true); // 清空队列。立即结束**正在运行的**动画。
        // $('#box').stop(true,true); // 让当前正在执行的动画**立即完成**，并且重设show和hide的原始样式，调用回调函数等。
        $('#box').finish(); 
        /*
            停止当前正在运行的动画，删除所有排队的动画，并完成匹配元素所有的动画。
            当.finish()在一个元素上被调用，立即停止当前正在运行的动画和所有排队的动画（如果有的话），并且他们的CSS属性设置为它们的目标值（所有动画的目标值）。所有排队的动画将被删除。
            .finish()方法和.stop(true, true)很相似，.stop(true, true)将清除队列，并且**目前的动画**跳转到其最终值。但是，不同的是，.finish() 会导致**所有排队的动画**的CSS属性跳转到他们的最终值。
        */         
    })



	