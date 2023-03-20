---
title: Vue3 中自定义插件
---
### 自定义全局方法
#### Vue2 中定义全局方法
在 Vue2 中，自定义全局方法的思路如下：
```js
Vue.prototype.postRequest = postRequest;
```
通过 Vue.prototype 将一个方法挂载为全局方法，这样，在具体的 .vue 文件中，我们就可以通过 this 来引用这个全局方法了：
```vue
this.postRequest('/doLogin', this.loginForm).then(resp => {
    this.loading = false;
    if (resp) {
        this.$store.commit('INIT_CURRENTHR', resp.obj);
        window.sessionStorage.setItem("user", JSON.stringify(resp.obj));
        let path = this.$route.query.redirect;
        this.$router.replace((path == '/' || path == undefined) ? '/home' : path);
    }else{
        this.vcUrl = '/verifyCode?time='+new Date();
    }
})
```

在 Vue2 中，我们可以将一个方法挂载为全局方法。
Vue3 这个写法则完全变了：
1. 定义的方式变了，不再是 Vue.prototype。
2. 引用的方式变了，因为在 Vue3 中，没法直接通过 this 去引用全局方法了。

#### Vue3 中定义全局方法
方法定义：
```js
/**
 * Vue3 中定义全局方法
 */
app.config.globalProperties.sayHello=()=>{
    console.log("hello wourld!");
}
```
定义好之后，需要引用，方式如下：
```vue
<template>
    <div>
        <div>{{age}}</div>
        <div>{{aa.name}}</div>
        <div>{{aa.author}}</div>
        <div>{{name}}</div>
        <div>{{author}}</div>
        <button @click="updateInfo">更新信息</button>
        <button @click="btnClick">ClickMe</button>
    </div>
</template>

<!--直接在 script 节点中定义 setup 属性，然后，script 节点就像以前 jquery 写法一样-->
<script setup>

    //getCurrentInstance 方法可以获取到当前的 Vue 对象
    import {ref, reactive, toRefs, onMounted, getCurrentInstance} from 'vue';

    //来自该方法的 proxy 对象则相当于之前的 this
    const {proxy} = getCurrentInstance();

    const age = ref(99);
    const aa = reactive({
        name: "1111",
        author: 'sihai'
    });
    const updateInfo = () => {
        //修改，注意，在 vue3 中，现在方法中访问变量，不再需要 this
        aa.name = '111123';
    }
    //展开的变量
    const {name, author} = toRefs(book);
    onMounted(() => {
        console.log(this);
    })
    const btnClick = () => {
        //想在这里调用全局方法
        proxy.sayHello();
    }
</script>

<style scoped>

</style>
```
1. 首先需要导入 getCurrentInstance() 方法。
2. 从第一步导入的方法中，提取出 proxy 对象，这个 proxy 对象就类似于之前在 Vue2 中用的 this。
3. 接下来，通过 proxy 对象就可以去引用全局方法了。<br/>
   其他一些曾经在 Vue2 中使用 this 的地方，现在都可以通过 proxy 来代替了。

<br/>

### 自定义插件
一些工具方法可以定义为全局方法，如果这个全局的工具，不仅仅是一个工具方法，里边还包含了一些页面等，<br/>
那么此时，全局方法就不适用了。这个时候我们需要定义插件。<br/>
上面的全局方法定义，可以理解成为一个简单的插件<br/>
Vue2 和 Vue3 中自定义插件的流程基本上都差不多，但是，插件内部的钩子函数不一样。
#### 注册全局组件
首先定义一个组件：
```vue
<template>
    <div>
        <div>
            <a href="https://github.com/Tsihai">github</a>
        </div>
        <div>
            <a href="https://www.baidu.com">baidu</a>
        </div>
    </div>
</template>

<script>
    export default {
        name: "MyBanner"
    }
</script>

<style scoped>

</style>
```
接下来，就可以在插件中导入组件并注册：
```js
// 在这里定义插件
//在插件中，可以引入 vue 组件，并注册（这里的注册，就相当于全局注册）
import MyBanner from "@/components/MyBanner";

export default {
    /**
     * @param app 这个就是 Vue 对象
     * @param options 这是一个可选参数
     *
     * 当项目启动的时候，插件方法就会自动执行
     */
    install: (app, options) => {
        console.log("第一个插件")
        //在这里完成组件的注册，注意，这是一个全局注册
        app.component('my-banner', MyBanner);
    }
}
```
最后，就可以在项目的任意位置使用这个组件了：
```vue
<template>
  <nav>
    <router-link to="/">Home</router-link> |
    <router-link to="/about">About</router-link>
  </nav>
  <div>
    <!--这里就可以直接使用插件中注册的全局组件了-->
    <my-banner></my-banner>
  </div>
  <router-view/>
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

nav {
  padding: 30px;
}

nav a {
  font-weight: bold;
  color: #2c3e50;
}

nav a.router-link-exact-active {
  color: #42b983;
}
</style>
```
#### 注册全局指令
首先注册全局指令：
```js
// 在这里定义插件
//在插件中，可以引入 vue 组件，并注册（这里的注册，就相当于全局注册）
import MyBanner from "@/components/MyBanner";

export default {
    /**
     * @param app 这个就是 Vue 对象
     * @param options 这是一个可选参数
     *
     * 当项目启动的时候，插件方法就会自动执行
     */
    install: (app, options) => {
        console.log("第一个插件")
        //在这里完成组件的注册，注意，这是一个全局注册
        app.component('my-banner', MyBanner);
        //自定义指令，第一个参数是自定义指令的名称，第二个参数自定义指令的逻辑
        //el 表示添加这个自定义指令的节点
        //binding 中包含了自定义指令的参数
        app.directive('font-size', (el, binding, vnode) => {
            let size = 18;
            //binding.arg 获取到的就是 small 或者 large
            switch (binding.arg) {
                case "small":
                    size = 14;
                    break;
                case "large":
                    size = 36;
                    break;
                default:
                    break;
            }
            //为使用了 v-font-size 指令的标签设置 font-size 的大小
            el.style.fontSize = size + 'px';

        })
    }
}
```
然后就可以在任意地方去使用这个全局指令了：
```vue
<template>
    <div>
        <div>
            <a href="https://github.com/Tsihai" v-font-size:large>github</a>
        </div>
        <div>
            <!--使用自定义指令，去指定文本的大小，指令的名字就是 font-size-->
            <a href="https://www.baidu.com" v-font-size:small>baidu</a>
        </div>
    </div>
</template>

<script>
    export default {
        name: "MyBanner"
    }
</script>

<style scoped>

</style>
```
#### 参数传递
自定义插件的时候，可以通过 options 传递参数到插件中：
```js
// 在这里定义插件
//在插件中，可以引入 vue 组件，并注册（这里的注册，就相当于全局注册）
import MyBanner from "@/components/MyBanner";

export default {
    /**
     * @param app 这个就是 Vue 对象
     * @param options 这是一个可选参数
     *
     * 当项目启动的时候，插件方法就会自动执行
     */
    install: (app, options) => {
        console.log("第一个插件")
        //在这里完成组件的注册，注意，这是一个全局注册
        app.component('my-banner', MyBanner);
        //自定义指令，第一个参数是自定义指令的名称，第二个参数自定义指令的逻辑
        //el 表示添加这个自定义指令的节点
        //binding 中包含了自定义指令的参数
        app.directive('font-size', (el, binding, vnode) => {
            let size = 18;
            //binding.arg 获取到的就是 small 或者 large
            switch (binding.arg) {
                case "small":
                    size = options.fontSize.small;
                    break;
                case "large":
                    size = options.fontSize.large;
                    break;
                default:
                    break;
            }
            //为使用了 v-font-size 指令的标签设置 font-size 的大小
            el.style.fontSize = size + 'px';

        })
    }
}
```
options.fontSize.small 就是插件在引用的时候传入的参数。
```js
createApp(App)
    //安装插件
    .use(plugins,{
        fontSize:{
            small: 6,
            large: 64
        }
    })
    .use(store)
    .use(router)
    .mount('#app')
```
#### provide 和 inject
可以通过 provide 去定义一个方法，然后在需要使用的使用，通过 inject 去注入这个方法然后使用。
方法定义：
```js
// 在这里定义插件
//在插件中，可以引入 vue 组件，并注册（这里的注册，就相当于全局注册）
import MyBanner from "@/components/MyBanner";

export default {
    /**
     * @param app 这个就是 Vue 对象
     * @param options 这是一个可选参数
     *
     * 当项目启动的时候，插件方法就会自动执行
     */
    install: (app, options) => {
        console.log("第一个插件")
        //在这里完成组件的注册，注意，这是一个全局注册
        app.component('my-banner', MyBanner);
        //自定义指令，第一个参数是自定义指令的名称，第二个参数自定义指令的逻辑
        //el 表示添加这个自定义指令的节点
        //binding 中包含了自定义指令的参数
        app.directive('font-size', (el, binding, vnode) => {
            let size = 18;
            //binding.arg 获取到的就是 small 或者 large
            switch (binding.arg) {
                case "small":
                    size = options.fontSize.small;
                    break;
                case "large":
                    size = options.fontSize.large;
                    break;
                default:
                    break;
            }
            //为使用了 v-font-size 指令的标签设置 font-size 的大小
            el.style.fontSize = size + 'px';
        })

        const clickMe = () => {
            console.log("========clickMe========")
        }
        //这里相当于是注册方法
        app.provide('clickMe', clickMe);
    }
}
```
注意定义的时候，方法要写在 install 中。
方法使用：
```js
<template>
    <div>
        <div>
            <a href="https://github.com/Tsihai" v-font-size:large>github</a>
        </div>
        <div>
            <!--使用自定义指令，去指定文本的大小，指令的名字就是 font-size-->
            <a href="https://www.baidu.com" v-font-size:small>baidu</a>
        </div>
        <div>
            <button @click="btnClick">ClickMe</button>
        </div>
    </div>
</template>

<script>
    import {inject} from 'vue';

    export default {
        name: "MyBanner",
        mounted() {
            //注入 clickMe 函数
            const clickMe = inject('clickMe');
            clickMe();
        },
        methods: {
            btnClick() {
            }
        }
    }
</script>

<style scoped>

</style>
```
建议，最好是利用 Vue3 中的 setup 一起来使用。