# Vue学习笔记

## 双向数据绑定

### Object.defineProperty

Object.defineProperty 方法，是es6(ECMAScript 2015) 中定义的的方法, 作用是为指定的对象定义一个新的属性.
以前我们给一个对象添加一个属性直接`obj.xx = yy` 这样就可以了，这样很灵活，`xx`可以被随意的修改或者删除，但是有时候我并不希望这样。
通过Object.defineProperty方法定义的属性的值默认是【不可以】被修改的!

如下代码

```javascript
var vm = {}
Object.defineProperty(vm, 'name', {}) // 给vm对象添加了一个属性name, 它的值为undefined  
// 尝试着修改vm.name 的值
vm.name = 18
console.log(vm.name) // undefined 输出的还是undefined，也就是说上一句的赋值操作并没有生效
```

Object.defineProperty 需要开发者提供一个get,和set方法

**Object.defineProperty(对象, 属性名, 描述)  **

* 参数1: 是我们想要为其添加属性的对象  
* 参数2: 是我们想要添加的属性 
* 参数3: 也是一个对象,开发者可以给该对象指定一个get方法和set方法，称之为getter，setter。

**给第三个参数添加get，set**

```javascript
var vm = {}
Object.defineProperty(vm, 'name', {
  get: function () {
    console.log('get方法执行了')
  },
  set: function (val) {
    console.log(val)
    console.log('set方法执行了')
  }
})
vm.name = 18
console.log(vm.name) // undefined 输出的还是undefined
// 但是执行 `vm.name = 18`时set方法执行了,且参数val就是18
// 执行console.log(vm.name)时 get方法执行了，输入的值是undefined, 这个undefined其实是get方法的返回值(没有return 则返回undefined)
```

使用Object.defineProperty的属性代理实现js中对象或变量中的值改变，页面元素的值也改变

###oninput

oinput是输入事件，当在文本中输入内容时触发!

**用oninput事件实现页面上的元素值改变，js对象或变量中的值也同时改变**

###封装双向数据绑定的实现代码

```javascript
// 定义DataBind
/**
 * 1.到dom中寻找有v-model属性的标签,
 * 2.根据v-model的值到options中找到对应的属性值
 * 3.把找到的属性赋值v-model所在标签的vlaue值
 */
function DataBind (options) {
  // console.log(this)
  var obj = options.data()
  // 我要用vm来代理这个obj对象
  // 要代理obj里的所有属性
  for (var key in obj) {
    DataBind.proxy(obj, key,this)
  }

  // 给v-model标签赋值
  DataBind.vModel(obj)
  // 给v-text赋值
  DataBind.vText(obj)
  DataBind.vModelChange(obj)
}

DataBind.proxy = function (obj, key, that) {
  // key就是obj的属性名
    // 假设key是'msg'时
    Object.defineProperty(that, key, {
      get: function () {
        // this.msg/vm.msg
        return obj[key]
      },
      set: function (val) {
        // this.msg = 18/ vm.msg = 18
        // set
        obj[key] = val
        DataBind.vModel(obj)
        DataBind.vText(obj)
      }
    })
}

DataBind.vModel = function (obj) {
  // 1.到dom中寻找有v-model属性的标签
  var oModels = document.querySelectorAll('[v-model]')
  // 2.根据v-model的值到options中找到对应的属性值
  for (var i = 0; i < oModels.length; i++) {
    var item = oModels[i] // 每一个元素
    var attrVal = item.getAttribute('v-model') // 获取属性值
    var val = obj[attrVal]
    // 3.把找到的属性赋值v-model所在标签的vlaue值
    item.value = val
  }
}

DataBind.vText = function (obj) {
  var oTexts = document.querySelectorAll('[v-text]')
  for (var i = 0; i < oTexts.length; i++) {
    var item = oTexts[i]
    var attrVal = item.getAttribute('v-text')
    // console.log(attrVal)
    // obj.msg/ obj['msg']
    // var val = obj[attrVal]
    var val = obj[attrVal]
    // console.log(val)
    item.innerText = val
  }
}

// 自动获取标签中的值
// 1.监听所有有v-model的属性的标签的oninput
// 2.在事件的回调函数里，读取v-model所在标签的value值
// 3.要把得到的value值赋值给obj中对应的的属性
DataBind.vModelChange = function (obj) {
  // 1.
  var oModels = document.querySelectorAll('[v-model]')
  for (var i = 0; i < oModels.length; i++) {
    var item = oModels[i]
    // oinput是输入事件，当我们在文本中输入内容时触发!
    item.oninput = function (e) {
      // 2.
      // item.value
      // e.target.value()
      // e.target就是这个item
      console.log(e.target.value)
      // 获取到这个元素的v-model属性的值
      var arrVal = e.target.getAttribute('v-model')
      // 将input的value值赋值给obj中与arrVal值同名的属性
      obj[arrVal] = e.target.value
    }
  }
}
```

**for循环中的代码如果不是立即执行，代码中所使用的for语句的变量值会是循环结束的变量值，可以在for循环中调用一个方法，把这个方法提取出来单独定义来解决这个问题**

使用封装的库

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
  <div>
    <input type="text" v-model="msg" value>
    <input type="text" v-model="name" value>
    <p v-text="msg"></p>
  </div>
</body>
</html>
<script src="./g.all.js"></script>
<script>
  // 用vm代理obj
  // 意思是如果想设置obj.xx,就通过vm.xx来设置
  // 如果想获取obj.xx,就通过vm.xx来获取

  // new 一个函数得到的返回值称之为这个构造函数的实例
  // 并且在执行时，这个构造函数里的this就是这个实例
  // vm === 函数里的 this
  // vm.xx === this.xxx
  var obj = {msg:'我是中国人', name: '小明'}
  var vm = new DataBind({
    data: function () {
      return obj
    }
  })
</script>
```

## 开始使用

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>开始使用Vue</title>
</head>
<body>
  <h1>vue快速开始</h1>
  <p v-text="age">xxx</p>
  <div id="app">
    <input type="text" v-model="age">
    <p v-text="age"></p>
    <p v-text="msg"></p>
  </div>
</body>
</html>
<!-- 1.引入vue.js, 引包-->
<script src="./vue.js"></script>
<script>
  // Vue 是vue.js中定义的构造函数是一个全局变量
  // 2.new Vue
  var vm = new Vue({
    data: function () {
      return {msg: '我是中国人，我爱自己的祖国', age: 18}
    }
    // 或者data:{msg:'我是中国人，我爱自己祖国',age:18}
    // 但是在组件中只能是一个方法，return一个对象
    methods: {
      sayHi: function () {
        console.log("Hello World")
      }
  })
  // 3.调用$mount方法让vue把data中的值添加到dom中
  // 参数是个选择器，作用是指定下Vue只作用于哪一部分的dom
  // 选择推荐选择body以内的元素
  vm.$mount('#app')		// 或者el：'#app'
</script>
```

## 指令

> vue中对html中特定的属性做了专门的处理，使得这些属性可以方便开发都书写项目，这些特殊的属性称之为指令！(vue中 v-开头的html属性)
>
> vue 的指令中除了直接写固定值外，还可以书写js表达式，vue还会把这个属性值当js解析，普通属性中写js表达式是无效的

###数据绑定v-text,v-html,{{}}

> 将数据绑定到标签之间的位置

* **v-text**	把解析后的结果赋值给标签的innerText，v-text的值中不可以使用BOM, 或者BOM对象中的属性，但是可以直接使用ecma中的对象
* **v-html**    作用和v-text几乎一样但是会赋值给标签的innerHTML
* **{{ }}**    

`v-text` 与 `{{}}` 及 `v-html` 的区别:  

- `v-text`是写在标签的属性上的, `v-text` 指定的表达式的值会被赋值标签的innerText  
- `{{}}` 是写在非标签的属性位置(标签之间), `{{}}` 指定的表达式的值会替换 {{}} 本身!,不会解析html
- `v-html`也是写在标签的属性上的, `v-html` 指定的表达式的值会被赋值给标签的innerHTML

```html
<p v-text="msg"></p>
<div v-html="msg"></div>
<div>
  {{msg}}
  {{1+1}}
</div>
```

###表单控件数据绑定v-model

> 将数据与表单控件进行绑定(如input,select)

**v-model**

- v-model写在**select**(下拉列表)中, 与option中的value值绑定，与哪个option的value值一致，哪个就会被选中，如果选择了某个option,它的value值就会被赋值给v-model对应的数据
- v-model写在**checkbox**(多选框)中，如果只有一个框，checkbox就两种值,true,false,对应选中与不选中，v-model对应的值如果是true,就选中，如果是false不选中
- v-model写在**checkbox**(多选框)中，选中的多选框的vlaue值会被添加进v-model指定的数组中，如果v-model指定的数组中的值, 与checkbox的value一致，这个checkbox就会被选中，必需指定一个value值
- radio一定要指定value值 , 这个value如果和v-model指定的值一致，就会被选中，点击某个radio,它的value就会被赋值给 v-model对应的值
- v-model写在**textarea**(文本域)中，v-model绑定的变量值就会渲染到文本域中，文本域中的value值也会被赋值给v-model对应的数据，保持同步

```html
<body>
  <h1>使用v-model绑定表单控件</h1>
  <div id="app">
    <h1>select 下拉</h1>
    <select v-model="action">
      <option value="chifan">吃饭</option>
      <option value="hejiu">喝酒</option>
    </select>
	
	<h1>checkbox 情况1: 一个框</h1>
    <!-- checkbox就两种值,true,false,对应选中与不选中-->
    <!-- v-model对应的值如果是true,就选中，如果是false不选中-->
    <input type="checkbox" v-model="isChecked">
	
    <h1>checkbox 情况2: 一组多选框</h1>
    吃饭<input type="checkbox" value="chifan" v-model="arrChecked">
    睡觉<input type="checkbox" value="shuijiao" v-model="arrChecked">
    打豆豆<input type="checkbox" value="dd" v-model="arrChecked">
    学习<input type="checkbox" value="xx" v-model="arrChecked">
    唱歌<input type="checkbox" value="cg" v-model="arrChecked">
    
    <h1>input-radio 单选框</h1>
    男<input name="xx" type="radio" value="99" v-model="sex"> <br>
    女<input name="xx" type="radio" value="1" v-model="sex">
    
	<h1>input-text</h1>
    <input type="text" v-model="msg">
    <textarea name="" id="" cols="30" rows="10" v-model="msg"></textarea>

    <button @click="sub">提交</button>
  </div>
</body>
</html>
<script src="./vue.js"></script>
<script>
  var vm = new Vue({
    el: '#app',  //$mount('#app')
    data: {
      msg: '小明你好',
      sex: 1,
      isChecked: true,
      arrChecked: [
        'chifan',
        'shuijiao',
        'xx'
      ],
      action: 'hejiu'
    },
    methods: {
      sub: function () {
        console.log(this.sex)
      }
    }
  })
</script>
```

### 属性绑定v-bind

**v-bind:**    动态的设置属性值，就要在属性名前加上  v-bind:

v-bind简写就是把v-bind去除，只保留一个冒号

```html
<p v-bind:id="age"></p>
<p :id="age"></p>
```

通过v-bind来给class属性赋值时，可以赋值一个对象，对象的属性名就是代理要添加的样式类名，如果属性值为 true, 那Vue会把对应的属性名追加给元素的class属性

```html
<div v-bind:class=" {active: isActive , hide: isHide}">根据</div>
```

###条件渲染v-if/v-else,v-show

* **v-if/v-else**	只要v-if对应的表达式，执行的结果是true,那么这个元素就存在，否则就不存在,这个元素不是被隐藏，是被删除了

  v-if 与 v-else配合使用, 只会显示一个标签，如果v-if所在标签显示，则v-else所在标签就不存在，如果v-if所在标签不存在，则v-else所在标签就显示    (选择性的显示元素)

  v-if 与 v-else所在标签一定要是 相联的兄弟元素

* **v-show**    v-show对应的表达式为true时，vue不对它做任何操作(如果是vue自己添加的display:none则会移除),为falsed vue会给这个元素添加display:none    (要么显示元素，要么隐藏元素)

  不要给用v-show的元素添加dispaly:none

```html
<body>
  <h1>v-if/v-else, v-show  控制元素的显示与否</h1>
  <!-- 只要v-if对应的表达式，执行的结果是true,那么这个元素就存在，
  否则就不存在,这个元素不是被隐藏，是被删除了! -->
  <div id="box">
    <h1 v-if="isExist">我是中国人，爱自己的祖国!</h1>
    <h1 v-if="sex === 0">男</h1>
    <h1 v-if="sex === 1">女</h1>
    <hr>
    <!-- v-if 与 v-else配合使用, 只会显示一个标签，如果v-if所在标签显示，则v-else所在标签就不存在
    如果v-if所在标签不存在，则v-else所在标签就显示 -->
    <!--*注意 v-if 与 v-else所在标签一定要是 相联的兄弟元素*-->
    <h1 v-if="sex === 0">男</h1>
    <!--<a href="">dd</a>-->
    <h1 v-else>女</h1>
    <hr>
    <h1>v-show</h1>

    <pre>要么显示元素，要么隐藏元素： display:none</pre>
    <!-- v-show对应的表达式为true时，vue不对它做任何操作(如果是vue自己添加的display:none则会移除),
    为falsed vue会给这个元素添加display:none-->
    <!-- *注意，不要给用v-show的元素添加dispaly:none*-->
    <p v-show="sex === 0">我出来了吗</p>
    <p v-show="true" class="myhide">我出来了吗???????</p>
  </div>
</body>
</html>
<!-- 1.引包 vue.js-->
<script src="./vue.js"></script>
<script>
// 2.实例化
var vm = new Vue({
  data: function () {
    return {
      isExist: true,
      sex: 0 // 0表示男，1表示女
    }
  }
})
.$mount('#box')
</script>
```

###列表渲染v-for

**v-for**    用来将数组中的数据渲染到页面

```html
<!--<div v-for="变量名 in 数组">-->
<!-- item是随便起的名字，表示数组中元素, list 是要遍历的数组, in 是固定写法-->
<div v-for="(item,index) in list">
  <h3>{{item.name}}</h3>
  <h5>{{item.age}}</h5>
</div>
<script>
  var vm = new Vue({
    data: function () {
      return {
        list: [
          {name: '小明明', age: 18},
          {name: '老王王', age: 16},
          {name: '王老吉', age: 97}
        ]
      }
    }
  }) 
  .$mount('#app')
</script>
```

###注册事件v-on

> v-on指令vue中用专门注册事件的指令

**v-on:**事件名

注册事件时，v-on指令可以简写，就是把 v-on: 简写为一个 @ 符号, 其他不变

```html
<button v-on:click="sayHi">点我</button>
<button v-on:click="hello(2,3,4, '嘻嘻', {a:1,b:2}, true)">带参数</button>
```

####事件修饰符

如果在事件名后加上.stop 表示阻止冒泡

如果在事件名后加上.prevent 表示阻止默认事件

1. .stop ：调用event.stopPropagation()阻止单击事件冒泡
2. .prevent ：调用 event.preventDefault()
3. .capture ：添加事件侦听器时使用 capture(捕获) 模式
4. .self ：只当事件在该元素本身（而不是子元素）触发时触发回调
5. .once ：只执行一次事件
6. 修饰符串联写法 ：v-on:click.stop.prevent
7. 只有修饰符 ：`<form v-on:submit.prevent></form>`

####按键修饰符

1. .enter（按回车键捕获）
2. .tab（按tab键捕获）
3. .esc（按esc键捕获）
4. .space（按空格键捕获）
5. .up（按↑键捕获）
6. .down（按↓键捕获）
7. .left（按←键捕获）
8. .right（按→键捕获）
9. .ctrl（按ctrl键捕获）
10. .alt（按alt键捕获）
11. .shift（按shift键捕获）
12. .字母（1.0.8+： 支持单字母按键别名）
13. 1.0.17+： 可以自定义按键别名：写法：Vue.directive('on').keyCodes.f1 = 112

##axios

默认发送的数据格式是json格式

'{"name:1","age":2}'  json格式  对应的请求头 application/json

jQuery默认发送的是formDate格式

'name=1&age=9&xx=11'  formData 对应的请求头 application/x-www-form-urlencoded

[文档地址](https://www.npmjs.com/package/axios)

```javascript
// 常规写法
axios({
    url: 'http://xx',
  	// get方式
  	// method: 'get',
  	// params: {id: myid, name: myname, age: 998},
    // post方式
  	method: 'post',
    // data里放post请求的参数!
    data:{
      name:1,age:2,sex:3
    }
}).then(function (res) {
  	// 返回的数据是res.data
	console.log(res.data)
  }).catch(function () {
	window.alert('请求失败')
  })
```

```javascript
// get请求简写
axios.get('/question?id=998')
  .then(function (response) {
    // 请求成功的回调函数
    console.log(response);
  })
  .catch(function (error) {
    // 请求失败的回调函数
    console.log(error);
  });

// get请求第二种简写
// 发送get请求时，也可以将url中的参数，以key,value的形式写在params属性中
// axios在发出请求之前，会自动将参数拼接到请求地址上去
axios.get('/question', {
    params: {
      id: 12345
    }
  })
  .then(function (response) {
    console.log(response)
  })
  .catch(function (error) {
    console.log(error)
  })
```

```javascript
// post请求简写
axios.post('/question', {
    age: 18,
    name: '小明'
  })
  .then(function (response) {
    console.log(response)
  })
  .catch(function (error) {
    console.log(error)
  })
```

###配置axios请求

```javascript
import axios from 'axios'

/**对axios发请求进行统一配置 */
// create得到的返回值就是一个新的axios,这个axios里的配置就是create中的参数
const myAxios = axios.create({
  // url: '/xx', 用myAxios发请求默认请求地址 就是 /xx
  // method: 'post', 加了这个属性，后，可以用myAxios来发请求，默认发的就是post请求,
  transformRequest: [function (data) {
    // 这里的参数data是用myAxios对象发请求时的data参数
    // 这个函数的返回值，就是要发给后端的最终的数据
    // return '小明' 
    // 我们要将data转换为formData的格式，再返回
    // data: {name: 1, age: 2} // name=1&age=2

    // let str = ''
    // for (let key in data) {
    //   // 请求的参数中多个&没有关系
    //   str += key + '=' + data[key] + '&' // name=1&,   name=1&age=2&
    // }
    
    let arr = []
    for (let key in data) {
      arr.push(key + '=' + data[key])  // ['name=1'], ['name=1', 'age=2']
    }
    return arr.join('&')  // 'name=1&age=2'
    // 以后如果希望发的请求的数据格式是formData格式，就使用 my-axios.js
    // 不需要引入axios这个包了
  }]
})

// 暴露这个对象
export default myAxios
```

## 过滤器filter

> 过滤器就是一个特定的方法,filters是固定的写法
>
> 在要显示的数据后加个|线和过滤器中的一个方法名，作用是相当于调用这个方法,并把|线左边的值传递给右边的方法 :  {{getAge(birthday)}}
>
> {{birthday | getAge}}
>
> filter中的this不是vue对象

**作用**

在数据被渲染之前，可以对其进行进一步处理，比如将字符反转，或者将小写统一转换为大写等!

HTML代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=<device-width>, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
  <h1>Vue的过滤器</h1>
  <div id="app">
    <!--第一种方式-->
	{{msg | my_filter}}
     <!--第二种方式-->
    {{msg | my_filter1}}
  </div>
</body>
</html>
```

JS代码

```javascript
<script>
  //第一种方式
  Vue.filter ('my_filter',function (val) {
  	return val+"----我说的，哈哈"
  })
  
  var vm = new Vue({
    el: '#app',
  	data: {name: 'xiaomin', count: 2, msg: '小明是坏蛋'},  	
	//第二种方式
	filters: {
		my_filter1: function (val) {
			return val + "-----我说的，咋滴";
		}
	}
  })
</script>
```

## 计算属性computed

> 如果想精确的控制一个数据的变化, 并在变化时做一些操作，则可以使用计算属性！ 
>
> 比如，想要在用户输入用户名时，实时的判断用户名是否符合要求
>
> computed的作用和filter相似，但是更强大一些!
>
> computed的属性可以是一个对象也可以是一个方法
>
> computed中的this是vue的对象

```html
<body>
  <h1>计算属性</h1>
  <div id="app">
    <input type="text" v-model="msg"> <br>
    {{myMsg}} <br>
    {{myname}} <br>
    <!--{{myname = 18}} <br>-->
    <input type="text" v-model="myname">
    <button>提交</button>
  </div>
</body>
</html>
<script src="./vue.js"></script>
<script>
  var vm = new Vue({
    el: '#app',
    data: function () {
      return {
        msg: '小明是坏蛋!!',
        name: '小明'
      }
    },
    computed: {
      // computed的作用和filter相似，但是更强大一些!
      // computed的属性可以是一个对象也可以是一个方法
      // 可以把这里的myMsg在dom当一个普通的数据来使用,不需要加()来调用
      // 方法的返回值就是myMsg的值 {{myMsg}}
      // 写成这个函数其实是Vue提供的简写, 这个函数其实是就get函数
      myMsg: function () {
        // 这里的this是vm
        return this.msg + '--小红说明'
      },
      // myMsg: {
      //   get: function () {
      //     return this.msg + '--小红说的'
      //   }
      // },
      // 当我们在页面使用这个myname值是，会自动调用这个get方法，返回值就是我们得到的值
      // 当修改这个myname的值时,set方法就会执行,set的参数就是这个设置的值
      myname: {
        get: function () {
          // console.log('get了')
          // return '小明'
          return this.name
        },
        set: function (val) {
          console.log(val)
          if (val.length > 6) {
            window.alert('太大了!,不能长于6')
            // this.myname = this.name
            return
          }
          this.name = val
          // console.log(val)
          // console.log('设置了')
        }
      }
    }
  })
</script>
```

##Vue的实例属性

* $refs	用了获取dom元素
* $el
* $emit和\$on

```html
// 操作dom
<body>
  <h1>Vue中的dom操作</h1>
  <div id="app">
    <!-- 1. 给要操作的dom元素添加ref属性，给一个值: 这个值不能重复-->
    <!-- vue在解析时，会自动获取有ref属性的dom元素,把这些元素赋值给实例的$refs对象-->
    <!-- vm.$refs.myipt 就是这个input标签-->
    <input ref="myipt" type="text" v-model="msg">
    <div>{{msg}}</div>
     <!-- vm.$refs.mybtn 就是这个button标签-->
    <button ref="mybtn">{{msg}}</button>
    <div class="a" ref="mydiv">
      <div>
        <div>
          <a href="" ref="mya"></a>
        </div>
      </div>
    </div>
  </div>
</body>
</html>
<script src="./vue.js"></script>
<script>
  var vm = new Vue({
    el: '#app',
    data: {
      msg: '哈哈'
    },
    // 要做dom操作的话
    // 通过把dom操作的代码在mounted中调用，或者在dom事件的回调函数中调用
    // Vue所控制的dom部分加载完成，之后Vue就会自动调用这个mounted方法!
    mounted: function () {
      // document.querySelector('input').focus()  // 可以
      // 这里的this也是vm
      // vm有个名为$refs的属性
      console.log(this.$refs)
      this.$refs.myipt.focus()
      // 如果通过dom操作修改的Vue中的模板，则修改部分的数据绑定就不生效了!
      this.$refs.mybtn.innerHTML = '我是中国人，我爱自己的祖国'
      console.log(this.$el)
    }
  })
</script>
```

## Vue的生命周期

从new Vue执行开始，Vue创建自己的实例，Vue的生命就开始

创建实例之后，Vue会用实例去代理data,methods,computed中的属性，但是在开始代理之前会调用beforeCreate方法

Vue在执行了beforeCreate方法之后，会对data,methods,computed中的属性进行代理，代理的代码执行完成之后，会自动调用created方法

Vue会读取el对应的元素的innerHTML, 然后Vue会解析这个字符串 ,会处理字符串中的{{}},指令等, 处理好之后，就会把字符串重新插入到el指定的元素的innerHTML中，但是,在插入之前，会调用一下beforeMount方法，在beforeMount方法执行时,解析后的html还没有插入到dom

执行完成beforeMount方法之后，就会将解析后的html插入到el指定的元素中也会帮我们把一些事件(点击事件等)注册好， 然后就会调用mounted方法

如果我们修改了data中的数据，Vue会重新解析模板,然后更新dom,但是在更新dom之前会先调用beforeUpdate方法

Vue在调用beforeUpdate之后会更新dom，更新dom之后,会调用这个updated方法，更新dom指的是，将data中的数据重新赋值到dom中

当$destroy执行时，Vue会终止数据绑定和取消事件,在终止和取消之前，Vue先调用一个beforeDestroy方法，此时数据绑定还没有终止

Vue终止数据绑定之后会调用destroyed方法，至此Vue的生命周期结束

Vue的实例中有个$destroy方法，这个方法调用的时候 Vue的生命周期就会终止，终止之后，页面在的数据绑定就不再生效，我们通过Vue注册的事件也会被Vue取消!

![Vue生命周期](E:\08-学习笔记\web前端\Images\Vue生命周期.png)

* beforeCreate  代理之前执行,此时 this.xx 无法操作data中的xx
* created       代理之后执行,此时this.xx 就是data中的xx
* beforeMount   Vue中的模板挂载到dom中之前, 此时无法操作el对应的dom
* mounted       Vue中的模板挂载到dom中之后, 此时可以操作el中的dom
* beforeUpdate  修改数据之后，dom更新之前.执行
* updated       修改数据之后，dom更新之后执行
* beforeDestroy 销毁之前执行(取消事件，取消数据绑定之前)
* destroyed     销毁之后执行

##组件

import form作用

1.告诉webpack要打包什么文件

2.告诉webpack要把什么文件中的变量，拿到当前文件这个作用域中!

```javascript
//es6暴露
export default obj
//es6引入包
import xx from '路径'    // 引入的是一个不完整的包，如果要引入完整的需要写完整的路径名

//CommonJS暴露
module.exports = obj
//CommonJS引入
require()
```

###模板

```html
<body>
  <h1>Vue中指定模板的几种写法</h1>
  <div id="app">
    <p>{{msg}}</p>
    <input type="text">
  </div>
  <template id="tmpl">
    <div class="嘻嘻">
      <h1>我是小白兔</h1>
      <h2>{{msg}}</h2>
    </div>
  </template>
</body>
</html>
<script src="./vue.js"></script>
<!-- 如果用script作为模板的话，要修改type属性，不能为text/javascript-->
<script id="xxa" type="text/html">
  <div class="xx">
    <h1>嘻嘻</h1>
  </div>
</script>
```

JavaScript代码

```javascript
/*情况1*/
//  var vm = new Vue({
//    el: '#app',
//    // 如果没有这个template属性, Vue会获取el指定的元素的innerHTML, 
//    // 解析完成之后，再插入到el指定的元素中去!
//    data: {
//      msg: '我是小明，我来啦001'
//    }
//  })
// var a = {}
// var c = {}

/*情况2*/
//  var vm = new Vue({
//    el: '#app',
//    // 如果写的template属性，那么Vue会将template对应的模板把el指定的元素替换掉!
//    template: '<h1>{{msg}}</h1>',
//    data: {
//      msg: '我是小明，我来啦002'
//    }
//  })

/*情况3*/
//  var vm = new Vue({
//    el: '#app',
//   //  如果写的template属性，那么Vue会将template对应的模板把el指定的元素替换掉!
//   // template属性的值也可以是一个选择器，Vue会读取所以template标签与这个选择器匹配的元素
//   // vue会根据这个选择找到id为tmpl的template标签,或者script,把找到的标签的innerHTML
//   // 替换el指定的元素
//    template: '#xxa',
//    data: {
//      msg: '我是小明，我来啦003'
//    }
//  })

/*情况4*/
 var vm = new Vue({
   el: '#app',
  // 如果有render方法，Vue会自动调用这个方法,Vue会把这个方法的返回值得到
  // 根据这个返回值生成对应的html，用来替换el指定的元素
  // template: '<h1>我是小明明</h1>'
   render: function (createElement) {
    //  return createElement('div', '我是小明明')
    // 相当于会创建一个h1标签，内容为: 我是小明明,创建后会用这个标签替换el指定的元素
     return createElement('h1', '我是小明明')
   },
   data: {
     msg: '我是小明，我来啦003'
   }
 })
</script>
```

### 全局组件

通过Vue.component创建的组件叫作全局组件!

参数1: 组件的名字,用小写英文, 组件的名字，在别的模板中可以以标签的形式存在:`<mybtn></mybnt>`

参数2: 是组件中使用到的在一些方法，数据，html等，这个对象的属性与new Vue传入的对象属性相似

**组件中的data属性必须是一个方法，不能是一个对象**，所有的template属性底层都变为了render方法

全局组件可以在任何组件的模板中直接使用，不用使用components起别名

```javascript
<script>
Vue.component('组件名', {
  template: '',
  data: function () {
      return {
        count: 0
      }
  },
  methods: {
      add: function () {
        this.count++
      }
    }
})

var vm = new Vue({
    el: '#app',
    data: { name:'小明', count: 2}
  })
</script>
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
  <h1>Vue中的组件</h1>
  <div id="app">
    <p>{{name}}</p>
    <mybtn></mybtn>
    <mybtn></mybtn>
    <mybtn></mybtn>
    <mybtn></mybtn>
    <mybtn></mybtn>
    <mybtn></mybtn>
    <mybtn></mybtn>
  </div>
</body>
</html>
<script src="./vue.js"></script>
<script>
  // 如果在Vue中以组件的方式写代码
  // 如果在Vue中定义一个组件
  // 方式1: Vue.component方法
  // 用组件写一个button按钮
  // 参数1: 组件的名字,用小写英文, 组件的名字，在别的模板
  // ，可以以标签的形式存在:<mybtn></mybnt>
  // 参数2: 是组件中使用到的在一些方法，数据，html等
  // 这个对象的属性与new Vue传入的对象属性相似
  // 全局组件的写法
  Vue.component('mybtn', {
    // 这个template字符串中也能用{{}},也能用指令，和之前的用法一致
    // 这个template字符串中使用的数据和方法都要到自己这个组件对象中去取
    // 就是到自己的data中自己的methods中取
    template: '<div><button @click="add">我是Btn: {{count}}</button></div>',
    // data必须是一个方法
    data: function () {
      return {
        count: 0
      }
    },
    methods: {
      add: function () {
        this.count++
      }
    }
    // 这里也可以有过滤器，计算属性等
    // 只不这些过滤器，计算属及methods中的方法，及data中的数据，*只能* 在自己这个
    // 组件的template中使用，不能使用别人的，别人也不能使用这里的
  })

  var vm = new Vue({
    el: '#app',
    data: { name:'小明', count: 2}
  })
</script>
```

### 局部组件

> 包含了template的对象就称之为局部组件
>
> 局部组件只能在引入这个组件的组件的模板中使用(也要配置别名)

```javascript
<script>
var Menu = {
    template: '<div>我是菜单: {{msg}}</div>',
    data: function () {			//在模板中写了data必须return一个对象，不能写成data: {msg: '我是菜单'}
      return {
        msg: '我是菜单'
      }
    }
  }
var vm = new Vue({
    el: '#app',
    // 给组件起别名!, 起了别名后，这个组件就可以
    // 以标签的形式写在当前组件或者vue实例的模板中
    components: {
      // 别名: 组件对象
      mnu: Menu
    }
})
</script>
```

**使用实例**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
  <h1>Vue组件</h1>
  <div id="app">
    <div>我是中国人</div>
    <btn></btn>
    <mnu></mnu>
  </div>
</body>
</html>
<!-- 局部组件-->
<!-- 一个包含了template属性的js对象就可以称之为组件-->
<!-- 组件的目的是为了把页面分面多份-->

<script src="./vue.js"></script>
<script>
  // 如果一个变量表示组件，推荐用大写
  // 定义一个组件
  // 每个组件都会有template,data,methods,computed,filters,及生命周期函数
  // 如果想让组件生效，就要把组件以【标签】的形式写在其他组件或者vue实例的模板中
  var Button = {
    // template称之为模板
    template: '<div>我是小明: {{msg}}</div>',
    data: function () {
      return {msg:'你好'}
    }
  }
  var Menu = {
    template: '<div>我是菜单: {{msg}}</div>',
    data: function () {
      return {
        msg: '我是菜单'
      }
    }
  }

  var vm = new Vue({
    el: '#app',
    // 给组件起别名!, 起了别名后，这个组件就可以以
    // 标签的形式写在当前组件或者vue实例的模板中
    components: {
      // 别名: 组件对象
      btn: Button,
      mnu: Menu
    }
  })
</script>
```

### 单文件组件

本质上是局部组件，只是换了一种写法！

组件中`<style></style>`的样式默认是全局的，如果希望样式只对当前组件有效果，需要给style标签添加一个scoped属性

```javascript
<template>
<div>
  我是中国人，我爱自己的祖国!!!!!!!
  {{ 1 + 1}}
</div>
</template>
<script>
// 单文件组件仅仅是局部组件的特殊写法
var Button = {
  // template: '<div>', 单文件组件不需要在对象中写template属性，
  // 把模板写到template标签中
  data: function () {
    return {}
  }
}
// 必需有下面这一句, 前两个单词是固定的
export default Button
</script>
<style>
/*这里专门用来写样式
Vue最终会把它插入到dom中的style标签!*/
/*webpack，可以将这个文件中的代码转换为一个js对象的形式,
也就是上边局部组件的写法*/
</style>
```

**在一个组件中使用另一个组件**

1.引入xx.vue这个组件import xx from './xx.vue' 

 2.给这个组件起个别名，任何地址使用这个组件都需要别名components: {xx:xxx}

3.以自定义标签的方式使用组件

```html
<script>
//  如果我想在btn组件中使用Swiper.vue这个组件
// 1.引入这个组件Swiper.vue
// 2.是给这个组件起个别！，任何地址使用这个组件都需要别名
import Swiper from './Swiper.vue'
var Btn = {
  // template:
  data: function () {
    return {msg:'嘻嘻'}
  },
  components: {
    Swiper: Swiper
  }
}
// 暴露这个Btn组件
export default Btn
</script>
```

####单文件组件的使用

index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
  <div id="app"></div>
  <--webpack打包后的js文件-->
  <script src="./dist/hebing.js"></script>
</body>
</html>
```

main.js

```javascript
// 引入Vue
import Vue from 'vue'
import App from './App.vue'
// 创建Vue的实例
var vm = new Vue({
  // 指定模板
  render: function (createElement) {
    // return createElement(组件对象)
    // App组件中的模板最终会替换index.html中#app这个元素
    return createElement(App)
  }
})
.$mount('#app')
```

App.vue

```vue
<template>
<!-- template 标签中只能有一个直接子元素-->
<div>
  <h1>我是App.vue组件</h1>
  <son></son>
</div>
</template>
<script>
  export default {
  components: {
    // 给组件起别名
    // 左边是名字，右边是组件对象
    Son: Son
  },
  data: function () {
    return {
     msg: 'Hello World'
    }
  }
}
</script>
<style>
</style>
```

son.vue

```vue
<template>
<div>
<h1>我是son标签</h1>
</div>
</template>
<script>
</script>
<style>
</style>
```

MVVM: Model(数据，及操作数据的方式...)，View(界面)，ViewModel(中间的桥梁)

##组件之间的数据传递

###props

> 通过props可以让子组件在被其他组件使用时，接收到到其他组件写在子组件标签上的属性值!
>
> 父子组件传值：**父往子传**
>
> 父组件在使用子组件时，在子组件的标签上写数据，然后，子组件中通过props来表示接收哪些数据

main.js		下边所有的mian.js入口文件都一样

```javascript
// 引入Vue
import Vue from 'vue'
import App from './App.vue'
// 创建Vue构造函数的实例
var vm = new Vue({
  render: function (createElement) {
    // return createElement(组件对象)
    // App组件中的模板最终会替换index.html中#app这个元素
    return createElement(App)
  }
})
.$mount('#app')
```

App.vue

```vue
<template>
<!-- template 标签中只能有一个直接子元素-->
<div>
  <h1>我是App.vue组件</h1>
  // 如果希望给Son.vue组件传递数据，可以直接将数据写在这个标签的属性上
  // 并不是写了就一定能传过去的, 需要在【特定的情况】下才能传递过去
  // 这些属性是传递给了Son.vue组件的实例对象
  
  // webpack进行打包时，是把这段内容变成了js代码
  // if Son组件中poprs数组中如果包含了这些属性
  // jiu  Son实例.name = '你好',  Son实例.sex = ' 我是小男孩 '
  <Son 
  name="小明"
  :age="age"
  :msg="msg"
  </Son>
</div>

</template>
<script>
import Son from './Son.vue'
export default {
  components: {
    // 给组件起别名
    // 左边是名字，右边是组件对象
    Son: Son
  },
  data: function () {
    return {
      // 把这个App.vue中的age，msg传递到Son组件中模板中呈现
      age: '18',
      msg: {a:'hello', b:'world'}
    }
  }
}
</script>
<style>
</style>
```

son.vue

```vue
<template>
<div>
<h1>我是son标签</h1>
</div>
</template>
<script>
export default {
  // 这里的props属性就是【特定的情况】
  // 中有Son标签中的属性名被包含在props数组中,
  // Vue才会将标签的属性值传递过来
  // 传递过来之后，会给Son的实例一个同名属性
  props:['name','age','msg'],
  created: function () {
    console.log(this.name)
    console.log(this.age)
    console.log(this.msg)
  }
}
</script>
<style>
</style>
```

###customEvent

> 什么是自定义事件?
>
> 也提供一个事件名，和一个事件的回调函数!, 通过代码来让事件触发
>
> 自定义事件的本质就是让一个名字和方法关联起来了

> 什么是事件? 发生的事情 v-on:click  @click   @xxxx
>
> 通过自定义事件，也可以实现两个组件之间数据的传递, 可以通过 `v-on:` 指令来给某个组件自定义事件
>
> **子给父传**		在父组件中使用子组件时,在子组件标签上给子组件定义了自定义事件

**封装自定义事件**

```javascript
var obj = {
  // 自定义了一个事件叫test
  test: function () {
    console.log('你好')
  }
}
// 封装一个用于定义事件的函数
// 事件要用名字和函数
function on(eventName, callback) {
  //obj.eventName
  obj[eventName] = callback
}

// obj.test() // 触发了一个事件
// 封装一个触发事件的方法
function emit(eventName) {
  //obj[]
  var fn = obj[eventName]
  fn()
}
// 定义事件
on('chifan', function () {aler("吃饭了")})

// 触发事件
emit('chifan')
```

**通过自定义事件传值**

App.vue

```vue
<template>
<!-- template 标签中只能有一个直接子元素-->
<div>
  <h1>我是App.vue组件</h1>
  <button @click="clickHandler">测试</button>
  // 如果这里写了Son，说明App.vue和Son.vue是父子关系
  // 给Son标签添加自定义事件，和添加普通事件的方式一样
  // 这里就是定义了一个事件 叫 helloWorld, 当事件触发时就会执行这个helloHandler函数
  // 如何触发这个自定义事件: 必需用Son标签对应的组件的实例来触发，Son实例.$emit(事件名)
  // 其实是 Son实例.$emit 方法内部会调用这里的事件的回调函数!
  <Son @helloWorld="helloHandler"></Son>
</div>
</template>
<script>
import Son from './Son.vue'
export default {
  // 给组件起别名
  components: {
    Son: Son
    // XX
  },
  methods: {
    clickHandler: function () {
      window.alert('嘻嘻，点我了')
    },
    helloHandler: function (arg) {
      console.log(arg)
      console.log('你点我了')
    }
  }
}
</script>
<style>
</style>
```

son.vue

```vue
<template>
<div>
<h1>我是Son.vue组件</h1>
<button @click="myclick">测试触发事件</button>
</div>
</template>
<script>
export default {
  // 这个Son组件的实例才能触发定义在Son标签上的自定义事件!
  created: function () {
    // 这样就会触发，helloWorld这个自定义事件
    // $emit第一个参数: 是要触发的事件名
    // $emit第二个参数: 会被作为参数传递给事件对应的回调函数!(参数可以是任意的数据类型)
    // 只要能用这个this实例，就可以调用这个$emit//,在created执行之后就可以调用$emit方法了
    this.$emit('helloWorld', {a:'你好'})
  },
  mounted: function () {
    this.$emit('helloWorld', 96998)
  },
  methods: {
    myclick: function () {
      this.$emit('helloWorld', '我是小男孩!!')
    }
  }
}
</script>
<style>
</style>
```

###$refs

> 子组件实例.xxx = 998 传值给子组件
>
> console.log(子组件实例.name) // 从子组件中获取数据
>
> 子组件实例.hello('你了吗')  // 即是传给子组件，也是获取数据, 传的是参数，获取的是方法的返回值!
>
> 使用$refs是可以得到子组件的实例!只要能得到一个对象，这个对象的属性方法，可以任意调用

App.vue

```vue
<template>
<!-- template 标签中只能有一个直接子元素-->
<div>
  <h1>我是App.vue组件</h1>
  <button ref="btn">测试</button>
  <Son ref="myson"></Son>
</div>
</template>
<script>
import Son from './Son.vue'
export default {
  components: {
    Son: Son
  },
  mounted: function () {
    console.dir(this.$refs.btn)
    // 这里的this.$refs.myson就是son组件的实例!
    console.log(this.$refs.myson)
    var that = this
    // 两秒后修改Son组件中的数据
    setTimeout(function () {
      // that.$refs.myson.msg = 998
      // myson是Son组件的实例
      var a = that.$refs.myson.changeMsg('你好吗')
      window.alert(a)
    }, 2000)
  }
}
</script>
<style>
</style>
```

son.vue

```vue
<template>
<div>
<h1>我是Son.vue组件</h1>
{{msg}}
</div>
</template>
<script>
// Vue底层会将这个对象作为一个参数，来创建一个实例，这个对象写在组件中，这个创建的实例就是组件的实例!
// 每个组件都会有自己的实例，都不一样!
export default {
  data: function () {
    return {
      msg: '我是中国人，我爱自己的Son.vue',
      name: '我是这个组件Son.vue'
    }
  },
  methods: {
    changeMsg: function (arg) {
      // 这里的this就是组件的实例
      // 是组件的实例的原因，是因为
      // 这个函数是组件的实例调用
      this.msg = Math.random() + arg
      return '处理好了'
    }
  }
}
// window.xx = obj.methods.changeMsg
// winow.xx() // window
</script>
<style>
</style>
```

###busEvent

> Vue new 出的实例有两个特殊的方法, 一个是`$on`,一个是`$emit`
>
> BusEvent, 是vue中兄弟组件（任何组件之间，无论是不是父子关系）之间传递数据的方式

bus.js

```javascript
// 不管别的文件中有没有引入vue
// 只要我这里使用了，就必须再引入一次
import Vue from 'vue'
var bus = new Vue()

// 暴露出bus, 让别的文件引入
// 把它写在一个文件中暴露的好处是: 任何文件中引入这个文件得到的bus对象是同一个对象!
export default bus
```

App.vue

```vue
<template>
<!-- template 标签中只能有一个直接子元素-->
<div>
  <h1>我是App.vue组件</h1>
  <MyA></MyA>
  <MyB></MyB>
</div>
</template>
<script>
// 要引入
import A from './A.vue'
import B from './B.vue'
export default {
  components: {
    MyA: A,
    MyB: B
  }
}
</script>
<style>
</style>
```

A.vue

```vue
<template>
<div>
<h1>我是A组件</h1>
</div>
</template>
<script>
// 引入bus(其实就是Vue的实例)
import bus from './bus.js'
export default {
  created: function () {
    // 这里要用bus注册事件
    bus.$on('chifan', function (arg) {
      window.alert(arg)
    })
  }
}
</script>
<style>
</style>
```

B.vue

```vue
<template>
<div>
<h1>我是B组件</h1>
<button @click="test">测试按钮</button>
</div>
</template>
<script>
// 这里引入得到bus和A.vue中引入得到的bus是同一个对象
import bus from './bus.js'
export default {
  methods: {
    test: function () {
      bus.$emit('chifan', '数据来了')
    }
  }
}
</script>
<style>
</style>
```

###vuex

Vuex 是一个专为 Vue.js 应用程序开发的**状态管理模式**。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。

Vuex相当于是提供了一个全局对象，任何组件中都可以对这个对象进行赋值和读取，只不过在赋值时并不是简单的使用=号赋值,需要使用Vuex提供的方式进行赋值!

#### 开始使用

#####创建store对象

1.下载Vuex,并引入, 可以通过npm下载 `npm install vuex --save`

引入下载好的包，得到Vuex对象,创建Vuex.Store的实例对象

```javascript
// store.js
import Vuex from 'vuex'
import Vue from 'vue'
Vue.use(Vuex) // 注意需要调用Vue.use
// 使用Vuex时，将数据统一保存到这个store对象中，在各个组件中引入这个对象就可以直接获取和设置store中的数据了
// 不过在store对象中获取和设置数据时，必须要【按照Vuex指定的方式进行】!
const store = new Vuex.Store({})
export default store
```

#####state属性

> 在Vuex中约定，要将共享的数据保存在state属性中，state也是一个对象

```javascript
// store.js
import Vuex from 'vuex'
import Vue from 'vue'
Vue.use(Vuex) // 注意需要调用Vue.use

const store = new Vuex.Store({
  // 约定将数据都保存到这个state对象中,这个state就是状态
  // 组件中需要使用的state的属性都提前在这里申明好
  state: {
    // 这里随便添加什么数据都可以
    count: 0,
    name: '小明',
    age: 18
  }
})

export default store
```

> 在组件中使用store对象中的状态:
>
> 在组件中使用store对象中的状态时，需要记得引入store `import store from './store.js'`
>
> 注意不可以直接在组件的模板中使用store.state中的数据
>
> 要在组件的模板中使用store中的状态的话，需要在方法或者计算属性中返回状态才可以使用 `foo () {return store.state.age}`

```vue
// App.vue
<template>
  <div class="app">
    <h1>{{count}}</h1>
    <h1>{{name}}</h1>
    <h1>{{age}}</h1>
  </div>
</template>
<script>
  // 需要引入才可以得到这个store
  import store from './store.js'
  export default {
    computed: {
      // 需要在计算属性中返回，才可以使用state中的数据
      count () {
        return store.state.count
      },
      name () {
        return store.state.name
      },
      age () {
        return store.state.age
      }
    }
  }
</script>
<style>
 .app {
   background: #f00;
   color: #fff;
  }
</style>
```

[点击查看示例](https://www.webpackbin.com/bins/-KqqgVvFCTAGzaFn5mH0)

#####为子组件注入store对象

> 如果在每个组件中使用store时，需要需要引入一次的话，就太麻烦了，Vue允许开发将store对象注入到每个组件的实例中去

> 在初始化Vue实例时，添加一个store属性，值为Vuex.Store创建出的store对象。
> 添加之后，Vue会将store对象赋值给所有子组件的实例的this.$store属性。
>
> 这样一来，在其他组件中使用store时就不需要引入store对象了，可以直接通过this.$store来得到store对象。

注入store对象

```javascript
// main.js
import Vue from 'vue'
import App from './App.vue'
import store from './store.js'
const app = new Vue({
  el: '#app',
  // 将sotre添加到这里之后，这个store对象就会被传递给所有子组件的实例
  // 在子组件中就可以不需要引入store.js,而直接使用this.$store来得到这个store对象了
  store, 
  render: h => h(App)
})
```

> 注入后，不需要引入store就可以直接通过this.$store使用store对象了

```vue
// App.vue
<template>
  <div class="app">
    <h1>{{count}}</h1>
    <h1>{{name}}</h1>
    <h1>{{age}}</h1>
  </div>
</template>
<script>
  export default {
    computed: {
      // 需要在计算属性中返回，才可以使用state中的数据
      count () {
        // 不需要引入store对象了，因为已经在创建Vue实例里注入了store对象
        return this.$store.state.count
      },
      name () {
        return this.$store.state.name
      },
      age () {
        return this.$store.state.age
      }
    }
  }
</script>
<style>
 .app {
   background: #f00;
   color: #fff;
  }
</style>
```

 [示例](https://www.webpackbin.com/bins/-Kqqj754AUJ9uRRxtQPR)

#####mutations属性

> 在Vuex，约定不允许直接修改store中的状态

```javascript
// *.vue
methods: {
  click () {
    store.state.age = 998 // 这样的代码是不允许的
  }
}
```

> 必须要通过store对象中的mutations属性中的方法来修改store中的状态!

```javascript
// store.js
import Vuex from 'vuex'
import Vue from 'vue'
Vue.use(Vuex)

const store = new Vuex.Store({
  state: {
    count: 0,
    name: '小明',
    age: 18
  },
  // 所有用来修改状态(state)的方法，都必须申明在mutations中
  // 不允通过mutations之外的方式来修改状态
  mutations: {
    // 自定义方法increment
    // 作用: 让count的值加1
    // 参数1: 就是上面的state
    increment (state) {
      state.count++
    },
    // 自定义方法changeName
    // 作用: 根据将传来的参数重新赋值给name
    // 参数1: 就是上面的state
    // 参数2: 是changeName被调用时传递过来的参数!
    changeName (state, arg) {
      state.name = arg
    }
  }
})

export default store
```

> 那么如何调用mutations中的方法呢?
>
> 通过store.commit('mutations中的方法名'), 就可以调用mutations中的方法了!

```vue
// App.vue
<template>
  <div class="app">
    <h1>{{count}}</h1>
    <h1>{{name}}</h1>
    <h1>{{age}}</h1>
    <button @click="clickHandler">修改store中的状态</button>
  </div>
</template>
<script>
  export default {
    computed: {
      // 需要在计算属性中返回，才可以使用state中的数据
      count () {
        // 不需要引入store对象了，因为已经在创建Vue实例里注入了store对象
        return this.$store.state.count
      },
      name () {
        return this.$store.state.name
      },
      age () {
        return this.$store.state.age
      }
    },
    methods: {
      clickHandler () {
        // 这里store.commit('increment')的作用是调用
        // mutations中的increment方法！
        // 参数1是要调用的方法名,
        this.$store.commit('increment')
        
        // 这里store.commit('increment')的作用是调用
        // mutations中的changeName方法!,
        // 参数1是要调用的方法名，
        // 参数2是要传递给changeName方法的参数
        this.$store.commit('changeName', '我是小明' + Math.random())
      }
    }
  }
</script>
<style>
 .app {
   background: #f00;
   color: #fff;
  }
</style>
```

[示例](https://www.webpackbin.com/bins/-KqqlWIrockR4otX_ywD)

> 还有一种调用mutations中方法的方式:

```javascript
// 传递一个对象(事件对象都会传递给mutations中的方法)，通过type属性指定要调用的方法
store.commit({
  // type属性名是固定的
  type: 'increment',
  // 可以任意添加添加属性
  age: 99
})
```

```javascript
const store = new Vuex.store({
  state: {
    count: 0
  },
  mutations: {
    increment (state, obj) {
      state.count += obj.age
    }
  }
})
```

*注意, 不允许在mutations的方法中使用异步代码!,*

#####getter

> 如果store中的state中有个数组users，我想获取users中所有成员的数据，代码可以这么写!

store.js

```javascript
// store.js
import Vuex from 'vuex'
import Vue from 'vue'
Vue.use(Vuex)

const store = new Vuex.Store({
  // 约定将数据都保存到这个state对象中,这个state就是状态
  state: {
    users: [
      {
        name: '何小明', age: 19, sex: 0
      },
      {
        name: '刘大河', age: 21, sex: 1
      },
      {
        name: '韩梅梅', age: 11, sex: 1
      },
      {
        name: '韩梅梅', age: 19, sex: 0
      }]
  }
})

export default store
```

> 下载是要使用store中数据的组件

```vue
// App.vue
<template>
  <div class="app">
    <ul>
      <li v-for="item in getAdults">
      {{item.name}}
      </li>
    </ul>
  </div>
</template>
<script>
  export default {
    computed: {
      // 从state.users中获取所有成年人
      // 直接写代码对users进行过滤就可以了!
      getAdults () {
        const users = this.$store.state.users
        const tmpArr = []
        for (let i = 0; i < users.length; i++) {
          const item = users[i]
          if (item.age >= 18) {
            tmpArr.push(item)
          }
        }
        return tmpArr
      }
    }
  }
</script>
<style>
 .app {
   background: #f00;
   color: #fff;
  }
</style>
```

> 上述案例中，如果在多个组件中都想从store.state.users中获取成年人的数据, 可以将代码封装到Vuex提供到getter中

如下:

```js
import Vuex from 'vuex'
import Vue from 'vue'
Vue.use(Vuex)

const store = new Vuex.Store({
  // 约定将数据都保存到这个state对象中,这个state就是状态
  state: {
    users: [
      {
        name: '何小明', age: 19, sex: 0
      },
      {
        name: '刘大河', age: 21, sex: 1
      },
      {
        name: '韩梅梅', age: 11, sex: 1
      },
      {
        name: '韩梅梅', age: 19, sex: 0
      }]
  },
  // 如果多个组件都需要对state中的数据做相同的操作，则可以将这些操作封装到getters中,
  // 然后在需要使用这个方法的组件中直接调用这个封装的方法就可以了! 
  // 调用getters.getAdultsstore时不要加括号: store.getters.getAdults
  getters: {
    // 自定义的方法getAdults
    // 从state.users中获取所有成年人
    getAdults: state => {
      const users = state.users
      const tmpArr = []
      for (let i = 0; i < users.length; i++) {
        const item = users[i]
        if (item.age >= 18) {
          tmpArr.push(item)
        }
      }
      return tmpArr
    }
  }
})

export default store
```

```vue
// App.vue
<template>
  <div class="app">
    <ul>
      <li v-for="item in getAdults">
      {{item.name}}
      </li>
    </ul>
  </div>
</template>
<script>
  export default {
    computed: {
      // 从state.users中获取所有成年人
      getAdults () {
        return this.$store.getters.getAdults
      }
    }
  }
</script>
<style>
 .app {
   background: #f00;
   color: #fff;
  }
</style>
```

[点击查看示例](https://www.webpackbin.com/bins/-Kqr-OAHAUwVgDhDWFe6)

#####actions

> actions和mutations作用相似，只是vuex希望开发者不要在mutations中写异步代码，如果非要写异步代码，则使用actions,然后再在actions中调用mutations

```javascript
// store.js
import Vuex from 'vuex'
import Vue from 'vue'
Vue.use(Vuex)

const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    },
    changeName (state, arg) {
      state.name = arg
    }
  },
  // 如果有异步的代码就，就使用actions，然后在actions中调用mutations
  actions: {
    // 参数context是一个与store有相同属性的对象,就把它当store一样使用
    increment (context) {
      // 异步的代码
      setTimeout(() => {
        // 调用mutations中的increment方法
        context.commit('increment')
      }, 1000)
    }
  }
})

export default store
```

> 调用actions中的方法: `store.dispatch('actions方法名')`, dispatch参数的形式和commit参数的形式一致

```vue
<template>
  <div class="app">
    <h1>{{count}}</h1>
    <button @click="clickHandler">异步的修改store中的状态</button>
  </div>
</template>
<script>
  export default {
    computed: {
      count () {
        return this.$store.state.count
      }
    },
    methods: {
      clickHandler () {
        // 调用actions中的increment方法
        this.$store.dispatch('increment')
      }
    }
  }
</script>
<style>
 .app {
   background: #f00;
   color: #fff;
  }
</style>
```

[点击查看示例](https://www.webpackbin.com/bins/-Kqrc-ZVET1MN5s36Wu-)

#####modules

> 由于使用单一状态树，应用的所有状态会集中到一个比较大的对象。当应用变得非常复杂时，store 对象就有可能变得相当臃肿。

> 为了解决以上问题，Vuex 允许我们将 store 分割成模块（module）。每个模块拥有自己的 state、mutation、action、getter、甚至是嵌套子模块——从上至下进行同样方式的分割：



```javascript
const moduleA = {
  state: { ... },
  mutations: { ... },
  actions: { ... },
  getters: { ... }
}

//
const moduleB = {
  state: { ... },
  mutations: { ... },
  actions: { ... }
}
              
//
const store = new Vuex.Store({
  modules: {
    a: moduleA,
    b: moduleB
  }
})
store.state.a // 得到moduleA中的状态
store.state.b // 得到moduleB中的状态
// 在调用moduleA或者moduleB中的mutations,actions,getters时和之前的调用方式一致，所以注意不要重名。
```

[点击查看示例](https://www.webpackbin.com/bins/-KqtbgVlCffHEc7vPcB9)

#####注意

> 使用 Vuex 并不意味着你需要将**所有的**状态放入 Vuex。虽然将所有的状态放到 Vuex 会使状态变化更显式和易调试，但也会使代码变得冗长和不直观。如果有些状态严格属于单个组件，最好还是作为组件的局部状态。你应该根据你的应用开发需要进行权衡和确定。

#####简写

> 使用state中数据时的简写

```javascript
import { mapState } from 'vuex'

export default {
  // ...
  computed: mapState({
    // 箭头函数可使代码更简练
    count: state => state.count,

    // 传字符串参数 'count' 等同于 `state => state.count`
    countAlias: 'count',

    // 为了能够使用 `this` 获取局部状态，必须使用常规函数
    countPlusLocalState (state) {
      return state.count + this.localCount
    }
  })
}
```

> 或者:

```javascript
computed: mapState([
  // 映射 this.count 为 store.state.count
  'count'
])
```

> 或者合适对象展开运行符

```javascript
computed: {
  localComputed () { /* ... */ },
  // 使用对象展开运算符将此对象混入到外部对象中
  ...mapState({
    // ...
  })
}
```



> 使用getters中方法的简写

```javascript
import { mapGetters } from 'vuex'

export default {
  // ...
  computed: {
  // 使用对象展开运算符将 getters 混入 computed 对象中
    ...mapGetters([
      'doneTodosCount',
      'anotherGetter',
      // ...
    ])
  }
}
```

> 调用mtations中方法的简写

```javascript
import { mapMutations } from 'vuex'

export default {
  // ...
  methods: {
    ...mapMutations([
      'increment' // 映射 this.increment() 为 this.$store.commit('increment')
    ]),
    ...mapMutations({
      add: 'increment' // 映射 this.add() 为 this.$store.commit('increment')
    })
  }
}
```

####使用实例

store.js

```javascript
// 1.引入Vuex
// 2.使用Vue.use方法
// 3.创建一个对象用来存储需要共享的数据
// 4.暴露3.中创建的对象

// 1.
import Vuex from 'vuex'
import Vue from 'vue'
// 2.
Vue.use(Vuex)
// 3.
// 数据保存到这个对象中
const store = new Vuex.Store({
  // 状态,Vuex中把要在多个组件中共享的数据叫状态
  state: {
    // 这里的数据随便添加,所有需要使用的属性，需要在这里【先定义好!】
    name: '小明',
    age: 18,
    sex: '小女孩'
  },
  // 约定: 所有要修改状态(state)中代码，都写在这个mutations中！
  mutations: {
    // 第二个参数，是调用store.commmit的第二个参数!
    changeName (state, arg) {
      if (!arg) {
      state.name = '小明损失了' + Math.random()
      }else {
      state.name = arg
      }
    },
    // 这里写自定义方法
    // 假如我要修改name,就必需定义一个方法来修改name
    // 这里每个方法，都会有这个形state，就是上面的state
    // 在需要让age值+1的地方，调用changeAge方法
    // store.commit('changeAge') 来调用changeAge方法
    changeAge (state) {
      // 约定，这里mutations中不可以写异步的代码!
      // 不要这么去写代码，这样会导致在使用Vue的调试工具
      // 看不到最终的数据
      // setTimeout(function () {
      //   console.log(11)
      //   state.age +=1
      // }, 1000)

      state.age +=1
      console.log('我执行了吗!')
      
    }
    
  },
  // actions和mutations中的作用是一致，都是为了修改状态!
  // actions中可以写异步代码，mutations中不可以写异步代码
  // actions中会调用mustations中的方法
  actions: {
    // 希望当前我点击按钮后，1秒后年龄+1
    // 这里的context其实也是上面store
    changeAge (context) {
      setTimeout(function () {
        // 调用mutations中的的方法
        // commit 方法会到mutations中找有没有和'changeAge'同名的方法，有的话，就调用
        context.commit('changeAge')
      }, 1000)
    }
  }
})
// 4. 
// 在其他组件中，引入这个store,就可以使用这里的数据
// store.state.name 
export default store
```

App.vue使用定义的数据

```vue
<template>
<div>
  <h1>我是中国人，我爱自己的祖国!</h1>
  <p>{{myname}}</p>
  <p>{{myage}}</p>
  <hr>
  <TestStore></TestStore>
</div>
</template>
<script>
import TestStore from './TestStore.vue'
//  如果要使用store.js中的数据，就引入这个js
import store from './store.js'
// store.state.name  '小明'
// store.state.age    18
// store.state.sex    '小女孩'
export default {
  components: {
    TestStore
  },
  data () {
    return {
      tmp: '嘻嘻',
      // test: store.state.name
    }
  },
  computed: {
    // 在store中的数据推荐通过组件的计算属性来在模板中使用!
    myname () {
      return store.state.name
    },
    myage () {
      return store.state.age
    }
  },
  filters: {}
}
</script>
<style>
</style>
```

main.js

```javascript
import Vue from 'vue'
import App from './App.vue'
import store from  './store.js'

new Vue({
  el: '#app',
  // 这里添加一个store属性，值为申明的store
  // 好处是，添加之后Vuex会将这个store对象
  // 赋值给每个组件的实例对象的$store属性
  // 组件实例.$store = store
  // 这样我们在组件中使用的时候就不需要引入store.js了
  // 直接使用实例的$store属性
  store,
  // router, this.$router
  render: function (createElement) {
    return createElement(App)
  },
  // handler
  render: h => h(App)
})
```

TestStroe.vue	使用数据

```vue
<template>
<div>
  <p>{{msg}}</p>
  {{myage}} <br>
  {{myname}}
  <button @click="addOne">值+1</button>
  <button @click="addOneSync">异步的值+1</button>
</div>
</template>
<script>
// 在Vue实例中加入store属性之后在组件中就不用引入store.js了
// import store from './store.js'
export default {
  data () {
    return {
      msg: '我是中国人，我爱自己的祖国!'
    }
  },
  computed: {
    myname () {
      // 返回一个名字
      return this.$store.state.name
    },
    myage () {
      // this.$store
      // return store.state.name
      // 这里不再需要引入script标签了，直接使用组件对象的$store
      return this.$store.state.age
    }
  },
  methods: {
    addOneSync () {
      // 这里的dispatch是为了调用actions中的方法的
      this.$store.dispatch('changeAge')
    },
    addOne () {
      // 下面的代码是可以的，但是不要这么使用
      // 在Vuex中如果要修改状态，必需通过vuex提供的方式来修改
      // this.$store.state.age +=1 // 有没有效果
      // 这里传递一个方法名,vuex会自动调用mutations
      // 中对应的方式
      this.$store.commit('changeAge')
      this.$store.commit('changeName', '新名字'+ Math.random())
      // this.$store.commit('changeName')
      // 希望这里还可以修改名字!
      // setTimeout( () => {
      // this.$store.commit('changeAge')

      // })
    }
  }
}
</script>
<style>
</style>
```

####getter使用实例

store.js

```javascript
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

const store = new Vuex.Store({
  state: {
    users: [
      {
        // 27天
        name: '小兔子', age: 27, sex: -1
      },
      {name: '小猫子', age: 17, sex: -12},
        
      {name: '小狗子', age: 20, sex: -1},
        
      {name: '小鼠子', age: 21, sex: -1},
        
      {name: '小田子', age: 16, sex: -11}
    ]
  },
  // mutations
  // getters中定义方法来对state的数据进行处理(不修改state中的数据)
  getters: {
    // 获取成年动物的方法, 如果有组件想获取成年的动物就直接调用这个方法，
    // 就需要在自己组件中再定义方法进行处理了!

    // 如果要调用这个getAdultAnimal方法，就直接使用store.getters.getAdultAnimal就可以
    // 调用时不需要添加括号,就可以调用这个方法
    getAdultAnimal (state) {
      // 如果多个组件中都写了下面的代码，是对状态进行处理的(不修改状态的值的!)
      // 此时推荐将我们的获取状态的代码放到store.getters中！
      // 所有的动物
      const allAnimal = state.users
      const tmpArr = []
      for (let i = 0; i < allAnimal.length; i++) {
        const item = allAnimal[i]
        if (item.age >= 18) {
          tmpArr.push(item)
        }
      }
      return tmpArr
    }
  }
})
// 暴露
export default store
```

main.js

```javascript
import Vue from 'vue'
import App from './App.vue'
import store from './store.js'
new Vue({
  store,
  render: h => h(App)
})
.$mount('#app')
```

App.vue

```vue
<template>
<div>
  <h1>我是中国人，我爱自己的祖国!</h1>
  <p v-for="item in getAdultAnimal">
  {{item.name}}, {{item.age}}</p>
  <hr>
  getA
  <p v-for="item in getAa">
  {{item.name}}, {{item.age}} </p>
</div>
</template>

<script>
export default {
  computed: {
    getAa () {
      // 当我们使用this.$store.getters.getAdultAnimal时，vue会监测时，然后会自动调用
      // getters中同名的方法
      return this.$store.getters.getAdultAnimal // 不需要添加括号()
    },
    getAdultAnimal () {
      // 如果多个组件中都写了下面的代码，是对状态进行处理的(不修改状态的值的!)
      // 此时推荐将我们的获取状态的代码放到store.getters中！
      // 所有的动物
      const allAnimal = this.$store.state.users
      const tmpArr = []
      for (let i = 0; i < allAnimal.length; i++) {
        const item = allAnimal[i]
        if (item.age >= 18) {
          tmpArr.push(item)
        }
      }
      return tmpArr
    }
  }
}
</script>
<style>
</style>
```



## Vue.use ()

use方法里传递的可以是一个函数!,这个函数称之为Vue的插件

###把Axios封装成一个Vue的插件

main.js

```javascript
import Vue from 'vue'
import App from './App.vue'
import vueAxios from './vue-axios.js'

// use方法里传递的可以是一个函数!,这个函数称之为Vue的插件

// 1.use方法中会调用这个vueAxios方法
// 2.use函数的参数就是Vue这个构造函数
// 3.Vue的原型属性会被组件的实例继承
Vue.use(vueAxios)

new Vue({
  render: h => h(App)
})
.$mount('#app')
```

vue-axios.js

```javascript
/**
 * 目标，将axios对象传递给组件的实例的$axios属性
 * 这样我们以后在组件中使用axios时，就不需要再引入它的
 */
import axios from 'axios'

// 参数是Vue这个构造函数!
function vueAxios(Vue) {
  console.log('嘻嘻，哈哈')
  // 如果给Vue这个构造函数添加原型属性!
  // Vue.prototype.hello = '小明你好'
  // 添加了原型属性后Vue的实例就会有一个hell属性
  // var vm  = new Vue() // vm.hello 小明你好
  // 组件的实例会继承vm这个实例的属性!

  Vue.prototype.$axios = axios 
  // 写了之后，一个是Vue的实例有$axios对象
  // 组件的实例会有一个$axios对象

}
export default vueAxios
```

App.vue

```vue
<template>
<div>
  <h1>我是中国人，我爱自己的祖国!</h1>
</div>
</template>
<script>
export default {
  created () {
    console.log('app.vue')
    // console.log(this.$axios)
    this.$axios({
      url: '/xx',
      method: 'post',
      data: {a:1,b:2}
    })
  }
}
</script>
<style>
</style>
```

##路由vue-router

> 在Vue中路由的作用是根据浏览器【地址栏中的地址】的不同，将不同的Vue组件呈现到页面的指定位置
>
> 如果只修改地址栏中的锚点名字的话，页面是不会跳转的
>
> 路由可以用于做单文件应用	SPA : single page application

**路由使用步骤**

1.下载并引用vue-router这个包

   `<script src="....vue-router.js"></script>` 或者 `import VueRouter from 'vue-router'`

2.使用`Vue.use(VueRouter)`

 如果是使用的webpack打包的方式使用的Vue，就需要在引入vue-router之后加上一行代码: `Vue.use(VueRouter)`

3.配置规则

规则的作用是用来告诉vue-router,当页面地址栏是什么内容时，就会将某个组件的模板解析后插入到页面【特定的位置】

在项目中添加一个js文件，添加如下代码，并引入到main.js， 或者直接将如下代码写到main.js中


```javascript
import A from './A.vue'
import B from './B.vue'
var vr = new VueRouter({
  // 规则就写在这个routes数组中
  // 数组中的一个元素就是一条规则
  // 每个规则至少有两个属性，一个是path:用于指定地址栏的地址,一个是component:用于指定组件对象
  // path属性通常以/开头，可以包含多个/, 注意component指定的组件一定要引入进来,才能使用
  routes: [
    // 这个规则表示，当页面地址栏中的锚点值为 '#/home' 时，就会将 组件A 中的模板解析后插入到页面【特定的位置】
    {path: '/home', component: A},
    // 这个规则表示，当页面地址栏中的锚点值为 '#/user/xiaoming' 时，就会将 组件B 中的模板解析后插入到页面【特定的位置】
    {path: '/user/xiaoming', component: B}
  ]
})
```

4.在new Vue({}) 时添加一个名为router的属性，值为3中的vr这个实例对象。

   ```javascript
   new Vue({
     ...
     router:vr // Vue将vr对象的一些属性和方法赋值给各个组件的实例
   }).$mount('#app')
   ```

5.指定组件插入的位置

在页面上想要显示A组件或者B组件对应的代码中写上`<router-view></router-view>`标签, 这个`router-view`标签也是一个组件，是引入的vue-router 这个包中声明的全局组件, 这个组件内部会根据3.中的规则去动态的展示相应的组件!

main.js

```javascript
// 1.引入vue-router
import VueRouter from 'vue-router'
// 引入Vue
import Vue from 'vue'
import App from './App.vue'

import XXX from './XXX.vue'
import YYY from './YYY.vue'

// 2.调用Vue.use方法
// 如果是在html中引入script标签的方式引入vue-router就不需要调用这个方法
// 如果是使用webpack打包的方式就要调用!
Vue.use(VueRouter)

// 3. 配置路由
// 指定一下，地址栏地址是什么时，才呈现什么组件!
// Vue中路由的作用是根据【地址栏地址】的不同，在页面指定的位置呈现【指定的组件】
var vr = new VueRouter({
  // 这个routes是固定的名字, 把配置写在这个routes数组中
  routes: [
    // 一个对象就是一个路由
    // 每个对象中至少有两个属性，
    // path: 用来匹配地址栏的地址
    // component: 是用来指定组件的，当地址栏的地址与path匹配时，component对应的组件就
    // 会呈现到页面上
    // path的值通常要以/开头
    {
      // 当地址的中锚点值为 '#/home/xx' 时, XXX组件就会被呈现到页面上【指定位置】
      path: '/home/xx', component: XXX
    },
    {
      // 当地址栏中的锚点值为 '#/user' 时，YYY组件就会被呈现到页面指定位置，
      // 否则这个组件就会从页面消失(组件被销毁)
      path: '/user', component: YYY
    }
  ]
})

// 4. 在new Vue({}) 时，在参数中添加一个属性叫router,
// router的值就是3中new出的对象
// 创建Vue的实例
var vm = new Vue({
  router: vr,// 让路由生效!
  render: function (createElement) {
    // return createElement(组件对象)
    // App组件中的模板最终会替换index.html中#app这个元素
    return createElement(App)
  }
})
.$mount('#app')
```

App.vue

```vue
<template>
<!-- template 标签中只能有一个直接子元素-->
<div>
  <h1>我是App.vue组件</h1>
  <a href="#/home/xx">用户管理</a>
  <a href="#/user">薪资管理</a>
  <!-- 这个router-view是vue-router这个包中提供的全局组件,这个组件的作用,
  就是根据我们对路由的配置在页面上呈现指定的组件
  -->
  <router-view></router-view>
</div>

</template>
<script>
</script>
<style>
</style>
```

模拟路由监听地址栏中锚点值的变化

```javascript
// 监听地址栏中锚点的变化
window.onhashchange = function () {
  // 得到地址栏的锚点值
  var hash = window.location.hash

  if (hash === '#/user') {
    document.querySelector('#view').innerHTML = '用户管理'
  } else if (hash === '#/salary') {
    document.querySelector('#view').innerHTML = '薪资管理'
  }
}
```

###动态路由

只配置一个路由，可以让多个不同的锚点值匹配同一个规则，同一个组件!

在上面路由基本使用中：当前锚点值为'#/user/xiaoming', 就会显示B组件。如果值变为其他内容时，B组件就会消失，

如果现在有这样的需求: 希望无论锚点值是'#/user/xiaoming', 还是 '#/user/honghong' 时，都能够显示B组件，也就是说,希望锚点值以'#/user/' 开头，时都会显示B组件。

这时就需要使用动态路由匹配来达到目的了

要达到上面的需求，只需要将原来的: `{path: '/user/xiaoming'}` 修改为 `{path: '/user/:name'}`, 这里的` :`号,有点像正则中通配符的意思, 通过表示任意字符，: 后面的name 没有什么特殊语义，只是一个普通字符，换成任意一个字符都可以。: 冒号后的name暂且我们叫它为路由参数!

main.js

```javascript
// 引入Vue
import Vue from 'vue'
import App from './App.vue'
// 1.引入 vue-router
import VueRouter from 'vue-router'
import UserInfo from './UserInfo.vue'
import Home from './Home.vue'

// 2. 使用use
// 实际是: vue-router内部会判断，如果Vue是全局对象，它就会自动调用Vue.use方法
// 但是用了webpack之后，这里的Vue就不是全局的了,需要我们自己添加这个Vue.use方法的调用
Vue.use(VueRouter)

// 3.配置路由
// 根据不同的的锚点值，将对应的组件的模板解析后呈现【指定的位置】
var vr = new VueRouter({
  routes: [
    {
      // #/user/a, #/user/b
      // path:'/a/b/c/d/f/e/g'
      path: '/user/:xxx',  component: UserInfo,
    },
    {
      path: '/home', component: Home
    }

  ]
})

// 4.要将vr这个实例与vm实例关联(newVue时添加一个router属性，值为vr)
// 创建Vue的实例
var vm = new Vue({
  router: vr,
  render: function (createElement) {
    // return createElement(组件对象)
    // App组件中的模板最终会替换index.html中#app这个元素
    return createElement(App)
  }
})
.$mount('#app')
```

userInfo.vue

```vue
<template>
<!-- template 标签中只能有一个直接子元素-->
<div>
  <h1>我是UserInfo.vue组件</h1>
  {{name}}
</div>

</template>
<script>
export default {
  data: function () {
    return {
      name: ''
    }
  },
  created: function () {
    console.log('userinfoCreated')
    // 要根据地址栏中锚点值判断,如果是/user/xiaoming就呈现小明的信息
    // 如果是/user/xiaohong就呈现小红的信息
    // this.$routes.params 这个对应是vue-router创建，它把地址锚点值的一部分获取到了，保存到了这里
    
  },
  mounted: function () {
    console.log('userinfomounted')
    // path: '/user/:xxx'
    console.log(this.$route.params) // {xxx: 'xiaoming'}
    if(this.$route.params.xxx === 'xiaohong') {
      this.name = '我是小红啊！'
    }else {
      this.name = '我不是小红啊!'
    }
  }
}
</script>
<style>
</style>
```

###嵌套路由

main.js

```javascript
// 引入Vue
import Vue from 'vue'
import App from './App.vue'
import A from './A.vue'
import B from './B.vue'
import C from './C.vue'
import D from './D.vue'
import E from './E.vue'

// 1. 引包
import Router from 'vue-router'

// 2.Vue.use
Vue.use(Router)

// 3.配置路由
var vr = new Router({
  routes: [
    // 如果希望在B组件再嵌套一些内容，就需要添加一个children属性, 再指定多个内容
    // childrend属性也是数组，数组中的内容和routes一样
    { 
      path: '/user/b',
      component: B, 
      // children中也是配置的路由规则，也可以直接访问,path是/开头就直接根据path的值访问
      // 如果地址栏输入 '#/xm',D组件就会呈现到页面指【定的位置】!
      // 这个组件件最终会呈现到组件B中(在组件B中也可以写个router-view)
      // 组件B中的router-view是用来呈现children中匹配到的组件的
      children: [{path: '/xm', component: D}, {path: '/xh', component: E}]
    },

    {path: '/user/c', component: C},
  ]
})

// 创建Vue的实例
var vm = new Vue({
  router: vr,
  render: function (createElement) {
    // return createElement(组件对象)
    // App组件中的模板最终会替换index.html中#app这个元素
    return createElement(App)
  }
})
.$mount('#app')
```

###路由导航

####声明式导航

* 可以使用一个router-link标签来代替a标签进行导航, router-link其实是vue-router这个包提供的全局组件, 

router-link组件内部会根据 to属性的值生成对应的a标签，a标签的href什是由 这里的to属性决定的。

`<router-link to="/xx">导航到/xx</router-link>`

* 也可以通过v-bind来指定to属性!

`<router-link :to=" '/xx' ">导航到/xx</router-link>`

```vue
<template>
<!-- template 标签中只能有一个直接子元素-->
<div>
  <a href="#/user/a">首页</a> 
  <a href="#/user/b">用户管理</a> 
  <a href="#/user/c">商品管理</a> 
  <br>
  // 这个router-link标签是vue-router提供的组件，这个组件中就只有一个a标签
  // 推荐用这个标签来代替a标签
  // router-link组件内部会根据这个to属性中path的值，显示一个a标签，a标签的href与
  这个path值是对应的!
  <router-link :to="{path: '/user/a' }">首页router-link</router-link>
  <!--上一行的router-link会生成一个a标签，<a href="#/user/a">首页router-link</a>-->
  <router-view></router-view>
  <h1>我是App.vue组件</h1>
  
</div>
</template>
<script>
</script>
<style>
</style>
```

####编程式导航

* 可以通过vue-router提供的方法来进行导航，参数中的path属性名是固定的

* 也可以再添加个query属性(也是固定的，也可以不添加, 有没有这个属性对导航的效果没有实质的影响)

  query属性的值需要是个对象，query属性的就是可以在进行导航时，在要导航到的组件中，通过`this.$routes.query` 来得到这个query属性值，也是组件之间传递数据的方式之一

  `this.$router.push({path: '/xx/yy', query: {a: 1, b: 2} })`  调用后地址栏地址变为了 '#/xx/yy?a=1&b=2'

```vue
<template>
<div class="b">
<h1>我是B组件</h1>
<a href="#/xm">xm</a>
<a href="#/xh">xh</a>
<router-view></router-view>
</div>

</template>
<style>
.b {
  color: white;
  background: red
}
</style>
<script>
export default {
  created: function () {
    // 我希望这个组件显示后3秒自动导航到A组件 /user/a
    // 这个push方法的作用，就是导航到 '/user/a'
    // 和点击a标签的效果一致
    var that = this
    setTimeout(function () {
      // 这个push方法内部就是改变了地址栏中的锚点值// window.location.hash = '#jjj'
      // query属性的作用，是给锚点最后添加一个?a=1&b=2
      // /user/a 对应的是A组件，在A组件中就可以得到这里的query这个值
      // 在A组件中调用this.$route.query 可以得到这里query的值
      that.$router.push({path: '/user/a',  query:{a:1,b:2}})
    }, 3000)
  }
}
</script>
```

A.vue

```vue
<template>
<div class="a">
<h1>我是A组件</h1>
</div>
</template>

<style>
.a {
  color: white;
  background: red
}
</style>
<script>
export default {
  created: function () {
    // 这里可以接收到B组件通过锚点传过来的数据
    console.log(this.$route.query)
  }
}
</script>
```

####命名路由

在写路由时，多添加一个name属性，有了name属性之后，声明式导航可以这么写:

`<router-link :to="{name: 'myhome'}">导航</router-link>`  

等价于 `<router-link :to="/home">导航</router-link`

```javascript
var vr = new VueRouter({
  routes: [
    // 这里的name的值，可以随便写个英文就可以，但是多个name的值不要相同。
    {name: 'myhome', path: '/home', component: XX},
    {name: 'myxiao', path: '/user/xiaoxiao', component: YY}
  ]
})
```

使用命令路由的意思在于若果后期修改了路由的地址，路由而配置不需要改动

```javascript
// 引入Vue
import Vue from 'vue'
import App from './App.vue'
import A from './A.vue'
import B from './B.vue'
import C from './C.vue'
import D from './D.vue'
import E from './E.vue'

// 1. 引包
import Router from 'vue-router'

// 2.Vue.use
Vue.use(Router)

// 3.配置路由
var vr = new Router({
  routes: [
    // 这里每一个，对象可以再添加一个name属性，添加了之后，它就是命令路由了
    // name属性的值不会重复,这里的name只是这个路由的别名!
    // 写了name属性之后，再进行导航时，可以不写链接，而是通过name属性的值来导航
    
    // 写name之前: 
    // <a href="#/user/test"></a> 
    // <router-link :to="{path: '/user/a'}"></router-link>
    //  this.$router.push({path: '/user/a'})

    // 写了name之后
    // <router-link :to="{name: 'mya'}"></router-link>  a:href="#/user/a"
    // <router-link :to="{name: 'mya'}"></router-link>
    // this.$router.push({name: 'mya'})
    {name: 'mya', path: '/user/a', component: A},

    // 如果希望在B组件再嵌套一些内容，就需要添加一个children属性, 再指定多个内容
    // childrend属性也是数组，数组中的内容和routes一样
    { 
      name: 'myb',
      path: '/user/b',
      component: B, 
      // children中也是配置的路由规则，也可以直接访问,path是/开头就直接根据path的值访问
      // 如果地址栏输入 '#/xm',D组件就会呈现到页面指【定的位置】!
      // 这个组件件最终会呈现到组件B中(在组件B中也可以写个router-view)
      // 组件B中的router-view是用来呈现children中匹配到的组件的
      children: [{path: '/xm', component: D}, {path: '/xh', component: E}]
    },

    {name: 'myc',path: '/user/c', component: C},
  ]
})
// 创建Vue的实例
var vm = new Vue({
  router: vr,
  render: function (createElement) {
    // return createElement(组件对象)
    // App组件中的模板最终会替换index.html中#app这个元素
    return createElement(App)
  }
})
.$mount('#app')
```

####重定向

```javascript
const router = new VueRouter({
  routes: [
    // redirect属性的作用是: 如果用户想跳转到/a时，会自动跳转到b
    { path: '/a', redirect: '/b' },
     // 如果用户想跳转到/a时，会自动跳转到名为foo的路由
    { path: '/c', redirect: { name: 'foo' }}
  ]
})
```

##vue-cookie

为vue提供的cookie操作插件，cookie只能存储字符串

对象转换为字符串，千万不要用toString()，用JSON.stringify ( )

###开始使用

下载安装：`npm install vue-cookie --save`

```javascript
import VueCookie from 'vue-cookie'
Vue.use(VueCookie)
```

然后在各个组件中就可以直接使用this.\$cookie.set方法和this.$cookie.get方法来操作cookie值

this.$cookie.set(key, value, [过期时间])



encodeURI ()

decodeURI ()

##数据驱动的思想

vue，angualr，react都是这种思想

不再推崇大量的dom操作，推荐的是直接操作数据, 如何呈现，改变，删除，添加【数据】，至于数据如何呈现到dom中，是框架底层帮我们做的dom操作。

##项目构建

1.新建`webpack.config.js`,生成`package.json` ，将项目代码放到一个统一的文件夹中,比如src

2.在`webpack.config.js`打要打包js的配置写好

3.在`webpack.config.js`使用 `html-webpack-plugin` 来将html复制到打包后的文件夹
并自动引入打包后的文件

4.将css打包进js中, css-loader, style-loader

5.如果使用的Vue单文件组件的形式: 要通过webapck将.vue还原成.js: `vue-loader`

6.将js由jses6 转换为es5, 开发时用es6, 打包时，将es6代码转换为es5的代码！

使用babel-loader：作用,是将各种形式的js转换为es5, TypeScript, coffeeScript 和js的关系 就是 sass/less 和css的关系

##vue-cli

安装: `npm install -g vue-cli`

生成基本的项目结构及配置文件! `npm init [项目类型] 项目文件夹路径` ，从github中可以看到有多少种[项目类型]

npm init webpack  project01	有些包第一次试的时选择N,不要选择Y!

npm init webpack-simple project02		生成简洁的项目结构