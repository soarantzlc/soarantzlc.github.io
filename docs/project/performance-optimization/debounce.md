# 防抖
防抖（Debounce）是一种常见的前端优化技术，用于限制某个函数在短时间内多次触发时的执行频率。
它常用于处理输入框的输入事件、窗口的缩放事件、滚动事件等场景，以减少性能开销和提升用户体验。

## 原理
当事件被触发时，设置一个延迟执行的定时器。如果在延迟时间内事件再次被触发，则重置定时器，只有当延迟时间结束后，事件处理函数才会执行一次

## javascript版

### 基础实现

```js
function debounce(func, wait, immediate = false) {
    let timeout; // 用于存储定时器的标识

    // 返回一个新的函数（闭包）
    return function (...args) {
        const context = this; // 保存当前的上下文

        // 清除之前的定时器
        if (timeout) clearTimeout(timeout);

        // 判断是否需要立即执行
        if (immediate) {
            // 如果定时器不存在（即第一次触发事件），则立即执行
            const callNow = !timeout;
            timeout = setTimeout(() => {
                timeout = null; // 清空定时器标识
            }, wait);

            if (callNow) {
                func.apply(context, args); // 立即执行
            }
        } else {
            // 非立即执行模式：等到延迟时间结束后执行
            timeout = setTimeout(() => {
                func.apply(context, args);
            }, wait);
        }
    };
}
```
- `func`：需要防抖处理的函数。
- `wait`：延迟时间，单位为毫秒。
- `immediate`：可选参数，默认为 `false`。
  - 如果设置为 `true`，则在事件触发时立即执行函数。
    - 当事件触发时，立即执行一次函数，然后在延迟时间内不再执行。
    - 当事件再次触发时，重置定时器，延迟时间结束后再次执行。
  - 如果设置为 `false`，则在延迟时间结束后执行函数。
    - 当事件触发时，设置一个定时器，延迟一段时间后执行函数。

### 使用示例

```js
    const inputElement = document.querySelector('input');

    // 监听输入事件，使用防抖函数
    inputElement.addEventListener('input', debounce((e) => {
        console.log('输入内容：', e.target.value);
    }, 300))

```

- 立即执行常用于监控防止功能按钮被重复点击
- 延迟执行常用于监控输入框的输入事件、窗口的缩放事件、滚动事件等场景


## react版

### 基础实现

由于React中封装了上下文导致了触发闭包机制，因此可以使用useCallback来实现防抖。

```jsx
import { useState, useEffect, useCallback } from "react";

// 自定义防抖 Hook
function useDebounce(callback, delay, immediate = false) {
  const [timer, setTimer] = useState(null);

  // 清理定时器
  const clearTimer = useCallback(() => {
    if (timer) {
      clearTimeout(timer);
      setTimer(null);
    }
  }, [timer]);

  // 返回一个防抖后的函数
  return useCallback(
          (...args) => {
            clearTimer(); // 清除之前的定时器

            if (immediate && !timer) {
              // 如果是立即执行模式且没有定时器，则直接执行
              callback(...args);
            }

            // 设置新的定时器
            const newTimer = setTimeout(() => {
              if (!immediate) {
                // 如果是延迟执行模式，则在延迟后执行
                callback(...args);
              }
              clearTimer(); // 清理定时器
            }, delay);

            setTimer(newTimer); // 保存定时器
          },
          [callback, delay, immediate, timer, clearTimer]
  );
}
```
### 使用示例

```jsx
import React, { useState } from "react";

function SearchBox() {
  const [inputValue, setInputValue] = useState("");

  // 使用防抖 Hook，延迟 500ms，延迟执行
  const handleSearch = useDebounce((query) => {
    console.log("Searching:", query);
  }, 500, false);

  // 使用防抖 Hook，延迟 500ms，立即执行
  const handleImmediateSearch = useDebounce((query) => {
    console.log("Immediate Searching:", query);
  }, 500, true);

  return (
    <div>
      <input
        type="text"
        value={inputValue}
        onChange={(e) => {
          setInputValue(e.target.value);
          handleSearch(e.target.value); // 延迟执行
        }}
        placeholder="Search (Debounced)"
      />

      <input
        type="text"
        value={inputValue}
        onChange={(e) => {
          setInputValue(e.target.value);
          handleImmediateSearch(e.target.value); // 立即执行
        }}
        placeholder="Search (Immediate)"
      />
    </div>
  );
}

export default SearchBox;
```
