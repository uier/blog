---
title: "筆記 React.js 中 useMemo Hook"
date: 2021-07-29T18:13:04+08:00
draft: false
tags: ["React.js"]
categories:
- "Front-end"
---

## 前言

暑假回去公司實習時把 React 文件拿出來重新複習一次，順便在一些可以優化的地方使用 `useMemo`, `useCallback`，順手做個紀錄。

## 被計算出來的狀態使用 `useMemo`

### 範例：計算按鈕何時應啟用 disabled 樣式

```javascript
// Before: 自己寫 useEffect 去監聽依賴，在依賴變動時重新賦值
const [input, setInput] = useState('')
const [isInputError, setIsInputError] = useState(false)
const [isBtnDisabled, setIsBtnDisabled] = useState(true)

useEffect(() => {
  setIsBtnDisabled(input === '' || isInputError)
}, [input, isInputError])

// ------------------------------------------------------
// After: 使用 useMemo

const [input, setInput] = useState('')
const [isInputError, setIsInputError] = useState(false)
const isBtnDisabled = useMemo(() => {
  return (input === '' || isInputError)
}, [input, isInputError])
```

`useMemo` 也可以 memorize 一個 functional component


### 備註

同樣是監聽一些 state 去響應另一個 state，`useMemo` 的改寫我覺得可讀性較高。

在 Vue.js 中，`computed` 屬性在做類似的事情，跟 React 不同的是，Vue.js 會自動追蹤計算的過程牽扯到哪些依賴，也就是不需要 dependency array。

老實說，我翻了一些的 realworld example，卻不常看到 `useMemo` 被使用，還蠻令我驚訝的，我想 `useMemo` 應該不僅僅是這樣，有空會再尋找它在實際上是如何被使用的。

參考資料：
- [Hooks API 參考](https://zh-hant.reactjs.org/docs/hooks-reference.html#usememo)
- [Vue computed](https://v3.vuejs.org/guide/computed.html#basic-example)

---


## Child Component 使用 React.memo

在 functional component 之前，React 提供 `React.Component` 和 `React.PureComponent` 來撰寫元件。

[React.Component - React](https://zh-hant.reactjs.org/docs/react-component.html)

其中的差異在於，`React.Component` 需自行實作 `shouldComponentUpdate` 這個 function 來控制在 props 或 states 更新後是否需要重新 render，而 `React.PureComponent` 則沒有 `shouldComponentUpdate`，會以 shallow compare 的方式來比較舊的 props, states 和新的 props, states 來決定是否重新 render。

在 functional component 中，當 parent component 觸發了重新渲染後，child component 也會重新 render，**即使 props 沒有更新**，若要避免這個問題，應該使用 `React.memo` 來包住 child component，他的用途是對於 props 做 shallow compare 來決定是否需要重新 render。

另外，`React.memo` 可以在第二個參數傳遞一個自定義的 compare function，也就是保留了彈性，當不想要用其預設的 shallow compare，可以自已撰寫邏輯。


### 範例：使用 `React.memo` 包裹子元件

我在 [codesandbox](https://codesandbox.io/s/react-memo-7mhki) 寫了一個範例，點進去後打開 console 可以發現，一開始初始化 App 後三個 component 都被渲染出來，之後在按下 add count1 按鈕時，`Child1` 跟 `Child2` 更新了，其中我們知道 `Child1` 有關注 `count1` 所以更新的合理，但 `Child2` 在這個情況下可以不必更新。
而用 `React.memo` 包起來的 `MemoChild2` 不會在每次父元件更新就跟著更新，而是做 shallow compare。

### 使用時機
沒有使用 `React.memo` 可能會導致子元件做不必要的更新，但使用了 `React.memo` 也有 shallow compare 帶來的成本。
> 問題：shallow compare 的成本跟重新渲染的成本相比是不是小很多？

假如父元件將更新 N 次，而其中 M 次 props 並沒有更新，另外 N-M 次 props 有更新，那麼在 M 較小時我們應該不使用 `React.memo`，應該在 M 較大時使用。

這樣來看，在 Container component 與 Presentational component 之間就不需要用 `React.memo` 包住 Presentational component，因為幾乎每次更新都是因為 props 更新。

想一個相反的情境，例如表單 component 有較多狀態的情況，若有包含其他 component 如錯誤時要跳出來的 Alert，因為他只關心少數狀態，就可以使用 `React.memo`，或者用 `useMemo`。

### 備註
在 Vue.js 中，子元件的重渲染機制預設就使用 shallow compare，目前看來在 React 中要自己 handle。

參考資料：
- [React 頂層 API](https://zh-hant.reactjs.org/docs/react-api.html)
- [我該如何實作 shouldComponentUpdate？](https://zh-hant.reactjs.org/docs/hooks-faq.html#how-do-i-implement-shouldcomponentupdate)
- [關於 props 的記憶，React Memo](https://ithelp.ithome.com.tw/articles/10240296)

