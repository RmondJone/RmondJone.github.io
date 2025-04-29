---
title: "React Native Hook使用详解"
date: 2025-04-29T12:57:34+08:00
draft: false
categories: ["移动端"]
tags: ["React Native"]
---

React Hooks 是 React 16.8 引入的一项特性，它让你在不编写 class 的情况下使用 state 和其他 React 特性。在 React Native 中，Hooks 同样适用，并且极大地简化了组件的编写方式。

## 基础 Hooks

### 1. useState

`useState` 是最基本的 Hook，用于在函数组件中添加局部状态。

```jsx
import React, { useState } from 'react';
import { View, Text, Button } from 'react-native';

const Counter = () => {
  const [count, setCount] = useState(0);

  return (
    <View>
      <Text>You clicked {count} times</Text>
      <Button 
        title="Click me" 
        onPress={() => setCount(count + 1)}
      />
    </View>
  );
};
```

### 2. useEffect

`useEffect` 用于处理副作用操作，相当于 class 组件中的 `componentDidMount`, `componentDidUpdate` 和 `componentWillUnmount` 的组合。

```jsx
import React, { useState, useEffect } from 'react';
import { View, Text } from 'react-native';

const Timer = () => {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds(seconds => seconds + 1);
    }, 1000);
    
    return () => clearInterval(interval); // 清理函数
  }, []); // 空数组表示只在组件挂载和卸载时执行

  return (
    <View>
      <Text>Seconds: {seconds}</Text>
    </View>
  );
};
```

### 3. useContext

`useContext` 用于访问 React 的 Context。

```jsx
import React, { useContext } from 'react';
import { View, Text } from 'react-native';

const ThemeContext = React.createContext('light');

const ThemedComponent = () => {
  const theme = useContext(ThemeContext);
  
  return (
    <View style={{ backgroundColor: theme === 'dark' ? '#333' : '#FFF' }}>
      <Text style={{ color: theme === 'dark' ? '#FFF' : '#000' }}>
        Current theme: {theme}
      </Text>
    </View>
  );
};
```

## 额外 Hooks

### 4. useReducer

`useReducer` 是 `useState` 的替代方案，适用于复杂的状态逻辑。

```jsx
import React, { useReducer } from 'react';
import { View, Text, Button } from 'react-native';

const initialState = { count: 0 };

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    default:
      throw new Error();
  }
}

const Counter = () => {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <View>
      <Text>Count: {state.count}</Text>
      <Button 
        title="+" 
        onPress={() => dispatch({ type: 'increment' })}
      />
      <Button 
        title="-" 
        onPress={() => dispatch({ type: 'decrement' })}
      />
    </View>
  );
};
```

### 5. useCallback

`useCallback` 返回一个 memoized 回调函数，避免不必要的重新渲染。

```jsx
import React, { useState, useCallback } from 'react';
import { View, Button } from 'react-native';

const ExpensiveComponent = React.memo(({ onClick }) => {
  console.log('ExpensiveComponent rendered');
  return <Button title="Click me" onPress={onClick} />;
});

const ParentComponent = () => {
  const [count, setCount] = useState(0);
  
  const increment = useCallback(() => {
    setCount(c => c + 1);
  }, []);

  return (
    <View>
      <Text>Count: {count}</Text>
      <ExpensiveComponent onClick={increment} />
    </View>
  );
};
```

### 6. useMemo

`useMemo` 返回一个 memoized 值，用于性能优化。

```jsx
import React, { useState, useMemo } from 'react';
import { View, Text } from 'react-native';

const computeExpensiveValue = (a, b) => {
  // 模拟复杂计算
  return a * b;
};

const Calculator = ({ a, b }) => {
  const result = useMemo(() => computeExpensiveValue(a, b), [a, b]);
  
  return (
    <View>
      <Text>Result: {result}</Text>
    </View>
  );
};
```

### 7. useRef

`useRef` 返回一个可变的 ref 对象，其 `.current` 属性被初始化为传入的参数。

```jsx
import React, { useRef } from 'react';
import { TextInput, Button, View } from 'react-native';

const TextInputWithFocusButton = () => {
  const inputRef = useRef(null);

  const focusTextInput = () => {
    inputRef.current.focus();
  };

  return (
    <View>
      <TextInput
        ref={inputRef}
        style={{ height: 40, borderColor: 'gray', borderWidth: 1 }}
      />
      <Button
        title="Focus the text input"
        onPress={focusTextInput}
      />
    </View>
  );
};
```

## React Native 特有 Hooks

### 8. useWindowDimensions

获取窗口的尺寸，并在尺寸变化时自动更新。

```jsx
import { useWindowDimensions } from 'react-native';

const ResponsiveComponent = () => {
  const { height, width } = useWindowDimensions();
  
  return (
    <View>
      <Text>Window width: {width}</Text>
      <Text>Window height: {height}</Text>
    </View>
  );
};
```

### 9. useColorScheme

检测用户的首选配色方案（浅色/深色模式）。

```jsx
import { useColorScheme } from 'react-native';

const ThemeAwareComponent = () => {
  const colorScheme = useColorScheme();
  
  return (
    <View style={{
      backgroundColor: colorScheme === 'dark' ? '#333' : '#FFF',
      flex: 1
    }}>
      <Text style={{
        color: colorScheme === 'dark' ? '#FFF' : '#000'
      }}>
        Current theme: {colorScheme}
      </Text>
    </View>
  );
};
```

## 自定义 Hooks

你可以创建自己的 Hooks 来复用状态逻辑。

```jsx
import { useState, useEffect } from 'react';
import { Dimensions } from 'react-native';

function useWindowOrientation() {
  const [orientation, setOrientation] = useState(
    Dimensions.get('window').width > Dimensions.get('window').height 
      ? 'LANDSCAPE' 
      : 'PORTRAIT'
  );

  useEffect(() => {
    const onChange = ({ window }) => {
      setOrientation(
        window.width > window.height ? 'LANDSCAPE' : 'PORTRAIT'
      );
    };
    
    Dimensions.addEventListener('change', onChange);
    
    return () => Dimensions.removeEventListener('change', onChange);
  }, []);

  return orientation;
}

// 使用自定义 Hook
const OrientationComponent = () => {
  const orientation = useWindowOrientation();
  
  return (
    <View>
      <Text>Current orientation: {orientation}</Text>
    </View>
  );
};
```

## Hooks 使用规则

1. **只在最顶层使用 Hooks**：不要在循环、条件或嵌套函数中调用 Hook
2. **只在 React 函数中调用 Hooks**：在 React 的函数组件或自定义 Hook 中调用 Hook

## 总结

React Hooks 为 React Native 开发带来了更简洁、更直观的代码组织方式。通过使用 Hooks，你可以：

- 在不编写 class 的情况下使用 state 和其他 React 特性
- 将组件拆分为更小的函数，而非强制按照生命周期划分
- 更轻松地复用状态逻辑
- 减少代码量，提高可读性

掌握 Hooks 的使用可以显著提升你的 React Native 开发效率和代码质量。