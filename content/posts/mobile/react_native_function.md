---
title: "React Native 组件类型详解：Class组件、函数组件与高阶组件"
date: 2025-04-29T13:13:24+08:00
draft: false
categories: ["移动端"]
tags: ["React Native"]
---
## 一、Class 组件（类组件）

### 基本结构
```jsx
class MyClassComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
  }

  componentDidMount() {
    console.log('组件已挂载');
  }

  handlePress = () => {
    this.setState({ count: this.state.count + 1 });
  };

  render() {
    return (
      <View>
        <Text>{this.state.count}</Text>
        <Button title="增加" onPress={this.handlePress} />
      </View>
    );
  }
}
```

### 特点
- **生命周期方法**：可以使用完整的生命周期方法（如 `componentDidMount`, `shouldComponentUpdate` 等）
- **状态管理**：通过 `this.state` 和 `this.setState` 管理内部状态
- **Ref 支持**：可以直接使用 ref 获取组件实例

### 优势
1. 完整的生命周期控制
2. 细粒度的性能优化（通过 `shouldComponentUpdate`）
3. 支持错误边界（Error Boundaries）

### 劣势
1. 代码相对冗长
2. `this` 绑定问题容易导致 bug
3. 逻辑复用较困难

## 二、函数组件（Function Component）

### 基本结构（带 Hooks）
```jsx
function MyFunctionComponent() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log('组件已挂载或count更新');
    return () => console.log('清理效果');
  }, [count]);

  const handlePress = useCallback(() => {
    setCount(c => c + 1);
  }, []);

  return (
    <View>
      <Text>{count}</Text>
      <Button title="增加" onPress={handlePress} />
    </View>
  );
}
```

### 特点
- **Hooks 支持**：使用 `useState`, `useEffect` 等 Hooks 实现状态和副作用
- **无实例**：没有 `this` 绑定问题
- **简洁语法**：通常代码量更少

### 优势
1. 代码更简洁易读
2. 更容易进行逻辑复用（通过自定义 Hooks）
3. 没有 `this` 绑定问题
4. React 团队推荐的主流写法

### 劣势
1. 在 Hooks 之前无法使用状态和生命周期
2. 某些高级功能（如错误边界）仍需类组件
3. 需要理解 Hooks 的依赖数组和闭包问题

## 三、高阶组件（Higher-Order Component, HOC）

### 基本结构
```jsx
function withLogger(WrappedComponent) {
  return class extends React.Component {
    componentDidMount() {
      console.log(`组件 ${WrappedComponent.name} 已挂载`);
    }

    render() {
      return <WrappedComponent {...this.props} />;
    }
  };
}

// 使用
const EnhancedComponent = withLogger(MyComponent);
```

### 特点
- **函数接收组件返回组件**：本质是一个函数，接收组件作为参数并返回新组件
- **props 代理**：可以操作和传递 props
- **逻辑复用**：用于横切关注点（如日志、鉴权等）

### 优势
1. 优秀的逻辑复用能力
2. 不影响原组件内部实现
3. 符合单一职责原则

### 劣势
1. 容易产生 props 命名冲突
2. 组件层级嵌套过深（"wrapper hell"）
3. 调试时组件名称可能不直观

## 三者的对比

| 特性                | Class 组件          | 函数组件            | 高阶组件            |
|---------------------|--------------------|--------------------|--------------------|
| **状态管理**         | this.state/setState | useState/useReducer | 通过props传递       |
| **生命周期**         | 完整支持            | useEffect等Hooks    | 依赖包装组件的类型   |
| **代码量**           | 较多               | 较少               | 中等（额外包装层）   |
| **逻辑复用**         | 困难               | 自定义Hooks        | 主要设计模式        |
| **性能优化**         | shouldComponentUpdate | React.memo, useMemo | 依赖实现方式        |
| **学习曲线**         | 中等               | 较低（基础部分）    | 较高               |
| **React推荐度**      | 旧版主流           | 当前主流           | 特定场景使用        |

## 使用场景建议

### 使用 Class 组件当：
1. 需要实现错误边界（Error Boundaries）
2. 维护遗留代码
3. 需要精细控制生命周期（如动画库）

### 使用函数组件当：
1. 新项目开发
2. 需要简洁的代码
3. 需要逻辑复用（自定义Hooks）
4. 大多数常规UI组件

### 使用高阶组件当：
1. 需要跨组件复用逻辑（如鉴权、日志）
2. 需要修改组件行为而不改变其实现
3. 开发第三方库/组件

## 代码示例对比

### 相同功能的三种实现

1. **Class 组件实现计数器**
```jsx
class Counter extends React.Component {
  state = { count: 0 };
  
  increment = () => this.setState({ count: this.state.count + 1 });
  
  render() {
    return (
      <View>
        <Text>{this.state.count}</Text>
        <Button title="增加" onPress={this.increment} />
      </View>
    );
  }
}
```

2. **函数组件实现计数器**
```jsx
function Counter() {
  const [count, setCount] = useState(0);
  const increment = useCallback(() => setCount(c => c + 1), []);
  
  return (
    <View>
      <Text>{count}</Text>
      <Button title="增加" onPress={increment} />
    </View>
  );
}
```

3. **高阶组件增强计数器**
```jsx
function withCounter(WrappedComponent) {
  return class extends React.Component {
    state = { count: 0 };
    increment = () => this.setState({ count: this.state.count + 1 });
    
    render() {
      return <WrappedComponent 
        count={this.state.count}
        increment={this.increment}
        {...this.props}
      />;
    }
  };
}

// 使用
const CounterDisplay = ({ count, increment }) => (
  <View>
    <Text>{count}</Text>
    <Button title="增加" onPress={increment} />
  </View>
);

const EnhancedCounter = withCounter(CounterDisplay);
```

## 现代 React Native 开发建议

1. **优先使用函数组件**：除非有明确需要使用类组件的理由
2. **合理使用Hooks**：
    - `useState` 管理状态
    - `useEffect` 处理副作用
    - `useCallback`/`useMemo` 优化性能
    - 自定义Hooks复用逻辑

3. **谨慎使用高阶组件**：
    - 考虑是否可以用Hooks替代
    - 注意避免props命名冲突
    - 保持高阶组件单一职责

4. **渐进式迁移**：
    - 新组件使用函数组件
    - 逐步重构旧类组件
    - 对复杂生命周期组件最后迁移

## 总结

React Native 的组件模型发展经历了从 Class 组件到函数组件+Hooks 的演进。当前最佳实践是：

- **主要使用函数组件**：利用 Hooks 实现简洁高效的代码
- **必要时使用类组件**：处理错误边界等特殊场景
- **合理应用高阶组件**：用于横切关注点的逻辑复用

理解这三种组件的区别和适用场景，能够帮助开发者根据具体需求选择最合适的实现方式，构建可维护、高性能的 React Native 应用。