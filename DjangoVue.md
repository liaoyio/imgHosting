



# ES6模块化和异步编程

为什么使用模块化？

> 规范代码，降低沟通的成本，方便各个模块之间的相互调用，利人利己。



回顾：`Node.js` 中如何实现模块化 :

- 遵循 CommonJS 的模块化规范
- 导入其它模块使用 `require()` 方法
- 模块对外共享成员使用 `module.exports` 对象



ES6 模块化规范中定义： 

- 每个 js 文件都是一个独立的模块 
- 导入其它模块成员使用 `import` 关键字 
-  向外共享模块成员使用 `export` 关键字



在 node.js 中体验 ES6 模块化

```
node.js > v14.15.1 
```

在文件夹下初始化包配置文件：

````
npm init -y
````

在 `package.json` 里添加 `"type": "module"` 节点，设置为 ES6 模块化规范即可体验。



## ES6 模块化的基本语法

1. 默认导出与默认导入 

   `export default` 默认导出的成员：

   默认导入：

   ```js
   import ml from./01_m1.js'
   ```

   注意： 每个模块中，只允许使用`唯一的一次` export default，否则会报错。

2. 按需导出与按需导入 

   `export` 按需导出的成员

   按需导入： `import { s1 } from` '模块标识符'

   

   注意：

   - 按需导入的成员名称必须和按需导出的名称保持一致

   - 模块中可以使用多次按需导入。

3. 直接导入并执行模块中的代码

   ```js
   //当前文件模块为05_m3.js
   
   //在当前模块中执行一个for循环操作
   for（leti=0；i<3；i++）{
   console.1og（i）
   }
   
   //直接导入并执行模块代码，不需要得到模块向外共享的成员
   import./05_m3.js
   ```



## Promise

### 回调地狱问题：

> `多层回调函数的相互嵌套`，就形成了回调地狱。

![QQ截图20211104132703](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/DjangoQQ%E6%88%AA%E5%9B%BE20211104132703.png)

### Promise 解决回调地狱

#### Promise 基本概念 ：

- Promise 是一个构造函数
- 每new 出一个Promise 实例对象，都代表一个`异步操作`
- `Promise.prototype`(原型) 上包含一个 `.then()` 方法，通过`原型链`访问。

#### .then () 方法：

- p.then( 成功的回调函数，失败的回调函数）
  - p.then (`result` => { }, `error` => { })
- 调用 .then() 方法时，成功的回调函数是必选的、失败的回调函数是可选的。



##### . then() 方法的特性：

> 如果上一个 .then() 方法中返回了一个新的 Promise 实例对象，则可以通过下一个 .then() 继续进行处理。通 过 .then() 方法的`链式调用`，就解决了回调地狱的问题。





【实例】有文件1、2、3.txt, 里面内容依次是111、222、333 ；依次按顺序读取文件，并输出：



使用 then-fs 提供的 readFile() 方法，可以异步地读取文件的内容，它的返回值是 Promise 的实例对象。

```
npm install then-fs
```

```js
import thenFs from 'then-fs'

//1.返回值是Promise的实例对象
thenFs.readFile('./files/1.txt','utf8')
	//2.通过.then为第一个Promise实例指定成功之后的回调函数
	.then((r1)=>{
    	console.log(r1)
    	//3.在第一个.then中返回一个新的Promise实例对象
		return thenFs.readFile('./files/2.txt','utf8')
})
// 4.继续调用.then,为上一个then的返回值(新的Promise实例)指定成功之后的回调函数
.then((r2) =>{
    console.log(r2)
    //5.在第二个.then中再返回一个新的Promise实例对象
	return thenFs.readFile('./files/3.txt','utf8')
})
 // 6.继续调用。then,为上一个.then的返回值(新的Promise实例)指定成功之后的回调函数     
.then((r3)=>{
         console.log(r3)                         
})
```



##### 通过 .catch 捕获错误

```js
import thenFs from 'then-fs'

thenFs.readFile('./files/1111.txt','utf8')   //文件不存在,后面三个then都不执行
	.then((r1)=>{
    	console.log(r1)
		return thenFs.readFile('./files/2.txt','utf8')
})
.then((r2) =>{
    console.log(r2)
	return thenFs.readFile('./files/3.txt','utf8')
})    
.then((r3)=>{
         console.log(r3)                         
})
.catch(err =>{
	console.log(err.message)  // 捕获错误信息并数出
})

```

如果不希望前面的错误导致后续的 .then 无法正常执行，则`可以将 .catch 的调用提前`：

```js
import thenFs from 'then-fs'

thenFs.readFile('./files/1111.txt','utf8')   
	.catch(err =>{  // 捕获并输出错误
	console.log(err.message)  // 因为错误被及时处理,不影响后续 then 执行
})
    .then((r1)=>{
    	console.log(r1)   // undefined
		return thenFs.readFile('./files/2.txt','utf8')
})
.then((r2) => {
  console.log(r2)  // 222
	return thenFs.readFile('./files/3.txt','utf8')
})    
.then((r3)=>{
         console.log(r3)     // 333                      
})
```



#### .all() 和 .race()

Promise.all() 方法:

- 等所有的异步操作全部结束后才会执行下一步的 .then  操作（`等待机制`）

  ```js
  import thenFs from 'then-fs'
  
  //1.定义一个数组，存放3个读文件的异步操作
  const promiseArr = [
    thenFs.readFile('./files/1.txt', 'utf8'),
    thenFs.readFile('./files/2.txt', 'utf8'),
    thenFs.readFile('./files/3.txt', 'utf8'),
  ]
  //2.将Promise的数组，作为Promise.all（）的参数
  Promise.all(promiseArr)
    .then(([r1, r2, r3]) => {
      //2.1所有文件读取成功（等待机制）
      console.log(r1, r2, r3)
    })
    .catch(err => {
      //2.2捕获Promise异步操作中的错误
      console.log(err.message)
    })
  ```

  

Promise.race() 方法：

- 只要任何一个异步操作完成，就立即执行下一步的 .then 操作（`赛跑机制`）

  ```js
  import thenFs from 'then-fs'
  
  const promiseArr = [
    thenFs.readFile('./files/1.txt', 'utf8'),
    thenFs.readFile('./files/2.txt', 'utf8'),
    thenFs.readFile('./files/3.txt', 'utf8'),
  ]
  
  Promise.race(promiseArr)
    .then(([r1, r2, r3]) => {
      // 只要任何一个异步操作完成，就立即执行成功的回调函数（赛跑机制）
      // 只输出一文件的数据  
      console.log(r1, r2, r3)
    })
    .catch(err => {
      console.log(err.message)
    })
  ```

### async/await

> async/await 简化 Promise 异步操作, 直接返回值。

```js
import thenFs from 'then-fs'

//按照顺序读取文件1,2,3的内容

async function getAllFile(){
const r1=await thenFs.readFile('./files/1.txt','utf8')
console.log(r1)
const r2=await thenFs.readFile('./files/2.txt","utf8')
console.log(r2)
const r3=await thenFs.readFile('./files/3.txt','utf8')
console.log(r3)
}

getA11File()
```

#### async / await 的使用注意事项:

- 如果在 function 中使用了 await，则 function `必须 `被 async 修饰 !
- 在 async 方法中，第一个 await 之前的代码会同步执行，await 之后的代码会异步执行:

```js

console.log('A');
async function getAllFile(){
  console.log('B');
const r1 =await thenFs.readFile('./files/1.txt','utf8')
console.log(r1)

const r2 =await thenFs.readFile('./files/2.txt','utf8')
console.log(r2)

const r3 =await thenFs.readFile('./files/3.txt','utf8')
console.log(r3)
console.log('D');
}

getA11File()
console.log('C');
```

输出结果：

```
A
B
C
111 222 333
D
```

--------



## EventLoop 事件循环

JavaScript 是一门**`单线程 `**(同一时间只能做一件事情) 执行语言。

### 同步任务和异步任务

> 为了防止`耗时任务`导致`程序假死`的问题，JavaScript 把待执行的任务分为了同步和异步任务。

同步任务：

- 只有前一个任务执行完毕，才能执行后一个任务
- 在主线程上排队执行

异步任务：

- 异步任务执行完成后，会`通知 JavaScript 主线程`执行异步任务的`回调函数`。
- 由 `JavaScript 委托给宿主环境`进行执行

![](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/DjangoQQ%E6%88%AA%E5%9B%BE20211104150808.png)

### 图解 EventLoop

![Snipaste_2021-11-04_15-32-16](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/DjangoSnipaste_2021-11-04_15-32-16.png)

![](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/Djangoe3313b4466b234f4f0_b.gif)

## 宏任务和微任务

### 什么是宏任务和微任务？

![QQ截图20211104154619](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/DjangoQQ%E6%88%AA%E5%9B%BE20211104154619.png)



1. 宏任务（macrotask） 
   - 异步 Ajax 请求  
   - setTimeout
   - setInterval
   - 文件操作  
2. 微任务（microtask） 
   - Promise.then、.catch 和 .finally 
   -  process.nextTick 

### 执行顺序

> 每一个宏任务执行完之后，都会检查是否存在待执行的微任务， 如果有，则执行完所有微任务之后，再继续执行下一个宏任务。

![](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/DjangoQQ%E6%88%AA%E5%9B%BE20211104162339.png)

经典案例：

1.

![](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/DjangoQQ%E6%88%AA%E5%9B%BE20211104163544.png)

2.

![](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/DjangoQQ%E6%88%AA%E5%9B%BE20211104163530.png)



# Vue 2

## Vue 介绍

### 什么是 vue ？

1. 构建用户界面
   + 用 vue 往 html 页面中填充数据，非常的方便
2. 框架
   + 框架是一套现成的解决方案，程序员只能遵守框架的规范，去编写自己的业务功能！
   + 要学习 vue，就是在学习 vue 框架中规定的用法！
   + vue 的指令、组件（是对 UI 结构的复用）、路由、Vuex、vue 组件库
   + 只有把上面老师罗列的内容掌握以后，才有开发 vue 项目的能力！



### vue 的两个特性

1. 数据驱动视图：

   + 数据的变化**会驱动视图**自动更新
   + 好处：程序员只管把数据维护好，那么页面结构会被 vue 自动渲染出来！

2. 双向数据绑定：

   > 在网页中，form 表单负责**采集数据**，Ajax 负责**提交数据**。

   + js 数据的变化，会被自动渲染到页面上
   + 页面上表单采集的数据发生变化的时候，会被 vue 自动获取到，并更新到 js 数据中

> 注意：数据驱动视图和双向数据绑定的底层原理是 MVVM（Mode 数据源、View 视图、ViewModel 就是 vue 的实例）



### MVVM 

> MVVM 是 vue 实现数据驱动视图和双向数据绑定的核心原理。

![QQ截图20211029215318](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/DjangoQQ%E6%88%AA%E5%9B%BE20211029215318.png)

###  MVVM 的工作原理

`ViewModel `作为 MVVM 的核心，是它把当前页面的`数据源`（Model）和`页面的结构`（View）连接在了一起。

![QQ截图2021102921513ff8](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/DjangoQQ%E6%88%AA%E5%9B%BE2021102921513ff8.png)



当<font>数据源发生变化时</font>，会被 ViewModel 监听到，VM 会根据最新的数据源自动更新页面的结构。

当<font>单元素的值发生变化</font>表时，也会被 VM 监听到，VM 会把变化过后最新的值自动同步到 Model 数据源中

### 起步

第一个Vue程序：

1.  导入开发版本的 Vue.js
2. 创建 Vue实例对象，设置`el`属性和`data`属性
3. 使用简洁的模板语法`把数据渲染到页面`上

```html
<body>
  <div id="app">
    {{message}}
  </div>
</body>
```

```html
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
  <script>
    // 创建 Vue 实例对象
    var app = new Vue({
      // 指定当前 app 实例要控制页面的哪个区域,接收的值是一个选择器
      // el 属性是固定的写法
      el:'#app',
      // 指定 Model 数据源，data对象就是要渲染到页面上的数据
      data:{
        message:'Hello Vue!'
      }
    })
  </script>
```



基本代码与 MVVM 的对应关系

<img src="https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/DjangoSnipaste_2021-10-29_22-00-53.png" style="zoom: 90%;" alt="shadow-f" />





## 插件库

[day.js](https://dayjs.fenxianglu.cn/)    **快速日期格式化**



解决格式化Vue文件时 逗号、分号问题 ：

![Snipaste_2021-11-01_21-54-30](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/DjangoSnipaste_2021-11-01_21-54-30.png)

在根目录中，添加一个 `.prettierrc.json`  配置文件，写入：

```json
{
	"singleQuote": true,  
    "semi": false,
    "trailingComma": "none"
}
```

一劳永逸方法：

1. 在电脑上的用户目录下新建一个 `.prettierrc.json` 文件，同理上面的参数。

2. 在 **Vs code **的 `setting.json` 中添加配置路径：

   ```js
    
   	// 设置 配置文件,解决格式化vue文件时逗号、分号爆红问题
   	// 这里注意配置路径，记得使自己计算机上的用户名
     "prettier.configPath": "C:/Users/mi/.prettierrc.json",
         
   ```



解决格式化时函数括号前的空格问题 ：

- Prettier 格式化插件无法处理函数括号前添加空格问题

解决办法一、在当前项目里的 `.eslintrc.js` 里 rules 规则里添加忽略加空格爆红提示 ：

```js
'space-before-function-paren':['error','never']
```

解决办法二： 使用 `Prettier now` 插件可以解决此问题。

## vue 指令

### 1. 内容渲染指令



1. `v-text` 设置标签的内容, 默认写法会 <font>覆盖元素内部原有的内容</font> , <font style="background-color:#8bc34a">内部支持写表达式</font>。
2. `{{ }}` 插值表达式：在实际开发中用的最多，只是内容的占位符，不会覆盖原有的内容！
3. `v-html` 指令的作用：可以把带有标签的字符串，渲染成真正的 HTML 内容！



> 注意：插值表达式只能用在元素的**内容节点**中，不能用在元素的**属性节点**中！





### 2. v-bind 属性绑定属性

+ 在 vue 中，可以使用 `v-bind:` 指令，为元素的属性动态绑定值；
+ 简写是英文的 `:`



1. 在使用 v-bind 属性绑定期间，如果绑定内容需要进行动态拼接，则字符串的外面应该包裹单引号，例如：

   

   ```vue
   <div :title="'box' + index">这是一个 div</div>
   ```

   

2. 使用 v-bind 在元素绑定时希望内传入 Number数值时，避免被解析成字符串

   

   ```vue
   <select    v-model='value'>
   	<option  ：value='数字'>...</option>
   </select>
   ```

   

### 3. v-on 事件绑定属性

1. `v-on:` 简写是 `@`

2. 语法格式为：

   

   ```vue
   <p> count的值是：{{ count }}</p>
   
   <button @click="add"></button>
   
   methods: {
      add() {
   			// 如果在方法中要修改 data 中的数据，可以通过 this 访问到
   			this.count += 1
      }
   }
   ```

   

#### $event

在事件绑定时，会有一个原生DOM 的事件对象 e，如果事件对象传入参数，默认的事件对象会被覆盖。

> `$event ` 的应用场景：如果默认的事件对象 e 被覆盖了，则可以手动传递一个  $event。

例如：

```vue
// 点击按钮让count值 递增，绑定点击事件并传参

<button @click="add(3, $event)"></button>

methods: {
   add(n, e) {
			// 如果在方法中要修改 data 中的数据，可以通过 this 访问到
			this.count += 1
   }
}
```

#### 事件修饰符：

> 事件绑定期间非常好玩的一个东西，对事件的触发进行控制。

   + `.prevent`

     ```xml
     <a @click.prevent="xxx">链接</a>
     ```

   + `.stop`

     ```xml
     <button @click.stop="xxx">按钮</button>
     ```




| 事件修饰符 | 说明                                                 |
| ---------- | ---------------------------------------------------- |
| `.prevent` | `阻止默认行为` (如：阻止a链接的跳转，阻止表单提交等) |
| `.stop`    | `阻止冒泡事件`                                       |
| .capture   | 以捕获模式触发当前的事件处理函数                     |
| .once      | 绑定的事件只触发一次                                 |
| .self      | 只有在event.target 是当前元素自身时触发事件处理函数  |



#### 按键修饰符

| 按键        | 键码值 | 使用                             |
| ----------- | ------ | -------------------------------- |
| Enter       | 13     | .enter                           |
| Tab         | 9      | .tab                             |
| Delete      | 46     | .delete (捕获“删除”和“退格”按键) |
| Esc         | 27     | .esc                             |
| BackSpace   | 8      | .space                           |
| Up Arrow    | 38     | .up                              |
| Left Arrow  | 37     | .left                            |
| Right Arrow | 39     | .right                           |
| Dw Arrow    | 40     | .down                            |



1.自定义其他的按键别名：

```js

Vue.config.keyCodes.f6 = 118

<input @keyup.f6="xxx" />  // 只有单击f6键才会触发xxx的回调
   
```

2.多个按键一并触发该事件

```js
@keyup.ctrl.enter="XXX"   //  按下ctrl和enter才触发事件执行
```



【案例应用】：表单输入后按回车键添,按esc则清空表单

```vue
<input type="text" v-model="newbrand" 
       @keyup.enter="add" 
       @keyup.esc="newbrand=''" />
```







### 4. v-model 表单绑定

> 在不操作 DOM 的前提下，实现表单元素和数据的双向绑定。



v-model 指令的修饰符

| 修饰符  | 作用                                           | 示例                             |
| ------- | ---------------------------------------------- | -------------------------------- |
| .number | 自动将用户输入值转为 Number                    | <input v-model.`number`="age" /> |
| .trim   | 去除首尾空白字符                               | <input v-model.`trim`="msg" />   |
| .lazy   | 表单输入后失去焦点时更新页面数据，而非实时更新 | <input v-model.`lazy`="msg" />   |



1. input 输入框

   - type="text"

   + type="radio"
   + type="checkbox"
   + type="xxxx"

2. textarea

3. select



### 5. 条件渲染指令

> 条件渲染指令用来辅助开发者按需控制 DOM 的显示与隐藏。



#### v-if 和 v-show

1. `v-show` 的原理是：动态为元素添加或移除<font style="background-color:#8bc34a">style= " display: none; "</font> 样式，从而控制元素的显示与隐藏。
   
   + 如果要频繁的切换元素的显示状态，用 v-show 性能会更好。
   
2. `v-if` 的原理是：<font>每次动态创建或移除 DOM 元素</font>，实现元素的显示和隐藏。
   
   + 如果在运行时条件很少改变，则使用 v-if 较好。





v-if 指令在使用的时候，有两种方式：

1. 直接给定一个布尔值 true 或 false

   ```xml
   <p v-if="true">被 v-if 控制的元素</p>
   ```

2. 给 v-if 提供一个判断条件，根据判断的结果是 true 或 false，来控制元素的显示和隐藏

   ```xml
   <p v-if="type === 'A'">良好</p>
   ```




#### v-else-if

```vue
<div v-if="type === 'A'">优秀</div> 
<div v-else-if="type === 'B'">良好</div> 
<div v-else-if="type === 'C'">及格</div> 
<div v-else>不及格</div>
```



### 6. 列表渲染指令

> 基于一个数组来循环渲染一个列表结构。

#### v-for

`v-for` 指令需要使用 `item in items` 形式的特殊语法

-  `items` 是源数据 (待循环) 的数组，而
-  `item`   被循环的每一项。



【示例】：动态渲染表单数据：

```html
 <link rel="stylesheet" href="./lib/bootstrap.css">

  <!-- 希望 Vue 能够控制下面的这个 div，帮我们把数据填充到 div 内部 -->
  <div id="app">
    <table class="table table-bordered table-hover table-striped">
      <thead>
        <th>索引</th>
        <th>Id</th>
        <th>姓名</th>
      </thead>
      <tbody>
        <!-- 官方建议：只要用到了 v-for 指令，那么一定要绑定一个 :key 属性 -->
        <!-- 而且，尽量把 id 作为 key 的值 -->
        <!-- 官方对 key 的值类型，是有要求的：字符串或数字类型 -->
        <!-- key 的值是千万不能重复的，否则会终端报错：Duplicate keys detected -->
        <tr v-for="(item, index) in list" :key="item.id">
          <td>{{ index }}</td>
          <td>{{ item.id }}</td>
          <td>{{ item.name }}</td>
        </tr>
      </tbody>
    </table>
  </div>
```





#### 使用 key 维护列表的状态

> 官方推荐, 使用 v-for 指令绑定一个` :key  ` 属性，key的作用是为了高效的更新虚拟DOM。

- key 的值类型，是有要求的：字符串或数字类型 

- 把数据项 id 属性的值作为 key 的值（因为 id 属性的值具有唯一性）

- <span alt="wavy">改变 data 顺序，index 会重新排序，所以 index 的值不具有唯一性。</span>



## 计算属性 computed

> `实时监听` data 中数据的变化，并 `return 一个计算后的新值`， 供组件渲染 DOM 时使用。
>
> > 可以被`模板结构` (插值、v-bind ) 或 `methods ` 方法使用。

- 但是在某些情况下，我们可能需要对数据进行一些转化后在显示，或者需要将多个数据结合起来进行显示，这时候我们可以使用计算属性。

实例 1：

```html
<div id="app">
  <h2>{{getFullName()}}</h2>
  <h2>{{fullName}}</h2>
</div>
```



```js
  const vm = new Vue({
    el: '#app',
    data: {
      firstName: 'lin',
      lastName: 'willen'
    },
    computed: {
      fullName () {
        return this.firstName + ' ' + this.lastName;
      }
    },
     // 使用 methods: 每次都会调用方法
    methods: {
      getFullName () {
        return this.firstName + ' ' + this.lastName;
      }
    }
  })
```



特点：

1. 定义的时候，要被定义为 **“方法”**。
2. 在使用计算属性的时候，当普通的属性使用即可
3. 实现了代码的复用，只要计算属性中依赖的数据源变化了，则计算属性会自动重新求值。



实例2：

```html

<div id="app">
  <h2>总价格：{{totalPrice}}</h2>
</div>
```



```js
const vm = new Vue({
    el: '#app',
    data: {
      books:[
        {id: 1001, name: 'Unix编程艺术',price: 119},
        {id: 1002, name: '代码大全',price: 105},
        {id: 1003, name: '深入理解计算机原理',price: 99},
        {id: 1004, name: '现代操作系统',price: 109}
      ]
    },
    computed: {
      totalPrice () {
        
        let totalPrice = 0;
        
        for (let i in this.books) {
          totalPrice += this.books[i].price;
        }
          
        // 也可以使用 for of 
        for (let book of this.books) {
          totalPrice += book.price;
        }
        return totalPrice;
      }
    }
  })
```



#### 计算属性 vs 方法 

`methods`和`computed`看起来都可以实现我们的功能，那么为什么还要多一个计算属性 ?

- methods:  每次使用都会调用方法
- computed: `计算属性会缓存计算的结果`, 不变的情况下只调用一次, 除非原属性发生改变，才会重新调用.



#### 计算属性 vs 侦听器

侧重的应用场景不同`侧重的应用场景不同`：

- 计算属性侧重于监听`多个值`的变化，最终计算并`返回一个新值`。
- 侦听器侧重于监听单个数据的变化，最终执行`特定的业务处理`，`不需要有任何返回值`。



## vue组件

什么是组件化开发 ？

> 根据封装的思想，把页面上可重用的 UI 结构封装为组件，方便项目的开发和维护。



vue 中的组件化开发

- vue 是一个支持组件化开发的前端框架。
- **vue** 中组件的后缀名是` .vue`

### 组件的构成

每个 .vue 组件都由 3 部分构成，分别是：

1. `template` ：组件的模板结构，且每个组件中必须包含template模板结构。
2. `script` ： 组件的 JavaScript 行为
3. `style` ：组件的样式

```js

<template>
<!-- 当前组件的 DOM 结构，需要定义到 template 标签内部 -->
</template>

<script>
 // 组件相关的 data数据、methods方法等，
 // 都要定义到 export default 所导出的对象中
export default {}
</script>

// 标签上添加 lang="less" 属性，即可使用 less 语法编写组件的样式
// scoped 防止样式冲突
<style lang='less' scoped>
</style>

```

#### script 节点：

（1）script 中的 `name` 节点

- 用来定义组件的名称，调试的时候`可以清晰的区分每个组件`。

（2）script 中的 `data` 节点

- 组件渲染期间需要用到的数据， data `必须是函数`, 不能直接指向对象数据。

（3）script 中的 `methods` 节点

- 组件中的事件处理函数（`方法`），必须定义到 `methods` 节点中



### 注册私有组件

>  通过 components 注册的是私有子组件，被注册的组件只能用在当前组件中。

1.使用 import 语法`导入需要的组件`

```js
import Left from '@/components/Left.vue'
```



2.在 script 标签中使用 `components` 节点注册组件

```vue
<script>
export default {
  comments:{
    Left
  }
}
</script>
```



3.以`标签的形式`使用刚才注册的组件

```vue
<template>
   <Left></Left>
</template>
```



### 注册全局组件

> 在 vue 项目的 `main.js` 入口文件中，通过 `Vue.component() `方法，可以注册**全局组件**。

注册：

```js
import Vue from 'vue'
import App from './App.vue'

// 导入需要被全局注册的那个组件
import Count from '@/components/Count.vue'

// 参数一： 组件的 '注册名称'，将来以标签形式使用时要求和这个名称一样。
// 参数二： 需要被全局注册的那个组件。
Vue.component('MyCount', Count)

// 消息提示的环境配置，设置为开发环境或者生产环境
Vue.config.productionTip = false

new Vue({
  // render 函数中，渲染的是哪个 .vue 组件，那么这个组件就叫做 “根组件”
  render: h => h(App)
}).$mount('#app')

```

使用：

```js
<template>
    // 这里需要注意 '注册名称'
    <MyCount ></MyCount>
</template>
```

### 组件注册时名称的大小写

在 Vue 定义组件注册名称的方式有两种：

1. 短横线命名法，例如 `my-swiper` 和 `my-search`
   - 使用组件时也必须使用短横线命名
2. 大驼峰命名法，例如 `MySwiper` 和 `MySearch`
   - 既可以按照大驼峰命名法使用，也可以`转化为短横线名称`进行使用。

注意： 在开发中，推荐使用`大驼峰命名法`为组件注册名称，因为它的适用性更强。







## 组件之间的样式冲突问题

> 默认情况下，写在 .vue 组件中的样式会全局生效，因此很容易造成多个组件之间的样式冲突问题。

#### scoped 属性

> 让当前组件的样式对其子组件是不生效。

Vue 中提供了在style 节点添加 `scoped `属性，来防止样式冲突问题：

- 原理是为每个组件分配唯一的自定义属性，在编写组件样式时，通过属性选择器来控制样式的作用域。

![QQ截图20211030133220](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/DjangoQQ%E6%88%AA%E5%9B%BE20211030133220.png)

```js
<template>
    <div class="container" data-v-001>
        <h3 data-v-001 > 轮播图组件件</h3>
	</div>
</template>

<style>
  // 通过中括号'属性选择器'，防止样式冲突问题
  // 因为每个组件分配的自定义属性是'唯一的'
 . container[data-v-001]{
      border: 1px solid red;
    }
</style>
```



为了提高开发效率和开发体验，直接在 style 节点使用 scoped 属性：

```js
<style lang="less" scoped>
</style>
```



#### /deep/ 样式穿透

> 让某些样 式对子组件生效。

使用场景： 当使用第三方组件库的时候，需要修改第三方组件默认样式的时候。

```js
<style lang="less" scoped>
    
/*不加 /deep/ 时，生成的选择器格式为 .title[data-v-052242de]*/
.title{
    color: blue;
}

/*加/deep/ 时，生成的选择器格式为 [data-v-052242de] .title*/
/deep/ .title {
  color: pink;
}
</style>
```



----------



## Class 与 Style 绑定

> 通过 `v-bind `动态操作元素样式。



### 1. 动态绑定 HTML 的 class：

通过三元表达式，动态的为元素绑定 class 的类名：

```vue
<h3 class="thin" :class="isItalic ?'italic':''">MyDeep 组件</h3>
<button @click="isItalic=!lisItalic"> Toggle Italic </button>

```

```js
data(){
	return { isItalic:true }
.thin{
	font-weight:200;
.italic{
	font-style:italic;
}
    
```



### 2. 以数组语法绑定 HTML 的 class

如果元素需要动态绑定多个 class 的类名，此时可以使用数组的语法格式

```vue
  
<h3 class="thin":class="[isItalic? 'italic': '',isDelete? 'delete':'']">MyDeep组件</h3>
<button @click="isItalic= !isItalic"> 字体变细 </button>
<button @click="isDelete= !isDelete"> 添加删除线 e</button>

<script>
export default {
  data () {
    return {
      isItalic: true,
      isDelete: false,
    }
  }
}
</script>
```

### 3. 以对象语法绑定 HTML 的 class：

```vue
<h3 class="thin":class="classObj">MyDeep组件</h3>
<button @click="classObj.isItalic = !classObjisItalic"> 字体变细 </button>
<button @click="classObj.isDelete = !classObj.isDelete"> 添加删除线 e</button>

<script>
export default {
  data () {
    return {
        classObj:{
            isItalic: true,
      		isDelete: false,    
        }
    }
  }
}
</script>
```



### 4. 以对象语法绑定内联的 style

命名可以用`驼峰式`或`短横线分隔` (记得用引号括起来) 来命名：

```vue
<div :style="{color:active, fontSize: fsize +'px','background-color': bgcolor}">
    Hello world!!
</div>

<button @click="fsize += 1">字号+1</button>
<button @click="fsize -= 1">字号-1</button>

```

```js
data () {
    return {
      active: 'red',
      fsize: 30, 
      bgcolor: 'pink'
    }
}
```



## 自定义属性 props

> `props `是组件的自定义属性，允许使用者通过自定义属性，为当前组件指定初始值，极大的提高组件的复用性。

#### 在组件中声明 prpos

my-article 组件的定义如下：

```vue

<template>
<h3>标题：{{title}}</h3>
<h5>作者：{{author}}</h5>
</template>

```
父组件传递给`my-article`组件的数据，必须在`props`节点中声明 :

```js
<script>
    export default {
		props:['title','author'],
</script>
```

#### 无法使用未声明的 props

> 如果父组件给子组件`传递了未声明的 props 属性`，则这些属性会被忽略，无法被子组件使用。

#### 动态绑定 props 的值

> 使用 v-bind 属性绑定的形式，可以为组件动态绑定 props 的值。

```vue
<！--通过V-bind属性绑定，为author动态赋予一个表达式的值
                                    
<my-article :title="info.title" :author="'post by'+info.author"></my-article>

```

#### props 的大小写命名

组件中如果使用“`camelCase (驼峰命名法)`”声明了 props 属性的名称，则有两种方式为其绑定属性的值：

```js
<script>
    export default {
		props:['pubTime'],  // 使用'驼峰命名'法为当前组件声明 pubTime 属性
</script>
```

使用时既可以用`驼峰命名`，亦可以用`短横线分隔命名`的形式为组件绑定属性的值 ：

```vue
<my-article pubTime="2021" ></my-article>
// 等价于
<my-article pub-time="2021" ></my-article>
```



#### props 验证

1. 基础的类型检查  `type`

   ```js
   props:{
       
      //支持的8种基础类型
   
      propA:String,    //字符串类型
      propB:Number,    //数字类型
      propC:Boolean,   //布尔值类型
      propD:Array,     //数组类型
      propE:Object,    //对象类型
      propF:Date,      //日期类型
      propG:Function,  //函数类型
      propH:Symbol     //符号类型
   
     }
   ```

   

2. 多个可能的类型 

3. 必填项校验  `required`

4. 属性默认值  `default`

   ```
   
   props:{
       // 通过数组形式，为当前属性定义多个可能的类型
       type: [String, Number],
       required: true,
       default: 0,
   }
   ```



##### 自定义验证函数

在封装组件时，可以为 prop 属性指定自定义的验证函数，从而对 prop 属性的值进行更加精确的控制：

```js
 props:{
   type:{
       // 通过 validataor函数，对type 属性进行校验，属性值 通过val形参接收
     validataor(val){
       // 必须匹配下列字符串中的一个
       return ['success','warning','danger'].indexOf(val) !== -1
     }
   } 
}
```







#### prpos 传入不同的初始值

在实际开发中我们经常会碰到下面的情况：

- 在不同组件`使用同一个注册的组件`时候希望赋值一个`不同的初始值`。

1.组件的封装者通过 props 允许使用者自定义初始值：

```js
<template>
  <div>
    <h5>Count是全局组件，将被Left 和 Right组件使用</h5>
    <p>count 的值是：{{ init }}</p>
    <button @click="count += 1">+1</button>
  </div>
</template>

<script>
export default {
  // props: ['init'],
  props: {
    // 自定义属性的名字，是封装者自定义的（只要名称合法即可）
    init: {
      // 如果外界使用 Count 组件的时候，没有传递 init 属性，则默认值生效
      default: 0,
      // 指定值类型必须是 Number 数字
      type: Number,
      // 必填项校验(表示必须传入值)
      required: true
    }
  },

  data() {
    return {
        //props是只读的且不可修改
       // 要想修改 props 的值，可以把 props 的值转存到 data中
      count: this.init
    }
  }
}
```

2.组件的使用者通过属性节点传入初始值 

Left 组件、Right 组件：

```js
<template> 
  <div class="left-container">
    <h3>Left 组件</h3>
    <hr />
    <MyCount :init="9"></MyCount>
  </div>
</template>
```

```js
<template>
  <div class="right-container">
    <h3>Right 组件</h3>
    <hr />
        
    <MyCount :init="6"></MyCount>
  </div>
</template>
```



#### props 里传参注意点

- props 可以通过`[] `给数据定义多个可能的数据类型 ;
- props 传入 `Object` 默认值必须是一个 `fn` 

```js
props: {
    commCount: {
      type: String,
      default: ''
    },
    pubdate: {
      // 通过数组形式，为当前属性定义多个可能的类型
      type: [String, Number],
      default: ''
    },
    cover: {
      type: Object,
      // 通过 default 函数，返回 cover 的默认值
      default: function() {
      // 这个 return 的对象就是 cover 属性的默认值
        return { cover: 0 }
      }
    }
  }
```



![QQ截图20211030131333](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/DjangoQQ%E6%88%AA%E5%9B%BE20211030131333.png)

- props 是`只读的`,想修改 props 的值，可以把 props 的值转存到 data 中。
- props 的三个属性值：default、type、required。



---------

## 自定义事件

封装组件时，为了让`组件的使用者`可以`监听到组件内状态的变化`，此时需要用到组件的`自定义事件`。



**vue2 中自定义事件的 3 个使用步骤：**

-  在封装组件时 （子组件）：`触发`自定义事件

-  在使用组件（父组件）时：`监听 `自定义事件

#### 触发自定义事件

```vue
// 子组件

<button @click="onBtnClick"> +1 </button>

<script>
export default {
    data() {
        // 子组件自己的数据，将来希望把 count 值传给父组件
        return { count: 0 }
    },
    methods: {
        onBtnClick() {
            this.count t= 1
            //修改数据时，通过 $emit()触发自定义事件
            // 当点击 '+1' 按钮时 调用this.$emit 触发自定义的 numchange 事件
            this.$emit( 'numchange')}
    }
}
</script>
```



#### 监听自定义事件

```js

// 父组件
<Son @numchange="getNewCount"></Son>

methods : {
	getNewCount(val) {
	console.log('监听到了 count 值的变化', val)
}，

```



#### 自定义事件传参

在调用 this.$emit() 方法触发自定义事件时，可以通过第 2 个参数为自定义事件传参:

```js
methods: {
    onBtnClick() {
        this.count t= 1
        this.$emit( 'numchange' , this.count)} // 触发自定义事件,通过第二个参数传参
}
```



## filter 过滤器

在 vue 3.x 的版本中`剔除了过滤器`相关的功能。

在 vue 3.x 使用计算属性或方法代替被剔除的过滤器功能。



> 过滤器（Filters）常用于文本的格式化。过滤器可以用在两个地方：`双括号插值表达式` 和` v-bind `属性绑定。



```vue
<!-- 在双花括号中通过 | 调用capitalize过滤器，对message值进行格式化  -->
{{ message | capitalize }}

<!-- 在 `v-bind` 中 --> 
<div v-bind:id="rawId | formatId"></div>
```



### 私有过滤器

> 在 filters 节点下定义的过滤器，称为“私有过滤器”，因为它只能在当前 vm 实例所控制的 el 区域内使用。

创建一个私有过滤器，示例代码如下：

```js
const vm = new Vue({
    el: '#app',
    data: {
        message: 'hello world!',
        info: 'title info'
    },
    // 在 filters 节点下定义过滤器
    filters: {
        // 把首字母转换为大写的过滤器
        capitalize (value) {  
            return value.charAt(0).toUpperCase() + value.slice(1)
        }
    }
```



### 全局过滤器

如果希望在多个 vue 实例之间共享过滤器，则可以按照如下的格式定义全局过滤器:

```js
// Vue.filters()方法接收两个参数：
// 第一个参数：过滤器名字   第二个参数：过滤器的处理函数

Vue.filter('capitalize',(str)=>{
    return value.charAt(0).toUpperCase() + value.slice(1)'
} )
```



### 调用多个过滤器：

```js
{{ message | filterA | filterB }}
```

- 先把 message 的值交给 filterA 处理，再把 filterA 处理结果交给 filterB  进行处理，最终把 filterB 的处理结果，作为最终值渲染到页面上。

  

### 过滤器传参

过滤器是 JavaScript 函数，因此可以接收参数：

```js
{{ message | filterA('arg1', arg2) }}

// 第一个参数永远是 '管道符' 前面待处理的值，第二个参数开始才是调用过滤器时传递的参数

Vue.filter('filterA',(msg,arg1,arg2)=>{
	// 过滤器的逻辑代码
})
```

这里，`filterA` 被定义为接收三个参数的过滤器函数。其中 `message` 的值作为第一个参数，普通字符串 `'arg1'` 作为第二个参数，表达式 `arg2` 的值作为第三个参数。



```js
<p>{{message | cap | maxl(5)}}</p>


Vue.filter('cap', (str) => {
    return str.charAt(0).toUpperCase() + str.slice(1) + '----'
})

Vue.filter('maxl', (str, len = 10) => {
    if (str.length <= len) return str
    return str.slice(0, len) + '....'
})
var app = new Vue({
    el: '#app',
    data: {
        message: 'hello Vue 2021年10月30日00:07:45!'
    }
```



### 过滤器的注意点

1. 在过滤器函数中，**一定要有 return 值**
2. 在过滤器的形参中，可以获取到“管道符”前面待处理的那个值
3. 如果全局过滤器和私有过滤器名字一致，此时按照“**就近原则**”，调用的是”私有过滤器“

### label的for属性

使用lable自带的属性进行单选钮的启用和禁用：

```html
<input type="checkbox" :id="'cb' + item.id" v-model="item.status">
<label  :for="'cb' + item.id" v-if="item.status">已启用</label>
<label  :for="'cb' + item.id" v-else>已禁用</label>
```



## watch 侦听器

> watch 侦听器 `监视数据的变化`，从而`针对数据的变化做特定的操作`。

### 侦听器的格式

1. 方法格式的侦听器
   + 无法在刚进入页面的时候，自动触发！！
   + 如果侦听的是一个对象，如果对象中的属性发生了变化，不会触发侦听器！
2. 对象格式的侦听器
   + 可以通过 **immediate** 选项，让侦听器自动触发！
   + 可以通过 **deep** 选项，让侦听器深度监听对象中每个属性的变化！



**使用方法格式创建的侦听器：**

监听 username 值的变化，并使用 axios 发起 Ajax 请求，检测当前输入的用户名是否可用： 

```js
import axios from 'axios'

export default{
    data(){
        return{ username: ''}
    }
},
watch: { 
    // newVal 是'变化后的新值'，oldVal 是'变化之前的旧值'
    async username(newVal,oldVal) {  
        if (newVal === '') return 
        // 使用 axios 发起请求，判断用户名是否可用 
        const { data: res } = await axios.get(`https://www.escook.cn/api/finduser/${newVal}` ) 
        console.log(res) 
    } 
}

```

> 使用方法创建时：组件在初次加载完毕后不会调用 watch 侦听器。

### immediate 选项

> 如果想让 watch 侦听器在浏览器打开时立即被调用，则需要使 用 `immediate` 选项。

**使用对象格式创建的侦听器**：

```js
watch: {
    username: {
        // handler 是固定写法，表示当 username 的值变化时，自动调用 handler 处理函数
        async handler(newVal,oldVal) {
            if (newVal === '') return
            const { data: res } = await axios.get('https://www.escook.cn/api/finduser/' + newVal)
            console.log(res)
        },
            // 表示页面初次渲染好之后，就立即触发当前的 watch 侦听器
            immediate: true
    }
}
```



1. 使用 `handler` 定义侦听器函数
2. `immediate`控制侦听器是否自动触发， 默认值为 `false `不立即触发。



### deep 选项 

> 如果 watch 侦听的是一个对象，如果对象中的属性值发生了变化，则无法被监听到。此时需要使用 deep 开启 **深度监听**。

```html
<input type="text" v-model.trim="username"/>
```

```js
data: {
   info: {username: 'admin'}
},
watch:{
    // 监听info对象的变化
	info:{
      	handler(newVal){
            console.log(newVal.username)
		},
	// 开启深度监听，监听每个属性值的变化，默认值为 false
	deep: true
	}
}
```



### 监听对象单个属性的变化

> 如果只想监听对象中单个属性的变化，则可以按照如下的方式定义 watch 侦听器。

```js
const vm = new Vue({
    el: '#app',
    data: {
        info: {username: 'admin',}
    },
    watch:{
        'info.username':{
            handler(newVal){
                console.log(newVal)
            },
        }
    }
})
```





## 组件的生命周期

- `生命周期`（Life Cycle）是指一个组件从创建 -> 运行 -> 销毁的整个阶段，强调的是一个时间段。
- `生命周期函数`：是由 vue 框架提供的内置函数，会伴随着组件的生命周期，自动按次序执行。

> `生命周期`强调的是时间段，`生命周期函数`强调的是时间点。

### 生命周期函数的分类



生命周期图示:

![lifecycle](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/Djangolifecycle.png)

需要注意的三个周期函数：

![生命周期函数](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/Django%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E5%87%BD%E6%95%B0.png)

>  在实际开发中，`created` 是最常用的生命周期函数 ！

## 组件之间的数据共享

![组件之间的数据共享](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/Django%E7%BB%84%E4%BB%B6%E4%B9%8B%E9%97%B4%E7%9A%84%E6%95%B0%E6%8D%AE%E5%85%B1%E4%BA%AB.png)

### 父→子

> 父组件通过 `v-bind` 属性绑定向子组件共享数据，子组件使用 `props`接收数据。

![](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/DjangoSnipaste_2021-11-06_14-09-51.png)

```js
// 父组件
// 注册子组件
<Son :msg="message" :user="userinfo"></Son>

<script>
    // 导入子组件
    import Son from '@component/Son.vue'

export default {
    data:{
        return{
        message:'hello vue.js'
        userinfo:{
        name: 'lilei',age: 21
    },
    components: {
        Son
    		}
		}
	}
}
</script>
```

```js
// 子组件
<template>
    <div>
    	<h5>Son组件</h5>
		<p>父组件传递过来的 msg值是: {{ msg }</p>
    	<p>父组件传递过来的user值是: {{ user }}</p>
    </div>
</template>

export default {
	props: [ 'msg' , 'user ']
}
```



### 子 → 父 

> 子组件向父组件共享数据使用自定义事件。

```js
// 子组件

export default {
    data() {
        // 子组件自己的数据，将来希望把 count 值传给父组件
        return { count: 0 }
    },
    methods: {
        add() {
            this.count t= 1
            //修改数据时，通过 $emit()触发自定义事件
            this.$emit( 'numchange' , this.count)}
    }
}
```



```js
// 父组件
<h1>App 根组件</h1>
<h3>子组件传过来的数据是 ： {{ countFromSon }}</h3>
<Son @numchange="getNewCount"> </Son>

<script>
import Son from '@component/Son.vue'

export default {
    data() {
        return { countFromSon: 0 }
    },
    methods : {
        getNewCount(val) {
            console.log('numchange 事件被触发了！', val)
            this.countFromSon = val
        }，
        components: {
        Son
    }
}
}
    </script>

```



### 兄弟组件数据共享

> 在 vue2.x 中，兄弟组件之间数据共享的方案是 `EventBus`。

![QQ截图20211030151214](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/DjangoQQ%E6%88%AA%E5%9B%BE20211030151214.png)

EventBus 的使用步骤

1. 创建 `eventBus.js `模块，并向外`共享一个 Vue 的实例对象`  
2. 在数据发送方，调用 `bus.$emit('事件名称', 要发送的数据) `方法触发自定义事件 。
3. 在数据接收方，调用 `bus.$on('事件名称', 事件处理函数) `方法注册一个自定义事件。





## ref 引用 操作DOM

> 不依赖于 jQuery 和调用 DOM API 的情况下，获取 `DOM 元素`或`组件的引用`。



每个 vue 的组件实例上，`都包含一个 $refs 对象`，里面存储着对应的 DOM 元素或组件的引用。默认情况下，组件的 $refs 指向一个空对象。



### 使用 ref 引用 DOM 元素



```js
<！--使用ref属性，为对应的DOM添加引用名称-->
    
<h3 ref="myh3">MyRef 组件</h3>
<button@click="getRef">获取$refs 引用</button>


methods:{
    getRef(){
    	//通过this.$refs.引用的名称可以获取到DOM元素的引用
        console.log(this.$refs.myh3)
    	//操作DOM元素，把文本颜色改为红色
    	this.$refs.myh3.style.color='red'
}，
```



### 使用 ref 引用组件实例

需求： 在根组件控制子组件

```js
<！--使用ref属性，为对应的“组件”添加引用名称-->
    
<my-counter ref="counterRef"> </my-counter>
<button @click="getRef"> 获取$refs 引用 </button>

methods:{
    getRef（）{
    	// 通过this.$refs.引用的名称可以引用组件的实例
        console.log（this.$refs.counterRef）
    	// 引用到组件的实例之后，就可以调用 子组件上的 methods 方法
        this.$refs.counterRef.add（）
}，
```





### 点击文本框自动获得焦点

> 添加 ref 引用，并调用原生 DOM 对象的` .focus()` 方法即可。

###  this.$nextTick(cb) 方法

> `$nextTick(cb) `保证 cb 回调函数可以操作到最新的 DOM 元素（推迟到下一个 DOM 更新周期之后执行）。

- 解决我们在页面没有渲染完成前使用 ref 操作DOM元素报错问题。

  

### 控制文本框和按钮的按需切换

点击按钮展示文本框，文本框输入时隐藏按钮：

```js
<template>
    <input type="text" v-if="inputVisible" ref="ipt">
    <button v-else @click="showInput">展示input输入框</button>
</template>
```

```js
<script>
export default{
	data(){
		return{
		//控制文本框和按钮的按需切换
		inputVisible:false，
}，
methods:{
    showInput(){
        //切换布尔值，显示文本框
        this.inputVisible=true
        
        //获取文本框的DOM引用，并调用.focus（）使其自动获得焦点
        
        // this.$refs.ipt.focus（） 错误，此时页面未渲染完毕，无法获取文本框
        
        //把对input文本框的操作，推迟到下次DOM更新之后。否则页面上根本不存在文本框元素
        this.$nextTick(() =>{
				this.$refs.ipt.focus（）
		}）
    }
}，
    </script>
```



## 动态组件 

> 实现不同组件之间的按需展示。( 动态切换组件显示和隐藏 ) ，类似于 `Vue-Router`。

### 动态组件的基本使用

Vue 提供了一个内置的  `<component>  `组件，专门用来实现动态组件的渲染 :

1. `component` 标签是 vue 内置的，作用：组件的占位符

   

2. 通过 `:is` 属性，动态指定要渲染的组件

   - is 属性的值，表示要渲染的组件的名字。

   - is 属性的值，应该是组件在 components 节点下的注册名称。

     

3. 使用 `keep-alive `保持组件的状态 （避免组件切换时重新渲染）。

   -  keep-alive 会把内部的组件进行缓存，而不是销毁组件。

     

4. 通过 `include` 指定哪些组件需要被缓存。

5. 通过 `exclude `属性指定哪些组件不需要被缓存。

   

```vue
// 点击按钮，动态切换组件的名称

<button @click="comName = 'Left'">展示 Left</button>
<button @click="comName = 'Right'">展示 Right</button>

<div class="box">
    <keep-alive exclude="MyRight">
        <component :is="comName"></component>
    </keep-alive>
</div>

<script>
    import Left from '@/components/Left.vue'
    import Right from '@/components/Right.vue'

    export default {
        data(){
            return {
                // comName 表示要展示的组件的名字
                comName: 'Left'
            }
        },
        components: {
            // 如果在“声明组件”的时候，没有为组件指定 name 名称，则组件的名称默认就是“注册时候的名称”
            // 组件的 “注册名称” 应用场景是：以标签的形式使用渲染到页面。
            Left,
            Right
        }
    }
</script>

```



### 在组件中定义 name 名称：

- 当提供了 name 属性之后，组件的名称就是 name 属性的值。

  

```js
export default {
  name: 'MyRight'
}
```



声明 name 应用场景：结合` <keep-alive> `标签实现组件缓存功能；以及在调试工具中看到组件的 name 名称。





------



## 插槽 slot

> 在签形式使用的`组件中内容节点`中`插入内容`。

- 通过  `slot` 元素 `定义插槽`，从而为用户预留内容占位符。
- 封装组件时，没有预留插槽的内容会被丢弃。



###  后备内容

> 在 `slot` 标签内添加的内容会被作为后备内容。

```vue
// 子组件 my-com
<template>
	<slot>这是后备内容</slot>
</template>
```

```js
// 使用插槽
	<my-com>
       // 如果用户没有提供内容，上面 slot标签 内定义的内容会生效，此时页面会有 "这是后备内容"
       // 如过提供了，下面的 img 将会被渲染到页面上
        <img src="../assets/log.png" alt="">
	</my-com>
```



### 具名插槽

```vue
// MyArticle 组件
<template>
<div class="container">
    <header>
        <slot name="header"></slot>
    </header>
    <main>
        <slot></slot>
    </main>
    <footer>
        <slot name="footer"></slot>
    </footer>
    </div>
</template>
```

- Vue 官方规定，每个 `slot`插槽，都要有一个 name名称，一个不带 `name` 的 `<slot>` 出口会带有隐含的名字“default”。

### 为具名插槽提供内容

如果要把内容填充到指定名称的插槽中，我们可以在一个 `<template>` 元素上使用 `v-slot` 指令，并以 `v-slot` 的参数的形式提供其名称：

```vue
<MyArticle>
    
    <template v-slot:header>
		// 把内容放在 MyArticle组件的 header 标签内
		<h1>如今最好，没有来日方长。</h1>
    </template>
    
	// 上一个代码块中，main 标签未指定具名插槽，所以默认渲染到 main标签中
    <p>现在是2021年11月7日 22点05分 星期天</p>
    <p>今天不仅是周日，也是冬至</p>

    <template #footer>   
		<p>最后，祝大家冬至快乐！</p>
    </template>
    
</MyArticle>
```

- `v-slot: `可以简写 ` #`。例如 v-slot:header 可以被重写为 #header。

- `v-slot` 属性只能放在 组件标签和 `<template>` 元素上 (否则会报错)。



### 作用域插槽

在封装组件的过程中，可以为预留的`<slot>`插槽绑定 props 数据，这种`带有 props 数据的<slot>`  叫做“`作用域插槽`”。示例代码如下：



```vue

<template>
	<div>
        <h1> 这是 Left 组件</h1>
		<！--下面的slot 是一个作用域插槽 -->
 		<slot v-for="item in list" :user="item"></slot> 
    </div>
  </template>
```



接收作用域插槽对外提供的数据,  使用`解构赋值`简化数据的接收过程 :

```vue
<！-- 使用自定义组件 -->
<Left>
  <！--作用域插槽对外提供的数据对象，可以通过“解构赋值”简化接收的过程-->
  <template #default="{user}">
        <tr>
            <td>{{user.id}}</td>
            <td>{{user.name}}</td>
            <td>{{user.state}}</td>
        </tr>
  </template>
</Left>
```





## 自定义指令

> vue 官方提供了 v-text、v-for、v-model、v-if 等常用的指令。除此之外 vue 还允许开发者自定义指令。



### 私有自定义指令 directives: { }

在每个 vue 组件中，可以在 `directives` 节点下声明私有自定义指令。

##### 1. 定义一个私有自定义指令：

```js
directives:{
    color:{
        //为绑定到的HTML元素设置红色的文字
        bind(el){
            //形参中的el是绑定了此指令的、原生的DOM对象
            el.style.color=‘red'
        }
```

##### 2. 使用自定义指令：

- 使用自定义指令时，需要加上`v-指令前缀`

```html
<！--声明自定义指令时，指令的名字是color 使用时就是 v-color -->
    
<h1 v-color>App组件</h1>
```



##### 3. 为自定义指令动态绑定参数值

在 `template` 结构中`使用自定义指令`时，可以通过等号 `=` 的方式，为当前指令动态绑定参数值：

```js
<template>
    
<！--在使用指令时，动态为当前指令绑定参数值 color-->
<h1 v-color="color">App组件</h1>

</template>

data(){
	return{
	color:'red’
}
```

##### 4. 通过 `binding` 获取指令的参数值：

在上面的实例中我们为自定义指令绑定了一个动态的参数值，如何拿到这个值呢 ？

> 在声明自定义指令时，可以通过形参中的第二个参数，来接收指令的参数值

```js

<h1 v-color="color">App组件</h1>

// 此时传入的是字符串，不会在data 数据中查找
<p v-color="'blue'" ></p>

directives:{
	color:{
		bind(el，binding){
            //通过binding对象的.value属性，获取动态的参数值
			el.style.color=binding.value
}
```



##### 5.  使用 update 函数更新 DOM

>  `update` 函数会在每次 DOM 更新时被调用。



注意：在 vue3 的项目中使用自定义指令时，  bind 必须改为 `mounted`  、  update 改为 `updated`   。



**bind** 函数只调用 1 次：当指令第一次绑定到元素时调用，**当 DOM 更新时 bind 函数不会被触发**。

```js
directives:{
	color:{
		//当指令第一次被绑定到元素时被调用
        bind(el，binding){
			el.style.color=binding.value
	}，
		//每次DOM更新时被调用
		update(el，binding){
			el.style.color=binding.value
	}
```



##### 6. 同时使用 bind 和 update 函数简写

如果 `bind` 和 `update` 函数中的 `逻辑完全相同`，则对象格式的自定义指令可以简写成函数格式：

```js
directives:{
    //在 bind 和 update 时，会触发相同的业务逻辑
    color( el，binding ){
        el.style.color=binding.value
}
```





### 全局自定义指令 Vue.directive ()

> 通过“ `Vue.directive()`” 进行声明 。

注意：在使用 Vue-cli ( 脚手架) 时，要把全局自定义指令写在`main.js`文件中：

```js
//参数1：字符串，表示全局自定义指令的名字
//参数2：对象，用来接收指令的参数值

Vue.directive('color'，function(el，binding){
		el.style.color=binding.value
})

// 简写
Vue.directive（'color'，(el，binding) =>{
		el.style.color=binding.value
})
```



# vue-cli

> [Vue CLI](https://cli.vuejs.org/zh/) 是官方发布的vue.js项目脚手架, 可以快速搭建vue开发环境以及Webpack配置。

### 安装和使用

一、安装Vue脚手架 

```
npm install -g @vue/cli 
```

二、创建项目 

```
vue create project(项目名称)
```

目录详解：

```js
|- public         // 静态页面目录
    |- index.html // 项目入口
|- src            // 源码目录
    |- assets     // 存放项目中用到的静态资源文件，例如：css 样式表、图片资源
    |- components     // 封装的、可复用的组件，都要放到 components 目录下
    |- App.vue        // 根组件
    |- main.js        // 项目的入口文件。整个项目的运行，要先执行
```



vue 项目的运行流程：

通过 `main.js `把 `App.vue `渲染到` index.html `的指定区域中。



1. **App.vue** 用来编写待渲染的`模板结构 `
2. **index.html** 中需要预留一个 `el 区域 `
3. **main.js** 把 App.vue 渲染到了 index.html 所预留的区域中



vue-cli 打开项目流程：

index.html  ==>  main.js ==> render（App.vue）

- 网页访问的是 index.html  ，它引入了  main.js
-  main.js 作为入口文件，它渲染了 App.vue。



# axios 

> axios 是一个专注于网络请求的库， 调用 axios 方法得到的返回值是 `Promise 对象`。



### axios 的基本使用

1. 发起 GET 请求：

   ```js
   axios({
     // 请求方式
     method: 'GET',
     // 请求的地址
     url: 'http://www.liulongbin.top:3006/api/getbooks',
       
     // URL 中的查询参数（GET请求传参）
     params: {
       id: 1
     }
   }).then(function (result) {
     console.log(result)
   })
   ```

   - `params`  表示传递到服务器端的数据，以`url参数`的形式拼接在请求地址后面
     - 如 ： params: { page:1,per:3 }
     - 最终生成：http://jsonplaceholder.typicode.com/?page=1&per=3

2. 发起 POST 请求：

   ```js
   document.querySelector('#btnPost').addEventListener('click', async function () {
       
     // 如果调用某个方法的返回值是 Promise 实例，则前面可以添加 await！
     // await 只能用在被 async “修饰”的方法中
     
     
     // 1. 调用 axios 之后，使用 async/await 进行简化
     // 2. 使用解构赋值，从 axios 封装的大对象中，把 data 属性解构出来
     // 3. 把解构出来的data属性，使用冒号进行重命名，一般都重命名为 { data: res }
       
     const { data: res } = await axios({
       method: 'POST', 
       url: 'http://www.liulongbin.top:3006/api/post',
         
       //  POST请求体传参
       data: {
         name: 'zs',
         age: 20
       }
     })
   
     console.log(res)
   })
   ```



-

 

axios  封装的 6 个属性：

> axios  在请求到数据之后，在真正的数据之外，套了一层外壳。

```json
{	
    config:{ },  
    // data 才是服务器返回的真实数据
    data:{ 
        status: 200,
        msg: "获取数据成功！",
        data: Array(6)
    }, 
    headers:{ ... },
    request:{ },
    status: 200,
    statusText: 'OK',    
}
```



### axios直接发起GET和POST请求

```js
document.querySelector('#btnGET').addEventListener('click', async function () {
    /* axios.get('url地址', {
        // GET 参数
        params: {}
      }) */

    const { data: res } = await axios.get('http://www.liulongbin.top:3006/api/getbooks', {
        params: { id: 1 }
    })
    console.log(res)
})

document.querySelector('#btnPOST').addEventListener('click', async function () {
    // axios.post('url', { /* POST 请求体数据 */ })
    const { data: res } = await axios.post('http://www.liulongbin.top:3006/api/post', { name: 'zs', gender: '女' })
    console.log(res)
})
```



### 在 Vue-cli 中使用 axios

一般我们会直接这么用：

```vue
// 在组件内
<template>
    <button @click="postInfo">发起 POST 请求</button>
</template>


<script>
// 1. 导入 axios 
import axios from 'axios' 
export default {
 //2. 在 methods 定义 axios请求方法
  methods: {
    async postInfo() {
      const { data: res } = await axios.post('http://www.liaoyia.top:3306/api/post', { name: 'zs', age: 20 })
      console.log(res)
    }
  }
}
</script>
```



缺点： 在每次使用时候都要导入 axios 文件，写请求地址 (对后期维护不友好) 。



#### 把 axios 挂载到 Vue原型上并配置请求根路径

> 避免重复导入axios 和重复写入完整请求地址。

在`main.js `中配置

```js
import Vue from 'vue'
import App from './App.vue'
import axios from 'axios'

Vue.config.productionTip = false

// 全局配置 axios 的请求根路径 (官方提供配置项)
axios.defaults.baseURL = 'http://www.liaoyia.top:3006'

// 把 axios 挂载到 Vue.prototype 上，供每个 .vue 组件的实例直接使用
Vue.prototype.$http = axios

// 今后，在每个 .vue 组件中要发起请求，直接调用 this.$http.xxx
// 但是，把 axios 挂载到 Vue 原型上，有一个缺点：不利于 API 接口的复用！！！

new Vue({
  render: h => h(App)
}).$mount('#app')

```

使用：

```vue
<template>
     <button @click="btnGetBooks">获取图书列表的数据</button>
</template>


<script>
export default {
  methods: {
    // 点击按钮，获取图书列表的数据
    async btnGetBooks() {
      const { data: res } = await this.$http.get('/api/getbooks')
      console.log(res)
    }
  }
}
</script>
```



但是把 axios 挂载到 Vue原型上并配置请求根路径也有缺点：

**`无法实现API接口的复用`** :  在多个组件想使用同一个请求方法(API)的时候，只能在每个组件重新定义一次。



### axios 封装 



1.如果项目中有多个请求地址，我们可以根据多个地址使用工厂模式封装 js 模块，创建多个 `axios` 实例对象，并设置请求根路径 (`baseURL`) ：

步骤如下： 在项目的 `src`目录下`创建utils`文件夹并新建一个 `request.js`文件：

```js
import axiox from 'axios'

const request =axiox.create({
    //baseURL会在发送请求的时候拼接在url参数的前面
    baseURL:'http://jsonplaceholder.typicode.com/',
    
    timeout:5000
})

// 向外导出
export default  request
```

- 使用这种方法时： 一般我们只会在一个 js 模块创建一个`axios` 实例对象，并向外导出。
- 如果有多个服务器地址，那就创建多个 js模块，并在里面创建axios实例对象。

2.为了实现复用性，我们还可以把所有请求，都封装在API模块里，在API模块中，按需导出一个方法，这个方法调用 request.get 或 request.post 来请求一个接口，最后return 一个Promise 对象。



比如想调用接口获取用户相关信息：

在根目录新建 `utils` 文件夹并在里面新建 `userAPI.js` 文件

````js
//导入 utils 文件夹下的 request.js

import request from '@/utils/request.js'

export const getArticleListAPI = function(_page, _limit) {
  return request.get('/articles', {
    params: {
      _page,
      _limit
    }
  })
}
````



![](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/DjangoSnipaste_2021-11-02_20-11-38.png)



【实例】：点击按钮发起 GET 请求并自动调用`请求拦截`和添加`响应拦截器`：

```vue
<template>
  <div class="home">
    <button @click="getByMineHandle">调用封装的get请求</button>
  </div>
</template>

<script>
 // 导入 get 方法
import  { get } from '../utils/request'

export default {
  name: 'Home',
  methods:{
    
    getByMineHandle(){
      get('',{page:3,per:2}).
      then(res=>console.log(res))
    }
  }
}
</script>
```



```js
import axiox from 'axios'

const instance =axiox.create({
    //baseURL会在发送请求的时候拼接在url参数的前面
    baseURL:'http://jsonplaceholder.typicode.com/',
    timeout:5000
})

//请求拦截
// 所有的网络请求都会先走这个方法
// 添加请求拦截器,所有的网络请求都会先走这个方法
// 我们可以在它里面为请求添加一些自定义的内容 比如 token 或者在 headers 提供信息
instance.interceptors.request.use(function (config) {
    // 在发送请求之前做些什么
    console.group('全局请求拦截')
    console.log(config)
    console.groupEnd()
    config.headers.token ='12343'
    return config;
}, function (error) {
    // 对请求错误做些什么
    return Promise.reject(error);
});

// 添加响应拦截器
//此处可以根据服务器的返回状态码做响应的处理
//404 404 500
instance.interceptors.response.use(function (response) {
    // 对响应数据做点什么
    console.group('全局响应拦截')
    console.log(response)
    console.groupEnd()
    return response;
}, function (error) {
    // 对响应错误做点什么
    return Promise.reject(error);
});

// 从服务器查看获取数据
export function get(url,params) {
    return instance.get(url,{
        params
    })
}

// 向服务器创建数据
export function post(url,data) {
    return instance.post(url,data)
}

// 从服务器删除数据
export  function del(url) {
    return instance.delete(url)
}

//  向服务器发送更新数据
export  function put(url,data) {
    return instance.put(url,data)
}
```

![QQ截图20211102113013](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/DjangoQQ%E6%88%AA%E5%9B%BE20211102113013.png)

-------



# Vue-router

### SPA 与前端路由 

`SPA  (单页面网页)`，所有组件的展示与切换都在这唯一的一个页面内完成。 

此时，不同组件之间的切换需要依赖 前端`路由(router)`来实现。 



什么是前端路由？

- `Hash 地址与组件之间的对应关系`，不同的Hash 展示不同的页面。

![QQ截图20211031144325](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/DjangoQQ%E6%88%AA%E5%9B%BE20211031144325.png)

前端路由的工作方式：

![QQ截图20211031143159](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/DjangoQQ%E6%88%AA%E5%9B%BE20211031143159.png)

### vue-router 的基本用法

[vue-router](https://router.vuejs.org/zh/) 是 vue.js 官方给出的路由解决方案。它只能结合 vue 项目进行使用，能够轻松的管理 SPA 项目中组件的切换。



1. 安装 vue-router 包 

   ```shell
   npm i vue-router@3.5.2 -S
   ```

   此时 src 源码目录下会新增一个 router 文件夹。

2. 创建路由模块 

   在 src 源代码目录下，新建 `router/index.js` 路由模块，并初始化：

   ```js
   
   //1.导入Vue和VueRouter的包import Vue from'vue'
   import VueRouter from‘vue-router'
   
   //2.调用Vue.use（）函数，把VueRouter 安装为Vue的插件
   Vue.use（VueRouter）
   
   //3.创建路由的实例对象
   const router=new VueRouter（）
   
   //4.向外共享路由的实例对象
   export default router
   
   ```

   

3. 导入并挂载路由模块 

   在 `src/main.js` 入口文件中，导入并挂载路由模块：

   ```js
   import Vue from'vue'
   import App from'./App.vue'
   
   //1.导入路由模块
   import router from@/router'
   
   new Vue ({
   render:h=>h（App），
       
   //2.挂载路由模块
   router:router
   }).$mount('#app')
   
   ```

   

4. 声明路由链接和占位符

   在 src/App.vue 组件中，使用 vue-router 提供的 `<router-link>`  和`<router-view> `声明路由链接和占位符:

   ```vue
   
   <tcmplate>
       <div class="app-container">
           <h1>App组件</h1>
           
           <！--1.定义路由链接-->
           <router-link to="/home">首页</router-link>
           <router-link to="/movie">电影</router-link>
           <router-link to="/about">关于</router-link>
           
           <！--2.定义路由的占位符-->
           <router-view></router-view>
       </div>
   </template>
           
   ```

   

5.  声明路由的`匹配规则`

   在 `src/router/index.js` 路由模块中，通过 `routes` 数组声明路由的匹配规则：

   ```js
   
   //导入需要使用路由切换展示的组件
   
   import Home from'@/components/Home.vue'
   import Movie from'@/components/Movie.vue'
   import About from'@/components/About.vue'
   
   //2.创建路由的实例对象
   const router=new VueRouter({
       //在routes数组中，声明路由的匹配规则
   	routes:[
           //path 表示要匹配的hash地址；component表示要展示的路由组件
           {path:'/home'，component: Home}，
   		{path:'/movie'，component: Movie}，
           {path:'/about'，component: About}
   ]
   })
   ```

   





### 路由重定向

> 当访问地址 A 的时候，`强制用户跳转` 到地址C, 从而展示特定的组件页面。

- 通过路由规则的 `redirect` 属性，指定一个新的路由地址来实现路由的重定向。



下面这个应用场景，当用户访问网页 / 目录时候，跳转到首页：

```js
const router=new VueRouter({
//在routes数组中，声明路由的匹配规则
    routes:[
		//当用户访问 / 的时候，通过 redirect 属性跳转到 /home 对应的路由规则
		{ path: '/'，redirect:'/home' }，
		{ path: '/home'，component: Home }，
		{ path: '/movie'，component: Movie }，
		{ path: '/about'，component: About }
})
```





### 路由高亮

可以通过如下的两种方式，将激活的路由链接进行高亮显示：

1.  使用`默认的`高亮 class 类 
2.  `自定义`路由高亮的 class 类



#### 默认的高亮 class 类

被激活的路由链接，默认会应用一个叫做 `router-link-active` 的类名, 可以使用此`类名选择器`，为激活 的路由链接设置高亮的样式：

```css
/*在index.css全局样式表中，重新router-link-active的样式*/

.router-link-active {
    color:#ef4238；
    font-weight:bold;
    border-bottom: 2px #ef4238 solid;
}
```

#### 自定义路由高亮的 class 类

在创建路由的实例对象时，开发者可以基于 `linkActiveClass` 属性，自定义路由链接被激活时所应用的类名：

```js
const router=createRouter({
	history:createlebHashHistory()，
	// 指定被激活的路由链接，会应用 router-active 这个类名，
	// 默认的 router-link-active 类名会被覆盖掉
    1inkActiveclass:'router-active'，
    routes:[
    	{ path: '/'，redirect: '/home' }，
		{ path: '/home'，component: Home }，
		{ path：'/movie'，component: Movie }，
		{ path: '/about'，component: About }，
]
})
```



### 嵌套路由

> 通过路由实现组件的嵌套展示，叫做嵌套路由。

![QQ截图20211031165158](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/DjangoQQ%E6%88%AA%E5%9B%BE20211031165158.png)

##### 1. 声明子路由链接和子路由占位符

上图中，我们在 `About.vue` 组件中套娃了 tab1 和 tab2 组件 ，如果我们想展示它，则需要声明`子路由链接`以及`子路由占位符 ` :

```vue
<template>
	<div class="about-container">
	 	<h3>About 组件</h3>
        
		<！--1.在关于页面中，声明两个子路由链接-->
		<router-link to="/about/tab1">tab1</router-link>
    	<router-link to="/about/tab2">tab2</router-link>

    	<hr/>
 		<！--2.在关于页面中，声明子路由的占位符-->
 		<router-view></router-view>
	</div>
</template>
```

到此为止，我们已经有了 `子路由链接`、`子路由占位符`，以及`链接对应的组件`，但是要在页面显示，我们还缺少对应关系，也就是`路由规则`。

##### 2.通过 children 属性声明子路由规则

在 `src/router/index.js` 路由模块中，导入需要的组件，并使用 `children` 属性声明子路由规则：

```js
// 导入组件
import Tab1 from '@/components/tabs/Tab1.vue'
import Tab2 from '@/components/tabs/Tab2.vue'

const router=new VueRouter({
    routes:[
        {	//about 页面的路由规则（父级路由规则）
            path: '/about'，
            component: About，
            //1.通过children属性，嵌套声明子级路由规则
            children:[
			{ path: 'tab1'，component: Tab1 }，//2.访问/about/tab1时，展示Tab1组件
			{ path: 'tab2'，component: Tab2 } //2.访问/about/tab2时，展示Tab2组件
}]
})
```

- 注意：使用 ` children ` 属性嵌套声明子级路由规则时，path 名称不需要加 `/`。



##### 3. 默认子路由

> 如果`children` 数组中，某个路由规则的 path 值为空字符串，则这条路由规则，叫作“默认子路由”。

```js
// src/router/index.js

const router=new VueRouter({
    routes:[
        {	
            path: '/about'，
            component: About，
            // rediect:  'about/table'  // 通过重定向设置默认展示页面
            children:[
            
            //  path值为空字符串,此时Tab1为默认子路由，一进入About页面，默认展示 Tab1
			{ path: ''，component: Tab1 }，
        	{ path: 'tab2'，component: Tab2 } 
	}]
})
```

- 所以，`默认子路由`和`路由重定向`都可以用来设置 `展示特定的组件页面` 。





### 动态路由匹配

> 动态路由指的是：把 Hash 地址中`可变的部分`定义为`参数项`，从而`提高路由规则的复用性`。



有如下3个路由链接：

```vue
<router-link to="/movie/1"> 电影1 </router-link>
<router-link to="/movie/2"> 电影2 </router-link>
<router-link to="/movie/3"> 电影3 </router-link>
```

如果定义下面3个路由规则，虽然可行，但太过繁琐且复用性差：

```js

{path:'/movie/1'，component:Movie}
{path:'/movie/2'，component:Movie}
{path:'/movie/3'，component:Movie}

```

使用动态路由，将上面创建3个路由规则，合并成一个，提高了路由规则的复用性：

```js
//  src/router/index.js 文件

//路由中的动态参数以 : 进行声明，冒号后面的是动态参数的名称
{ path:'/movie/:id'，component:Movie}

```

- 在 vue-router 中使用英文的冒号（`:`）来定义路由的参数项 。



##### `$route.params `  访问动态匹配的参数值

在 `动态路由` 渲染出来的组件中，可以使用 `this.$route.params` 对象访问到动态匹配的参数值：

```vue
<template>
	<div class="movie-container">
		<！--this.$route是路由的“参数对象”-->
		<h3> Movie 组件-- {{this.$route.params.id}} </h3>
	</div>
</template>

<script>
	export default{
		name:'Movie'
	}
</script>
```



##### 使用 props 接收路由参数

为了简化路由参数的获取形式，vue-router 允许在路由规则中开启 `props` 传参, 在定义路由规则时，声明`props : true` 选项 ,

```js

// router下的 index.js 文件
// 1. 声明 props : true 选项
{ path:'/movie/：id'，component: Movie，props:true }

```

定义好后，即可在Movie组件中，以props的形式接收到路由规则匹配到的参数项。

```vue
<template>
	<！-- 3、直接使用props中接收的路由参数 -->
    <h3> MyMovie组件--{{id}}</h3>
</template>

<script>
export default{
    //2、使用props接收路由规则中匹配到的参数项
	props:['id']
</script>
```



### 编程式导航  API

vue-router 提供了许多编程式导航的 API，其中最常用的导航 API 分别是：

1. this.$router. `push`('hash 地址') 
   -  跳转到指定 hash 地址，并`增加`一条历史记录 。
2. this.$router.`replace`('hash 地址') 
   - 跳转到指定的 hash 地址，并`替换掉当前`的历史记录 。
3. this.$router. `go` (数值 n) 
   - 实现导航历史前进、后退。
   - $router.`back()`，后退到上一个页面。
   - $router.`forward()` ，前进到下一个页面。



调用 `this.$router.push() ` 或者 `$router.replace()`方法，可以跳转到指定的 hash 地址，展示对应的组件页面 :

```vue

<template>
	<div class="home-container">
		<h3> Home组件 </h3>
		<button @click="gotoMovie">跳转到Movie页面</button>
	</div>
</template>

<script>
export default{
	methods:{
		gotollovie(){
        this.$router.push("/movie/1")
    }
}
</script>
```



push 和 replace 的区别： 

- push 会 `增加一条历史记录 `
- replace 不会增加历史记录，而是`替换掉当前的历史记录`。



调用 `this.$router.go()` 方法，可以在浏览历史中前进和后退：

```vue
<template>
	<h3> MyMovie组件---{{id}}</h3>
	<button @click="goBack" >后退</button>
</template>

<script>
export default{
	props:['id']，
    methods:{
		goBack(){
            this.$router.go(-1) //后退到之前的组件页面
        }
}，
</script>
```



### 导航守卫

![](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/DjangoQQ%E6%88%AA%E5%9B%BE20211031202624.png)

每次发生路由的导航跳转时，都会触发全局前置守卫。因此，在全局前置守卫中，我们可以对每个路由进行访问权限的控制：

```js
//创建路由实例对象
const router = new VueRouter（{..…}）

//调用路由实例对象的beforeEach方法，即可声明“全局前置守卫"
//每次发生路由导航跳转的时候，都会自动触发这个“回调函数”

router.beforeEach((to,from,next) =>{
    /* 必须调 next 函数 */   
})
```



守卫方法的 3 个形参：

- `to` 是将要访问的路由的信息 (对象)
- `from` 是将要离开的路由的信息对象
- `next`  是一个函数，一定要调用 `next ()` 才表示放行，允许这次路由导航 。



注意： 

1.   在守卫方法中如果`不声明 next` 形参，则默认`允许用户访问每一个路由`！ 
2.   在守卫方法中如果`声明了 next 形参`，则`必须调用 next() 函数`，否则不允许用户访问任何一个路由！





#### next 函数的 3 种调用方式

![](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/DjangoSnipaste_2021-11-08_10-45-14.png)



1.  直接放行：`next()` 
2. `强制其停留在当前页面`：next(`false`) 
3. `强制其跳转到登录页面`：next('/login')



#### 结合 token 控制后台主页的访问权限

```js

router.beforeEach((to，from，next) => {
     //获取浏览器缓存的用户信息
    const userInfo = window.localStorage.getItem('token');
	if (to.path === '/main' && !userInfo ){ //访问的是 main 页面且 token 不存在
            // 访问的是后台主页，但是没有token的值,跳转到登入页
			next('/login')  
	} else{
	   next()//访问的不是后台主页，直接放行
}）
```

上面代码中，如果项目中有多个页面都需要设置访问权限怎么办呢 ?

- 使用路由导航守卫 判断访问的是否是一个有权限的 Hash 地址问题：

```js
// 第一种: 使用 || 多重判断，代码臃肿

router.beforeEach((to，from，next) => {
	if (to.path === '/main'  || to.path === '/user' || to.path === 'seeting'){
          // code....
		} else{
          // ....  
	} 
}）                 
```

我们可以把多个地址存入到一个`数组`或者一个`json`文件、`js`文件：

```js

const patArr = ['/mian', '/home', '/home/users', '/home/rights']

router.beforeEach((to，from，next) => {
	if (patArr.indexOf(to.path)  !== -1){
          // code....
		} else{
          // ....  
	} 
}）  
```

```js
// 存入 js 使用

// 1. 定义一个 pathArr.js文件
export default ['/mian', '/home', '/home/users', '/home/rights']
// 2. 使用时导入 
import  pathArr from '@/router'

```

### meta 路由原信息 

上面的例子中，还是不够方便，vue-router 中给我们提供了 `meta` 字段，传入一个对象，设置`requiresAuth: true` ，就相当于开起来跳转验证。

```js
const router = new VueRouter({
  routes: [
    {
      path: '/blog',
      component: Blog,
          path: 'bar',
          component: Bar,
          // 设置 meta ，开启跳转验证
          meta: { requiresAuth: true }
    }
  ]
})
```

全局守卫中定义页面权限：

```js
router.beforeEach((to, from, next) => {
	console.log(to) 
    //判断 $route.matched数组 是否中有 meta 字段值
    if (to.matched.some(record => record.meta.requiresAuth)) {
        如果当前页面的 是否有 user
        if (!localStorage.getItem("user")) {
            next({
                path: '/login',
                //传入完整的路径
                query: { redirect: to.fullPath }
            })
        } else {
            next()
        }
    } else {
        next() // 确保一定要调用 next() 这次放行说明在白名单
    }
})
```

更改登入页面的逻辑

````js
// Login.vue
methods:{
    handLeLonge(){
	// 获取用户名和密码
        setTimeout(()=>{
            let data = {
                user: this.user
            }
        // 保存用户名到本地
            localStorage.setItem('user',JSON.stringify(data))
            this.$route.push({
                path: this.$route.query.redirect
            })
        })
    }

}

````





### 拓展 - 网页链接详解

```vue
 
 // APP 组件
 <router-link to="/movie/1"> 洛基 </router-link>
 <router-link to="/movie/2?name=zs&age=20"> 雷神< /router-link>
 <router-link to="/movie/3"> 复联 </router-link>
 <router-link to="/about">   关于 </router-link>

```

```vue

<template>
  <div class="movie-container">
    <!-- this.$route  是路由的“参数对象” -->
    <!-- this.$router 是路由的“导航对象” -->
    <h3>Movie 组件 --- {{ $route.params.mid }} --- {{ mid }}</h3>
      
    <button @click="showThis"> 打印 this</button>
      
    <!-- 在行内使用编程式导航跳转的时候，this 必须要省略，否则会报错！ -->
    <button @click="$router.back()">back 后退</button>
    <button @click="$router.forward()">forward 前进</button>
      
  </div>
</template>

<script>
export default {
  name: 'Movie',
  // 接收 props 数据
  props: ['mid'],
  methods: {
    showThis() {
      console.log(this)
    },
  }
}
</script>
```



点击雷神电影打印出的 this 里，有一个 `$route` 对象, 如下：

![网页链接详解](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/DjangoQQ%E6%88%AA%E5%9B%BE20211101114216.png)

1. 在 hash 地址中，` / `后面的参数项，叫做 **“路径参数”**
   - 在路由“参数对象”中，需要使用 `this.$route.params` 来访问路径参数
   - 但是我们一般会使用  `props`传参来接收路径参数。
2. 在 hash 地址中，`?` 后面的参数项，叫做**“查询参数”**
   - 在路由“参数对象”中，需要使用 `this.$route.query`  来访问查询参数
3. 在 `this.$route` 中，path 只是路径部分；fullPath 是完整的地址。



开发思路：

- 要想从一个地址跳到另一个地址 (如： 登录后立即跳转到首页，并且首页里设置默认的展示的页面)

  - 先要确定 `离开谁` 、`要去哪儿`
  - 找到 要离开 的页面，给它添加  `redirect `指向新地址。

  

注意点 ：

- 标签节点不平级时无法使用 v-if 和 v-else 结合來判断

- 在 vue 中：

  - 在使用组件的时候，如果某个属性名是“小驼峰”形式，则绑定属性的时候，建议改写成“连字符”格式。例如  `cmtCount` 建议写成 `cmt-count` 

    ![](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/DjangoSnipaste_2021-11-02_21-00-08.png)
  
  
  
  

-------

# Vuex



### Vuex 是什么

> `vuex` 是终极的组件之间的数据共享方案。

- Vuex 是实现组件全局状态（数据）管理的一种机制，可以方便的实现组件之间数据的共享。

![](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/Djangosrc=http___image.mamicode.com_info_201812_20181229191144131679.png&refer=http___image.mamicode.jpg)

<img src="https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/Django20210311150835126.png" alt="20210311150835126" style="zoom:67%;" />

优点：

1. 集中管理共享的数居，易于开发和维护
2. 高效地实现组件之间的数据共享，提高开发效率
3. 响应式数据，与页面的同步



##### 什么样的数据适合存储到 Vuex 中 ？

一般情况下，只有组件之间共享的数据，才有必要存储到 vuex 中；对于组件中的私有数据，依旧存储在组件自身的data中即可 ！



##### Vuex 的基本使用：

1. 安装 vuex 依赖包

   ```shell
    npm install vuex --save
   ```

   

2. 在 `store / index.js` 导入包 并创建 store 对象

   ```js
   //导入 vuex 包
   import Vuex from 'vuex'
   Vue.use(Vuex)
   
   // 创建 store 对象
   const store = new Vuex.Store({
       // state 中存放的就是全局共享的数据
       state: { count: 0 }，
       mutations: {},
       actions: {},
       modules: {}
   })
   
   ```

   

3. 将 store 对象挂载到 vue 实例`main.js`中

   ```js
   
   import store from './store/index';
   
   new Vue({
    el: '#app',
    render: h => h(app),
    router,
    // 将创建的共享数据对象，挂载到 Vue 实例中
    // 所有的组件，就可以直接从 store 中获取全局的数据了
    store
   })
   
   ```

   

一张图看懂 vuex：

![QQ截图20211114171546](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/DjangoQQ%E6%88%AA%E5%9B%BE20211114171546.png)



### state 数据源

> State 提供唯一的公共数据源，所有共享的数据都要统一放到 `Store`的 `state` 中进行存储。

访问 state 中数据两种方式 ：

##### this.$store.state.全局数据名称 访问：

```js

// this.$store.state.全局数据名称
<h3>当前最新的count值为：{{$store.state.count}}</h3>

```

- 由于在模板字符串中，是不需要写 this 的。



##### mapState 映射为计算属性：

通过 mapState 函数，将当前组件需要的全局数据，映射为 `computed` 计算属性：

```js

// 1. 在想使用数据的组件中 从 vuex 中按需导入 mapState 函数
import { mapState } from 'vuex'

// 2. 将全局数据，映射为当前组件的计算属性
computed: {
 ...mapState(['count'])
}
```



### mutations 变更数据

> `mutation` 用于变更 store 中的数据。

- 只能通过 mutation 变更 Store 数据，不可以直接操作 Store 中的数据。 
- 虽然操作起来稍微繁琐，但可以集中监控所有数据的变化。



##### this.$store.commit() 触发 mutations

```js
// 定义 Mutation
const store = new Vuex.Store({
    state: {
        count: 0
    },
    mutations: {
        add(state) {
            // 变更状态
            state.count++
        }
    }
})

```

```js
// 在组件中 触发 mutation
methods: {
    handle1() {
        // 触发 mutations 的第一种方式
        this.$store.commit('add')
    }
}
```



可以在触发 mutations 时`传递参数`：

```js
// 定义Mutation
const store = new Vuex.Store({
    state: {
        count: 0
    },
    mutations: {
        addN(state, step) {
            // 变更状态
            state.count += step
        }
    }
})
```



```js
// 触发mutation
methods: {
    handle2() {
        // 在调用 commit 函数，
        // 触发 mutations 时携带参数
        this.$store.commit('addN', 3)
    }
}
```





#####  mapMutations 映射为方法

1. 从vuex中按需导入 mapMutations函数

   ```js
   import { mapMutations } from 'vuex'
   ```

   

2. 通过刚才按需导入的 mapMutations 函数，映射为当前组件的`methods`函数。

   ```js
   
   // 2. 将指定的 mutations 函数，映射为当前组件的 methods 函数
   methods: {
    ...mapMutations(['add', 'addN'])
   }
   
   ```



实例：

```js
// store
mutations: {
  add(state) {
    // 变更状态
    state.count++
  },
  sub(state) {
    state.count--
  },
  addN(state, step) {
    // 变更状态
    state.count += step
  },
  subN(state, step) {
    state.count -= step
  }
},

// 组件A
import { mapState,mapMutations } from 'vuex'
methods:{
  ...mapMutations(['sub','subN']),
  decrement(){
    // 调用 
    this.sub()
  },
  decrementN(){
    this.subN(5)
  }
}
```



### Actions 处理异步操作

> 如果通过**异步**操作变更数据，**必须通过 Action**,而不能使用Mutation,但是在 Action中还是要**通过触发Mutation**的方式间接**变更数据**。

注意： `在Actions 中不能直接修改 state中的数据，要通过 mutations修改`。



##### this.$store.dispatch 触发 Actions

```js
// store/index.js 定义 Action

const store = new Vuex.store({
  // ...省略其他代码
  mutations: {
    // 只有 mutations中的函数才有权利修改 state。
    // 不能在 mutations里执行异步操作。
    add(state) {
      state.count++
    }
  },
  actions: {
    // 在Actions 中不能直接修改 state中的数据，要通过 mutations修改。
    addAsync(context) {
      setTimeout(() => {
        context.commit('add')
      }, 1000);
    }
  },
})
```

```js
// 在组件中 触发 Action
methods:{
  handle(){
    // 触发 actions 的第一种方式
    this.$store.dispatch('addAsync')
  }
}

```



##### mapActions 映射为方法

```js
// 1. 从Vuex中按需导入 mapActions 函数。

import {mapActions} from 'vuex'

// 2. 将指定的 actions 函数，映射为当前组件 methods 的方法。
methods:{
  ...mapActions(['subAsync']),
  decrementAsync(){
    this.subAsync()
  }
}

```

```js
// store/index.js
actions: {
 // 在Actions 中不能直接修改 state中的数据，要通过 mutations修改。
  subAsync(context){
    setTimeout(() => {
      context.commit('sub')
    }, 1000);
  }
}

```





### Getter 按需展示数据

> Getter 用于对 Store中的数据进行加工处理形成新的数据。筛选或者排序显示。

1. Getter **不会修改 Store 中的原数据**，它只起到一个包装器的作用，将Store中的数据加工后输出出来。
2. Store 中数据发生变化， Getter 的数据也会跟着变化。



```js
//定义 Getter
const store = new Vuex.Store({
  state:{
	count:0
  },
  getters: {
    showNum(state) {
      return '当前最新的数量是【' + state.count + '】'
    }
  },
})
```

#####  this.$store.getters.名称 访问

```js
this.$store.getters.名称
```



##### mapGetters 映射为计算属性

```js
import { mapGetters } from 'vuex'

computed:{
	...mapGetters(['showNum'])
}
```



### 简写 !!!

其实，通过`mapState,mapMutations,mapActions,mapGetters`映射过来的计算属性，或者方法都可以直接调用，不用在 `commit` 或者 `dispatch`。

正常写法：

```js

<button @click="decrementAsync"> -1Async</button>

import {mapActions} from 'vuex'



methods: {
  ...mapActions(['subAsync']),
  decrementAsync(){
    this.subAsync()
  }
},

```

其实可以简写成：

```js
<button @click="decrementAsync"> -1Async</button>

import {mapActions} from 'vuex'

//...省略一些代码

methods: {
  ...mapActions(['subAsync']),
},

```

有参数的时候，也可以直接把参数带上，就像这样：

```html
<button @click="subAsync(5)"> +5 </button>
```

```js
import {mapActions} from 'vuex'

//...省略一些代码

methods: {
  ...mapActions(['addAsync']),
},

```



-----------

# Vue 3

### 使用  vite+vue3.0搭建项目

```shell

npm init vite-app 项目名称

cd 项目名称
npm install 
npm run dev

```

![Snipaste_2021-11-23_17-35-55](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/DjangoSnipaste_2021-11-23_17-35-55.png)

### vue3 新特性：

- `fragments`   Template 支持多个根标签

- 使用 createApp()，而 Vue 2 的是 new Vue()

  ```
  
  createApp(组件)
  
  new Vue({template, render})
  ```

  

- 更好的 ts 支持 `defineAsyncComponent`

  ```js
  import { ref,defineAsyncComponent } from 'vue'
  
  /*
  vue2中的写法:
    export default {
      data:{},
      methods: {},
    }
  */
  // vue3中更好的 ts语法支持,类型提示。
  
  defineAsyncComponent({
    data() {
      return{
        
      }
    },
    methods: {},
    computed:{},
    components:{}
  })
  defineProps({
    msg: String
  })
  const count = ref(0)
  ```

- 更好的`tree shaking`  配合 (esm) 按需导入，精简打包代码的大小。

- teleport 传送门 [示例：](https://codepen.io/team/Vue/pen/gOPNvjR)

- `custom renderer`    自定义渲染器

  - [飞机大战案例 (vue3 + canvas)](https://github.com/cuixiaorui/teaching-vuejs/tree/main/packages/play-plane)

- **composition api ** 质变

### 组件之间的数据共享

#### Vue 3 中的自定义事件

在vu3中使用使用自定义事相较vu2中多了一个  `emits` 节点声明:

vue 3中使用自定义事件:



在封装组件时:  

	1. 在`emits`节点声明自定义事件
	2. `触发`监听事件

在使用组件时: `监听`自定义事件

声明自定义事件:

```vue
// 子组件
<button @click="onBtnClick"> +1 </button>

<script>
export default {
    // 自定义事件必须声明到 emits 节点中
    emits: ['chage']
}
</script>
```

触发自定义事件:

```vue
// 子组件
<button @click="onBtnClick"> +1 </button>

<script>
export default {
    // 自定义事件必须声明到 emits 节点中
    emits: ['chage'],
    data() {
        // 子组件自己的数据，将来希望把 count 值传给父组件
        return { count: 0 }
    },
    methods: {
        onBtnClick() {
            this.count t= 1
            //修改数据时，通过 $emit()触发自定义事件
            // 当点击 '+1' 按钮时 调用this.$emit 触发自定义的 numchange 事件
            this.$emit( 'numchange')}
    }
}
</script>

```

监听自定义事件:

```js
// 父组件
// 使用 v-on 指令绑定监听事件
<Son @numchange="getNewCount"></Son>

methods : {
	getNewCount(val) {
	console.log('监听到了 count 值的变化', val)
}，
```



#### 组件上的 v-model

当`需要维护组件内外数据的同步时`，可以在组件上使用 v-model 指令。示意图：

![](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/DjangoQQ%E6%88%AA%E5%9B%BE20211106140507.png)

#### 父子组件数据共享

> 实现父子组件数据双向同步。



子组件向父组件共享数据：

- 父组件通过 `v-bind` 属性绑定向子组件共享数据，子组件使用 `props`接收数据。

  ![](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/DjangoSnipaste_2021-11-06_14-09-51.png)

父组件向子组件共享数据：

- 通过自定义事件 `emits` ，然后触发 `$emit()`, 子组件监听。

  ![](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/DjangoQQ%E6%88%AA%E5%9B%BE20211106142725.png)

1. 在 v-bind: 指令之前添加 v-model 指令 
2. 在子组件中声明 emits 自定义事件，格式为必须 update:xxx 
3. 调用 $emit() 触发自定义事件，更新父组件中的数据



使 v-model 实现子→父组件共享数据好处：

在父组件中 不用再`监听自定义事件`了，也不用再`额外声明事件处理`函数。



父组件：

```js
// 使用双向数据绑定指令
<my-son v-model:number="count"> </my-son>

data(){
	return{
		count: 0
	}
}
```

子组件：

```vue
// 子组件

<button @click="add"> +1 </button>

<script>
export default {
    prpos:['number']
    emits:['update:number']
    methods: {
        add() {
            this.$emit( 'update:number',this.num+1)}
    }
}
</script>
```



#### 后代关系组件之间的数据共享

> 指的是`父节点的组件`向其`子孙组件`共享数据。

使用步骤：

1. 父节点通过 `provide` 方法，对子孙组件共享数据。

   ```js
   export default {
     data() {
       return {
         color: 'red', // 1. 定义"父组件"要向"子孙组件"共享的数据
       }
     },
     provide() {  // provide函数 返回要共享的数据对象
       return {
         color: this.color,
         count: 1,
       }
     },
   }
   ```

   

2. 子孙节点通过 `inject` 接收数据。

   ```js
   
   <template>
       <h5>三级组件 --- {{ color }} --- {{ count }}</h5>
   </template>
   
   <script>
   export default {
     // 子孙组件，使用 inject 接收父节点共享的数据
     inject: ['color', 'count'],
   }
   </script>
   
   ```

   

#### 父节点对外共享响应式的数据

父节点使用 provide 向下共享数据时`并非响应式`，我们可以结合 `computed 函数`向下`共享响应式的数据`：

```js

import { computed } from 'vue' // 1. 从vue中按需导入 computed 函数

export default {
  data() {
    return {
      color: 'red',
    }
  },
  provide() {  
    return {
      // 2. 使用 computed 函数，把共享数据包装为"响应式"的数据
      color: computed(() => this.color),
      count: 1,
    }
  },
}
```

子孙节点使用响应式数据, 注意接收的响应式数据必须以 `.value` 的形式使用：

```js
<template>
    // 响应式数据，必须以.value 的形式进行使用
    <h5>子孙组件 --- {{ color.value }} --- {{ count }}</h5>
</template>

<script>
export default {
  // 子孙组件，使用 inject 接收父节点共享的数据
  inject: ['color', 'count'],
}
</script>

```



#### 全局数据共享

`vuex` 是终极的组件之间的数据共享方案。

- 解决大量、频繁的共享数据麻烦问题。
- vuex 就是用来管理组件中`需要共享的数据`。



#### 数据共享总结

父子关系：



1. 父 → 子 ： `自定义属性`
2. 子 → 父  ：`自定义事件`
3. 父子组件数据共享  使用组件上的 `v-model`



兄弟关系:  `EvenBus`



后代关系：`provide` & `inject`



全局数据共享： `vuex`



### 在vue3中全局配置 axios



```js
import { createApp } from 'vue'

import './assets/css/bootstrap.css'
import './index.css'

import axios from 'axios'

// 创建一个单页面应用程序实例
const app = createApp(App)

// 配置全局请求根路径
axios.defaults.baseURL = 'https://www.escook.cn'

// 全局挂载到app 根路径中
// $http 是模仿 vue封装成员的方式
app.config.globalProperties.$http = axios

// 实现挂载
app.mount('#app')

```



在组件中使用：

```js
methods: {
    async getInfo() {
      const { data: res } = await this.$http.get('/api/get', {
        // get 请求必须通过 params 传参
        params: {
          name: 'ls',
          age: 33,
        },
      })
      console.log(res)
    },
  },
```





### vue3 中创建路由



-  vue3 中需要安装 4.x的版本

  - 在 src 目录下创建 `router.js` 文件并配置

  ```js
  import { createRouter, createWebHashHistory } from 'vue-router'
  
  import Login from './components/MyLogin.vue'
  import Home from './components/Home.vue'
  
  
  // 创建路由实例对象
  const router = createRouter({
    history: createWebHashHistory(),   // 指定通过 hash 管理路由的切换
    routes: [
      { path: '/home', redirect: '/login' },
      { path: '/login', component: Login, name: 'login' },
    ],
  })
  
  export default router   //4. 向外共享路由对象
  ```

  

  在 `src/main.js` 入口文件中，导入并挂载路由模块：

  ```js
  import { createApp } from 'vue'
  import App from './App.vue'
  
  import router from './router'
  
  // 创建 app 实例
  const app = createApp(App)
  
  app.use(router)
  
  // 挂载 app 实例
  app.mount('#app')
  
  ```

  





(1) 创建一个创库

(2) git 全局设置

```
git config --global user.name "liaoyia"
git config --global user.email "2417276459@qq.com"
```

(3) git commit 命令用来将本地暂存的修改提交到版本库

```
git commit -m '提交信息'。
```

**git commit -a -m ‘提交信息’**

- 我们知道-m参数是输入提交信息的，-a 参数就是可以把还没有执行add命令的修改一起提交。

**git commit --amend**

这个命令就比较优秀了。经过个人的探索，我总结了它的两个功能

1.可以修改上一次的提交信息。

- 输入命令之后弹出一个vim编辑器的界面，有提交信息，提示，提交时间，修改的文件。然后我们将之前的222进行修改。
- 通过`git log`查看我们的提交信息。

2.可以将最近的修改追加到上一次的提交上。

- 我们在上一次修改的基础上再做一些修改。查看当前的状态。









