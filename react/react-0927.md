#react

##1.下载
react v16.0.0

版本是向下兼容的，老版本的东西在新版本里都可以使用，如果有变化的会告知

1. 下载html页面，里面有三个地址，保存到本地

	1. react.development.js  -----核心代码
	
	2. react-dom.development.js ---- 渲染DOM（如果放在浏览器平台，解析成浏览器认识的，放在安卓平台，就解析成安卓认识的）

2. 为什么核心代码的渲染DOM要拆开？
	因为：react native；在react native里写的还是html代码，但是安卓和苹果不认识

##2.语法和方法

###2-1.生成节点


**React.createElement()方法**

**参数**：要按顺序写

1. 标签名字（类型），可以是自定义标签
2. 给标签添加的属性,（不添加的话写null）
3. 标签的内容（可以是标签）

html

	<div id="box">

	</div>

js

	let span = React.createElement('span',null,'hi')
    let h2 = React.createElement('h2',{className:'yellow'},span)
	console.log(h2); // 是一个对象
   
这样创建的元素是react Element,是一个**虚拟标签**，是一个**对象**

**渲染进页面：ReactDOM.render()方法**

**参数：**

1. 要渲染的标签
2. 渲染到的目标容器

把h2渲染进box里

    ReactDOM.render(
        h2,
        document.getElementById('box')
    )

###2-2.在js中写html结构（jsx）

> 使用createElement如果遇到复杂的结构，需要写很多，更加易读易写的形式就是写成html结构
> 
> 但是写的html结构只是虚拟DOM，本质上内部还是使用createElement把虚拟DOM转成对象（存这个虚拟DOM的信息）
> 
> 在浏览器端，就会根据该对象生成浏览器认识的html标签 渲染到页面里，（浏览器根据对象里的这些信息生成真实的DOM）
> 
> 类比原生的DOM更好理解：
DOM是文档对象模型，把文档转成对象，只需要操作对象即可，所以console.dir(document.documentElement)可以看到它是一个对象（document.documentElement就是html）

	ReactDOM.render(
        <h2>hello,sky</h2>, // 不行，会报错
        document.getElementById()
    )

在js里写xml结构，要用jsx ，xml包含html

**jsx**

jsx = javascript + xml 

是js的语法扩展，在js中直接写xml结构；在react里推荐用jsx来描述用户界面：

**描述用户界面**：在写页面其实是在使用不同的标签来描述界面，比如用a标签描述连接

要在js中直接写xml结构，需要有一个解析器解析
所以有一个库： babel.min.js

		<script type="text/babel"></script>

type默认是javascript`<script type="text/javascript">`

这个库解析时，如果哪个script标签有`type="text/babel"`，那么这个script标签就归babel这个库管理，babel会把这里的语法转成浏览器可执行的js代码

这时就可以直接写xml结构 ,比如：`let h2 = <h2>hi></h2>`   会经过babel转义之后才被浏览器解析（**此时h2仍然是一个虚拟dom**）


> **注意单标签必须写结束标签！！**
> 
> **jsx元素必须有一个顶层标签包裹**

###2-3.插值 {}

怎么动态渲染结构？

react中的插值：**{表达式}**  

{}里可以直接写表达式，不用像vue里用v-bind

---------例子：写一个标题和列表---------

1. 结构可以存在变量里

		<script type="text/babel">
			let title="你好"
			let arr = [1,2,3,4]; 
			// 如果数据比较多，可以写成对象的形式
						
			// 根据数组渲染列表结构：在模板里不能写for循环
	        // 循环时一定要加上**key值**，代表这个标签时唯一的，之后做数据对比时，不会吧原来的标签干掉，而是会复用标签
	        // {}里可以直接写表达式，不用像vue里用v-bind
	        let liHtml = arr.map((item,index) => {
	            return <li key={index}>{item}</li>
	        })
			console.log(liHtml); // [Object, Object, Object, Object]

			ReactDOM.render(
	            <h2>
	                <span>{title}</span>
	                <img src="./img3.png" /> 
	                <ul>
	                    { liHtml }
	                </ul>
	            </h2>,
	            document.getElementById('box')
	        )
		</script>

2. 结构可以存在数组里

        let liHtml2 = [<li key="1">1111</li>,<li key="2">2222</li>]
		console.log(liHtml2); // [Object, Object]
		ReactDOM.render(
            <h2>
                <span>{title}</span>
                <img src="./img3.png" /> 
                <ul>
                    {liHtml2}
                </ul>
            </h2>,
            document.getElementById('box')
        )
3. 也可以直接在花括号里写结构（比如上面的ul里的结构）
	
		<ul>
            {
                arr.map((item,index) => {
                    return <li key={index}>{item}</li>
                })
            }
        </ul>



##3.组件

###3-1.例子：把标题和ul封装成组件

- 1.必须要继承
- 2.最起码要有一个render方法
- 3.要把自定义标签要和组件区分开，所以组件的首字母必须大写`<Hello></Hello>`

		class Hello extends React.Component {
            render(){
                return <h2>123</h2>
                // return 后面就是界面的描述
            }
            ReactDOM.render(
                <hello></hello>,
                Document.getElementById('box')
            )
        }
		此时<hello></hello>被解析为自定义标签
- 4.return 后面就是界面的描述，也可以用小括号包起来
- 5.渲染组件时单标签和双标签均可，有各自的用途
	- **注意jsx元素必须有一个顶层标签包裹**

代码

	<script type="text/babel">
		/*
            父组件Hello，包括子组件List
        */
		class List extends React.Component {
            render(){
                return (
                    <ul>
                        <li>1</li>
                        <li>2</li>
                        <li>3</li>
                        <li>4</li>
                    </ul>
                )
            }
        }
        class Hello extends React.Component {
            render(){
                return (
                    <div>
                        <h2>hello</h2>  
                        <List />  
                    </div>
                )
                // return 后面就是界面的描述
            }           
        }
        ReactDOM.render(
            <div>
	          <Hello></Hello>
	          <Hello></Hello>
	        </div>,       
            document.getElementById('box')
        )
	</script>

###3-2.组件定制数据

* **外面传进来的数据会自动放在当前组件的实例对象的props属性上**
* props是一个对象
* 默认值的设置：在单花括号里用 || 即可
	* `{this.props.title||'hello'}`
* 注意 `title={t1}` 和 `title="t1"`，前者t1被解析为变量，后者t1被解析为字符串

代码

	<script type="text/babel"> 
        let t1 = '音乐';
        let t2 = '电影';
        let list1 = [1,2,3,4];
        let list2 = ['hi','hello'];

        class List extends React.Component {
            render(){
                return (
                    <ul>      
                        {
                            this.props.abc.map((item,index) => {
                                return <li key={index}>{item}</li>
                            })
                        }
                    </ul>
                )
            }
        }
        class Hello extends React.Component {
            render(){
                console.log(this.props); //Object {title: "音乐"}  Object {title: "电影"}
                return (
                    <div>
                        <h2>{this.props.title || 'hello'}</h2>  
                        <List abc={this.props.list} />  
                    </div>
                )
                // return 后面就是界面的描述
            }           
        }
        ReactDOM.render(
            <div>
                <Hello title={t1} list={list1}></Hello>
                <Hello title={t2} list={list2}></Hello>
            </div>,
            document.getElementById('box')
        )
        
    </script>


* 定制多个数据，写成对象的形式


		let obj1 = {
            title:'音乐',
            list: [1,2,3,4]
        }
        let obj2 = {
            title:'电影',
            list: ['hi','hello']
        }
       

        class List extends React.Component {
            render(){
                return (
                    <ul>      
                        {
                            this.props.abc.map((item,index) => {
                                return <li key={index}>{item}</li>
                            })
                        }
                    </ul>
                )
            }
        }
        class Hello extends React.Component {
            render(){
				// this指向当前组件的实例对象
                console.log(this.props); //Object {title: "音乐", list: Array(4)}     Object {title: "电影", list: Array(2)}
                return (
                    <div>
                        <h2>{this.props.title || 'hello'}</h2>  
                        <List abc={this.props.list} />  
                    </div>
                )
                // return 后面就是界面的描述
            }           
        }
        ReactDOM.render(
            <div>
                 <Hello title={obj1.t1} list={obj1.list1}></Hello>
                 <Hello title={obj2.t2} list={obj2.list2}></Hello>
            </div>,
            
            document.getElementById('box')
        )

如果属性多的话，要在组件标签的行间手动写很多，比较臃肿，所以采用**扩展运算符...**


		<Hello {...obj1}></Hello>
        <Hello {...obj2}></Hello>

{...obj1}  会把obj1扩展到this.props这个对象上


###3-3.组件定制样式和结构

类比默认的那些标签`<input/>`，`<input/>`是系统提供的可定制的组件：
type指定为text，则为输入框，指定为button，则为按钮


		<input type="text"/>
		<input type="button"/>

----------例子：封装一个类似input的组件（根据type属性的值设定不同的标签）-----

**react里的虚拟标签的行间样式style的写法:**

	 style={{width:'100px',height:'100px'}}
因为style属性的值本来就是一个对象{}

	 <script type="text/babel">
		class Hello extends React.Component {
            render(){  
                // let html = <div style={{width:'100px',height:'100px',background:'yellow'}}>{this.props.title||'提交'}</div>

                // 或者用一个变量来存
                let cssStyle = {width:'100px',height:'100px',background:'yellow'};
                let html = <div style={cssStyle}>{this.props.title||'提交'}</div>
                if(this.props.type === 'text'){
                    html = <input type="text"/>
                }else if(this.props.type === 'btn'){
                    html = <button>我是按钮</button>
                }
                return html;
            }
        }
        
        ReactDOM.render(
            <div>
                <Hello type="text"></Hello>
				<Hello type="btn"></Hello>
                <Hello title="普通按钮"></Hello>
            </div>,
            document.getElementById('box')
        )
	</script>


###3-4.外面的交互改变组件内的状态

每个组件有自己的状态，外面的交互改变组件的状态时，要重新渲染组件，react有自己的渲染方式，和vue不同

--------例子：点击按钮，改变按钮的颜色-------

分析：

1. 组件内的状态不是立刻变化，是在某个交互时改变，那么怎么知道什么时候改变？
	* **监听**：用事件和事件处理函数来订阅和监听
	* 原生的事件只能用在原生的标签上，在虚拟标签上不能用，要用虚拟事件（驼峰命名法）：比如onClick
2. 怎么改变？
	* 在组件内定义一个状态**state**（是一个对象），要更新组件的状态，必须通过state
	* 不能重新赋值，否则改了状态之后并没有重新渲染，要调用**setState()方法**
3. 发现，每次更新时，都会调用组件内的render()方法，但是并不是整块结构都改变了，只是更新的地方变了。
	* （内部有一个diff算法），哪个地方变了会更新哪个地方，不变的地方不动


代码

	<script type="text/babel">

        class Hello extends React.Component {
            constructor(props){
                // 子类必须在constructor方法中调用super方法，否则新建实例时会报错。
                super(props) // 如果外面传进来属性，一定要传进super
                this.state = { // 定义一个状态，是一个对象， 要更新组件的状态，必须通过state
                    color: 'red'
                }
            }
            changeColor () {
                console.log(this); // 函数里的this默认指向undefined，要在调用时用bind方法使this指向实例
                // 改变颜色
                // this.state.color = 'green'; // 如果重新赋值，这样改了状态，并没有从新渲染
                // 要调用setState()方法
                this.setState({
                    color: 'green',
                    val: '我改变了'
                })
            }
            render(){
                console.log("我渲染了")
                console.log(this); // 这里的this指向Hello实例
                let cssStyle = {width:'200px',height:'200px',background:this.state.color};
                return (
                    <section>
                        <div style={cssStyle} onClick={this.changeColor.bind(this)}>{this.props.title||'默认提交'}</div>
                        <ul>
                            <li>{this.state.val}</li>
                            <li>222</li>
                            <li>333</li>
                        </ul>
                    </section>
                )
            }
        }
        
        ReactDOM.render(
            <div>
                <Hello title="点击我，变颜色"></Hello>
            </div>,
            document.getElementById('box')
        )
                
    </script>