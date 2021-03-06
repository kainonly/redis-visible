# 管道

## 定义

- @Pipe({name: 'Defined'})
- value 数值
- Return `boolean`

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

## 未定义

- @Pipe({name: 'Undefined'})
- value 数值
- Return `boolean`

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

## 空数组

- @Pipe({name: 'EmptyArray'})
- value 数组
- Return `boolean`

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

## 空对象

- @Pipe({name: 'EmptyObject'})
- value 对象
- Return `boolean`

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

## 对象转数组

- @Pipe({name: 'ObjectToArray'})
- value 对象
- Return `any[]`

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

## 对象转 Map 对象

- @Pipe({name: 'ObjectToMap'})
- value 对象
- Return `Map`

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

## 多语言填补显示

- @Pipe({name: 'JSONChose'})
- value 多语言类型的 JSON 字符串
- locale 语言包标识

例如，在接口直接返回多语言类型的 JSON 字符串

```typescript
const data = `{"zh_cn":"我","en_us":"me"}`;
```

如果指定某个语言为默认的显示

```html
<p>{{data|JSONChose:'en_us'}}</p>
<!-- display me -->
```

当然如果多语言类型的 JSON 字符串有缺失的语言存在

```typescript
const data = `{"zh_cn":"","en_us":"","ru-ru":"Я"}`;
```

这种情况则显示可以填补的语言

```html
<p>{{data|JSONChose}}</p>
<!-- display Я -->
```

## JSON 字符串转数组

- @Pipe({name: 'JSONParse'})
- value JSON 字符串
- chkey 子键

例如，在接口直接返回 JSON 字符串数组

```typescript
const data = `{"name":"bit","version":1}`;
```

在这种情况下直接使用管道可以减少遍历处理

```html
<p>{{data|JSONParse:name}}</p>
<!-- display bit -->
```

## 字符串分割为数组

- @Pipe({name: 'Split'})
- text 字符串
- symbol 分割符号
- Return `string`

例如，在一些场景下存在字符串

```typescript
const str = "a1|b2|c3";
```

那么可以通过该管道进行字符串分割

```html
<p>{{(str|Split:'|')[0]}}</p>
<!-- display a1 -->
```

## 数组拼接为字符串

- @Pipe({name: 'Join'})
- origin 字符串数组
- symbol 拼接符号
- Return `string`

例如，在一些场景下存在字符串数组

```typescript
const data = ["a1", "b2", "c3"];
```

那么可以通过该管道进行字符串拼接

```html
<p>{{data|Join:','}}</p>
<!-- display a1,b2,c3 -->
```
