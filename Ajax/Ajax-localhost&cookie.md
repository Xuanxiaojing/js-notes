

##1.localStorage（是一个全局对象）

不用localStorage时，只要刷新，代码重新执行，做不到数据持久化

本地存储：把数据存在本地，**以文本的形式存储**

每个域名都有，不同域名之间不能互相访问

查看：浏览器F12 - Applicaiton - storage - localStorage

localStorage 方法存储的数据没有时间限制。第二天、第二周或下一年之后，数据依然可用。

作用：数据持久化

应用：

1.把网站一些不变的数据，存储在localStorage里

2.可以存储文件数据，css，js，基础库存在localStorage里，不用每次都加载，节省流量

###1-1.对本地存储的操作

> * 添加内容
> 
>     	localStorage.setItem(key,value)
>     如果存数组或对象，要用JSON.stringify()转成字符串
> * 查找内容
> 
>     	localStorage.getItem(key)
>     如果查找到了，就返回对应的value值，如果没有查找到，返回null；
> * 改内容
> * 
>     	给同一个key值赋值
> * 删内容
> 
>     	localStorage.removeItem(key)

>* 存的是文本，如果存数组，对象，转成字符串

###购物车的简单实现

需求：同一个购物车的页面，如果在多个窗口打开，在其中一个窗口里添加商品，那么另外的窗口也都显示这些加进去的商品

思路：

用localStorage作为第三方：

在某个窗口点击添加商品时，把所加商品的数据存入localStorage，
其他窗口监听到storage事件后，从localStorage里拿数据，渲染页面结构

分析：

* localStorage里存的是json字符串
* 渲染结构要用**数组**形式，每次点击商品，往数组里push数据
* 所以要把字符串用JSON.stringify()转成数组

具体实现：

* 定义一个空数组dataArr
* 一开页面或者刷新页面，拿localStorage里的数据转成数组赋给dataArr
* 点击商品，往数组里push数据，拿数组渲染页面，新数组转成json字符串存进localStorag约定好的key值对应的属性值
* 其他页面监听到storage事件后，拿localStorage里的数据转成数组给dataArr赋新值，渲染结构

关于初始化：

* 刚一打开页面，或者刷新页面时
	* localStorage里有相关数据：数据转成数组赋给dataArr,拿dataArr渲染页面
	* localStorage里没有相关数据，dataArr依然为初始值空，不渲染


		window.addEventListener('storage',function(){
            console.log('我改变了');
            let str = localStorage.getItem('shop');
            dataArr = JSON.parse(str);
            let html = dataArr.map((item)=>{
                return `<li>${item}</li>`
            });
            list.innerHTML = html.join('');
        })

        let str = localStorage.getItem('shop');//一开始拿到localhost里的数据（是一个json字符串）
        console.log(localStorage.getItem('shop'));
        let dataArr = [];//定义一个空数组，把数据转成数组之后存在这里
        if(str){//如果localhost里有数据的话
            dataArr = JSON.parse(str);//把数据转成数组
            //循环数组渲染结构
            let html = dataArr.map((item)=>{
                return `<li>${item}</li>`
            });
            list.innerHTML = html.join('');
        }
        
        let spans = goods.getElementsByTagName('span');
        
        for(let i=0; i<spans.length; i++){
            //点击span
            spans[i].onclick = function(){
                dataArr.push(this.innerHTML);//往数组里push新数据
                //用新数组渲染结构
                let html = dataArr.map((item)=>{
                    return `<li>${item}</li>`
                });
                list.innerHTML = html.join('');
                //把新数组转json字符串存到localStorage里
                localStorage.setItem('shop',JSON.stringify(dataArr));
            }
        }


**老师的方法**

		window.addEventListener('storage',function (){
			console.log('我改变了数据');
			// 触发这个事件的时候，拿到另外窗口添加到localStorage中的数据
			
			let data = localStorage.getItem('shop');  //[]
			// 把data转成数组 JSON.parse()
			let dataArr = JSON.parse(data);

			console.log(arr,'22222');
			// 2. 更新
			arr = dataArr;  // 当触发了storage说名有新数据，所以更新一下
			//否则在这个窗口点击span时arr是其他窗口的arr，内容和当前localStorage里转出来的数组不同
			/*
			如果不写arr = dataArr;会出错，原因如下

				窗口1和窗口2的arr相互独立了，arr没有从localStorage里拿数据，点击商品时是按arr渲染的结构，所以效果不对

				窗口1点鼠标，arr=[“鼠标”]；窗口2里arr是[]
				窗口2里点键盘，arr是["键盘"]；窗口1获取到arr = ['鼠标']
				窗口1再点mac电脑，arr是["鼠标", "mac电脑"]；窗口2里获取的arr是["键盘"] 
				窗口2点mac电脑，arr是["键盘", "mac电脑"] ；窗口1 的arr是["鼠标","mac电脑"]
			*/
			let html = dataArr.map((item) => {
				return `<li>${item}</li>`
			})

			list.innerHTML = html.join("");
		});

		let spans = box.getElementsByTagName("span");
		let list = document.getElementById('list');

		// 一进到页面中，先从localStorage中拿到数据，渲染在页面中

		let data = localStorage.getItem('shop');
		let dataArr = [];

		// 1. 先拿值

		// 如果数据存在是一个json的字符串
		if(data){
			// 把data转成数组 JSON.parse()
			dataArr = JSON.parse(data);

			let html = dataArr.map((item) => {
				return `<li>${item}</li>`
			})

			list.innerHTML = html.join("");
		}


		// arr初始值应该是从localStorage拿到的数组
		let arr = dataArr;  // 【键盘】

		for( var i = 0; i < spans.length; i++ ){
			spans[i].onclick = function (){

				arr.push(this.innerHTML)
				console.log(arr,'11111')
				let newLi = `<li>${this.innerHTML}</li>`;
				list.innerHTML += newLi;

				// 放到localStorage中,每次存的时候不能是单独的一个值，而是一个数组
				//[1,2,3,4] = '1,2,3,4'
				// 2. 存值
				localStorage.setItem('shop',JSON.stringify(arr))
			};
		}

####storage事件

> * 当同源页面的某个页面修改了localStorage,其余的同源页面只要注册了storage事件，就会触发，当前操作localStorage数据的页面不触发这个事件
> 
> * 适用条件：
> 
>   * 同一浏览器打开了两个同源页面
> 
>   * 其中一个网页修改了localStorage
>   * 另一网页监听了storage事件（绑定了监听器，也就是事件处理函数）





##2cookie

> http无状态的,使用cookie来记录登录状态，这个时候你在一个页面登录了，在同源下其他的页面也登陆了
> 
> responseHeader(返回的响应头)里有set-cookie
> 一旦登录成功，后端会设置cookie
> 
> cookie也是和域名有关系，必须是同源才拿到cookie
> 
> 大小有限制 在kb之前

> 浏览器F12 - Application - Cookies  可以看到cookie



###2-1.操作cookie


####2-1-1.存cookie

		setCookie()


####2-1-2.删除cookie

		removeCookie()


####2-1-3.改cookie

		setCookie()


####2-1-4.查找cookie

		getCookie()


###2-2。cookie的生命周期

只有会话时间，浏览器会话结束，就删除

浏览器会话结束指的是关闭浏览器，不是关闭某个窗口

####2-3。设置cookie的过期时间
expires

> cookie和过期时间之间用分号和空格隔开
> 
> 转为UTC 世界标准时间

当前的时间向后推1分钟后过期

        let d = new Date();
        d.setMinutes(d.getMinutes()+1);

        document.cookie = 'test=123; expires=' + d.toUTCString(); 


###2-4封装cookie方法
> 
> setCookie()
> 
> getCookie()
> 
> removeCookie()

这些方法原生js里没有现成的，需要自己封装

####2-4-1.setCookie()
存cookie

参数：传三个参数，前两个是字符串，最后一个是数字；第一个是cookie的key值，第二个是value值，第三个是过期时间（以天为单位）

返回值：无

        function setCookie(key,value,n){ 
            
            //调用函数传实参的时候，可能传n，也可能不需要设置过期时间就不传n，所以要判断一下
            if(n){//如果传进来n了，说明要设置过期时间
                let d = new Date();
                d.setDate(d.getDate()+n);
                document.cookie = key + '=' + value + '; expires=' + d.toUTCString();
            }else{//如果没有传进来n，说明不需要设置过期时间
                document.cookie = key + '=' + value;
            }
        }
        setCookie('miaov','ketang',1);


####2-4-2.getCookie()
* 作用：获取cookie

* 使用时：传进来一个cookie的key值，函数返回出一个对应的value值，

* 参数：cookie的key值

* 返回值：对应的value值

* 思路：比如传进来test，要从document.cookie这个字符串里截取到123

* 步骤：
    * 1.用split方法 以分号加空格为界限 把字符串放到数组里  得到 [test=123,user=2,miaov=ketang];
    
    * 2.循环数组，对每项用split方法 以等号=为单位 转成数组  得到[test,123],[user,2],[miaov.ketang];

####2-4-3.removeCookie()

* 作用：删除cookie

* 使用时，传进来一个key值，删除该key值对应的cookie
* 思路：把过期时间设置为-1，这样一打开页面，该cookie就会消失

* 参数：cookie的key值

* 返回值：无


		function removeCookie(key){
            setCookie(key,null,-1);
        }
        removeCookie('miaov');