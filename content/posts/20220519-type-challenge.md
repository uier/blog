---
title: "type challenge 作答紀錄"
date: 2022-05-19T22:06:07+08:00
---

[See type challenge repo on GitHub](https://github.com/type-challenges/type-challenges)

---

## easy

### Pick

<details>
  <summary>Spoiler warning</summary> 
  
  ```typescript
  type MyPick<T, K extends keyof T> = {
    [key in K]: T[key]
  }
  ```
</details>

### Readonly

<details>
  <summary>Spoiler warning</summary> 
  
  ```typescript
  type MyReadonly<T> = {
    readonly [K in keyof T]: T[K]
  }
  ```
</details>

### Tuple to Object

<details>
  <summary>Spoiler warning</summary> 
  
  ```typescript
  type TupleToObject<T extends readonly string[]> = {
    [K in T[number]]: K
  }
  ```
</details>

### First of Array

<details>
  <summary>Spoiler warning</summary> 
  
  ```typescript
  type First<T extends any[]> = T extends never[] ? never : T[0]
  // or
  type First<T extends any[]> = T['length'] extends 0 ? never : T[0]
  ```
</details>

### Length of Tuple

<details>
  <summary>Spoiler warning</summary> 
  
  ```typescript
  type Length<T extends readonly any[]> = T['length']
  ```
</details>

### Exclude

<details>
  <summary>Spoiler warning</summary> 
  
  ```typescript
  type MyExclude<T, U> = T extends U ? never : T
  ```
</details>

### Awaited

<details>
  <summary>Spoiler warning</summary> 
  
  ```typescript
  type MyAwaited<T extends Promise<any>> =
    T extends Promise<infer V>
      ? V extends Promise<any>
        ? MyAwaited<V>
        : V
      : never
  ```
</details>

### If

<details>
  <summary>Spoiler warning</summary> 
  
  ```typescript
  type If<C extends boolean, T, F> = C extends true ? T : F
  ```
</details>

### Concat

<details>
  <summary>Spoiler warning</summary> 
  
  ```typescript
  type Concat<T extends any[], U extends any[]> = [...T, ...U]
  ```
</details>

### Includes

<details>
  <summary>Spoiler warning</summary> 
  
  ```typescript
  type Includes<T extends readonly any[], U> = 
    true extends { [K in keyof T]: Equal<T[K], U> }[number]
      ? true
      : false 
  ```
</details>

### Push

<details>
  <summary>Spoiler warning</summary> 
  
  ```typescript
  type Push<T extends any[], U> = [...T, U]
  ```
</details>

### Unshift

<details>
  <summary>Spoiler warning</summary> 
  
  ```typescript
  type Unshift<T extends any[], U> = [U, ...T]
  ```
</details>

### Parameters

<details>
  <summary>Spoiler warning</summary> 
  
  ```typescript
  type MyParameters<T extends (...args: any[]) => any> = 
    T extends (...args: infer Args) => any ? Args : 1
  ```
</details>

## medium

### Get Return Type

<details>
  <summary>Spoiler warning</summary> 
  
  ```typescript
  type MyReturnType<T> =
    T extends (...args: any) => infer Return ? Return : never
  ```
</details>

### Omit

<details>
  <summary>Spoiler warning</summary> 
  
  ```typescript
  type MyOmit<T, K extends keyof T> = {
    [key in Exclude<keyof T, K>]: T[key]
  }
  ```
</details>

### Readonly 2

<details>
  <summary>Spoiler warning</summary> 
  
  ```typescript
  type MyReadonly2<T, K extends keyof T = keyof T> =
    { readonly [key in keyof T]: T[key] } &
    { [key in Exclude<keyof T, K>]: T[key] }
  ```
</details>

### Deep Readonly

<details>
  <summary>Spoiler warning</summary> 
  
  ```typescript
  type DeepReadonly<T> =
    keyof T extends never
      ? T
      : { readonly [K in keyof T]: DeepReadonly<T[K]> }
  ```
</details>

### Tuple to Union

<details>
  <summary>Spoiler warning</summary> 
  
  ```typescript
  type TupleToUnion<T extends unknown[]> = T[number]
  // or 
  type TupleToUnion<T> = T extends Array<infer F> ? F : never
  ```
</details>

### Chainable Options

<details>
  <summary>Spoiler warning</summary> 
  
  ```typescript
  type Chainable<T = {}> = {
    option<K extends string, V>(
      key: K extends keyof T ? never : K,
      value: V,
    ): Chainable<T & { [k in K]: V }>
    get(): T
  }
  ```
</details>

### Last of Array

<details>
  <summary>Spoiler warning</summary> 
  
  ```typescript
  type Last<T extends any[]> = T extends [...any, infer F] ? F : never
  ```
</details>

### Pop

<details>
  <summary>Spoiler warning</summary> 
  
  ```typescript
  type Pop<T extends any[]> = T extends [...infer F, any] ? F : never
  ```
</details>

### Promise.all 

<details>
  <summary>Spoiler warning</summary> 
  
  ```typescript
  declare function PromiseAll<T extends unknown[]>(
    values: readonly [...T]
  ): Promise<{
    [index in keyof T]: T[index] extends Promise<infer F> ? F : T[index]
  }>
  ```
</details>

### Type LookUp

<details>
  <summary>Spoiler warning</summary> 
  
  ```typescript
  type LookUp<U, T extends string> =
    U extends (infer D) ?
    D extends { type: string } ?
    D['type'] extends T ? D : never : never : never
  ```
</details>

### Trim Left

<details>
  <summary>Spoiler warning</summary> 
  
  ```typescript
  type TrimLeft<S extends string> =
    S extends ` ${infer T}` | `\n${infer T}` | `\t${infer T}` ? TrimLeft<T> : S
  ```
</details>

### Trim

<details>
  <summary>Spoiler warning</summary> 
  
  ```typescript
  type Trim<S extends string> = S extends
    ` ${infer T}` | `\t${infer T}` | `\n${infer T}` |
    `${infer T} ` | `${infer T}\t` | `${infer T}\n` ? Trim<T> : S
  ```
</details>

### Capitalize

<details>
  <summary>Spoiler warning</summary> 
  
  ```typescript
  type MyCapitalize<S extends string> =
    S extends `${infer P}${infer T}` ? `${Uppercase<P>}${T}` : S
  ```
</details>

### Replace

<details>
  <summary>Spoiler warning</summary> 
  
  ```typescript
  type Replace<S extends string, From extends string, To extends string> = 
    From extends ''
    ? S
    : S extends `${infer Pre}${From}${infer Suf}`
      ? `${Pre}${To}${Suf}`
      : S
  ```
</details>

### ReplaceAll

<details>
  <summary>Spoiler warning</summary> 
  
  ```typescript
  type ReplaceAll<S extends string, From extends string, To extends string> =
    From extends ''
      ? S
      : S extends `${infer Pre}${From}${infer Suf}`
        ? `${Pre}${To}${ReplaceAll<Suf, From, To>}`
        : S
  ```
</details>