# Vue 面试题

## 基础概念

### 1. Vue 实例和生命周期
**问题**: Vue 实例的生命周期有哪些阶段？

**答案**:
```javascript
// Vue 2 生命周期
export default {
  // 创建阶段
  beforeCreate() {
    // 实例初始化后，数据观测和事件配置之前
    console.log('beforeCreate: 实例刚创建');
  },
  
  created() {
    // 实例创建完成，数据观测、属性和方法运算完成
    // 可以访问 data、computed、methods
    console.log('created: 实例创建完成，可以进行数据请求');
    this.fetchData();
  },
  
  // 挂载阶段
  beforeMount() {
    // 挂载开始之前，render 函数首次被调用
    console.log('beforeMount: 挂载前');
  },
  
  mounted() {
    // 实例挂载完成，DOM 已经渲染
    console.log('mounted: 挂载完成，可以操作 DOM');
    this.$refs.myInput.focus();
  },
  
  // 更新阶段
  beforeUpdate() {
    // 数据更新时调用，发生在虚拟 DOM 重新渲染之前
    console.log('beforeUpdate: 数据更新前');
  },
  
  updated() {
    // 数据更新导致的虚拟 DOM 重新渲染后调用
    console.log('updated: 数据更新完成');
  },
  
  // 销毁阶段
  beforeDestroy() {
    // 实例销毁之前调用，清理定时器、解绑事件等
    console.log('beforeDestroy: 销毁前清理');
    clearInterval(this.timer);
  },
  
  destroyed() {
    // 实例销毁后调用
    console.log('destroyed: 实例已销毁');
  }
}

// Vue 3 生命周期（Composition API）
import { onMounted, onUpdated, onUnmounted } from 'vue'

export default {
  setup() {
    onMounted(() => {
      console.log('mounted')
    })
    
    onUpdated(() => {
      console.log('updated')
    })
    
    onUnmounted(() => {
      console.log('unmounted')
    })
    
    // beforeCreate 和 created 由 setup 替代
  }
}
```

### 2. Vue 响应式原理
**问题**: Vue 2 和 Vue 3 的响应式原理有什么区别？

**答案**:
```javascript
// Vue 2 响应式原理 - Object.defineProperty
function defineReactive(obj, key, val) {
  const dep = new Dep()
  
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get() {
      // 依赖收集
      if (Dep.target) {
        dep.depend()
      }
      return val
    },
    set(newVal) {
      if (newVal === val) return
      val = newVal
      // 派发更新
      dep.notify()
    }
  })
}

// Vue 2 的限制
const data = {
  list: [1, 2, 3],
  obj: { a: 1 }
}

// 无法检测数组索引赋值
data.list[0] = 10 // 不会触发更新

// 无法检测新增属性
data.obj.b = 2 // 不会触发更新

// 需要使用特殊方法
Vue.set(data.obj, 'b', 2) // 正确方式
data.list.splice(0, 1, 10) // 正确方式

// Vue 3 响应式原理 - Proxy
function reactive(target) {
  return new Proxy(target, {
    get(target, key, receiver) {
      // 依赖收集
      track(target, key)
      const result = Reflect.get(target, key, receiver)
      
      // 如果是对象，递归处理
      if (typeof result === 'object' && result !== null) {
        return reactive(result)
      }
      
      return result
    },
    
    set(target, key, value, receiver) {
      const oldValue = target[key]
      const result = Reflect.set(target, key, value, receiver)
      
      // 派发更新
      if (oldValue !== value) {
        trigger(target, key)
      }
      
      return result
    },
    
    deleteProperty(target, key) {
      const hadKey = hasOwnProperty.call(target, key)
      const result = Reflect.deleteProperty(target, key)
      
      if (result && hadKey) {
        trigger(target, key)
      }
      
      return result
    }
  })
}

// Vue 3 优势
const state = reactive({
  list: [1, 2, 3],
  obj: { a: 1 }
})

// 支持数组索引操作
state.list[0] = 10 // 正常触发更新

// 支持新增属性
state.obj.b = 2 // 正常触发更新

// 支持删除属性
delete state.obj.a // 正常触发更新
```

### 3. 计算属性和侦听器
**问题**: computed 和 watch 的区别和使用场景

**答案**:
```javascript
// Vue 2
export default {
  data() {
    return {
      firstName: 'John',
      lastName: 'Doe',
      userList: []
    }
  },
  
  computed: {
    // 计算属性 - 基于依赖缓存
    fullName() {
      // 只有当 firstName 或 lastName 改变时才重新计算
      return this.firstName + ' ' + this.lastName
    },
    
    // 计算属性的 getter 和 setter
    fullNameWithSetter: {
      get() {
        return this.firstName + ' ' + this.lastName
      },
      set(newValue) {
        const names = newValue.split(' ')
        this.firstName = names[0]
        this.lastName = names[names.length - 1]
      }
    }
  },
  
  watch: {
    // 侦听器 - 数据变化时执行
    firstName(newVal, oldVal) {
      console.log(`firstName changed from ${oldVal} to ${newVal}`)
    },
    
    // 深度侦听
    userList: {
      handler(newVal, oldVal) {
        console.log('userList changed')
      },
      deep: true, // 深度侦听对象内部变化
      immediate: true // 立即执行一次
    },
    
    // 侦听对象的某个属性
    'user.name'(newVal, oldVal) {
      console.log('user name changed')
    }
  }
}

// Vue 3 Composition API
import { ref, computed, watch, watchEffect } from 'vue'

export default {
  setup() {
    const firstName = ref('John')
    const lastName = ref('Doe')
    const userList = ref([])
    
    // 计算属性
    const fullName = computed(() => {
      return firstName.value + ' ' + lastName.value
    })
    
    // 可写计算属性
    const fullNameWithSetter = computed({
      get: () => firstName.value + ' ' + lastName.value,
      set: (newValue) => {
        const names = newValue.split(' ')
        firstName.value = names[0]
        lastName.value = names[names.length - 1]
      }
    })
    
    // 侦听单个数据源
    watch(firstName, (newVal, oldVal) => {
      console.log(`firstName changed from ${oldVal} to ${newVal}`)
    })
    
    // 侦听多个数据源
    watch([firstName, lastName], ([newFirst, newLast], [oldFirst, oldLast]) => {
      console.log('name changed')
    })
    
    // 侦听响应式对象
    watch(
      userList,
      (newVal, oldVal) => {
        console.log('userList changed')
      },
      { deep: true, immediate: true }
    )
    
    // watchEffect - 自动收集依赖
    watchEffect(() => {
      console.log(`Full name is: ${firstName.value} ${lastName.value}`)
    })
    
    return {
      firstName,
      lastName,
      fullName,
      fullNameWithSetter
    }
  }
}

// 使用场景对比
/*
computed：
- 模板中的复杂表达式
- 基于其他数据计算得出的值
- 需要缓存的计算结果
- 依赖的数据不变时不重新计算

watch：
- 数据变化时需要执行异步操作
- 数据变化时需要执行开销较大的操作
- 需要在数据变化时执行副作用
- 需要访问变化前后的值
*/
```

## 组件通信

### 4. 父子组件通信
**问题**: Vue 中父子组件之间如何通信？

**答案**:
```javascript
// 父组件
<template>
  <div>
    <!-- 1. Props 向下传递数据 -->
    <child-component 
      :message="parentMessage"
      :user="userInfo"
      @update-message="handleUpdateMessage"
      @custom-event="handleCustomEvent"
    />
    
    <!-- 2. 使用 v-model -->
    <input-component v-model="inputValue" />
    
    <!-- 3. 使用 .sync 修饰符 (Vue 2) -->
    <dialog-component :visible.sync="dialogVisible" />
    
    <!-- Vue 3 中使用 v-model:propName -->
    <dialog-component v-model:visible="dialogVisible" />
  </div>
</template>

<script>
import ChildComponent from './ChildComponent.vue'
import InputComponent from './InputComponent.vue'
import DialogComponent from './DialogComponent.vue'

export default {
  components: {
    ChildComponent,
    InputComponent,
    DialogComponent
  },
  
  data() {
    return {
      parentMessage: 'Hello from parent',
      userInfo: { name: 'John', age: 30 },
      inputValue: '',
      dialogVisible: false
    }
  },
  
  methods: {
    // 2. 监听子组件事件
    handleUpdateMessage(newMessage) {
      this.parentMessage = newMessage
    },
    
    handleCustomEvent(payload) {
      console.log('Custom event received:', payload)
    }
  }
}
</script>

// 子组件
<template>
  <div>
    <p>{{ message }}</p>
    <p>User: {{ user.name }}, Age: {{ user.age }}</p>
    
    <button @click="updateParentMessage">
      Update Parent Message
    </button>
    
    <button @click="emitCustomEvent">
      Emit Custom Event
    </button>
  </div>
</template>

<script>
export default {
  // 1. 接收 Props
  props: {
    message: {
      type: String,
      required: true,
      default: 'Default message'
    },
    user: {
      type: Object,
      default: () => ({})
    }
  },
  
  methods: {
    updateParentMessage() {
      // 2. 向父组件发射事件
      this.$emit('update-message', 'Updated from child')
    },
    
    emitCustomEvent() {
      this.$emit('custom-event', {
        timestamp: Date.now(),
        data: 'Custom data'
      })
    }
  }
}
</script>

// Input 组件 (v-model 实现)
<template>
  <input 
    :value="value"
    @input="$emit('input', $event.target.value)"
  />
</template>

<script>
export default {
  props: ['value']
}
</script>

// Vue 3 中的 v-model
<template>
  <input 
    :value="modelValue"
    @input="$emit('update:modelValue', $event.target.value)"
  />
</template>

<script>
export default {
  props: ['modelValue'],
  emits: ['update:modelValue']
}
</script>

// Dialog 组件 (.sync 修饰符)
<template>
  <div v-if="visible" class="dialog">
    <button @click="close">Close</button>
  </div>
</template>

<script>
export default {
  props: ['visible'],
  
  methods: {
    close() {
      // Vue 2 .sync
      this.$emit('update:visible', false)
      
      // Vue 3 v-model:visible
      this.$emit('update:visible', false)
    }
  }
}
</script>

// Vue 3 Composition API 版本
<script setup>
import { defineProps, defineEmits } from 'vue'

const props = defineProps({
  message: String,
  user: Object
})

const emit = defineEmits(['update-message', 'custom-event'])

const updateParentMessage = () => {
  emit('update-message', 'Updated from child')
}

const emitCustomEvent = () => {
  emit('custom-event', {
    timestamp: Date.now(),
    data: 'Custom data'
  })
}
</script>
```

### 5. 跨组件通信
**问题**: Vue 中如何实现非父子组件通信？

**答案**:
```javascript
// 1. Event Bus (Vue 2)
// event-bus.js
import Vue from 'vue'
export const EventBus = new Vue()

// 组件 A - 发送事件
import { EventBus } from './event-bus'

export default {
  methods: {
    sendMessage() {
      EventBus.$emit('message-sent', {
        from: 'ComponentA',
        content: 'Hello from A'
      })
    }
  }
}

// 组件 B - 接收事件
import { EventBus } from './event-bus'

export default {
  created() {
    EventBus.$on('message-sent', this.handleMessage)
  },
  
  beforeDestroy() {
    EventBus.$off('message-sent', this.handleMessage)
  },
  
  methods: {
    handleMessage(payload) {
      console.log('Message received:', payload)
    }
  }
}

// 2. Provide/Inject
// 祖先组件
export default {
  provide() {
    return {
      theme: 'dark',
      api: this.api,
      // 提供响应式数据
      reactiveData: this.reactiveData
    }
  },
  
  data() {
    return {
      api: {
        getData: () => fetch('/api/data'),
        postData: (data) => fetch('/api/data', { method: 'POST', body: data })
      },
      reactiveData: 'some value'
    }
  }
}

// 后代组件
export default {
  inject: ['theme', 'api', 'reactiveData'],
  
  created() {
    console.log('Theme:', this.theme)
    this.api.getData().then(data => {
      console.log('Data:', data)
    })
  }
}

// 3. Vuex 状态管理
// store/index.js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

export default new Vuex.Store({
  state: {
    message: '',
    user: null,
    notifications: []
  },
  
  mutations: {
    SET_MESSAGE(state, message) {
      state.message = message
    },
    
    SET_USER(state, user) {
      state.user = user
    },
    
    ADD_NOTIFICATION(state, notification) {
      state.notifications.push(notification)
    }
  },
  
  actions: {
    async fetchUser({ commit }, userId) {
      const user = await api.getUser(userId)
      commit('SET_USER', user)
    },
    
    sendNotification({ commit }, notification) {
      commit('ADD_NOTIFICATION', {
        id: Date.now(),
        ...notification,
        timestamp: new Date()
      })
    }
  },
  
  getters: {
    unreadNotifications: state => {
      return state.notifications.filter(n => !n.read)
    }
  }
})

// 组件中使用 Vuex
import { mapState, mapMutations, mapActions, mapGetters } from 'vuex'

export default {
  computed: {
    ...mapState(['message', 'user']),
    ...mapGetters(['unreadNotifications'])
  },
  
  methods: {
    ...mapMutations(['SET_MESSAGE']),
    ...mapActions(['fetchUser', 'sendNotification']),
    
    updateMessage(newMessage) {
      this.SET_MESSAGE(newMessage)
    }
  }
}

// 4. Vue 3 中的替代方案
// 使用 mitt 作为事件总线
import mitt from 'mitt'

const emitter = mitt()

// 发送事件
emitter.emit('message', { content: 'Hello' })

// 监听事件
emitter.on('message', (payload) => {
  console.log('Message:', payload)
})

// 5. Vue 3 Composition API + Provide/Inject
// 祖先组件
import { provide, ref, reactive } from 'vue'

export default {
  setup() {
    const theme = ref('dark')
    const user = reactive({ name: 'John', role: 'admin' })
    
    const api = {
      updateUser: (newData) => {
        Object.assign(user, newData)
      }
    }
    
    provide('theme', theme)
    provide('user', user)
    provide('api', api)
    
    return { theme, user }
  }
}

// 后代组件
import { inject } from 'vue'

export default {
  setup() {
    const theme = inject('theme')
    const user = inject('user')
    const api = inject('api')
    
    const updateUserName = () => {
      api.updateUser({ name: 'Jane' })
    }
    
    return {
      theme,
      user,
      updateUserName
    }
  }
}
```

## 指令和插槽

### 6. 自定义指令
**问题**: 如何创建和使用自定义指令？

**答案**:
```javascript
// Vue 2 自定义指令
// 全局指令
Vue.directive('focus', {
  // 只调用一次，指令第一次绑定到元素时调用
  bind(el, binding, vnode) {
    console.log('bind')
  },
  
  // 被绑定元素插入父节点时调用
  inserted(el, binding, vnode) {
    el.focus()
  },
  
  // 所在组件的 VNode 更新时调用
  update(el, binding, vnode, oldVnode) {
    console.log('update')
  },
  
  // 指令所在组件的 VNode 及其子 VNode 全部更新后调用
  componentUpdated(el, binding, vnode, oldVnode) {
    console.log('componentUpdated')
  },
  
  // 只调用一次，指令与元素解绑时调用
  unbind(el, binding, vnode) {
    console.log('unbind')
  }
})

// 局部指令
export default {
  directives: {
    focus: {
      inserted(el) {
        el.focus()
      }
    }
  }
}

// Vue 3 自定义指令
const app = createApp({})

app.directive('focus', {
  // 在绑定元素的 attribute 前或事件监听器应用前调用
  created(el, binding, vnode, prevVnode) {},
  
  // 在元素被插入到 DOM 前调用
  beforeMount(el, binding, vnode, prevVnode) {},
  
  // 在绑定元素的父组件及他自己的所有子节点都挂载完成后调用
  mounted(el, binding, vnode, prevVnode) {
    el.focus()
  },
  
  // 绑定元素的父组件更新前调用
  beforeUpdate(el, binding, vnode, prevVnode) {},
  
  // 在绑定元素的父组件及他自己的所有子节点都更新后调用
  updated(el, binding, vnode, prevVnode) {},
  
  // 绑定元素的父组件卸载前调用
  beforeUnmount(el, binding, vnode, prevVnode) {},
  
  // 绑定元素的父组件卸载后调用
  unmounted(el, binding, vnode, prevVnode) {}
})

// 常用自定义指令示例

// 1. 权限指令
Vue.directive('permission', {
  inserted(el, binding) {
    const { value } = binding
    const userPermissions = store.state.user.permissions
    
    if (!userPermissions.includes(value)) {
      el.style.display = 'none'
      // 或者移除元素
      // el.parentNode && el.parentNode.removeChild(el)
    }
  }
})

// 使用: <button v-permission="'admin'">管理员按钮</button>

// 2. 懒加载指令
Vue.directive('lazy', {
  bind(el, binding) {
    const io = new IntersectionObserver(entries => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          const img = entry.target
          img.src = binding.value
          img.onload = () => {
            img.classList.add('loaded')
          }
          io.unobserve(img)
        }
      })
    })
    
    el.io = io
    io.observe(el)
  },
  
  unbind(el) {
    el.io.disconnect()
  }
})

// 使用: <img v-lazy="imageUrl" src="placeholder.jpg">

// 3. 复制到剪贴板指令
Vue.directive('copy', {
  bind(el, binding) {
    el.addEventListener('click', () => {
      const text = binding.value
      navigator.clipboard.writeText(text).then(() => {
        console.log('复制成功')
      }).catch(err => {
        console.error('复制失败', err)
      })
    })
  }
})

// 使用: <button v-copy="textToCopy">复制</button>

// 4. 防抖指令
Vue.directive('debounce', {
  inserted(el, binding) {
    const { value, arg } = binding
    const delay = arg || 300
    
    let timer = null
    
    el.addEventListener('click', () => {
      clearTimeout(timer)
      timer = setTimeout(() => {
        value()
      }, delay)
    })
  }
})

// 使用: <button v-debounce:500="handleClick">防抖按钮</button>

// 5. 拖拽指令
Vue.directive('drag', {
  bind(el) {
    el.style.position = 'absolute'
    el.style.cursor = 'move'
    
    let isDragging = false
    let startX, startY, initialX, initialY
    
    el.addEventListener('mousedown', (e) => {
      isDragging = true
      startX = e.clientX
      startY = e.clientY
      initialX = el.offsetLeft
      initialY = el.offsetTop
    })
    
    document.addEventListener('mousemove', (e) => {
      if (!isDragging) return
      
      const deltaX = e.clientX - startX
      const deltaY = e.clientY - startY
      
      el.style.left = initialX + deltaX + 'px'
      el.style.top = initialY + deltaY + 'px'
    })
    
    document.addEventListener('mouseup', () => {
      isDragging = false
    })
  }
})

// 使用: <div v-drag>可拖拽元素</div>

// 6. 长按指令
Vue.directive('longpress', {
  bind(el, binding) {
    let timer = null
    
    const start = () => {
      timer = setTimeout(() => {
        binding.value()
      }, 1000)
    }
    
    const cancel = () => {
      clearTimeout(timer)
    }
    
    el.addEventListener('mousedown', start)
    el.addEventListener('mouseup', cancel)
    el.addEventListener('mouseleave', cancel)
    el.addEventListener('touchstart', start)
    el.addEventListener('touchend', cancel)
  }
})

// 使用: <button v-longpress="handleLongPress">长按我</button>
```

### 7. 插槽
**问题**: Vue 插槽的使用方法和作用域插槽

**答案**:
```vue
<!-- 1. 基础插槽 -->
<!-- 子组件 Card.vue -->
<template>
  <div class="card">
    <div class="card-header">
      <slot name="header">默认标题</slot>
    </div>
    
    <div class="card-body">
      <slot>默认内容</slot>
    </div>
    
    <div class="card-footer">
      <slot name="footer"></slot>
    </div>
  </div>
</template>

<!-- 父组件使用 -->
<template>
  <Card>
    <!-- 具名插槽 -->
    <template v-slot:header>
      <h3>自定义标题</h3>
    </template>
    
    <!-- 默认插槽 -->
    <p>自定义内容</p>
    
    <!-- 具名插槽简写 -->
    <template #footer>
      <button>确认</button>
      <button>取消</button>
    </template>
  </Card>
</template>

<!-- 2. 作用域插槽 -->
<!-- 子组件 List.vue -->
<template>
  <div class="list">
    <div 
      v-for="item in items" 
      :key="item.id"
      class="list-item"
    >
      <!-- 将数据传递给插槽 -->
      <slot 
        :item="item" 
        :index="index"
        :isEven="index % 2 === 0"
      >
        <!-- 默认内容 -->
        <span>{{ item.name }}</span>
      </slot>
    </div>
  </div>
</template>

<script>
export default {
  props: ['items']
}
</script>

<!-- 父组件使用作用域插槽 -->
<template>
  <List :items="userList">
    <!-- 解构插槽 prop -->
    <template v-slot="{ item, index, isEven }">
      <div :class="{ 'even': isEven }">
        <img :src="item.avatar" alt="">
        <h4>{{ item.name }}</h4>
        <p>{{ item.email }}</p>
        <span>第 {{ index + 1 }} 个用户</span>
      </div>
    </template>
  </List>
</template>

<!-- 3. 动态插槽名 -->
<template>
  <Card>
    <template v-slot:[dynamicSlotName]>
      <p>动态插槽内容</p>
    </template>
  </Card>
</template>

<script>
export default {
  data() {
    return {
      dynamicSlotName: 'header'
    }
  }
}
</script>

<!-- 4. 复杂的作用域插槽示例 - 表格组件 -->
<template>
  <div class="table">
    <table>
      <thead>
        <tr>
          <th v-for="column in columns" :key="column.key">
            {{ column.title }}
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(row, rowIndex) in data" :key="rowIndex">
          <td v-for="column in columns" :key="column.key">
            <!-- 列插槽 -->
            <slot 
              :name="column.key"
              :row="row"
              :column="column"
              :rowIndex="rowIndex"
              :value="row[column.key]"
            >
              <!-- 默认显示 -->
              {{ row[column.key] }}
            </slot>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<!-- 使用表格组件 -->
<template>
  <DataTable :columns="columns" :data="tableData">
    <!-- 自定义状态列 -->
    <template #status="{ value }">
      <span :class="getStatusClass(value)">
        {{ getStatusText(value) }}
      </span>
    </template>
    
    <!-- 自定义操作列 -->
    <template #actions="{ row, rowIndex }">
      <button @click="editRow(row, rowIndex)">编辑</button>
      <button @click="deleteRow(rowIndex)">删除</button>
    </template>
    
    <!-- 自定义头像列 -->
    <template #avatar="{ value, row }">
      <img 
        :src="value || defaultAvatar" 
        :alt="row.name"
        class="avatar"
      >
    </template>
  </DataTable>
</template>

<script>
export default {
  data() {
    return {
      columns: [
        { key: 'avatar', title: '头像' },
        { key: 'name', title: '姓名' },
        { key: 'email', title: '邮箱' },
        { key: 'status', title: '状态' },
        { key: 'actions', title: '操作' }
      ],
      tableData: [
        {
          id: 1,
          name: 'John Doe',
          email: 'john@example.com',
          status: 'active',
          avatar: 'avatar1.jpg'
        }
      ]
    }
  },
  
  methods: {
    getStatusClass(status) {
      return {
        'status-active': status === 'active',
        'status-inactive': status === 'inactive'
      }
    },
    
    getStatusText(status) {
      const statusMap = {
        active: '激活',
        inactive: '未激活'
      }
      return statusMap[status] || status
    },
    
    editRow(row, index) {
      console.log('编辑', row, index)
    },
    
    deleteRow(index) {
      this.tableData.splice(index, 1)
    }
  }
}
</script>

<!-- 5. Vue 3 中的多个默认插槽内容 -->
<template>
  <!-- Vue 3 支持多个根节点 -->
  <header>
    <slot name="header"></slot>
  </header>
  
  <main>
    <slot></slot>
  </main>
  
  <footer>
    <slot name="footer"></slot>
  </footer>
</template>

<!-- 6. 渲染函数中的插槽 -->
<script>
// Vue 2
export default {
  render(h) {
    return h('div', [
      h('header', this.$slots.header),
      h('main', this.$slots.default),
      h('footer', this.$slots.footer)
    ])
  }
}

// Vue 3
import { h } from 'vue'

export default {
  render() {
    return h('div', [
      h('header', this.$slots.header?.()),
      h('main', this.$slots.default?.()),
      h('footer', this.$slots.footer?.())
    ])
  }
}
</script>
```

## 高级特性

### 8. 混入 (Mixins)
**问题**: Vue 混入的使用方法和注意事项

**答案**:
```javascript
// 1. 基础混入
const formMixin = {
  data() {
    return {
      isLoading: false,
      errors: {}
    }
  },
  
  methods: {
    resetForm() {
      this.isLoading = false
      this.errors = {}
      // 重置表单数据
      if (this.formData) {
        Object.keys(this.formData).forEach(key => {
          this.formData[key] = ''
        })
      }
    },
    
    validateForm() {
      const errors = {}
      
      // 通用验证逻辑
      if (this.requiredFields) {
        this.requiredFields.forEach(field => {
          if (!this.formData[field]) {
            errors[field] = `${field} 是必填项`
          }
        })
      }
      
      this.errors = errors
      return Object.keys(errors).length === 0
    },
    
    async submitForm(url, data) {
      if (!this.validateForm()) return
      
      this.isLoading = true
      try {
        const response = await fetch(url, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(data)
        })
        const result = await response.json()
        this.handleSubmitSuccess(result)
      } catch (error) {
        this.handleSubmitError(error)
      } finally {
        this.isLoading = false
      }
    },
    
    handleSubmitSuccess(result) {
      // 子组件可以重写这个方法
      console.log('提交成功', result)
    },
    
    handleSubmitError(error) {
      // 子组件可以重写这个方法
      console.error('提交失败', error)
    }
  }
}

// 2. 生命周期混入
const loggerMixin = {
  created() {
    console.log(`${this.$options.name || 'Component'} created`)
  },
  
  mounted() {
    console.log(`${this.$options.name || 'Component'} mounted`)
  },
  
  beforeDestroy() {
    console.log(`${this.$options.name || 'Component'} before destroy`)
  }
}

// 3. 工具方法混入
const utilsMixin = {
  methods: {
    formatDate(date, format = 'YYYY-MM-DD') {
      // 日期格式化逻辑
      return moment(date).format(format)
    },
    
    debounce(func, delay = 300) {
      let timeoutId
      return function (...args) {
        clearTimeout(timeoutId)
        timeoutId = setTimeout(() => func.apply(this, args), delay)
      }
    },
    
    deepClone(obj) {
      return JSON.parse(JSON.stringify(obj))
    },
    
    isEmail(email) {
      const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
      return re.test(email)
    }
  }
}

// 4. 组件中使用混入
export default {
  name: 'UserForm',
  mixins: [formMixin, loggerMixin, utilsMixin],
  
  data() {
    return {
      formData: {
        name: '',
        email: '',
        age: ''
      },
      requiredFields: ['name', 'email']
    }
  },
  
  methods: {
    async handleSubmit() {
      // 使用混入中的方法
      await this.submitForm('/api/users', this.formData)
    },
    
    // 重写混入中的方法
    handleSubmitSuccess(result) {
      console.log('用户创建成功', result)
      this.$router.push('/users')
    },
    
    handleSubmitError(error) {
      this.errors = error.response?.data?.errors || {}
    }
  }
}

// 5. 全局混入
Vue.mixin({
  mounted() {
    console.log('全局混入：组件已挂载')
  },
  
  methods: {
    $message(text, type = 'info') {
      // 全局消息方法
      this.$notify({
        message: text,
        type: type
      })
    }
  }
})

// 6. 高级混入模式
const apiMixin = {
  data() {
    return {
      apiCache: new Map()
    }
  },
  
  methods: {
    async apiCall(url, options = {}) {
      const cacheKey = `${url}_${JSON.stringify(options)}`
      
      // 检查缓存
      if (this.apiCache.has(cacheKey)) {
        return this.apiCache.get(cacheKey)
      }
      
      try {
        const response = await fetch(url, {
          headers: {
            'Authorization': `Bearer ${this.$store.state.auth.token}`,
            'Content-Type': 'application/json'
          },
          ...options
        })
        
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`)
        }
        
        const data = await response.json()
        
        // 缓存结果
        this.apiCache.set(cacheKey, data)
        
        return data
      } catch (error) {
        console.error('API 调用失败:', error)
        throw error
      }
    },
    
    clearApiCache() {
      this.apiCache.clear()
    }
  },
  
  beforeDestroy() {
    // 清理缓存
    this.clearApiCache()
  }
}

// 7. 混入的注意事项和限制
/*
合并策略：
- data: 递归合并，组件数据优先
- methods, components, directives: 组件优先
- 生命周期钩子: 混入先执行，然后是组件
- watch: 都会执行

问题：
1. 命名冲突
2. 依赖不明确
3. 难以追踪来源
4. 可能导致复杂的继承链

替代方案（Vue 3）：
- Composition API
- 组合式函数 (Composables)
*/

// Vue 3 中的替代方案 - Composables
import { ref, reactive, onMounted, onUnmounted } from 'vue'

// 表单组合式函数
export function useForm(initialData = {}, requiredFields = []) {
  const formData = reactive({ ...initialData })
  const errors = ref({})
  const isLoading = ref(false)
  
  const resetForm = () => {
    Object.assign(formData, initialData)
    errors.value = {}
    isLoading.value = false
  }
  
  const validateForm = () => {
    const newErrors = {}
    
    requiredFields.forEach(field => {
      if (!formData[field]) {
        newErrors[field] = `${field} 是必填项`
      }
    })
    
    errors.value = newErrors
    return Object.keys(newErrors).length === 0
  }
  
  return {
    formData,
    errors,
    isLoading,
    resetForm,
    validateForm
  }
}

// 在组件中使用
export default {
  setup() {
    const { formData, errors, isLoading, resetForm, validateForm } = useForm(
      { name: '', email: '' },
      ['name', 'email']
    )
    
    return {
      formData,
      errors,
      isLoading,
      resetForm,
      validateForm
    }
  }
}
```

### 9. 渲染函数和 JSX
**问题**: Vue 中渲染函数的使用方法

**答案**:
```javascript
// Vue 2 渲染函数
export default {
  name: 'MyComponent',
  
  props: {
    level: {
      type: Number,
      required: true
    },
    title: String
  },
  
  render(h) {
    // h 函数 (createElement)
    return h(
      `h${this.level}`, // 标签名
      {
        // 属性对象
        class: {
          'heading': true,
          [`heading-${this.level}`]: true
        },
        style: {
          color: 'blue',
          fontSize: `${20 + this.level * 2}px`
        },
        attrs: {
          id: `heading-${this.level}`
        },
        domProps: {
          innerHTML: this.title
        },
        on: {
          click: this.handleClick
        }
      },
      this.$slots.default // 子节点
    )
  },
  
  methods: {
    handleClick() {
      this.$emit('heading-click', this.level)
    }
  }
}

// 更复杂的渲染函数示例
export default {
  props: ['items'],
  
  render(h) {
    return h('div', { class: 'list-container' }, [
      // 创建标题
      h('h2', '列表标题'),
      
      // 创建列表
      h('ul', { class: 'item-list' }, 
        this.items.map((item, index) => 
          h('li', {
            key: item.id,
            class: {
              'item': true,
              'item-active': item.active
            },
            on: {
              click: () => this.handleItemClick(item, index)
            }
          }, [
            // 项目图标
            h('i', { class: `icon-${item.type}` }),
            
            // 项目内容
            h('span', { class: 'item-text' }, item.text),
            
            // 条件渲染按钮
            item.showButton && h('button', {
              on: { click: (e) => this.handleButtonClick(e, item) }
            }, '操作')
          ])
        )
      ),
      
      // 条件渲染
      this.items.length === 0 && h('p', { class: 'empty' }, '暂无数据')
    ])
  },
  
  methods: {
    handleItemClick(item, index) {
      console.log('Item clicked:', item, index)
    },
    
    handleButtonClick(event, item) {
      event.stopPropagation()
      console.log('Button clicked:', item)
    }
  }
}

// Vue 3 渲染函数
import { h, ref, reactive } from 'vue'

export default {
  props: {
    level: Number,
    title: String
  },
  
  setup(props, { emit, slots }) {
    const handleClick = () => {
      emit('heading-click', props.level)
    }
    
    return () => h(
      `h${props.level}`,
      {
        class: `heading heading-${props.level}`,
        onClick: handleClick
      },
      slots.default?.()
    )
  }
}

// Vue 2 中使用 JSX (需要 babel 插件)
export default {
  props: ['message', 'type'],
  
  render() {
    const messageClass = `message message-${this.type}`
    
    return (
      <div class={messageClass}>
        <div class="message-header">
          {this.$slots.header || <span>默认标题</span>}
        </div>
        
        <div class="message-body">
          {this.message}
        </div>
        
        <div class="message-footer">
          <button onClick={this.handleClose}>关闭</button>
        </div>
      </div>
    )
  },
  
  methods: {
    handleClose() {
      this.$emit('close')
    }
  }
}

// Vue 3 中使用 JSX
import { defineComponent, ref } from 'vue'

export default defineComponent({
  props: {
    items: Array,
    loading: Boolean
  },
  
  setup(props, { emit }) {
    const selectedItems = ref([])
    
    const toggleItem = (item) => {
      const index = selectedItems.value.indexOf(item.id)
      if (index > -1) {
        selectedItems.value.splice(index, 1)
      } else {
        selectedItems.value.push(item.id)
      }
    }
    
    const renderItem = (item) => {
      const isSelected = selectedItems.value.includes(item.id)
      
      return (
        <div 
          class={['item', { 'item-selected': isSelected }]}
          onClick={() => toggleItem(item)}
        >
          <input 
            type="checkbox" 
            checked={isSelected}
            onChange={() => toggleItem(item)}
          />
          <span class="item-title">{item.title}</span>
          <span class="item-description">{item.description}</span>
        </div>
      )
    }
    
    return () => (
      <div class="item-selector">
        {props.loading ? (
          <div class="loading">加载中...</div>
        ) : (
          <div class="item-list">
            {props.items?.map(renderItem)}
          </div>
        )}
        
        <div class="selected-count">
          已选择: {selectedItems.value.length} 项
        </div>
      </div>
    )
  }
})

// 函数式组件
// Vue 2
export default {
  functional: true,
  
  props: {
    type: String,
    size: String
  },
  
  render(h, { props, children, data }) {
    return h(
      'button',
      {
        ...data,
        class: [
          'btn',
          `btn-${props.type || 'default'}`,
          `btn-${props.size || 'medium'}`
        ]
      },
      children
    )
  }
}

// Vue 3 函数式组件
import { h } from 'vue'

const Button = (props, { slots }) => {
  return h(
    'button',
    {
      class: [
        'btn',
        `btn-${props.type || 'default'}`,
        `btn-${props.size || 'medium'}`
      ]
    },
    slots.default?.()
  )
}

Button.props = {
  type: String,
  size: String
}

export default Button

// 高阶组件 (HOC)
function withLoading(WrappedComponent) {
  return {
    props: {
      loading: Boolean,
      ...WrappedComponent.props
    },
    
    render(h) {
      if (this.loading) {
        return h('div', { class: 'loading' }, '加载中...')
      }
      
      return h(WrappedComponent, {
        props: this.$props,
        on: this.$listeners,
        scopedSlots: this.$scopedSlots
      })
    }
  }
}

// 使用高阶组件
const LoadableUserList = withLoading(UserList)

// 在模板中使用
// <LoadableUserList :loading="isLoading" :users="userList" />
```

### 10. 性能优化
**问题**: Vue 应用的性能优化技巧

**答案**:
```javascript
// 1. 组件懒加载
// 路由懒加载
const routes = [
  {
    path: '/user',
    component: () => import('./views/User.vue')
  },
  {
    path: '/admin',
    component: () => import(/* webpackChunkName: "admin" */ './views/Admin.vue')
  }
]

// 组件懒加载
export default {
  components: {
    HeavyComponent: () => import('./HeavyComponent.vue')
  }
}

// 2. keep-alive 缓存组件
<template>
  <keep-alive :include="['ComponentA', 'ComponentB']" :max="10">
    <router-view />
  </keep-alive>
  
  <!-- 动态缓存 -->
  <keep-alive :include="cachedComponents">
    <component :is="currentComponent" />
  </keep-alive>
</template>

<script>
export default {
  data() {
    return {
      cachedComponents: ['UserList', 'ProductList']
    }
  },
  
  methods: {
    addToCache(componentName) {
      if (!this.cachedComponents.includes(componentName)) {
        this.cachedComponents.push(componentName)
      }
    },
    
    removeFromCache(componentName) {
      const index = this.cachedComponents.indexOf(componentName)
      if (index > -1) {
        this.cachedComponents.splice(index, 1)
      }
    }
  }
}
</script>

// 3. v-if vs v-show 优化
<template>
  <!-- 频繁切换使用 v-show -->
  <div v-show="isVisible">频繁切换的内容</div>
  
  <!-- 条件很少改变使用 v-if -->
  <div v-if="userRole === 'admin'">管理员内容</div>
  
  <!-- 昂贵的条件渲染 -->
  <template v-if="shouldRenderExpensiveComponent">
    <ExpensiveComponent />
  </template>
</template>

// 4. 列表渲染优化
<template>
  <!-- 使用唯一 key -->
  <div v-for="item in items" :key="item.id">
    {{ item.name }}
  </div>
  
  <!-- 避免在 v-for 中使用 v-if -->
  <!-- 不好的写法 -->
  <div v-for="user in users" :key="user.id" v-if="user.active">
    {{ user.name }}
  </div>
  
  <!-- 好的写法 -->
  <div v-for="user in activeUsers" :key="user.id">
    {{ user.name }}
  </div>
</template>

<script>
export default {
  computed: {
    // 预过滤数据
    activeUsers() {
      return this.users.filter(user => user.active)
    }
  }
}
</script>

// 5. 计算属性缓存优化
export default {
  data() {
    return {
      users: [],
      filter: ''
    }
  },
  
  computed: {
    // 缓存计算结果
    filteredUsers() {
      if (!this.filter) return this.users
      
      return this.users.filter(user => 
        user.name.toLowerCase().includes(this.filter.toLowerCase())
      )
    },
    
    // 避免在模板中进行复杂计算
    expensiveCalculation() {
      return this.users.reduce((sum, user) => {
        return sum + this.calculateUserScore(user)
      }, 0)
    }
  },
  
  methods: {
    calculateUserScore(user) {
      // 复杂计算逻辑
      return user.posts * 10 + user.comments * 2
    }
  }
}

// 6. 事件监听器优化
export default {
  mounted() {
    // 使用 passive 监听器
    this.$el.addEventListener('scroll', this.handleScroll, { passive: true })
    
    // 防抖滚动事件
    this.debouncedScroll = this.debounce(this.handleScroll, 100)
    window.addEventListener('scroll', this.debouncedScroll)
  },
  
  beforeDestroy() {
    // 清理事件监听器
    this.$el.removeEventListener('scroll', this.handleScroll)
    window.removeEventListener('scroll', this.debouncedScroll)
  },
  
  methods: {
    handleScroll() {
      // 滚动处理逻辑
    },
    
    debounce(func, delay) {
      let timeoutId
      return function (...args) {
        clearTimeout(timeoutId)
        timeoutId = setTimeout(() => func.apply(this, args), delay)
      }
    }
  }
}

// 7. 虚拟列表 (大数据渲染)
<template>
  <div class="virtual-list" @scroll="handleScroll" ref="container">
    <div :style="{ height: totalHeight + 'px' }">
      <div 
        v-for="item in visibleItems" 
        :key="item.index"
        :style="{ 
          position: 'absolute',
          top: item.top + 'px',
          height: itemHeight + 'px'
        }"
      >
        <slot :item="item.data" :index="item.index">
          {{ item.data }}
        </slot>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    items: Array,
    itemHeight: {
      type: Number,
      default: 50
    },
    containerHeight: {
      type: Number,
      default: 400
    }
  },
  
  data() {
    return {
      scrollTop: 0
    }
  },
  
  computed: {
    totalHeight() {
      return this.items.length * this.itemHeight
    },
    
    visibleCount() {
      return Math.ceil(this.containerHeight / this.itemHeight) + 2
    },
    
    startIndex() {
      return Math.floor(this.scrollTop / this.itemHeight)
    },
    
    visibleItems() {
      const start = this.startIndex
      const end = Math.min(start + this.visibleCount, this.items.length)
      
      return this.items.slice(start, end).map((item, index) => ({
        data: item,
        index: start + index,
        top: (start + index) * this.itemHeight
      }))
    }
  },
  
  methods: {
    handleScroll(event) {
      this.scrollTop = event.target.scrollTop
    }
  }
}
</script>

// 8. 组件拆分和按需渲染
export default {
  data() {
    return {
      showDetails: false,
      activeTab: 'basic'
    }
  },
  
  components: {
    // 按需加载重型组件
    UserDetails: () => import('./UserDetails.vue'),
    UserCharts: () => import('./UserCharts.vue')
  },
  
  template: `
    <div>
      <BasicUserInfo />
      
      <!-- 只有需要时才渲染 -->
      <UserDetails v-if="showDetails" />
      
      <!-- 标签页按需渲染 -->
      <component :is="currentTabComponent" />
    </div>
  `,
  
  computed: {
    currentTabComponent() {
      const components = {
        basic: 'BasicInfo',
        charts: 'UserCharts',
        settings: 'UserSettings'
      }
      return components[this.activeTab]
    }
  }
}

// 9. 内存泄漏防护
export default {
  data() {
    return {
      timer: null,
      observer: null
    }
  },
  
  mounted() {
    // 定时器
    this.timer = setInterval(() => {
      this.updateData()
    }, 1000)
    
    // 观察器
    this.observer = new IntersectionObserver(this.handleIntersect)
    this.observer.observe(this.$el)
  },
  
  beforeDestroy() {
    // 清理定时器
    if (this.timer) {
      clearInterval(this.timer)
    }
    
    // 清理观察器
    if (this.observer) {
      this.observer.disconnect()
    }
    
    // 清理事件监听器
    this.$bus?.$off('custom-event', this.handleCustomEvent)
  }
}

// 10. Vue 3 性能优化
import { ref, computed, watchEffect, onUnmounted } from 'vue'

export default {
  setup() {
    const count = ref(0)
    const expensiveValue = ref(null)
    
    // 使用 watchEffect 自动追踪依赖
    const stopWatcher = watchEffect(() => {
      if (count.value > 100) {
        expensiveValue.value = performExpensiveCalculation(count.value)
      }
    })
    
    // 清理
    onUnmounted(() => {
      stopWatcher()
    })
    
    return {
      count,
      expensiveValue
    }
  }
}

// 11. 代码分割和预加载
// webpack 配置
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all'
        },
        common: {
          name: 'common',
          minChunks: 2,
          priority: 5,
          reuseExistingChunk: true
        }
      }
    }
  }
}

// 预加载关键路由
const routes = [
  {
    path: '/',
    component: Home,
    meta: { preload: true }
  },
  {
    path: '/user',
    component: () => import(
      /* webpackPreload: true */
      './views/User.vue'
    )
  }
]
```

---

## Vue 3 Composition API 深度解析

### 11. watch vs watchEffect 深度对比

**问题**: watch 和 watchEffect 的区别是什么？各自的使用场景？

**答案**:

#### 基本差异对比

```javascript
import { ref, reactive, watch, watchEffect } from 'vue'

export default {
  setup() {
    const count = ref(0)
    const state = reactive({ name: 'Vue', version: 3 })

    // 1. watch - 显式指定依赖
    watch(count, (newVal, oldVal) => {
      console.log(`count changed: ${oldVal} -> ${newVal}`)
    })

    // 监听多个数据源
    watch([count, () => state.name], ([newCount, newName], [oldCount, oldName]) => {
      console.log('Multiple values changed:', { newCount, newName, oldCount, oldName })
    })

    // 2. watchEffect - 自动依赖收集
    watchEffect(() => {
      console.log(`watchEffect: count is ${count.value}, name is ${state.name}`)
    })

    return { count, state }
  }
}
```

#### 详细特性对比

| 特性 | watch | watchEffect |
|------|--------|-------------|
| **依赖追踪** | 显式指定监听源 | 自动收集依赖 |
| **执行时机** | 数据变化时执行 | 立即执行 + 依赖变化时执行 |
| **访问旧值** | 可以访问 oldValue | 无法访问旧值 |
| **性能** | 精确控制，性能更好 | 自动追踪，便捷但可能过度触发 |
| **调试难度** | 依赖明确，易调试 | 依赖隐式，调试相对困难 |

#### 高级使用模式

```javascript
// watch 高级用法
export default {
  setup() {
    const user = reactive({
      profile: {
        name: 'Vue',
        age: 3
      },
      settings: {
        theme: 'dark'
      }
    })

    // 深度监听
    watch(
      () => user.profile,
      (newProfile, oldProfile) => {
        console.log('Profile changed:', newProfile)
      },
      {
        deep: true, // 深度监听
        immediate: true, // 立即执行
        flush: 'sync' // 同步执行
      }
    )

    // 条件性监听
    const shouldWatch = ref(true)
    const stopWatcher = watch(
      () => user.profile.name,
      (newName) => {
        if (!shouldWatch.value) return
        console.log('Name changed to:', newName)
      }
    )

    // 手动停止监听
    const handleStop = () => {
      stopWatcher()
    }

    return { user, handleStop }
  }
}
```

```javascript
// watchEffect 高级用法
export default {
  setup() {
    const apiData = ref(null)
    const loading = ref(false)
    const userId = ref(1)

    // 副作用清理
    watchEffect((onInvalidate) => {
      const controller = new AbortController()
      
      // 设置清理函数
      onInvalidate(() => {
        controller.abort()
        console.log('Request cancelled')
      })

      // 执行副作用
      loading.value = true
      fetch(`/api/user/${userId.value}`, {
        signal: controller.signal
      })
      .then(response => response.json())
      .then(data => {
        apiData.value = data
      })
      .catch(error => {
        if (error.name !== 'AbortError') {
          console.error('Fetch error:', error)
        }
      })
      .finally(() => {
        loading.value = false
      })
    })

    // 控制执行时机
    watchEffect(
      () => {
        console.log('DOM updated:', document.title)
      },
      {
        flush: 'post' // DOM 更新后执行
      }
    )

    return { apiData, loading, userId }
  }
}
```

#### 性能优化技巧

```javascript
// 性能优化实例
export default {
  setup() {
    const expensiveData = ref(null)
    const trigger = ref(0)

    // ✅ 使用 watch 精确控制依赖
    watch(trigger, async () => {
      expensiveData.value = await computeExpensiveValue()
    })

    // ❌ watchEffect 可能造成不必要的执行
    // watchEffect(async () => {
    //   // 这里可能会因为其他响应式数据变化而重复执行
    //   expensiveData.value = await computeExpensiveValue()
    // })

    // 防抖监听
    const debouncedWatch = debounce(() => {
      console.log('Debounced execution')
    }, 300)

    watch(trigger, debouncedWatch)

    return { expensiveData, trigger }
  }
}

// 防抖工具函数
function debounce(fn, delay) {
  let timeoutId
  return function (...args) {
    clearTimeout(timeoutId)
    timeoutId = setTimeout(() => fn.apply(this, args), delay)
  }
}
```

#### 使用场景选择指南

**使用 watch 的场景：**
- 需要访问变化前后的值对比
- 监听特定数据源，避免不必要的触发
- 执行异步操作，需要精确控制时机
- 性能敏感的场景
- 需要条件性执行监听逻辑

**使用 watchEffect 的场景：**
- 编写声明式的副作用逻辑
- 依赖关系复杂，手动指定困难
- 原型开发或快速迭代阶段
- 副作用逻辑相对简单
- 需要立即执行的初始化逻辑

### 12. 响应式 API 深度理解：ref、reactive、toRef、toRefs

**问题**: 详细说明 ref、reactive、toRef、toRefs 的区别和使用场景？

**答案**:

#### ref - 基本类型响应式包装

```javascript
import { ref, isRef, unref } from 'vue'

// 基本用法
const count = ref(0)
const message = ref('Hello Vue3')
const user = ref({ name: 'John', age: 25 })

console.log(count.value) // 访问值需要 .value
console.log(isRef(count)) // true

// ref 的特殊性
export default {
  setup() {
    const num = ref(10)
    const obj = ref({ a: 1, b: 2 })

    // 在模板中自动解包
    return { num, obj }
  },
  template: `
    <!-- 模板中不需要 .value -->
    <div>{{ num }}</div>
    <div>{{ obj.a }}</div>
  `
}
```

```javascript
// ref 高级特性
export default {
  setup() {
    // 1. 自定义 ref
    function useCounter(initialValue = 0) {
      return customRef((track, trigger) => {
        let value = initialValue
        return {
          get() {
            track() // 追踪依赖
            return value
          },
          set(newValue) {
            value = newValue
            trigger() // 触发更新
          }
        }
      })
    }

    // 2. 防抖 ref
    function useDebouncedRef(value, delay = 200) {
      let timeout
      return customRef((track, trigger) => {
        return {
          get() {
            track()
            return value
          },
          set(newValue) {
            clearTimeout(timeout)
            timeout = setTimeout(() => {
              value = newValue
              trigger()
            }, delay)
          }
        }
      })
    }

    const counter = useCounter(5)
    const debouncedInput = useDebouncedRef('')

    return { counter, debouncedInput }
  }
}
```

#### reactive - 对象响应式代理

```javascript
import { reactive, isReactive, readonly, isReadonly } from 'vue'

// 基本用法
const state = reactive({
  user: {
    name: 'Vue',
    profile: {
      email: 'vue@example.com',
      preferences: {
        theme: 'dark',
        language: 'zh-CN'
      }
    }
  },
  settings: {
    notifications: true
  }
})

// 深度响应式
state.user.profile.preferences.theme = 'light' // 会触发响应式更新

console.log(isReactive(state)) // true
console.log(isReactive(state.user)) // true（深度代理）
```

```javascript
// reactive 注意事项和最佳实践
export default {
  setup() {
    // ✅ 正确用法
    const state = reactive({
      list: [],
      selectedId: null,
      loading: false
    })

    // ✅ 方法也可以是响应式对象的一部分
    const actions = reactive({
      async fetchData() {
        state.loading = true
        try {
          const response = await api.getData()
          state.list = response.data
        } finally {
          state.loading = false
        }
      },
      
      selectItem(id) {
        state.selectedId = id
      }
    })

    // ❌ 避免解构赋值破坏响应式
    // const { list, loading } = state // 失去响应式

    // ✅ 正确的解构方式
    const { list, loading } = toRefs(state)

    return { state, actions, list, loading }
  }
}
```

```javascript
// 响应式代理的边界情况
export default {
  setup() {
    const state = reactive({
      nested: {
        count: 0
      }
    })

    // 注意：重新赋值会破坏响应式连接
    // ❌ 错误做法
    // state.nested = { count: 10 } // 新对象不是响应式的

    // ✅ 正确做法
    Object.assign(state.nested, { count: 10 })
    // 或者
    state.nested.count = 10

    return { state }
  }
}
```

#### toRef - 单个属性响应式转换

```javascript
import { reactive, toRef } from 'vue'

export default {
  setup() {
    const state = reactive({
      user: {
        name: 'John',
        age: 25
      },
      settings: {
        theme: 'dark'
      }
    })

    // 将响应式对象的属性转为独立的 ref
    const userName = toRef(state.user, 'name')
    const userAge = toRef(state.user, 'age')
    const theme = toRef(state.settings, 'theme')

    // 保持响应式连接
    userName.value = 'Jane' // 会同时更新 state.user.name

    console.log(state.user.name) // 'Jane'

    return { userName, userAge, theme }
  }
}
```

```javascript
// toRef 实际应用场景
export default {
  setup(props) {
    // 1. Props 转 ref（保持响应式）
    const title = toRef(props, 'title')
    
    // 2. 组合式函数中使用
    function useUserProfile(user) {
      const name = toRef(user, 'name')
      const email = toRef(user, 'email')
      
      const displayName = computed(() => 
        name.value ? name.value.toUpperCase() : 'Unknown'
      )
      
      return { name, email, displayName }
    }

    const userState = reactive({
      name: 'Vue Developer',
      email: 'dev@vue.com'
    })

    const { name, email, displayName } = useUserProfile(userState)

    return { title, name, email, displayName }
  }
}
```

#### toRefs - 批量属性响应式转换

```javascript
import { reactive, toRefs } from 'vue'

export default {
  setup() {
    const state = reactive({
      count: 0,
      message: 'Hello',
      user: {
        name: 'Vue',
        age: 3
      },
      settings: {
        theme: 'dark',
        lang: 'zh-CN'
      }
    })

    // 批量转换为 ref
    const stateRefs = toRefs(state)
    
    // 现在可以安全解构
    const { count, message, user, settings } = stateRefs

    // 保持响应式连接
    count.value++ // 同时更新 state.count

    return {
      count,
      message,
      user,
      settings
    }
  }
}
```

```javascript
// toRefs 在组合式函数中的应用
function useCounter(initialValue = 0) {
  const state = reactive({
    count: initialValue,
    doubleCount: computed(() => state.count * 2),
    isEven: computed(() => state.count % 2 === 0)
  })

  const increment = () => state.count++
  const decrement = () => state.count--
  const reset = () => { state.count = initialValue }

  // 返回时使用 toRefs 保持响应式
  return {
    ...toRefs(state),
    increment,
    decrement,
    reset
  }
}

export default {
  setup() {
    // 可以直接解构使用
    const { count, doubleCount, isEven, increment, decrement, reset } = useCounter(10)

    return {
      count,
      doubleCount,
      isEven,
      increment,
      decrement,
      reset
    }
  }
}
```

#### 响应式 API 选择指南

```javascript
// 完整的响应式数据管理示例
export default {
  setup() {
    // 1. 基本类型使用 ref
    const loading = ref(false)
    const error = ref(null)
    const searchQuery = ref('')

    // 2. 对象/数组使用 reactive
    const state = reactive({
      users: [],
      pagination: {
        page: 1,
        pageSize: 10,
        total: 0
      },
      filters: {
        status: 'active',
        role: 'all'
      }
    })

    // 3. 需要解构时使用 toRefs
    const stateRefs = toRefs(state)

    // 4. 单个属性需要传递时使用 toRef
    const currentPage = toRef(state.pagination, 'page')

    // 组合逻辑
    const fetchUsers = async () => {
      loading.value = true
      error.value = null
      
      try {
        const response = await api.getUsers({
          page: state.pagination.page,
          pageSize: state.pagination.pageSize,
          query: searchQuery.value,
          ...state.filters
        })
        
        state.users = response.data
        state.pagination.total = response.total
      } catch (err) {
        error.value = err.message
      } finally {
        loading.value = false
      }
    }

    // 监听搜索查询
    watch(searchQuery, () => {
      state.pagination.page = 1 // 重置页码
      fetchUsers()
    }, { debounce: 300 })

    return {
      loading,
      error,
      searchQuery,
      ...stateRefs,
      currentPage,
      fetchUsers
    }
  }
}
```

#### 性能优化建议

```javascript
// 响应式数据性能优化
export default {
  setup() {
    // 1. 使用 shallowRef 优化大对象
    const largeData = shallowRef({
      // 大量数据...
    })

    // 手动触发更新
    const updateLargeData = (newData) => {
      largeData.value = newData
      triggerRef(largeData) // 手动触发
    }

    // 2. 使用 shallowReactive 优化深层对象
    const config = shallowReactive({
      api: {
        baseURL: 'https://api.example.com',
        timeout: 5000
      },
      ui: {
        theme: 'dark',
        locale: 'zh-CN'
      }
    })

    // 3. 只读数据使用 readonly
    const constants = readonly({
      APP_NAME: 'Vue App',
      VERSION: '1.0.0'
    })

    // 4. 计算属性缓存优化
    const expensiveValue = computed(() => {
      // 昂贵的计算...
      return heavyComputation(largeData.value)
    })

    return {
      largeData,
      config,
      constants,
      expensiveValue,
      updateLargeData
    }
  }
}
```

---

## Vue2 vs Vue3 全面对比与优化

### 1. 架构设计革命
**问题**: Vue3相比Vue2在架构设计上有哪些根本性改变？

**答案**:

#### 1.1 响应式系统重写

```javascript
// Vue 2 - Object.defineProperty 实现
function defineReactive(obj, key, val) {
  Object.defineProperty(obj, key, {
    get() {
      // 依赖收集
      Dep.target && dep.addSub(Dep.target);
      return val;
    },
    set(newVal) {
      if (newVal === val) return;
      val = newVal;
      // 通知更新
      dep.notify();
    }
  });
}

// Vue 2 的限制
const data = { count: 0 };
defineReactive(data, 'count', 0);

// ❌ 无法检测新增属性
data.newProp = 'value'; // 不会触发响应式

// ❌ 无法检测数组索引变化
data.items[0] = 'newValue'; // 不会触发响应式

// ✅ 需要使用特殊方法
Vue.set(data, 'newProp', 'value');
Vue.set(data.items, 0, 'newValue');
```

```javascript
// Vue 3 - Proxy 实现
function reactive(target) {
  return new Proxy(target, {
    get(target, key, receiver) {
      // 依赖收集
      track(target, key);
      const result = Reflect.get(target, key, receiver);
      
      // 深度响应式
      if (isObject(result)) {
        return reactive(result);
      }
      
      return result;
    },
    
    set(target, key, value, receiver) {
      const oldValue = target[key];
      const result = Reflect.set(target, key, value, receiver);
      
      // 触发更新
      if (value !== oldValue) {
        trigger(target, key, value, oldValue);
      }
      
      return result;
    },
    
    deleteProperty(target, key) {
      const hadKey = hasOwn(target, key);
      const result = Reflect.deleteProperty(target, key);
      
      if (result && hadKey) {
        trigger(target, key, undefined);
      }
      
      return result;
    }
  });
}

// Vue 3 优势
const state = reactive({ 
  count: 0, 
  items: [1, 2, 3],
  nested: { prop: 'value' }
});

// ✅ 可以检测所有变化
state.newProp = 'value';        // ✅ 响应式
state.items[0] = 'newValue';    // ✅ 响应式
delete state.count;             // ✅ 响应式
state.nested.newProp = 'value'; // ✅ 深度响应式
```

#### 1.2 编译时优化

```javascript
// Vue 2 编译结果
function render() {
  with (this) {
    return _c('div', {
      staticClass: "container"
    }, [
      _c('h1', [_v(_s(title))]),
      _c('p', [_v(_s(content))]),
      _l((items), function(item) {
        return _c('div', {
          key: item.id
        }, [_v(_s(item.name))])
      })
    ], 2)
  }
}
```

```javascript
// Vue 3 编译结果 - 带优化标记
import { createElementVNode as _createElementVNode, 
         createTextVNode as _createTextVNode,
         renderList as _renderList,
         Fragment as _Fragment,
         openBlock as _openBlock,
         createElementBlock as _createElementBlock } from "vue"

function render(_ctx, _cache) {
  return (_openBlock(), _createElementBlock("div", {
    class: "container"
  }, [
    // 静态提升
    _hoisted_1, // <h1>静态标题</h1>
    
    // 动态内容带优化标记
    _createElementVNode("p", null, _toDisplayString(_ctx.content), 1 /* TEXT */),
    
    // 列表渲染优化
    (_openBlock(true), _createElementBlock(_Fragment, null, 
      _renderList(_ctx.items, (item) => {
        return (_openBlock(), _createElementBlock("div", {
          key: item.id
        }, _toDisplayString(item.name), 1 /* TEXT */))
      }), 128 /* KEYED_FRAGMENT */))
  ]))
}

// 静态提升 - 编译时移动到外部
const _hoisted_1 = /*#__PURE__*/_createElementVNode("h1", null, "静态标题", -1 /* HOISTED */)
```

### 2. Composition API vs Options API

```javascript
// Vue 2 Options API
export default {
  name: 'UserProfile',
  data() {
    return {
      user: null,
      loading: false,
      posts: [],
      followers: 0
    }
  },
  
  computed: {
    displayName() {
      return this.user?.name || 'Unknown User';
    },
    
    userStats() {
      return {
        posts: this.posts.length,
        followers: this.followers
      };
    }
  },
  
  methods: {
    async fetchUser(id) {
      this.loading = true;
      try {
        this.user = await api.getUser(id);
        this.posts = await api.getUserPosts(id);
        this.followers = await api.getUserFollowers(id);
      } catch (error) {
        console.error(error);
      } finally {
        this.loading = false;
      }
    },
    
    async updateUser(userData) {
      await api.updateUser(this.user.id, userData);
      Object.assign(this.user, userData);
    }
  },
  
  async mounted() {
    await this.fetchUser(this.$route.params.id);
  },
  
  watch: {
    '$route.params.id': {
      handler(newId) {
        this.fetchUser(newId);
      },
      immediate: true
    }
  }
}
```

```javascript
// Vue 3 Composition API - 更好的逻辑复用
import { ref, computed, watch, onMounted } from 'vue';
import { useRoute } from 'vue-router';

// 可复用的用户数据逻辑
function useUser() {
  const user = ref(null);
  const loading = ref(false);
  
  const displayName = computed(() => {
    return user.value?.name || 'Unknown User';
  });
  
  const fetchUser = async (id) => {
    loading.value = true;
    try {
      user.value = await api.getUser(id);
    } catch (error) {
      console.error(error);
    } finally {
      loading.value = false;
    }
  };
  
  const updateUser = async (userData) => {
    await api.updateUser(user.value.id, userData);
    Object.assign(user.value, userData);
  };
  
  return {
    user,
    loading,
    displayName,
    fetchUser,
    updateUser
  };
}

// 可复用的用户帖子逻辑
function useUserPosts(userId) {
  const posts = ref([]);
  const loading = ref(false);
  
  const fetchPosts = async () => {
    if (!userId.value) return;
    
    loading.value = true;
    try {
      posts.value = await api.getUserPosts(userId.value);
    } catch (error) {
      console.error(error);
    } finally {
      loading.value = false;
    }
  };
  
  // 自动响应 userId 变化
  watch(userId, fetchPosts, { immediate: true });
  
  return {
    posts,
    loading: loading,
    fetchPosts
  };
}

// 组件中使用
export default {
  name: 'UserProfile',
  setup() {
    const route = useRoute();
    const userId = computed(() => route.params.id);
    
    // 组合逻辑
    const { 
      user, 
      loading: userLoading, 
      displayName, 
      fetchUser, 
      updateUser 
    } = useUser();
    
    const { 
      posts, 
      loading: postsLoading 
    } = useUserPosts(userId);
    
    // 用户统计
    const userStats = computed(() => ({
      posts: posts.value.length,
      followers: user.value?.followers || 0
    }));
    
    // 监听路由变化
    watch(userId, (newId) => {
      fetchUser(newId);
    }, { immediate: true });
    
    return {
      user,
      posts,
      loading: computed(() => userLoading.value || postsLoading.value),
      displayName,
      userStats,
      updateUser
    };
  }
}
```

### 3. 性能优化对比

#### 3.1 Bundle Size 优化

```javascript
// Vue 2 - 全量引入
import Vue from 'vue';
import VueRouter from 'vue-router';
import Vuex from 'vuex';

Vue.use(VueRouter);
Vue.use(Vuex);

// Bundle size: ~34KB (Vue) + ~8KB (VueRouter) + ~2KB (Vuex) = ~44KB
```

```javascript
// Vue 3 - Tree Shaking 友好
import { createApp } from 'vue';
import { createRouter, createWebHistory } from 'vue-router';
import { createStore } from 'vuex';

// 只引入使用的功能
import { ref, computed, watch } from 'vue';

const app = createApp(App);

// Bundle size: ~16KB (核心) + 按需引入 = 显著减少
```

#### 3.2 运行时性能对比

```javascript
// 性能测试组件
// Vue 2 实现
export default {
  data() {
    return {
      items: Array.from({ length: 10000 }, (_, i) => ({
        id: i,
        name: `Item ${i}`,
        active: i % 2 === 0
      }))
    }
  },
  
  computed: {
    filteredItems() {
      return this.items.filter(item => item.active);
    }
  },
  
  methods: {
    toggleItem(id) {
      const item = this.items.find(item => item.id === id);
      item.active = !item.active;
    }
  }
}

// Vue 3 实现 - 更好的性能
import { ref, computed } from 'vue';

export default {
  setup() {
    const items = ref(Array.from({ length: 10000 }, (_, i) => ({
      id: i,
      name: `Item ${i}`,
      active: i % 2 === 0
    })));
    
    // 更高效的计算属性
    const filteredItems = computed(() => {
      return items.value.filter(item => item.active);
    });
    
    const toggleItem = (id) => {
      const item = items.value.find(item => item.id === id);
      item.active = !item.active;
    };
    
    return {
      items,
      filteredItems,
      toggleItem
    };
  }
}
```

### 4. TypeScript 支持对比

```typescript
// Vue 2 + TypeScript - 复杂且有限制
import Vue from 'vue';
import Component from 'vue-class-component';
import { Prop, Watch } from 'vue-property-decorator';

interface User {
  id: number;
  name: string;
  email: string;
}

@Component
export default class UserComponent extends Vue {
  @Prop({ required: true }) readonly userId!: number;
  
  user: User | null = null;
  loading: boolean = false;
  
  get displayName(): string {
    return this.user?.name || 'Unknown';
  }
  
  @Watch('userId', { immediate: true })
  onUserIdChange(newId: number) {
    this.fetchUser(newId);
  }
  
  async fetchUser(id: number): Promise<void> {
    this.loading = true;
    try {
      this.user = await api.getUser(id);
    } finally {
      this.loading = false;
    }
  }
}
```

```typescript
// Vue 3 + TypeScript - 原生支持，类型推导更强
import { ref, computed, watch, onMounted, PropType } from 'vue';
import { defineComponent } from 'vue';

interface User {
  id: number;
  name: string;
  email: string;
}

export default defineComponent({
  props: {
    userId: {
      type: Number,
      required: true
    },
    config: {
      type: Object as PropType<UserConfig>,
      default: () => ({})
    }
  },
  
  setup(props, { emit }) {
    // 完整的类型推导
    const user = ref<User | null>(null);
    const loading = ref(false);
    
    // 自动推导返回类型
    const displayName = computed(() => {
      return user.value?.name || 'Unknown';
    });
    
    // 类型安全的函数
    const fetchUser = async (id: number): Promise<void> => {
      loading.value = true;
      try {
        user.value = await api.getUser(id);
        emit('user-loaded', user.value);
      } catch (error) {
        emit('error', error);
      } finally {
        loading.value = false;
      }
    };
    
    // 类型安全的 watch
    watch(
      () => props.userId,
      (newId: number) => fetchUser(newId),
      { immediate: true }
    );
    
    // 返回类型自动推导
    return {
      user,
      loading,
      displayName,
      fetchUser
    };
  }
});
```

### 5. 生态系统升级

#### 5.1 Vue Router 对比

```javascript
// Vue 2 + Vue Router 3
import VueRouter from 'vue-router';

const router = new VueRouter({
  mode: 'history',
  routes: [
    {
      path: '/user/:id',
      component: UserProfile,
      beforeEnter: (to, from, next) => {
        // 路由守卫
        if (isAuthenticated()) {
          next();
        } else {
          next('/login');
        }
      }
    }
  ]
});

// 组件中使用
export default {
  created() {
    console.log(this.$route.params.id);
    this.$router.push('/dashboard');
  }
}
```

```javascript
// Vue 3 + Vue Router 4
import { createRouter, createWebHistory } from 'vue-router';

const router = createRouter({
  history: createWebHistory(),
  routes: [
    {
      path: '/user/:id',
      component: UserProfile,
      beforeEnter: (to, from) => {
        // 更简洁的路由守卫
        if (!isAuthenticated()) {
          return { name: 'Login' };
        }
      }
    }
  ]
});

// 组件中使用 Composition API
import { useRoute, useRouter } from 'vue-router';

export default {
  setup() {
    const route = useRoute();
    const router = useRouter();
    
    // 响应式的路由参数
    const userId = computed(() => route.params.id);
    
    const navigateToDashboard = () => {
      router.push('/dashboard');
    };
    
    return {
      userId,
      navigateToDashboard
    };
  }
}
```

#### 5.2 状态管理对比

```javascript
// Vue 2 + Vuex 3
import Vuex from 'vuex';

const store = new Vuex.Store({
  state: {
    user: null,
    loading: false
  },
  
  mutations: {
    SET_USER(state, user) {
      state.user = user;
    },
    SET_LOADING(state, loading) {
      state.loading = loading;
    }
  },
  
  actions: {
    async fetchUser({ commit }, id) {
      commit('SET_LOADING', true);
      try {
        const user = await api.getUser(id);
        commit('SET_USER', user);
      } finally {
        commit('SET_LOADING', false);
      }
    }
  },
  
  getters: {
    displayName: state => state.user?.name || 'Unknown'
  }
});

// 组件中使用
import { mapState, mapActions, mapGetters } from 'vuex';

export default {
  computed: {
    ...mapState(['user', 'loading']),
    ...mapGetters(['displayName'])
  },
  
  methods: {
    ...mapActions(['fetchUser'])
  }
}
```

```javascript
// Vue 3 + Pinia (推荐) 或 Vuex 4
import { defineStore } from 'pinia';

// Pinia - 更简洁的状态管理
export const useUserStore = defineStore('user', () => {
  const user = ref(null);
  const loading = ref(false);
  
  const displayName = computed(() => {
    return user.value?.name || 'Unknown';
  });
  
  const fetchUser = async (id) => {
    loading.value = true;
    try {
      user.value = await api.getUser(id);
    } finally {
      loading.value = false;
    }
  };
  
  return {
    user,
    loading,
    displayName,
    fetchUser
  };
});

// 组件中使用
export default {
  setup() {
    const userStore = useUserStore();
    
    // 直接访问 store 状态和方法
    return {
      user: userStore.user,
      loading: userStore.loading,
      displayName: userStore.displayName,
      fetchUser: userStore.fetchUser
    };
  }
}
```

### 6. 开发体验优化

#### 6.1 开发工具对比

```javascript
// Vue 2 开发配置
module.exports = {
  configureWebpack: {
    devtool: 'source-map',
    resolve: {
      alias: {
        '@': path.resolve(__dirname, 'src')
      }
    }
  },
  
  devServer: {
    hot: true,
    overlay: {
      warnings: false,
      errors: true
    }
  }
}
```

```javascript
// Vue 3 + Vite 开发配置
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
  plugins: [vue()],
  
  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src')
    }
  },
  
  server: {
    hmr: true, // 更快的热重载
    open: true
  },
  
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['vue', 'vue-router'],
          utils: ['lodash', 'axios']
        }
      }
    }
  }
});

// 开发速度对比：
// Vue 2 + Webpack: 冷启动 ~15-30s, 热重载 ~1-3s
// Vue 3 + Vite: 冷启动 ~1-3s, 热重载 ~50-200ms
```

### 7. 迁移策略和兼容性

```javascript
// Vue 3 迁移构建 - 兼容 Vue 2 语法
import { createApp, configureCompat } from '@vue/compat';

// 启用兼容模式
configureCompat({
  MODE: 2, // Vue 2 兼容模式
  GLOBAL_MOUNT: false,
  GLOBAL_EXTEND: false
});

const app = createApp(App);

// 逐步迁移策略
// 1. 升级构建工具和依赖
// 2. 使用兼容构建运行现有代码
// 3. 逐个组件迁移到 Vue 3 语法
// 4. 移除兼容模式

// 迁移辅助工具
const migrationHelper = {
  // 检查不兼容的 API 使用
  checkDeprecatedAPIs(component) {
    const warnings = [];
    
    if (component.data && typeof component.data !== 'function') {
      warnings.push('data 必须是函数');
    }
    
    if (component.mixins?.length) {
      warnings.push('考虑用 Composition API 替换 mixins');
    }
    
    return warnings;
  },
  
  // 自动转换简单情况
  convertToCompositionAPI(optionsAPI) {
    // 工具函数，辅助迁移
  }
};
```

### 8. 性能基准测试对比

```javascript
// 性能测试结果对比
const performanceBenchmarks = {
  bundleSize: {
    vue2: {
      runtime: '34KB',
      fullBuild: '63KB'
    },
    vue3: {
      runtime: '16KB', // 减少 53%
      fullBuild: '34KB' // 减少 46%
    }
  },
  
  renderingPerformance: {
    initialRender: {
      vue2: '100ms (baseline)',
      vue3: '65ms (35% faster)'
    },
    updatePerformance: {
      vue2: '50ms (baseline)', 
      vue3: '25ms (50% faster)'
    },
    memoryUsage: {
      vue2: '12MB (baseline)',
      vue3: '8MB (33% less)'
    }
  },
  
  developmentExperience: {
    hotReload: {
      vue2Webpack: '1-3s',
      vue3Vite: '50-200ms'
    },
    buildTime: {
      vue2Webpack: '30-60s',
      vue3Vite: '5-15s'
    }
  }
};
```

## 总结

Vue3相比Vue2的改进是全方位的：

### 🚀 **性能提升**
- Bundle体积减少 40-50%
- 渲染性能提升 35%+  
- 内存使用减少 33%
- 支持Tree Shaking

### 🔧 **开发体验**
- TypeScript原生支持
- Composition API提供更好的逻辑复用
- Vite提供极速开发体验
- 更好的IDE支持

### ⚡ **技术架构**
- Proxy替代Object.defineProperty
- 编译时优化(静态提升、补丁标记)
- 更小的运行时核心
- 更好的Tree Shaking支持

### 🌟 **生态升级**
- Vue Router 4带来更好的路由体验
- Pinia替代Vuex提供更简洁的状态管理
- 更现代的构建工具链
- 向后兼容的迁移路径

这些优化使Vue3不仅在性能上有显著提升，在开发体验和可维护性方面也有质的飞跃。