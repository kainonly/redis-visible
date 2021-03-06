# 管道

## Defined 定义

- @Pipe({name: 'Defined'})
- value `any` 数值
- return `boolean`

例如，假设存在一个未定义属性

```typescript
export class AnyComponent {
  test;
}
```

在模版中判断使用

```html
<div *ngIf="test|Defined">
  <!-- here no display -->
</div>
```

## Undefined 未定义

- @Pipe({name: 'Undefined'})
- value `any` 数值
- return `boolean`

例如，假设存在一个未定义属性

```typescript
export class AnyComponent {
  test;
}
```

在模版中判断使用

```html
<div *ngIf="test|Undefined">
  <!-- here display -->
</div>
```

## EmptyArray 空数组

- @Pipe({name: 'EmptyArray'})
- value `any[]` 数组
- return `boolean`

例如，假设存在一个空数组属性

```typescript
export class AnyComponent {
  test = [];
}
```

在模版中判断使用

```html
<div *ngIf="!(test|EmptyArray)">
  <!-- here customize -->
</div>
```

## EmptyObject 空对象

- @Pipe({name: 'EmptyObject'})
- value `any` 对象
- return `boolean`

例如，假设存在一个空对象属性

```typescript
export class AnyComponent {
  test = {};
}
```

在模版中判断使用

```html
<div *ngIf="!(test|EmptyObject)">
  <!-- here customize -->
</div>
```

## ObjectToArray 对象转数组

- @Pipe({name: 'ObjectToArray'})
- value `any` 对象
- return `any[]`

例如，假设一个对象

```typescript
export class AnyComponent {
  some = {
    car1: "blue",
    car2: "red",
  };
}
```

在模版中判断使用

```html
<div *ngFor="let x of some|ObjectToArray">
  <p>{{x.key}}</p>
  <p>{{x.rows}}</p>
</div>
```

## ObjectToMap 对象转 Map 对象

- @Pipe({name: 'ObjectToMap'})
- value `any` 对象
- return `Map`

例如，假设一个对象

```typescript
export class AnyComponent {
  some = {
    car1: "blue",
    car2: "red",
  };
}
```

在模版中判断使用

```html
<ng-container *ngIf="some|ObjectToMap as x">
  <p>{{x.get('car1')}}</p>
</ng-container>
```

## Locale 语言包显示

- @Pipe({name: 'Locale'})
- value `string | object` JSON 字符串或 JSON
- locale `string` 语言包标识

例如，在接口直接返回多语言类型的 JSON 字符串

```typescript
const data = `{"zh_cn":"我","en_us":"me"}`;
```

如果指定某个语言为默认的显示

```html
<p>{{data|Locale:'en_us'}}</p>
<!-- display me -->
```

## JSONParse JSON 字符串转数组

- @Pipe({name: 'JSONParse'})
- value `string` JSON 字符串
- chkey `string` 子键

例如，在接口直接返回 JSON 字符串数组

```typescript
const data = `{"name":"bit","version":1}`;
```

在这种情况下直接使用管道可以减少遍历处理

```html
<p>{{data|JSONParse:name}}</p>
<!-- display bit -->
```

## Split 字符串分割为数组

- @Pipe({name: 'Split'})
- text `string` 字符串
- symbol `string` 分割符号
- return `string`

例如，在一些场景下存在字符串

```typescript
const str = "a1|b2|c3";
```

那么可以通过该管道进行字符串分割

```html
<p>{{(str|Split:'|')[0]}}</p>
<!-- display a1 -->
```

## Join 数组拼接为字符串

- @Pipe({name: 'Join'})
- origin `string[]` 字符串数组
- symbol `string` 拼接符号
- return `string`

例如，在一些场景下存在字符串数组

```typescript
const data = ["a1", "b2", "c3"];
```

那么可以通过该管道进行字符串拼接

```html
<p>{{data|Join:','}}</p>
<!-- display a1,b2,c3 -->
```
