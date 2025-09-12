# React 面试题

## 基础概念

### 1. React 基本概念
**问题**: 什么是React？React的核心特性有哪些？

**答案**:
React是一个用于构建用户界面的JavaScript库，主要特性包括：

- **组件化**: 将UI拆分成独立、可复用的组件
- **声明式**: 描述UI应该是什么样的，而不是如何实现
- **虚拟DOM**: 提高性能的关键机制
- **单向数据流**: 数据自上而下流动，便于调试和理解

```jsx
// 函数组件示例
function Welcome(props) {
  return <h1>Hello, {props.name}!</h1>;
}

// 类组件示例
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}
```

### 2. JSX 语法
**问题**: 什么是JSX？为什么使用JSX？

**答案**:
JSX是JavaScript语法扩展，允许在JavaScript中编写类似HTML的代码：

```jsx
// JSX 语法
const element = <h1>Hello, world!</h1>;

// 编译后的JavaScript
const element = React.createElement(
  'h1',
  null,
  'Hello, world!'
);

// JSX中的JavaScript表达式
const name = 'React';
const element = <h1>Hello, {name}!</h1>;

// 条件渲染
const isLoggedIn = true;
const greeting = (
  <div>
    {isLoggedIn ? <h1>Welcome back!</h1> : <h1>Please sign up.</h1>}
  </div>
);

// 列表渲染
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li key={number.toString()}>{number}</li>
);
```

### 3. 组件和Props
**问题**: React组件如何接收和使用props？

**答案**:
```jsx
// 函数组件接收props
function UserCard({ name, age, email }) {
  return (
    <div className="user-card">
      <h2>{name}</h2>
      <p>年龄: {age}</p>
      <p>邮箱: {email}</p>
    </div>
  );
}

// 使用组件
function App() {
  return (
    <UserCard 
      name="张三" 
      age={25} 
      email="zhangsan@example.com" 
    />
  );
}

// Props默认值
function Button({ children, variant = 'primary' }) {
  return (
    <button className={`btn btn-${variant}`}>
      {children}
    </button>
  );
}

// Props类型检查（使用PropTypes）
import PropTypes from 'prop-types';

UserCard.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number,
  email: PropTypes.string
};
```

### 4. State 状态管理
**问题**: 如何在React组件中管理状态？

**答案**:
```jsx
// 类组件的state
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
  }

  increment = () => {
    this.setState({ count: this.state.count + 1 });
  }

  render() {
    return (
      <div>
        <p>计数: {this.state.count}</p>
        <button onClick={this.increment}>增加</button>
      </div>
    );
  }
}

// 函数组件使用useState Hook
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  const increment = () => {
    setCount(count + 1);
    // 或者使用函数式更新
    // setCount(prevCount => prevCount + 1);
  };

  return (
    <div>
      <p>计数: {count}</p>
      <button onClick={increment}>增加</button>
    </div>
  );
}
```

## React Hooks

### 5. useState Hook
**问题**: useState Hook的用法和注意事项

**答案**:
```jsx
import { useState } from 'react';

function Form() {
  const [user, setUser] = useState({
    name: '',
    email: '',
    age: 0
  });

  // 更新对象状态
  const handleInputChange = (field, value) => {
    setUser(prevUser => ({
      ...prevUser,
      [field]: value
    }));
  };

  // 数组状态示例
  const [items, setItems] = useState([]);
  
  const addItem = (newItem) => {
    setItems(prevItems => [...prevItems, newItem]);
  };

  const removeItem = (index) => {
    setItems(prevItems => prevItems.filter((_, i) => i !== index));
  };

  return (
    <form>
      <input
        value={user.name}
        onChange={(e) => handleInputChange('name', e.target.value)}
        placeholder="姓名"
      />
      <input
        value={user.email}
        onChange={(e) => handleInputChange('email', e.target.value)}
        placeholder="邮箱"
      />
    </form>
  );
}
```

### 6. useEffect Hook
**问题**: useEffect的用法和生命周期对应关系

**答案**:
```jsx
import { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  // 相当于 componentDidMount 和 componentDidUpdate
  useEffect(() => {
    const fetchUser = async () => {
      setLoading(true);
      try {
        const response = await fetch(`/api/users/${userId}`);
        const userData = await response.json();
        setUser(userData);
      } catch (error) {
        console.error('获取用户信息失败:', error);
      } finally {
        setLoading(false);
      }
    };

    fetchUser();
  }, [userId]); // 依赖数组，只有userId变化时才重新执行

  // 相当于 componentWillUnmount
  useEffect(() => {
    const timer = setInterval(() => {
      console.log('定时器执行');
    }, 1000);

    // 清理函数
    return () => {
      clearInterval(timer);
    };
  }, []);

  if (loading) return <div>加载中...</div>;
  if (!user) return <div>用户不存在</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

### 7. useContext Hook
**问题**: 如何使用useContext进行状态共享？

**答案**:
```jsx
import { createContext, useContext, useState } from 'react';

// 创建Context
const ThemeContext = createContext();
const UserContext = createContext();

// Provider组件
function App() {
  const [theme, setTheme] = useState('light');
  const [user, setUser] = useState(null);

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <UserContext.Provider value={{ user, setUser }}>
        <Header />
        <Main />
      </UserContext.Provider>
    </ThemeContext.Provider>
  );
}

// 使用Context的组件
function Header() {
  const { theme, setTheme } = useContext(ThemeContext);
  const { user } = useContext(UserContext);

  return (
    <header className={`header-${theme}`}>
      <h1>欢迎, {user?.name || '游客'}</h1>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        切换主题
      </button>
    </header>
  );
}

// 自定义Hook封装Context
function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme必须在ThemeProvider内使用');
  }
  return context;
}
```

### 8. useMemo 和 useCallback
**问题**: 如何使用useMemo和useCallback优化性能？

**答案**:
```jsx
import { useState, useMemo, useCallback, memo } from 'react';

function ExpensiveComponent({ items, onItemClick }) {
  // useMemo 缓存计算结果
  const expensiveValue = useMemo(() => {
    console.log('执行昂贵计算');
    return items.reduce((sum, item) => sum + item.value, 0);
  }, [items]);

  // useCallback 缓存函数
  const handleClick = useCallback((id) => {
    onItemClick(id);
  }, [onItemClick]);

  return (
    <div>
      <p>总值: {expensiveValue}</p>
      {items.map(item => (
        <ItemComponent
          key={item.id}
          item={item}
          onClick={handleClick}
        />
      ))}
    </div>
  );
}

// 使用memo优化子组件
const ItemComponent = memo(({ item, onClick }) => {
  console.log(`渲染Item ${item.id}`);
  return (
    <div onClick={() => onClick(item.id)}>
      {item.name}: {item.value}
    </div>
  );
});

function App() {
  const [items, setItems] = useState([
    { id: 1, name: '商品1', value: 100 },
    { id: 2, name: '商品2', value: 200 }
  ]);
  const [filter, setFilter] = useState('');

  // 过滤后的商品列表
  const filteredItems = useMemo(() => {
    return items.filter(item => 
      item.name.toLowerCase().includes(filter.toLowerCase())
    );
  }, [items, filter]);

  const handleItemClick = useCallback((id) => {
    console.log('点击商品:', id);
  }, []);

  return (
    <div>
      <input
        value={filter}
        onChange={(e) => setFilter(e.target.value)}
        placeholder="过滤商品"
      />
      <ExpensiveComponent
        items={filteredItems}
        onItemClick={handleItemClick}
      />
    </div>
  );
}
```

## 性能优化

### 9. React.memo 和组件优化
**问题**: 如何避免不必要的组件重新渲染？

**答案**:
```jsx
import { memo, useState, useCallback } from 'react';

// 使用memo包装组件
const ChildComponent = memo(({ name, age, onButtonClick }) => {
  console.log('ChildComponent重新渲染');
  return (
    <div>
      <p>{name} - {age}岁</p>
      <button onClick={onButtonClick}>点击</button>
    </div>
  );
});

// 自定义比较函数
const AdvancedChild = memo(({ user, settings }) => {
  return (
    <div>
      <p>{user.name}</p>
      <p>设置: {settings.theme}</p>
    </div>
  );
}, (prevProps, nextProps) => {
  // 只有name或theme变化时才重新渲染
  return prevProps.user.name === nextProps.user.name &&
         prevProps.settings.theme === nextProps.settings.theme;
});

function Parent() {
  const [count, setCount] = useState(0);
  const [user] = useState({ name: '张三', age: 25 });

  // 使用useCallback避免函数重新创建
  const handleButtonClick = useCallback(() => {
    console.log('按钮被点击');
  }, []);

  return (
    <div>
      <p>计数: {count}</p>
      <button onClick={() => setCount(count + 1)}>增加计数</button>
      
      {/* 即使count变化，ChildComponent也不会重新渲染 */}
      <ChildComponent
        name={user.name}
        age={user.age}
        onButtonClick={handleButtonClick}
      />
    </div>
  );
}
```

### 10. 虚拟DOM和协调算法
**问题**: React的虚拟DOM是如何工作的？

**答案**:
```jsx
// React的协调(Reconciliation)过程示例
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map(todo => (
        // key是协调算法的重要提示
        <TodoItem 
          key={todo.id}  // 使用稳定的唯一ID
          todo={todo}
        />
      ))}
    </ul>
  );
}

// 错误的key使用
function BadExample({ items }) {
  return (
    <ul>
      {items.map((item, index) => (
        // 使用index作为key可能导致性能问题
        <li key={index}>{item.name}</li>
      ))}
    </ul>
  );
}

// 正确的key使用
function GoodExample({ items }) {
  return (
    <ul>
      {items.map(item => (
        // 使用唯一且稳定的ID
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}

// 虚拟DOM diff示例
// 旧虚拟DOM:
// <div>
//   <span>Hello</span>
//   <span>World</span>
// </div>

// 新虚拟DOM:
// <div>
//   <span>Hello</span>
//   <span>React</span>
//   <span>World</span>
// </div>

// React会智能地只更新变化的部分
```

## 高级概念

### 11. 高阶组件 (HOC)
**问题**: 什么是高阶组件？如何实现和使用？

**答案**:
```jsx
import { useState, useEffect } from 'react';

// 认证HOC
function withAuth(WrappedComponent) {
  return function AuthComponent(props) {
    const [isAuthenticated, setIsAuthenticated] = useState(false);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
      // 检查认证状态
      const checkAuth = async () => {
        try {
          const token = localStorage.getItem('token');
          if (token) {
            // 验证token
            const isValid = await validateToken(token);
            setIsAuthenticated(isValid);
          }
        } catch (error) {
          setIsAuthenticated(false);
        } finally {
          setLoading(false);
        }
      };

      checkAuth();
    }, []);

    if (loading) {
      return <div>加载中...</div>;
    }

    if (!isAuthenticated) {
      return <div>请先登录</div>;
    }

    return <WrappedComponent {...props} />;
  };
}

// 使用HOC
const ProtectedDashboard = withAuth(function Dashboard() {
  return <div>仪表盘内容</div>;
});

// 数据获取HOC
function withData(url) {
  return function(WrappedComponent) {
    return function DataComponent(props) {
      const [data, setData] = useState(null);
      const [loading, setLoading] = useState(true);
      const [error, setError] = useState(null);

      useEffect(() => {
        fetch(url)
          .then(response => response.json())
          .then(data => {
            setData(data);
            setLoading(false);
          })
          .catch(error => {
            setError(error);
            setLoading(false);
          });
      }, []);

      return (
        <WrappedComponent
          {...props}
          data={data}
          loading={loading}
          error={error}
        />
      );
    };
  };
}

// 使用数据HOC
const UserList = withData('/api/users')(function UserListComponent({ data, loading, error }) {
  if (loading) return <div>加载中...</div>;
  if (error) return <div>错误: {error.message}</div>;

  return (
    <ul>
      {data.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
});
```

### 12. Render Props 模式
**问题**: 什么是Render Props模式？

**答案**:
```jsx
// Render Props组件
function DataFetcher({ url, render }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetch(url)
      .then(response => response.json())
      .then(data => {
        setData(data);
        setLoading(false);
      })
      .catch(error => {
        setError(error);
        setLoading(false);
      });
  }, [url]);

  return render({ data, loading, error });
}

// 使用Render Props
function App() {
  return (
    <div>
      <DataFetcher
        url="/api/users"
        render={({ data, loading, error }) => {
          if (loading) return <div>加载中...</div>;
          if (error) return <div>错误: {error.message}</div>;
          
          return (
            <ul>
              {data.map(user => (
                <li key={user.id}>{user.name}</li>
              ))}
            </ul>
          );
        }}
      />

      {/* 可以复用相同的数据获取逻辑，但渲染不同的UI */}
      <DataFetcher
        url="/api/posts"
        render={({ data, loading, error }) => {
          if (loading) return <span>帖子加载中...</span>;
          if (error) return <span>加载失败</span>;
          
          return <div>共有 {data.length} 篇帖子</div>;
        }}
      />
    </div>
  );
}

// 使用children作为render prop
function Toggle({ children }) {
  const [isOpen, setIsOpen] = useState(false);

  return children({
    isOpen,
    toggle: () => setIsOpen(!isOpen)
  });
}

// 使用示例
function Modal() {
  return (
    <Toggle>
      {({ isOpen, toggle }) => (
        <div>
          <button onClick={toggle}>
            {isOpen ? '关闭' : '打开'}模态框
          </button>
          {isOpen && (
            <div className="modal">
              <p>模态框内容</p>
              <button onClick={toggle}>关闭</button>
            </div>
          )}
        </div>
      )}
    </Toggle>
  );
}
```

### 13. 错误边界
**问题**: 如何在React中处理错误？

**答案**:
```jsx
import { Component } from 'react';

// 类组件错误边界
class ErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null, errorInfo: null };
  }

  static getDerivedStateFromError(error) {
    // 更新状态以显示错误UI
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // 记录错误信息
    console.error('错误边界捕获到错误:', error, errorInfo);
    
    this.setState({
      error: error,
      errorInfo: errorInfo
    });

    // 可以将错误信息发送到错误报告服务
    // logErrorToService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-boundary">
          <h2>哎呀，出现了错误！</h2>
          <details style={{ whiteSpace: 'pre-wrap' }}>
            {this.state.error && this.state.error.toString()}
            <br />
            {this.state.errorInfo.componentStack}
          </details>
          <button 
            onClick={() => this.setState({ hasError: false, error: null, errorInfo: null })}
          >
            重试
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

// 使用错误边界
function App() {
  return (
    <div>
      <h1>我的应用</h1>
      <ErrorBoundary>
        <UserProfile userId="123" />
      </ErrorBoundary>
      
      <ErrorBoundary>
        <ProductList />
      </ErrorBoundary>
    </div>
  );
}

// 可能抛出错误的组件
function BuggyComponent() {
  const [count, setCount] = useState(0);

  if (count >= 3) {
    // 故意抛出错误
    throw new Error('计数器爆炸了！');
  }

  return (
    <div>
      <p>计数: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        增加计数
      </button>
    </div>
  );
}

// 函数组件版本的错误处理
function useErrorHandler() {
  const [error, setError] = useState(null);

  const resetError = () => setError(null);

  const captureError = (error) => {
    setError(error);
    console.error('捕获到错误:', error);
  };

  return { error, resetError, captureError };
}

function SafeComponent() {
  const { error, resetError, captureError } = useErrorHandler();

  if (error) {
    return (
      <div>
        <h3>出现错误: {error.message}</h3>
        <button onClick={resetError}>重试</button>
      </div>
    );
  }

  return (
    <div>
      <button 
        onClick={() => {
          try {
            // 可能出错的操作
            throw new Error('测试错误');
          } catch (err) {
            captureError(err);
          }
        }}
      >
        触发错误
      </button>
    </div>
  );
}
```

## 状态管理

### 14. Context vs Redux
**问题**: 何时使用Context，何时使用Redux？

**答案**:
```jsx
// Context适用场景：简单的全局状态
import { createContext, useContext, useReducer } from 'react';

// 创建Context
const AppContext = createContext();

// Reducer函数
function appReducer(state, action) {
  switch (action.type) {
    case 'SET_USER':
      return { ...state, user: action.payload };
    case 'SET_THEME':
      return { ...state, theme: action.payload };
    case 'TOGGLE_SIDEBAR':
      return { ...state, sidebarOpen: !state.sidebarOpen };
    default:
      return state;
  }
}

// Provider组件
function AppProvider({ children }) {
  const [state, dispatch] = useReducer(appReducer, {
    user: null,
    theme: 'light',
    sidebarOpen: false
  });

  return (
    <AppContext.Provider value={{ state, dispatch }}>
      {children}
    </AppContext.Provider>
  );
}

// 自定义Hook
function useAppContext() {
  const context = useContext(AppContext);
  if (!context) {
    throw new Error('useAppContext must be used within AppProvider');
  }
  return context;
}

// 使用Context的组件
function Header() {
  const { state, dispatch } = useAppContext();

  return (
    <header>
      <h1>欢迎, {state.user?.name}</h1>
      <button 
        onClick={() => dispatch({ 
          type: 'SET_THEME', 
          payload: state.theme === 'light' ? 'dark' : 'light' 
        })}
      >
        切换主题
      </button>
    </header>
  );
}

// Redux适用场景：复杂的状态管理
// store/userSlice.js (使用Redux Toolkit)
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// 异步action
export const fetchUser = createAsyncThunk(
  'user/fetchUser',
  async (userId) => {
    const response = await fetch(`/api/users/${userId}`);
    return response.json();
  }
);

const userSlice = createSlice({
  name: 'user',
  initialState: {
    data: null,
    loading: false,
    error: null
  },
  reducers: {
    updateProfile: (state, action) => {
      state.data = { ...state.data, ...action.payload };
    },
    clearError: (state) => {
      state.error = null;
    }
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUser.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchUser.fulfilled, (state, action) => {
        state.loading = false;
        state.data = action.payload;
      })
      .addCase(fetchUser.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message;
      });
  }
});

export const { updateProfile, clearError } = userSlice.actions;
export default userSlice.reducer;
```

### 15. 自定义Hooks
**问题**: 如何创建和使用自定义Hooks？

**答案**:
```jsx
import { useState, useEffect, useCallback, useRef } from 'react';

// 数据获取Hook
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        const response = await fetch(url);
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        const result = await response.json();
        setData(result);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, [url]);

  return { data, loading, error };
}

// 本地存储Hook
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error('读取localStorage错误:', error);
      return initialValue;
    }
  });

  const setValue = useCallback((value) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error('设置localStorage错误:', error);
    }
  }, [key, storedValue]);

  return [storedValue, setValue];
}

// 防抖Hook
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);

  return debouncedValue;
}

// 之前值Hook
function usePrevious(value) {
  const ref = useRef();
  
  useEffect(() => {
    ref.current = value;
  });
  
  return ref.current;
}

// 组件挂载状态Hook
function useIsMounted() {
  const isMountedRef = useRef(true);
  
  useEffect(() => {
    return () => {
      isMountedRef.current = false;
    };
  }, []);
  
  return useCallback(() => isMountedRef.current, []);
}

// 使用自定义Hooks的组件
function SearchComponent() {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebounce(query, 500);
  const [searchHistory, setSearchHistory] = useLocalStorage('searchHistory', []);
  const { data, loading, error } = useFetch(
    debouncedQuery ? `/api/search?q=${debouncedQuery}` : null
  );
  const previousQuery = usePrevious(debouncedQuery);
  const isMounted = useIsMounted();

  useEffect(() => {
    if (debouncedQuery && debouncedQuery !== previousQuery) {
      // 添加到搜索历史
      setSearchHistory(prev => {
        const newHistory = [debouncedQuery, ...prev.filter(item => item !== debouncedQuery)];
        return newHistory.slice(0, 10); // 只保留最近10条
      });
    }
  }, [debouncedQuery, previousQuery, setSearchHistory]);

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="搜索..."
      />
      
      {loading && <div>搜索中...</div>}
      {error && <div>错误: {error}</div>}
      {data && (
        <ul>
          {data.map(item => (
            <li key={item.id}>{item.title}</li>
          ))}
        </ul>
      )}
      
      <div>
        <h3>搜索历史</h3>
        <ul>
          {searchHistory.map((term, index) => (
            <li key={index} onClick={() => setQuery(term)}>
              {term}
            </li>
          ))}
        </ul>
      </div>
    </div>
  );
}
```

## React 生态系统

### 16. React Router
**问题**: 如何使用React Router进行路由管理？

**答案**:
```jsx
import { 
  BrowserRouter as Router, 
  Routes, 
  Route, 
  Link, 
  useNavigate, 
  useParams,
  useLocation,
  Navigate
} from 'react-router-dom';

// 路由配置
function App() {
  return (
    <Router>
      <div>
        <nav>
          <Link to="/">首页</Link>
          <Link to="/about">关于</Link>
          <Link to="/users">用户</Link>
          <Link to="/products">产品</Link>
        </nav>

        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/users" element={<Users />}>
            <Route path=":userId" element={<UserDetail />} />
          </Route>
          <Route path="/products" element={<Products />} />
          <Route path="/login" element={<Login />} />
          
          {/* 受保护的路由 */}
          <Route 
            path="/dashboard" 
            element={
              <ProtectedRoute>
                <Dashboard />
              </ProtectedRoute>
            } 
          />
          
          {/* 404页面 */}
          <Route path="*" element={<NotFound />} />
        </Routes>
      </div>
    </Router>
  );
}

// 编程式导航
function NavigationExample() {
  const navigate = useNavigate();
  const location = useLocation();

  const handleLogin = () => {
    // 登录成功后导航
    navigate('/dashboard');
    // 或者导航到之前的页面
    // navigate(location.state?.from || '/dashboard');
  };

  const goBack = () => {
    navigate(-1); // 返回上一页
  };

  return (
    <div>
      <button onClick={handleLogin}>登录</button>
      <button onClick={goBack}>返回</button>
    </div>
  );
}

// 动态路由参数
function UserDetail() {
  const { userId } = useParams();
  const { data, loading } = useFetch(`/api/users/${userId}`);

  if (loading) return <div>加载中...</div>;

  return (
    <div>
      <h1>用户详情</h1>
      <p>用户ID: {userId}</p>
      <p>用户名: {data?.name}</p>
    </div>
  );
}

// 路由守卫
function ProtectedRoute({ children }) {
  const isAuthenticated = useAuth(); // 自定义Hook检查认证状态
  const location = useLocation();

  if (!isAuthenticated) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  return children;
}

// 嵌套路由
function Users() {
  return (
    <div>
      <h2>用户列表</h2>
      {/* 子路由会在这里渲染 */}
      <Outlet />
    </div>
  );
}
```

### 17. React测试
**问题**: 如何测试React组件？

**答案**:
```jsx
// 使用React Testing Library
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import '@testing-library/jest-dom';
import Counter from './Counter';
import UserProfile from './UserProfile';

// 基本组件测试
describe('Counter组件', () => {
  test('初始计数为0', () => {
    render(<Counter />);
    expect(screen.getByText('计数: 0')).toBeInTheDocument();
  });

  test('点击按钮增加计数', async () => {
    const user = userEvent.setup();
    render(<Counter />);
    
    const button = screen.getByRole('button', { name: '增加' });
    await user.click(button);
    
    expect(screen.getByText('计数: 1')).toBeInTheDocument();
  });

  test('计数达到5时显示警告', async () => {
    const user = userEvent.setup();
    render(<Counter />);
    
    const button = screen.getByRole('button', { name: '增加' });
    
    // 点击5次
    for (let i = 0; i < 5; i++) {
      await user.click(button);
    }
    
    expect(screen.getByText('警告：计数过高！')).toBeInTheDocument();
  });
});

// 异步操作测试
describe('UserProfile组件', () => {
  test('加载用户数据', async () => {
    // Mock API响应
    global.fetch = jest.fn(() =>
      Promise.resolve({
        ok: true,
        json: () => Promise.resolve({
          id: 1,
          name: '张三',
          email: 'zhangsan@example.com'
        })
      })
    );

    render(<UserProfile userId="1" />);
    
    // 初始状态
    expect(screen.getByText('加载中...')).toBeInTheDocument();
    
    // 等待数据加载完成
    await waitFor(() => {
      expect(screen.getByText('张三')).toBeInTheDocument();
    });
    
    expect(screen.getByText('zhangsan@example.com')).toBeInTheDocument();
  });

  test('处理API错误', async () => {
    global.fetch = jest.fn(() =>
      Promise.reject(new Error('网络错误'))
    );

    render(<UserProfile userId="1" />);
    
    await waitFor(() => {
      expect(screen.getByText(/错误/)).toBeInTheDocument();
    });
  });
});

// Hook测试
import { renderHook, act } from '@testing-library/react';
import useCounter from './useCounter';

describe('useCounter Hook', () => {
  test('初始值为0', () => {
    const { result } = renderHook(() => useCounter());
    expect(result.current.count).toBe(0);
  });

  test('increment增加计数', () => {
    const { result } = renderHook(() => useCounter());
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });

  test('自定义初始值', () => {
    const { result } = renderHook(() => useCounter(10));
    expect(result.current.count).toBe(10);
  });
});

// 快照测试
test('Button组件快照', () => {
  const { container } = render(
    <Button variant="primary" size="large">
      点击我
    </Button>
  );
  expect(container.firstChild).toMatchSnapshot();
});

// 集成测试
describe('TodoApp集成测试', () => {
  test('完整的添加和删除流程', async () => {
    const user = userEvent.setup();
    render(<TodoApp />);
    
    // 添加待办事项
    const input = screen.getByPlaceholderText('输入待办事项');
    const addButton = screen.getByRole('button', { name: '添加' });
    
    await user.type(input, '学习React');
    await user.click(addButton);
    
    expect(screen.getByText('学习React')).toBeInTheDocument();
    
    // 删除待办事项
    const deleteButton = screen.getByRole('button', { name: '删除' });
    await user.click(deleteButton);
    
    expect(screen.queryByText('学习React')).not.toBeInTheDocument();
  });
});
```

### 18. React性能监控
**问题**: 如何监控和优化React应用性能？

**答案**:
```jsx
import { Profiler, useState, useEffect } from 'react';

// React Profiler使用
function App() {
  const onRenderCallback = (id, phase, actualDuration, baseDuration, startTime, commitTime) => {
    console.log('组件渲染信息:', {
      id,           // 组件标识
      phase,        // "mount" 或 "update"
      actualDuration, // 本次渲染耗时
      baseDuration,   // 估计渲染耗时
      startTime,      // 开始渲染时间
      commitTime      // 提交时间
    });
    
    // 可以将性能数据发送到分析服务
    // sendToAnalytics({ id, phase, actualDuration });
  };

  return (
    <div>
      <Profiler id="Header" onRender={onRenderCallback}>
        <Header />
      </Profiler>
      
      <Profiler id="MainContent" onRender={onRenderCallback}>
        <MainContent />
      </Profiler>
    </div>
  );
}

// 性能监控Hook
function usePerformanceMonitor(componentName) {
  useEffect(() => {
    const startTime = performance.now();
    
    return () => {
      const endTime = performance.now();
      const renderTime = endTime - startTime;
      
      if (renderTime > 16) { // 超过一帧时间(16ms)
        console.warn(`${componentName} 渲染时间过长: ${renderTime}ms`);
      }
    };
  });
}

// 内存使用监控
function useMemoryMonitor() {
  useEffect(() => {
    const checkMemory = () => {
      if (performance.memory) {
        const memInfo = {
          used: Math.round(performance.memory.usedJSHeapSize / 1048576), // MB
          total: Math.round(performance.memory.totalJSHeapSize / 1048576), // MB
          limit: Math.round(performance.memory.jsHeapSizeLimit / 1048576) // MB
        };
        
        console.log('内存使用:', memInfo);
        
        if (memInfo.used / memInfo.limit > 0.9) {
          console.warn('内存使用率过高！');
        }
      }
    };

    const interval = setInterval(checkMemory, 10000); // 每10秒检查一次
    return () => clearInterval(interval);
  }, []);
}

// 组件渲染次数统计
function useRenderCount(componentName) {
  const renderCount = useRef(0);
  
  useEffect(() => {
    renderCount.current += 1;
    console.log(`${componentName} 渲染次数: ${renderCount.current}`);
  });
  
  return renderCount.current;
}

// 长列表虚拟化示例
import { FixedSizeList as List } from 'react-window';

function VirtualizedList({ items }) {
  const Row = ({ index, style }) => (
    <div style={style}>
      {items[index].name}
    </div>
  );

  return (
    <List
      height={600}        // 容器高度
      itemCount={items.length}
      itemSize={35}       // 每项高度
      width="100%"
    >
      {Row}
    </List>
  );
}

// 使用示例
function PerformanceOptimizedComponent() {
  usePerformanceMonitor('PerformanceOptimizedComponent');
  useMemoryMonitor();
  const renderCount = useRenderCount('PerformanceOptimizedComponent');
  
  const [items] = useState(
    Array.from({ length: 10000 }, (_, i) => ({ id: i, name: `Item ${i}` }))
  );

  return (
    <div>
      <p>渲染次数: {renderCount}</p>
      <VirtualizedList items={items} />
    </div>
  );
}
```

## 常见问题和最佳实践

### 19. React中的key属性
**问题**: 为什么列表渲染需要key？key应该如何选择？

**答案**:
```jsx
// 错误的key使用 - 使用索引
function BadList({ items }) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>{item.name}</li> // ❌ 可能导致性能问题
      ))}
    </ul>
  );
}

// 正确的key使用 - 使用唯一ID
function GoodList({ items }) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.name}</li> // ✅ 使用稳定的唯一标识
      ))}
    </ul>
  );
}

// key的重要性演示
function DynamicList() {
  const [items, setItems] = useState([
    { id: 1, name: '苹果', count: 0 },
    { id: 2, name: '香蕉', count: 0 },
    { id: 3, name: '橙子', count: 0 }
  ]);

  const shuffleItems = () => {
    setItems([...items].sort(() => Math.random() - 0.5));
  };

  const addItem = () => {
    const newItem = {
      id: Date.now(),
      name: `新水果${items.length + 1}`,
      count: 0
    };
    setItems([newItem, ...items]);
  };

  return (
    <div>
      <button onClick={shuffleItems}>打乱顺序</button>
      <button onClick={addItem}>添加项目</button>
      
      <ul>
        {items.map(item => (
          <li key={item.id}>
            <span>{item.name}</span>
            <CounterInput initialValue={item.count} />
          </li>
        ))}
      </ul>
    </div>
  );
}

// 有状态的子组件，用于演示key的重要性
function CounterInput({ initialValue }) {
  const [value, setValue] = useState(initialValue);
  
  return (
    <input
      type="number"
      value={value}
      onChange={(e) => setValue(parseInt(e.target.value) || 0)}
    />
  );
}
```

### 20. React组件通信
**问题**: React组件之间如何进行通信？

**答案**:
```jsx
// 1. 父子组件通信 - Props
function Parent() {
  const [message, setMessage] = useState('来自父组件的消息');
  
  const handleChildMessage = (childMessage) => {
    console.log('收到子组件消息:', childMessage);
  };

  return (
    <Child 
      message={message}
      onMessage={handleChildMessage}
    />
  );
}

function Child({ message, onMessage }) {
  return (
    <div>
      <p>{message}</p>
      <button onClick={() => onMessage('来自子组件的消息')}>
        发送消息给父组件
      </button>
    </div>
  );
}

// 2. 兄弟组件通信 - 状态提升
function App() {
  const [sharedData, setSharedData] = useState('共享数据');

  return (
    <div>
      <ComponentA data={sharedData} onDataChange={setSharedData} />
      <ComponentB data={sharedData} onDataChange={setSharedData} />
    </div>
  );
}

// 3. 跨层级组件通信 - Context
const MessageContext = createContext();

function GrandParent() {
  const [messages, setMessages] = useState([]);
  
  const addMessage = (message) => {
    setMessages(prev => [...prev, message]);
  };

  return (
    <MessageContext.Provider value={{ messages, addMessage }}>
      <Parent />
    </MessageContext.Provider>
  );
}

function DeepChild() {
  const { messages, addMessage } = useContext(MessageContext);
  
  return (
    <div>
      <button onClick={() => addMessage('新消息')}>
        添加消息
      </button>
      <ul>
        {messages.map((msg, index) => (
          <li key={index}>{msg}</li>
        ))}
      </ul>
    </div>
  );
}

// 4. 事件总线通信
class EventBus {
  constructor() {
    this.events = {};
  }

  on(event, callback) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(callback);
  }

  emit(event, data) {
    if (this.events[event]) {
      this.events[event].forEach(callback => callback(data));
    }
  }

  off(event, callback) {
    if (this.events[event]) {
      this.events[event] = this.events[event].filter(cb => cb !== callback);
    }
  }
}

const eventBus = new EventBus();

function ComponentA() {
  const sendMessage = () => {
    eventBus.emit('message', '来自组件A的消息');
  };

  return <button onClick={sendMessage}>发送消息</button>;
}

function ComponentB() {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    const handleMessage = (message) => {
      setMessages(prev => [...prev, message]);
    };

    eventBus.on('message', handleMessage);
    
    return () => {
      eventBus.off('message', handleMessage);
    };
  }, []);

  return (
    <ul>
      {messages.map((msg, index) => (
        <li key={index}>{msg}</li>
      ))}
    </ul>
  );
}
```