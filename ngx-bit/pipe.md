# 其他

## 管道

### Empty 空判断

通过管道判断数值是否为空

```
{{ value_expression | Empty }}
```

- value `any` 任何
- Return `boolean`

假设存在一个空数组 `lists`（`undefined` `''` `0` `false` `null` `[]` `{}` 都被判断为空）

```typescript
import { Component, OnInit } from "@angular/core";

@Component({
  selector: "app-welcome",
  templateUrl: "./welcome.component.html",
  styleUrls: ["./welcome.component.scss"],
})
export class WelcomeComponent implements OnInit {
  lists: any[] = [];

  ngOnInit(): void {}
}
```

在模版中判断使用

```html
<div *ngIf="lists|Empty">
  <!-- 数组为空显示内容 -->
</div>
```

### Split 字符串分割为数组

通过管道使用字符串分割为数组

```
{{ value_expression | Split: symbol }}
```

- value `string` 字符串
- symbol `string` 分割符号
- Return `string`

假设存在一个字符串集合 `text`

```typescript
import { Component, OnInit } from "@angular/core";

@Component({
  selector: "app-welcome",
  templateUrl: "./welcome.component.html",
  styleUrls: ["./welcome.component.scss"],
})
export class WelcomeComponent implements OnInit {
  text = "1|2|3|4|5|6";

  ngOnInit(): void {}
}
```

那么可以通过该管道进行字符串分割

```html
<ng-container *ngFor="let val of text|Split:'|'">
  <p>{{val}}</p>
</ng-container>
```

### Join 数组拼接为字符串

通过管道使用数组拼接为字符串

```
{{ value_expression | Join: symbol }}
```

- value `string[]` 字符串数组
- symbol `string` 拼接符号
- Return `string`

假设存在一个字符串数组 `lists`

```typescript
import { Component, OnInit } from "@angular/core";

@Component({
  selector: "app-welcome",
  templateUrl: "./welcome.component.html",
  styleUrls: ["./welcome.component.scss"],
})
export class WelcomeComponent implements OnInit {
  lists: string[] = ["nodejs", "php", "golang", "java", "python"];

  ngOnInit(): void {}
}
```

那么可以通过该管道进行字符串拼接

```html
<p>{{lists|Join:'+'}}</p>

<!-- display nodejs+php+golang+java+python -->
```

### object 转换对象

通过管道转换对象，目标可以是 JSON 字符串或对象，也可作用语言包显示

```
{{ value_expression | object?: locale }}
```

- value `string | object` JSON 字符串或 JSON
- locale `string` 语言包标识
- Return `string`

假设存在一个多语言 JSON 字符串 `lang`

```typescript
import { Component, OnInit } from "@angular/core";

@Component({
  selector: "app-welcome",
  templateUrl: "./welcome.component.html",
  styleUrls: ["./welcome.component.scss"],
})
export class WelcomeComponent implements OnInit {
  lang = `{"zh_cn":"你好世界","en_us":"Hello World"}`;

  ngOnInit(): void {}
}
```

如果指定某个语言为默认的显示

```html
<p>{{lang|object:'zh_cn'}}</p>
<!-- display 你好世界 -->
<p>{{lang|object:'en_us'}}</p>
<!-- display Hello World -->
```

当然如果 `lang` 为相同的 json 对象也可以输出相同的内容，在项目中的某个多语言字段，通常配置 `bit.locale` 实现输出控制

```html
<p>{{data.name|object:bit.locale}}</p>
```

`object` 也可以用于 JSON 显示，有时数据返回并不是那么友好，例如返回的字段中 JSON 为字符串

```typescript
import { Component, OnInit } from "@angular/core";

@Component({
  selector: "app-welcome",
  templateUrl: "./welcome.component.html",
  styleUrls: ["./welcome.component.scss"],
})
export class WelcomeComponent implements OnInit {
  lists: any = [
    { id: 1, extra: '{"ttl":3600,"on":true,"plus":20}' },
    { id: 2, extra: '{"ttl":1800,"on":false,"plus":10}' },
  ];

  ngOnInit(): void {}
}
```

在这种情况下可直接使用管道

```html
<ng-container *ngFor="let x of lists">
  <p>{{(x.extra|object)['ttl']}}</p>
</ng-container>
```

### Print 字符串模板

通过管道打印字符串模板，可适用于多语言模板

```
{{ value_expression | print: vars }}
```

- value `string` 字符串文本
- vars `any[]` 变量，替换$0,$1,$2....$N
- Return `string`

假设存在一个语言包模板

```typescript
import { Component, OnInit } from "@angular/core";

@Component({
  selector: "app-welcome",
  templateUrl: "./welcome.component.html",
  styleUrls: ["./welcome.component.scss"],
})
export class WelcomeComponent implements OnInit {
  lang = `$0 是遵循 $1 设计规范的 $2 组件库`;

  ngOnInit(): void {}
}
```

打印语言包

```html
<p>{{lang|print:['ng-zorro-antd', 'Ant Design', 'Angular UI']}}</p>
<!-- display ng-zorro-antd 是遵循 Ant Design 设计规范的 Angular UI 组件库 -->
```

### Privacy 字符串脱敏

通过管道对字符串脱敏

```
{{ value_expression | Privacy: range }}
```

- value `string` 字符串
- range `string` 切片范围
- Return `string`

假设存在一个待脱敏的字符串 `text`

```typescript
import { Component, OnInit } from "@angular/core";

@Component({
  selector: "app-welcome",
  templateUrl: "./welcome.component.html",
  styleUrls: ["./welcome.component.scss"],
})
export class WelcomeComponent implements OnInit {
  text = "123456789";

  ngOnInit(): void {}
}
```

针对该字符串 3 到 6 的索引进行脱敏

```html
<p>{{text|Privacy:'3,6'}}</p>

<!-- display 123***789 -->
```