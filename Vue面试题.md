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