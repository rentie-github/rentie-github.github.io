title: Vue3新知识
author: Mario
date: 2024-11-16 16:03:34
tags:
---
关于Vue2到Vue3的一些更新记录
<!--more-->

### 1、Vue3特点
1、性能提升
``` bash
    打包大小减少41%
    初次渲染快51%，更新渲染快133%
    内存占用减少54%
``` 

2、源码升级
``` bash
    使用Proxy 代替Object.defineProperty 实现双向数据绑定
    重写虚拟DOM的实现和Tree-Shaking
    vue3 更好的支持typescript
```  
3、新的特性
composition API（组合式api）
``` bash
    setup配置
    ref和reactive
    watch 和watchEffect
    provide和inject
    提供了新的内置组件
    Fragment
    Teleport
    Suspense
```  
其他改变
    新的生命周期钩子
    移除keyCode 作为v-on的修饰符
    
##### 创建Vue3工程项目
``` bash
// 1. 确保你的vue-cli 脚手架版本在4.5.0以上，这样才能创建
// 2. 检查脚手架版本  vue -V 或 vue --version
// 3. 如果版本低话，重新安装vue-cli   npm install -g @vue/cli  
// 4. vue create vue3-project
// 5. cd vue3-project
// 6. npm run serve 启动项目
```
###### vite模板搭建vue3项目
``` bash
npm init vue@latest
```
![upload successful](/images/pasted-35.png)

###### vue3目录结构分析
``` bash
// vue3 入口文件 main.js
// 引入的不是Vue构造函数了，而是createApp 这个工厂函数（工厂函数就是返回值为一个对象的函数）
import { createApp } from 'vue'  
import App from './App.vue'  
const app = createApp(App).mount('#app') // app 相当于vue2中的vm，但是比vm轻，里面绑定的属性和方法更少，而不是像vm那样用的不用的都绑上，显的有点重
 
// vue2入口文件 main.js
import Vue from 'vue'
import App from './App.vue'
const vm = new Vue({
  render: h => h(App),
}).$mount('#app')
 
// 比较 
// vue3 引入的是createApp 这个函数，而不是vue2中的的 Vue 构造函数啦。
// vue3中的 app 类似于之前vue2中的vm, 但是app应用实例对象比vm 更轻，里面绑定的属性和方法更少
```
打开app.vue，发现vue3模板结构可以不使用根标签。
``` bash
<template>
  <img alt="Vue logo" src="./assets/logo.png">
  <HelloWorld msg="Welcome to Your Vue.js App"/>
</template>
```
### 2、Composition API
Composition API 也叫组合式api 主要包括如下及部分：
##### 1、setup函数

- 它是vue3新增的一个配置项，值为一个函数,所有的组合式API 方法都写在在setup函数中。
- 组件中用到的数据,方法 计算属性，事件方法，监听函数，都配置在setup中。
- setup的函数值为一个函数，该函数的返回值为一个对象，对象中的属性和方法均可以在模板中直接使用。代码如下：

``` bash
<template>
  <div>
    <p>姓名:{{ name }}</p>
    <p>年龄:{{ age }}</p>
    <p @click="sayHello(10)">说话</p>
  </div>
</template>
 
<script>
export default {
  name: "App",
  setup() { //为一个函数
     //定义变量 
    let name = "张三";
    let age = 20;
     // 定义方法
    function sayHello(m) {
      alert(`${name}--${age}--${m}`);
    }
    return { //setup函数返回值为一个对象
      name,
      age,
      sayHello,
    };
  },
};
</script>
```
<strong> 注意：setup 函数尽量不要和vue2.x混合使用</strong>
 - vue2.x中的（data,methods,computed）可以访问到 setup中的属性和方法
 - 在setup中不能访问vue2.x中的配置数据和方法如（data,methods,computed）代码如下：
 
``` bash
<template>
  <div>
    <p>姓名:{{ name }}</p>
    <p>年龄:{{ age }}</p>
    <p @click="sayHello(10)">说话</p>
    <p>{{ height }}</p>
  </div>
  <!-- vue2.x中的事件 -->
  <div @click="test1">vue2中的事件test1</div>
  <!-- vue3中的事件 -->
  <div @click="test2">vue3中的事件test2</div>
</template>
 
<script>
export default {
  name: "App",
  data() {
    return {
      sex: "男", // vue2中的变量
      height: 186, // vue2中的变量
    };
  },
  methods: {
    test1() {
      console.log(this.name); // "张三"
      console.log(this.age); // 20;
      console.log(this.sayHello); // 函数
      console.log(this.sex); // 男
    },
  },
  setup() {
    let name = "张三";
    let age = 20;
    let height = 176; // setup中的变量
    function sayHello(m) {
      alert(`${name}--${age}--${m}`);
    }
    function test2() {
      console.log(this.name); // "张三
      console.log(this.sayHello); // 函数
      console.log(this.sex); // undefined  vue3中的setup 访问不了vue2中数据
      console.log(this.test1);// undefined vue3中的setup 访问不了vue2中方法
    }
    return {
      name,
      age,
      sayHello,
      test2,
      height,// setup中的变量
    };
  },
};
```
##### 2、setup执行期间
 setup 在beforeCreate之前执行，且setup中获取不到this this为undefined
 ``` bash
 beforeCreate() {
    console.log("beforeCreate");
    console.log(this); // proxy 实例对象
  },
  setup() {
    console.log("setup");
    console.log(this); // undefined
  }
```
##### 3、ref 响应式函数
###### 1、引言
 如下代码：当点击执行changeFn函数，修改setup中定义的变量时，发现页面中的name和age的数据并没有修改，说明该数据<strong>不是响应式数据</strong>
 ``` bash
<template>
  <div>111</div>
  <p>{{ name }}--{{ age }}</p>
  <button @click="changeFn">changeFn</button>
</template>
 
<script>
export default {
  name: "App",
  setup() {
    //定义变量
    let name = "张三";
    let age = 20;
    // 定义方法
    function changeFn() {
      name = "李四";
      age = 30;
    }
    return {
      //setup函数返回值为一个对象
      name,
      age,
      changeFn,
    };
  },
};
</script>
```
###### 2、ref 函数
- 它是 vue3中的一个函数，一般用于将基本类型数据处理成响应式数据。
- 作用：定义一个基本类型的响应式的数据，只有数据成为响应式数据，这样当数据变化时，才能被监测到。
- 使用时需要从vue中引入
``` bash
<script>
   import { ref } from "vue"; // 引入响应式函数ref
   ...
</script>
```
- 语法：const xxx =ref(数据变量)；结果 返回一个 RefImpl 的引用对象，获取时为 xxx.value
- 在页面模板中使用数据直接 使用插值表达式，不需要加value
``` bash
//因为vue3会自动帮你.value,所以可以拿到值
姓名:{{ name }}
```
- ref 函数实现数据响应式的原理还是利用了vue2的Object.defineProperty() 给数据设置 get set 监听函数，如下图：

![upload successful](/images/pasted-36.png)
- 接收的数据类型可以是基本类型(<strong>实现响应式原理为Object.defineProperty()</strong>），也可以是对象类型(<strong>当为对象时，实现响应式的原理就是Proxy不是Object.defineProperty()</strong>)

点击如下change事件，修改name 和age
``` bash
<template>
  <div>
    <!--name这个ref 引用对象在使用时不需要加value,vue3会自动帮你加value,所以可以拿到值-->
    <p>{{ name }}--{{ age }}</p>
    <p>{{ job.type }}--{{job.salary}}</p>
    <p @click="change">说话</p>
  </div>
</template>
<script>
import { ref } from "vue"; // 引入响应式函数ref
export default {
  name: "App",
  setup() {
    let name = ref("张三"); //返回一个 ref 引用对象
    let age = ref(20);
    console.log(name)
    // 当ref传的参数为对象时
    let job = ref({
      type: "前端工程师",
      salary: "20k",
    });
    function change() {
      name.value = "李四"; // ref对象.value 修改其值
      age.value = 30;
      job.value.type = "后端开发工程师";
      job.value.salary = "30k";
    }
    return {
      name,
      age,
      change,
      job
    };
  },
};
</script>
```
##### 4、reactive 函数
1、定义一个对象类型的响应式数据，返回一个Proxy 实例对象,不能用于基本数据类型，否则报错。（基本类型响应式数据使用ref)。
语法：const 代理对象= reactive(源对象) ，接收一个对象或数组，返回一个代理对象（即Proxy源对象）
- 使用时先从vue中引入
``` bash
<script>
import { reactive } from "vue";
...
</script>
```
- 代码如下：

``` bash
<template>
  <p>{{ job.type }} --{{ job.salary }}--{{ job.like.a.b }}--{{job.content}}</p>
  <p v-for="(item, index) in foodArr" :key="index">{{ item }}</p>
  <button @click="changeFn">changeFn</button>
</template>
 
<script>
import { reactive } from "vue";
export default {
  name: "App",
  components: {},
  setup() {
    //定义对象
    let job = reactive({
      type: "前端工程师",
      salary: "20k",
      like:{
          a:{
              b:'不告诉你'
          }
      }
    });
    console.log(job);
    //定义数组
    let foodArr = reactive(["刷抖音", "敲代码"]);
    console.log(foodArr);
    // 定义方法
    function changeFn() {
      job.type = "后端开发工程师";
      job.salary = "30k";
      job.content = "写代码"; // 给对象添加属性
      foodArr[0]='买包包' // 通过下标修改数组
    }
    return {
      //setup函数返回值为一个对象
      job,
      changeFn,
      foodArr
    };
  },
};
</script>
```
- vue3中使用proxy 实现的数据响应去掉了vue2中不能检测到对象添加属性和通过下标修改数组而无法检测的情况

##### 5、vue3响应式原理
- 实现原理：通过Object.defineProperty()对属性的读取，修改进行拦截。（也叫数据劫持）即对数据的操作进行监听

``` bash
   let data = {
        name: "张三",
        age: 30
    }
    let p = {} 
    Object.defineProperty(p, 'name', {    
        get() {
            console.log('访问了name属性');
            return data.name
        },
        set(val) {
            console.log('设置name属性');
            data.name = val
        }
    })

//如上存在2个问题：
// 无法添加和删除属性
// 无法捕获到修改数组下标改变对应数组
```
###### 1、vue2.x存在的问题
- 1、对象数据新增属性和删除属性，界面不会响应更新

``` bash
<template>
  <div id="app">
    <p>{{ person.name }}</p>
    <p>{{ person.age }}</p>
    <p v-if="person.sex">{{ person.sex }}</p>
    <button @click="addSex">添加属性</button>
    <button @click="deleteAge">删除属性</button>
  </div>
</template>
<script>
import Vue from "vue";
export default {
  name: "App",
  data() {
    return {
      person: {
        name: "张三",
        age: 20,
      },
    };
  },
  components: {},
  methods: {
    addSex() {
      // this.person.sex = "男";
      // console.log(this.person); // 数据被修改，但是vue监测不到
      // this.$set(this.person, "sex", "男"); // 修改为第一种方式，vue可以检测到
      Vue.set(this.person, "sex", "男"); // 修改为第二种方式，vue也可以检测到
      console.log(this.person);
    },
    deleteAge() {
      // delete this.person.age;  // age被删除改，但是vue监测不到
      this.$delete(this.person, "age");  // 修改为第一种方式，vue可以检测到
      Vue.delete(this.person, "age"); // 修改为第二种方式，vue可以检测到
      console.log(this.person);
    },
  },
};
</script>
```
- 2、数组数据直接通过修改下标，界面不会响应更新

``` bash
 <div id="app">
    <p v-for="(item, index) in nameArr" :key="index">{{ item }}</p>
    <button @click="update">添加属性</button>
  </div>
<script>
import Vue from "vue";
export default {
  name: "App",
  data() {
    return {
      nameArr: ["刘备", "关羽"],
    };
  },
  methods: {
    update() {
      // this.nameArr[0] = "张飞"; // 数据被修改了，但是vue 检测不到
      // this.nameArr.splice(0, 1, "张飞"); // 修改为第一种方式，数据可以被检测到
      // this.$set(this.nameArr, 0, "张飞"); // 修改为第二种方式，数据可以被检测到
      Vue.set(this.nameArr, 0, "张飞"); // 修改为第三种方式，数据可以被检测到
      console.log(this.nameArr);
    },
  },
};
</script>
```
######  2、vue3响应式原理
1、首先看一下vue3是否还存在vue2.x 中的问题。
- 对象数据新增属性和删除属性，不存在vue2.x 中的问题了。

``` bash
<template>
  <div>
    <p>{{ person.name }}</p>
    <p>{{ person.age }}</p>
    <p>{{ person.sex }}</p>
    <button @click="addSex">添加属性</button>
    <button @click="deleteSex">删除属性</button>
  </div>
</template>
 
<script>
import { reactive } from "vue";
export default {
  name: "App",
  components: {},
  setup() {
    let person = reactive({
      name: "张三",
      age: 20,
    });
    console.log(person);
    //定义添加属性
    function addSex() {
      person.sex = "男";
      console.log(person);
    }
    // 定义删除属性
    function deleteSex() {
      delete person.sex;
      console.log(person);
    }
    return {
      person,
      addSex,
      deleteSex,
    };
  },
};
</script>
```
- 数组数据直接通过修改下标，修改数组，不存在vue2.x 中的问题了。

``` bash
<template>
  <div>
    <p v-for="(item, index) in person.like" :key="index">{{ item }}</p>
    <button @click="change">修改数组的值</button>
  </div>
</template>
 
<script>
import { reactive } from "vue";
export default {
  name: "App",
  components: {},
  setup() {
    let person = reactive({  
      name: "张三",
      age: 20,
      like: ["打球", "敲代码"],
    });
    console.log(person); // proxy 实例对象
    function change() {
      person.like[0] = "打台球";
    }
    return {
      person,
      change,
    };
  },
};
</script>
```
2、vue3响应式原理
首先说一下Reflect的作用。
``` bash
// Reflect是window下的一个内置对象
// 1. 使用reflect 访问数据
    let obj = {
        name: '张三',
        age: 20
    }
    console.log(Reflect.get(obj, 'name')); // 张三
// 2.使用Reflect 修改数据
    Reflect.set(obj, 'age', 50)
    console.log(obj);

//3.使用Reflect删除数据
    Reflect.deleteProperty(obj, 'name') 
    console.log(obj);
```
- vue3响应原理代码：

通过Proxy代理，拦截对象中任意属性的变化，包括属性的读取，修改、设置、删除。

通过Reflect 反射对被代理对象的属性进行操作。

``` bash
    let data = {
        name: "张三",
        age: 30
    }
    console.log(Proxy);
    // 使用p 对象代理data， Proxy为window 下的内置代理函数
    let p = new Proxy(data, {
        // 读取属性
        get(target, propName) {
            // target 就是 data
            console.log(`读取p上个${propName}属性`);
            return Reflect.get(target, propName)
        },
        // 修改和设置属性
        set(target, propName, value) {
            // value 为赋的值
            console.log(`修改p的${propName}属性`);
            // target[propName] = value
            Reflect.set(target, propName, value)

        },
        //删除属性
        deleteProperty(target, propName) {
            console.log(`删除p上的${propName}属性`);
            // return delete target[propName]
            return Reflect.deleteProperty(target, propName)
        }
    })
```
##### 6、Reactive 与 ref 对比

1、数据定义角度对比：
``` bash
    1、ref用来定义：基本数据类型
    2、reactive 用来定义： 对象或数组类型数据
    3、注意：ref也可以定义对象或数组类型数据，它内部还是通过reactive转换成代理对象
```
2、原理角度对比：
``` bash
   1、ref通过Object.defineProperty() 的get和set 实现数据的响应式 即（数据劫持）
   2、reactive 通过Proxy 实现数据响应式的，并通过Reflect 来操作源对象数据的。
```
3、从使用角度对比：
``` bash
  1、ref 定义的数据操作时需要使用 .value, 而插值表达式中使用时，不需要使用 .value
  2、reactive 定义的数据操作都不需要 .value
```
##### 7、vue3中的组件传参
``` bash
组件关系：
父子 props、$panrent
子父 emit自定义事件 refs
兄弟 eventbus中央事件总线 vue3如果需要实现eventbus 安装第三方库mitt
跨层级 provider inject
组件状态共享工具： vuex pinia
```
###### 1、父传子
- 在父组件中给子组件设置自定义属性 tit，将要传递的参数赋值给tit属性

``` bash
<!--父组件  -->
<template>
  <p></p>
  <Testvue3 :tit="schoolName">
    <span>123</span>
  </Testvue3>
</template>
 
<script>
import Testvue3 from "@/components/Testvue3";
export default {
  name: "App",
  components: { Testvue3 },
  setup() {
    let schoolName = "千锋"; // 定义要传给子组件的数据  
    return {
      schoolName, // 要使用的变量抛出去，这样就可以在页面模板中使用该变量
    };
  },
};
</script>
```
- 在子组件中接收传过来的属性通过props ,这个和vue2 一样没有变化。

``` bash
<!-- 子组件 -->
<template>
  <p>{{ tit }}</p>
  <button>点击事件,子传父</button>
</template>
 
<script>
export default {
  data() {
    return {};
  },
  props: ["tit"],
  setup(props) { 
    // 参数props即为父组件传过来的参数
     console.log(props)
    return {
      //setup函数返回值为一个对象
    };
  },
};
</script>
```
###### 2、子传父
- 给子组件绑定自定义事件，然后在setup中定义该事件对应的方法，因为setup中没有this ,this为undefined,所以vue的开发者为了解决该问题，在setup中提供了2个形参，prop和context
``` bash
   props 为父传子的参数
   context 上下文对象，里面有emit 方法，可以实现子传父
```
- 子组件中多了 emits选项，该选项类似于props,接收父组件给子组件绑定的自定义方法，如果不加该选项，vue3 会提示警告。但不影响功能
``` bash
  <!-- 子组件 -->
<template>
  <p>{{ tit }}</p>
  <button @click="emit">点击事件,子传父</button>
</template>
<script>
import { reactive } from "vue";
export default {
  data() {
    return {};
  },
  emits: ["transfer"], // 在子组件中使用emits配置项，接收父组件给我绑定的自定义事件，用法类似于props,						// 不加该配置项，控制台会提示警告
  setup(props, context) {
    console.log(11, props);
    console.log(22, context);
    // 定义方法
    function emit() {
      // 子传父 此处不用this,使用context上下文对象
      context.emit("transfer", 666);
    }
    return {
      //setup函数返回值为一个对象
      emit,
    };
  },
};
</script>
```
- 在父组件接收自定义事件，该事件对应的执行函数的形参就是传过来的数据，这个就和vue2一样啦。
``` bash
<!--父组件  -->
<template>
  <p></p>
  <Testvue3 @transfer="showdata">
    <span>123</span>
  </Testvue3>
</template>
<script>
import Testvue3 from "@/components/Testvue3";
export default {
  name: "App",
  components: { Testvue3 },
  setup() {
    function showdata(value) {
      console.log(value);
    }
    return {
      showdata,
    };
  },
};
</script>
```
##### 8、vue3中的计算属性
同vue2不同，使用计算属性需要引入computed 方法

``` bash
<template>
  <p>姓：<input type="text" v-model="data.firstname" /></p>
  <p>名：<input type="text" v-model="data.lastname" /></p>
  <p>姓名：<input type="text" v-model="data.fullname" /></p>
</template>
 
<script>
// 引入对应的计算属性方法
import { reactive, computed } from "vue";
export default {
  name: "App",
  setup() {
    let data = reactive({
      firstname: "",
      lastname: "",
      fullname: "",
    });
    // 计算属性--简写
    // data.fullname = computed(() => {
    //   return data.firstname + data.lastname;
    // });
    // 计算属性--完整写法
    data.fullname = computed({
      get() {
        return data.firstname + data.lastname;
      },
      set(value) {
        console.log(value);
        data.firstname = value.substr(0, 1);
        data.lastname = value.substr(1);
      },
    });
    return {
      data,
    };
  },
};
</script>
```
##### 9、vue3 中的 watch监听器
vue3 中的watch 也是 组合式api中的一个方法，所以使用时，需要引入

``` bash
<template>
  <p>{{ sum }} <button @click="sum++">sum++</button></p>
  <p>{{ fullname }} <button @click="fullname += '-'">修改姓名</button></p>
  <p>
    {{ userinfo.name }}--{{ userinfo.age }}--{{ userinfo.job.type }}--{{
      userinfo.job.salary
    }}K
    <button @click="userinfo.age++">修改年龄</button>
    <button @click="userinfo.job.salary++">修改薪水</button>
  </p>
</template>
<script>
// 引入对应的计算属性方法
import { ref, watch, reactive } from "vue";
export default {
  name: "App",
  setup() {
    let sum = ref(0);
    let fullname = ref("张三");
    let userinfo = reactive({
      name: "李四",
      age: 20,
      job: {
        type: "web开发",
        salary: 20,
      },
    });
    //1、监听ref定义的响应式数据 immediate初始化就执行watch
     watch(sum, (newvalue, oldvalue) => {
       console.log(newvalue, oldvalue);
      },{immediate:true});
      
    //2、 监听ref定义的多个响应式数据,immediate初始化就执行watch
     watch([sum, fullname], (newvalue, oldvalue) => {
         console.log(newvalue, oldvalue);
       }, { immediate: true });
 
    //3、 监听reactive 定义的响应式数据
    // 注意：此处oldvalue 无效（新值与旧值一样），默认是深度监听,immediate初始化就执行watch
    watch(
      userinfo,
      (newvalue, oldvalue) => {
        console.log(newvalue, oldvalue);
      },
      { immediate: true });
    
    return {
      sum,
      fullname,
      userinfo,
    };
  },
};
</script>
```
``` bash
watch 和 watchEffect 都能响应式地执行有副作用的回调。它们之间的主要区别是追踪响应式依赖的方式：
    watch 只追踪明确侦听的数据源。它不会追踪任何在回调中访问到的东西。另外，仅在数据源确实改变时才会触发回调。watch 会避免在发生副作用时追踪依赖，因此，我们能更加精确地控制回调函数的触发时机。
    watchEffect，则会在副作用发生期间追踪依赖。它会在同步执行过程中，自动追踪所有能访问到的响应式属性。这更方便，而且代码往往更简洁，但有时其响应性依赖关系会不那么明确。
```
##### 10、vue3生命周期

1、vue3中的生命周期钩子函数与vue2中基本相同，只有如下发生变化，其他都不变.
``` bash
    beforeDestory => beforeUnmount
    destoryed => unmounted
```  
- 第一种方式：不使用组合式api的方式使用生命周期函数
``` bash
<!-- 子组件 -->
<template>
  <p>{{ sum }} <button @click="sum++">sum++</button></p> <!--修改sum数据触发update生命周期 -->
</template>
<script>
import { ref } from "vue";
export default {
  setup() {
    let sum = ref(0);
    return {
      sum,
    };
  },
  beforeCreate() { 
    console.log("beforeCreate");
  },
  created() {
    console.log("created");
  },
  beforeMount() {
    console.log("beforeMount");
  },
  mounted() {
    console.log("mounted");
  },
  beforeUpdate() {
    console.log("beforeUpdate"); //页面数据发生改变触发
  },
  updated() {
    console.log("updated"); //页面数据发生改变触发
  },
  beforeUnmount() {
    console.log("beforeUnmount"); //组件卸载前触发
  },
  unmounted() {
    console.log("unmounted"); //组件卸载后触发
  },
};
</script>
```
在app.vue 中引入子组件
``` bash
<template>
  <Testvue3 v-if="flag"></Testvue3>
  <button @click="flag = !flag">显示/隐藏</button>  <!--修改flag 触发 beforeUnmount和 		  		                                             unmounted 生命周期-->
</template>
<script>
import { ref } from "vue";
import Testvue3 from "@/components/Testvue3";
export default {
  name: "App",
  components: {
    Testvue3,
  },
  setup() {
    let flag = ref(true);// 修改flag
    return {
      flag,
    };
  },
};
</script>
```
- 第二种方式：通过使用组合式api的形式使用生命周期 ，vue3和vue2.x的生命周期对照表如下：

<strong>setup 等同于 beforeCreate 和created</strong>

![upload successful](/images/pasted-37.png)
``` bash
<!-- 子组件 -->
<template>
  <p>{{ sum }} <button @click="sum++">sum++</button></p>
</template>
<script>
 //引入对应的生命周期钩子函数
import { 
  ref,
  onBeforeMount,
  onMounted,
  onBeforeUpdate,
  onUpdated,
  onBeforeUnmount,
  onUnmounted,
} from "vue";
export default {
  setup() {
    let sum = ref(0);
 
    onBeforeMount(() => {
      console.log("beforeMount");
    });
    onMounted(() => {
      console.log("mounted");
    });
    onBeforeUpdate(() => {
      console.log("beforeUpdate");
    });
    onUpdated(() => {
      console.log("updated");
    });
    onBeforeUnmount(() => {
      console.log("beforeUnmount");
    });
    onUnmounted(() => {
      console.log("unmounted");
    });
    return {
      sum,
    };
  },
};
</script>
```
##### 11、vue3中自定义hook函数

hook函数定义：本质是一个函数，将setup中的公共逻辑抽离出来放在一个单独的js文件，这样哪个组件使用导入即可。实现一个点击页面任意位置，在页面中输出当前点击位置的横纵坐标
- 实现代码：未使用hook 钩子函数前
``` bash
<!-- 子组件 -->
<template>
  <p>
    <span>鼠标点击位置的X坐标为：{{ point.x }}</span>
    <span>鼠标点击位置的Y坐标为：{{ point.y }}</span>
  </p>
</template>
<script>
import { reactive, onMounted, onBeforeUnmount } from "vue";
export default {
  setup() {
    let point = reactive({
      x: 0,
      y: 0,
    });
    onMounted(() => {
      document.onclick = function (event) {
        point.x = event.pageX;
        point.y = event.pageY;
      };
    });
    onBeforeUnmount(() => {
      document.onclick = null;
    });
 
    return {
      point,
    };
  },
};
</script>
```
- 使用hook 钩子函数 ，首先在src 目录下创建一个hooks文件夹，然后再该目录下创建usepoint.js文件，该文件用来存放抽来出来的公共方法。代码如下：
``` bash
import { reactive, onMounted, onBeforeUnmount } from "vue";
export function usePoint() {
    let point = reactive({
        x: 0,
        y: 0,
    });
    onMounted(() => {
        document.onclick = function (event) {
            point.x = event.pageX;
            point.y = event.pageY;
        };
    });
    onBeforeUnmount(() => {
        document.onclick = null;
    });
 
    return point // 最后抛出该point
}
```

``` bash
<!-- 子组件 -->
<template>
  <p>
    <span>鼠标点击位置的X坐标为：{{ point.x }}</span
    >,
    <span>鼠标点击位置的Y坐标为：{{ point.y }}</span>
  </p>
</template>
<script>
 // 在子组件中引入该hook方法
import { usePoint } from "@/hooks/usepoint"; 
export default {
  setup() {
    let point = usePoint();
    return {
      point,
    };
  },
};
</script>
```
##### 12、toRef 与 toRefs

定义：toRef 创建一个ref 响应数据

语法：let name = toRef(person，'name') 将响应对象person中的name属性单独拿出来变成响应属性。

应用：一般用于将响应对象中的某个属性单独提供给外部使用

- <strong>如下是使用toRef 前的代码：</strong> 插值表达式模板中的 person 有点繁琐

``` bash
<!-- 子组件 -->
<template>
  <div>
    <p>
      {{ person.name }} -- {{ person.age }} -- {{ person.job.type }} --
      {{ person.job.salary }}
    </p>
  </div>
</template>
 
<script>
import { reactive } from "vue";
export default {
  setup() {
    let person = reactive({
      name: "张三",
      age: 20,
      job: {
        type: "web前端开发",
        salary: 30,
      },
    });
    return {
      person,
    };
  },
};
</script>
```
- 如下是使用toRef 后 的代码

``` bash
<!-- 子组件 -->
<template>
  <div>
    <p>
     <!-- 在模板中直接使用name,age,type,salary -->
      {{ name }} -- {{ age }} -- {{ type }} --
      {{ salary }}
    </p>
    <p>
      <button @click="name += '-'">修改name</button>
      <button @click="salary++">修改薪水</button>
    </p>
  </div>
</template>
 
<script>
import { reactive, toRef } from "vue";
export default {
  setup() {
    let person = reactive({
      name: "张三",
      age: 20,
      job: {
        type: "web前端开发",
        salary: 30,
      },
    });
      
	// 将person 中的name 属性转换成ref 响应式数据，这样就可以直接在模板中使用了,以此类推
    let name = toRef(person, "name"); 
    let age = toRef(person, "age");
    let type = toRef(person.job, "type");
    let salary = toRef(person.job, "salary");
    return {
      name,
      age,
      type,
      salary,
    };
  },
};
</script>
<style scoped>
/* @import url(); 引入css类 */
</style>
```
- 使用toRefs 可以将对象中的多个属性转换成响应数据，代码如下：

``` bash
<!-- 子组件 -->
<template>
  <div>
    <p>
      {{ name }} -- {{ age }} -- {{ job.type }} --
      {{ job.salary }}
    </p>
    <p>
      <button @click="name += '-'">修改name</button>
      <button @click="job.salary++">修改薪水</button>
    </p>
  </div>
</template>
 
<script>
import { reactive, toRefs } from "vue";
export default {
  setup() {
    let person = reactive({
      name: "张三",
      age: 20,
      job: {
        type: "web前端开发",
        salary: 30,
      },
    });
    //toRefs返回一个响应对象，该对象中每个属性都变成了响应属性了。这样就可以直接拿来在模板插值表达式中使用了
    let person1 = toRefs(person);
    // console.log(person1);
    return {
      ...person1, // 使用后扩展运算符展开对象
    };
  },
};
</script>
<style scoped>
/* @import url(); 引入css类 */
</style>
```
``` bash
关于数据响应式设置的问题：
1、如何设置一个响应式数据？ ref reactive
2、如何将非响应式数据转为响应式数据？ toRef toRefs
3、如何将数据设置为只读数据？不能够修改 readOnly
```
##### 13、vue3 中的路由
1、首先下载安装vue3 对应的vue-router
``` bash
npm install vue-router@4
```
2、在src目录下新建router目录，在该目录下新建index.js 文件，代码如下
``` bash
//1. 从vue-router 引入对应的方法createRouter 和 createWebHashHistory
import { createRouter, createWebHashHistory } from "vue-router";
//2.配置路由规则
const routes = [
    { path: '/home', component: () => import('@/views/Home') },
    { path: '/category', component: () => import('@/views/Category') },
]
//3.创建路由对象
const router = createRouter({
    // 4. Provide the history implementation to use. We are using the hash history for simplicity here.
    history: createWebHashHistory(),
    routes, // short for `routes: routes`
})
 
export default router
```
3、在main.js 中引入并绑定
``` bash
// 引入路由 
import router from '@/router'
// 在全局实例上挂载router，这样每个页面都可以访问路由
createApp(App).use(store).use(router).mount('#app')
```
路由参数获取

``` bash
<template>
    <div>
        新闻页
    </div>
</template>
 
<script lang="ts">
import { defineComponent } from 'vue'
import { useRoute } from 'vue-router';
export default defineComponent({
    setup() {
        // useRoute获取路由参数
        const route = useRoute()
        console.log(route);
        console.log(route.query.id);
        return {}
    }
})
</script>
 
<style scoped></style>
```
路由方法使用
``` bash
<template>
    <div>
        Index首页
        <!-- <button @click="$router.push('/news')">新闻</button> -->
        <button @click="goNews">新闻</button>
    </div>
</template>
 
<script lang="ts">
import { defineComponent } from 'vue'
// 导入路由相关的组合式函数hook use开头的函数
import { useRouter } from 'vue-router'
export default defineComponent({
    setup() {
        // 组合式函数必须先调用一次
        // useRouter 获取路由操作方法
        const router = useRouter()
        const goNews = () => {
            // this.$router.push()
            // console.log(router);
            router.push('/news')
        }
        return { goNews }
    }
})
</script>
 
<style scoped></style>
```
##### 14、vue3 中的vuex
1、首先需要安装vuex
``` bash
npm install vuex@next --save
```
2、在src目录下创建store 文件夹,该目录下创建index.js文件，代码如下：
``` bash
import { createStore } from 'vuex'
// console.log(createStore);
export default createStore({
    state: {
        count: 100
    },
    getters: {
        getCount(state) {
            return state.count * 2
        }
    },
    mutations: {
        setTestA(state, value) {
            state.count = value
        }
    },
    actions: {
    },
    modules: {
 
    }
})
```
3、在main.js 引入然后使用挂载
``` bash
import { createApp } from 'vue'
import App from './App.vue'
// 引入 store
import store from '@/store'
console.log(store);
// 挂载使用 这样全局就可以使用store了
createApp(App).use(store).mount('#app')
// console.log(createApp(App));
```
4、在页面组件中使用store
``` bash
<!-- 子组件 -->
<template>
  {{ count }} ----{{ double }}
  <button @click="updateFun">修改数据</button>
</template>
 
<script>
import { computed } from "vue";
import { useStore } from "vuex";
export default {
  setup() {
    const store = useStore();
     // 必须以计算属性形式输出，否则无法修改state中的数据 如count
    const count = computed(() => {
      return store.state.count;
    });
    const double = computed(() => {
      return store.getters.getCount;
    });
 
    const updateFun = () => {
      store.commit("setTestA", 20);
    };
    return {
      count,
      double,
      updateFun,
    };
  },
};
</script>
```
<strong>vue官方在使用vue3时，更加推荐使用Pinia状态管理</strong>。Pinia和vuex实际一个开发团队在维护。vuex最终版本是4，可以将Pinia看作是vuex5。

src\stores\counter.ts
``` bash
import { ref, computed } from 'vue'
// defineStore store构造函数
import { defineStore } from 'pinia'
 
export const useCounterStore = defineStore('counter', () => {
  // 初始化state
  const count = ref(0)
  // 使用计算属性获取结果
  const doubleCount = computed(() => count.value * 2)
  // actions方法  修改state的操作方法
  function increment() {
    count.value++
    // console.log(count)
  }
 
  // 将需要外部使用到的值和方法按需导出
  return { count, doubleCount, increment }
})
```
src\App.vue
``` bash
<template>
  <div>
    <button @click="increment"> {{ store.count }}</button>
    <div>{{ store.doubleCount }}</div>
  </div>
</template>
 
<script lang="ts">
import { defineComponent, toRef } from 'vue'
// 将usestore引入
import { useCounterStore } from './stores/counter'
export default defineComponent({
  setup() {
    // 调用store获取对象
    const store = useCounterStore()
    console.log(store);
    // 从store对象中获取state值和修改state的方法
    return { store, increment: store.increment }
  }
})
</script>
 
<style scoped></style>
```