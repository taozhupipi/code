### <font face="宋体">vue3笔记</font>
```javascript
/**
 * 在Vue3中setup函数在beforeCreate钩子函数之前执行,并且只执行一次，此时组件还没有创建，所以没有this--->undefind
 */
```

```javascript
 /** 不能通过this去访问 data、computed、methods、props等内容
 *
 * setup的返回值是一个对象 内部创建的属性和方法是给当前的html模板（template）使用
 * 
 * setup(props,contest)
 * 
 * props参数是一个对象，里面有父组件给子组件传递的数据
 * 
 * contest参数是一个对象 里面有attrs（）/emit(分发事件)/slots（插槽）
 *
 * ref/reactive ref中如果放入了一个对象，内部会自动将对象、数组转换为reactive代理对象
 */
```

```javascript
 // Vue3中computed中如果需要get和set操作  
 const name= computed({
    get(){}
    set(val){}
 })
 // Vue3中computed中如果只需要get操作 
 const name= computed(()=>{

 })
 //Vue3中监听事件
 watch('监听的对象',(newValue,oldValue)=>{
 
 },{deep,true,immediate:true}) // 深度监听、默认执行一次watch事件
 
```


### <font face="宋体">封装一个简单的hook函数</font>
```javascript
/** hooks/useCkick */
import { onBeforeUnmount, onMounted, ref } from "vue";
export default function () {
    const x = ref(-1);
    const y = ref(-1);
    const clickHandler = (e) => {
        x.value = e.layerX;
        y.value = e.layerY;
    };
    onMounted(() => {
        window.addEventListener("click", clickHandler);
    });
    onBeforeUnmount(() => {
        window.removeEventListener("click", clickHandler);
    });
    return {
        x,
        y,
    };
}

/* 在app.vue中使用自定义的hook函数*/
<template>
  <h1>自定义hook函数操作</h1>
  <h2>X:{{ x }},Y:{{ y }}</h2>
</template>

<script>
import { defineComponent } from "vue";
import useClick from "./hooks/useClick";
export default defineComponent({
  setup() {
    const { x, y } = useClick;
    return {
      x,
      y,
    };
  },
});
</script> 

<style>
</style>
```

## <font face="宋体">toRefs的使用</font>
##### <font face="宋体" color="#666">toRefs可以把一个响应式对象（state）转化为普通对象 并将普通对象的每个property都是一个ref</font> #####
``` javascript 

<template>
  <h1>toRefs的使用</h1>
  <!-- <h3>name:{{ state.name }}</h3> -->
  <!-- <h3>age:{{ state.age }}</h3> -->
  <h3>name:{{ name }}</h3>
  <h3>age:{{ age }}</h3>
</template>

<script>
import { defineComponent, reactive, toRef, toRefs } from "vue";
export default defineComponent({
  setup() {
    const state = reactive({
      name: "zs",
      age: 18,
    });
    const state2 = toRefs(state); // const {name,age}=toRefs(state)
    setInterval(() => {
      // state.name += "====";  模板使用{{state.name}}时可以正常显示
      state2.name.value += "==="; //name.value += '==='
    }, 1000);
    return {
      // ...state,
      /**
       * 上方模板中直接使用 {{name}}将state解构 数据无法响应式
       * 得到的对象是{name:'zs',age:18}---->不是响应式
       */

      /**
       * toRefs可以把一个响应式对象（state）转化为普通对象 并将普通对象的每个property都是一个ref
       */
      ...state2,
      /**
       * name,
       * age
       */
    };
  },
});
</script> 

<style>
</style>

```


## <font face="宋体">shallowReactive和shallowRef</font>
``` javascript
    <template>
  <h1>shallowReactive,shallowRef的使用</h1>
  <h2>{{ m1 }}</h2>
  <h2>{{ m2 }}</h2>
  <h2>{{ m3 }}</h2>
  <h2>{{ m4 }}</h2>
  <hr />
  <button @click="update">update</button>
</template>

<script>
import {
  defineComponent,
  reactive,
  shallowReactive,
  shallowRef,
  ref,
} from "vue";
export default defineComponent({
  setup() {
    const update = () => {
      // 更改m1的数据---reactive
      // m1.car.name += "=="; // √ 深度响应式
      // 更改m2的数据---shallowReactive
      m2.car.name += "=="; // × 浅响应式
      // 更改m3的数据---ref
      // 更改m4的数据---shallowRef
    };
    const m1 = reactive({
      name: "zs",
      age: 18,
      car: {
        color: "red",
        name: "劳斯莱斯",
      },
    });
    const m2 = shallowReactive({
      name: "zs",
      age: 18,
      car: {
        color: "red",
        name: "劳斯莱斯",
      },
    });
    const m3 = ref({
      name: "zs",
      age: 18,
      car: {
        color: "red",
        name: "劳斯莱斯",
      },
    });
    const m4 = shallowRef({
      name: "zs",
      age: 18,
      car: {
        color: "red",
        name: "劳斯莱斯",
      },
    });
    return {
      m1,
      m2,
      m3,
      m4,
      update,
    };
  },
});
</script>
```

## <font face="宋体">readonly和shallowReadonly</font>
###### 一个对象中存在对象，存在的对象中具有属性，shallowReadonly可以进行修改
```js
const state = reactive( {
  name : 'zs',
  age : 18,
  car: {
    name : '奔驰',
    price: '888'
  }
} )
const state2 = readonly(state) 
const state3 = shallowReadonly(state)
/***
 * 点击按钮更改state2的数据 无法修改---->只读
 * 
 * 点击按钮更改state3的数据 无法修改，但是点击修改 state.car.name += ' ====' 可以修改
 * /

```

## <font face="宋体">toRaw和markRaw</font>
###### markRaw标记的对象，从此以后不能成为代理对象，toRaw将代理对象变为普通对象


### <font face="宋体">provide和inject</font>
```javascript
//App.vue
<template>
  <h1>provide和inject</h1>
  <p>当前的颜色：{{ color }}</p>
  <button @click="color = 'red'">红色</button>
  <button @click="color = 'yellow'">黄色</button>
  <button @click="color = 'green'">绿色</button>
  <son>
</template>

<script>
import { defineComponent, provide, ref } from "vue";
import Son from "./components/Son.vue";
export default defineComponent({
  name: "APP",
  components: {
    Son,
  },
  setup() {
    const color = ref("red");
    // 提供数据给孙子组件
    provide("color", color);
    return {
      color,
    };
  },
});
</script> 

// ./components/Son.vue
<template>
  <h3>SON子组件</h3>
  <hr />
  <grand-son>
</template>

<script>
import { defineComponent } from "vue";
import GrandSon from "./GrandSon.vue";
export default defineComponent({
  name: "Son",
  components: {
    GrandSon,
  },
  setup() {
    return {};
  },
});
</script>

// ./components/GrandSon.vue
<template>
  <h3 :style="{ color }">GrandSon孙子组件</h3>
  <hr />
</template> 

<script>
import { defineComponent, inject } from "vue";
export default defineComponent({
  name: "GrandSon",
  setup() {
    // 孙子组件接受数据
    const color = inject("color");
    return {
      color,
    };
  },
});
</script>
```
