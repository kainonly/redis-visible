# 生产

## Operates 操作库

辅助框架还提供一些常用到的操作库

### 创建异步验证器

- asyncValidator(req: Observable<any>, field = 'duplicated'): Observable<any> 
  - req `Observable<any>` 请求对象
  - field `string` 自定义返回

假设验证key字段是否唯一，`response` 是模拟请求的响应值，通常这样使用请求服务

```typescript
import { Component, OnInit } from '@angular/core';
import { AbstractControl, FormBuilder, FormGroup } from '@angular/forms';
import { asyncValidator } from 'ngx-bit/operates';
import { of } from 'rxjs';

@Component({
  selector: 'app-welcome',
  templateUrl: './welcome.component.html',
  styleUrls: ['./welcome.component.scss']
})
export class WelcomeComponent implements OnInit {
  form: FormGroup;

  constructor(
    private fb: FormBuilder
  ) {
  }

  ngOnInit(): void {
    this.form = this.fb.group({
      key: [null, [], [this.existsKey]]
    });
  }

  existsKey = (control: AbstractControl) => {
    const response = of({
      error: 0,
      data: true
    });
    return asyncValidator(response);
  };
}
```

### 判断数组是否为空 

- emptyArray(array: any[])
  - array `any[]` 数组
  - Return `boolean`

``` typescript
const data = [];

emptyArray(data);
// true
```

### 判断对象是否为空

- emptyObject(object: any)
  - object `any` 对象
  - Return `boolean`

``` typescript
const data = {};

emptyObject(data);
// true

emptyObject([]);
// false
```

### 生产 BitConfigService 

- factoryBitConfig(config: BitConfig)
- config `BitConfig`

通常使用在环境配置文件中，例如 `src\environments\environment.ts`

```typescript
import { en_US, zh_CN } from 'ng-zorro-antd';
import { factoryBitConfig } from 'ngx-bit/operates';

const bit = factoryBitConfig({
  url: {
    api: 'http://localhost:9501',
    static: 'https://cdn.example.com/',
    icon: 'https://cdn.example.com/'
  },
  api: {
    namespace: '/system',
    upload: '/system/main/uploads',
    withCredentials: true
  },
  col: {
    label: {
      nzXXl: 4,
      nzXl: 5,
      nzLg: 6,
      nzMd: 7,
      nzSm: 24
    },
    control: {
      nzXXl: 8,
      nzXl: 9,
      nzLg: 10,
      nzMd: 14,
      nzSm: 24
    },
    submit: {
      nzXXl: { span: 8, offset: 4 },
      nzXl: { span: 9, offset: 5 },
      nzLg: { span: 10, offset: 6 },
      nzMd: { span: 14, offset: 6 },
      nzSm: { span: 24, offset: 0 }
    }
  },
  locale: {
    default: 'zh_cn',
    mapping: new Map<number, string>([
      [0, 'zh_cn'],
      [1, 'en_us']
    ]),
    bind: new Map<string, any>([
      ['zh_cn', zh_CN],
      ['en_us', en_US]
    ])
  },
  i18n: {
    default: 'zh_cn',
    contain: ['zh_cn', 'en_us'],
    switch: [
      {
        i18n: 'zh_cn',
        name: {
          zh_cn: '中文',
          en_us: 'Chinese'
        }
      },
      {
        i18n: 'en_us',
        name: {
          zh_cn: '英文',
          en_us: 'English'
        }
      }
    ]
  },
  page: 20
});

export const environment = {
  production: false,
  bit
};
```

### 生产语言包工具

- factoryLocales(dataset: object, mapping: Map< number, string >)
  - dataset `object` 语言包数据源
  - mapping `Map<number, string>` 语言包文件中的索引映射

```typescript
import('../common.language').then(result => {
    const lang = factoryLocales(result.default, environment.bit.locale.mapping);
});
```

### 返回 QuerySchema 

- getQuerySchema(options: SearchOption[]): any[]
  - options `SearchOption[]` 条件数组

通过 `SearchOption[]` 组合成 Laravel Query 的条件数组（ThinkPHP 同样支持）

```typescript
let schema = getQuerySchema([
    { field: 'username', op: '=', value: '' }
]);

// console: []

schema = getQuerySchema([
    { field: 'username', op: '=', value: '', must: true }
]);

// console: ['username', '=', '']]

schema = getQuerySchema([
    { field: 'username', op: '=', value: 'kain' }
]);

// console: [['username', '=', 'kain']]

schema = getQuerySchema([
    { field: 'username', op: '=', value: null },
    { field: 'type', op: '=', value: 0 },
    { field: 'ids', op: 'in', value: [] },
    { field: 'error', op: '=', value: {} }
]);

// console: []
```

### 通过URL获取路由标签

- getSelectorFormUrl(url: string, match: any[])
  - url `string` URL地址
  - match `any[]` 筛选规则

```typescript
let key = getSelectorFormUrl('/%7Bacl-index%7D', ['%7B', '%7D']);

// console: acl-index

key = getSelectorFormUrl('/%7Bacl-edit%7D/1', ['%7B', '%7D']);

// console: acl-edit

key = getSelectorFormUrl('/%7Bacl-edit%7D/1/2', ['%7B', '%7D']);

// console: acl-edit
```

### JSONSchema 验证器 

- validate(schema: string | boolean | object, data: any): any
  - schema `string | boolean | object` 验证规则
  - data `any` 验证数据

```typescript
let valid = validate({
    enum: [1, 2, 3, 4]
}, 1);

// console: valid.error false

valid = validate({
    enum: [1, 2, 3, 4]
}, 5);

// console: valid.error true
```

---

## ListByPage 分页列表

ListByPage 提供了分页列表结构所需的基本条件，创建通常需要 `BitService` 协助，例如：

```typescript
import { Component, OnInit } from "@angular/core";
import { ListByPage } from "ngx-bit/factory";
import { BitService } from "ngx-bit";

@Component({
  selector: "app-welcome",
  templateUrl: "./welcome.component.html",
  styleUrls: ["./welcome.component.scss"],
})
export class WelcomeComponent implements OnInit {
  lists: ListByPage;

  constructor(public bit: BitService) {}

  ngOnInit(): void {
    this.lists = this.bit.listByPage({
      id: "test",
      limit: 10,
      query: [{ field: "username", op: "=", value: "" }],
    });
  }
}
```

| 属性            | 说明                     | 类型                                | 默认值      |
| --------------- | ------------------------ | ----------------------------------- | ----------- |
| `search`        | 搜索字段定义             | `{ [field: string]: SearchOption }` | `{}`        |
| `data`          | 分页列表数据             | `any[]`                             | `[]`        |
| `loading`       | 分页列表加载状态         | `boolean`                           | `true`      |
| `limit`         | 分页记录数量             | `number`                            | `0`         |
| `totals`        | 分页总数                 | `number`                            | `0`         |
| `index`         | 分页页码                 | `number`                            | `1`         |
| `checked`       | 分页列表是否全被选中     | `boolean`                           | `false`     |
| `indeterminate` | 分页列表是否不完全被选中 | `boolean`                           | `false`     |
| `batch`         | 是否可进行批量处理       | `boolean`                           | `false`     |
| `checkedNumber` | 分页列表被选中的数量     | `number`                            | `0`         |

### 设置分页列表数据

- setData(data: any[])
  - data `any[]` 数据源

当分页列表请求返回时设置

```typescript
getLists(refresh = false) {
    service.lists(
        this.lists,
        refresh,
        event !== undefined
    ).subscribe(data => {
        this.lists.setData(data);
    });
}
```

### 判断是否包含该字段的搜索条件

- hasSearch(field: string)
  - field `string` 字段名称

通常这被使用在模板上

```html
<nz-space-item *ngIf="lists.hasSearch('username')">
  <nz-input-group nzSearch [nzAddOnAfter]="nzAddOnAfter" style="width: 320px">
    <input
      nz-input
      [bitSearchStart]="lists"
      [placeholder]="bit.l['search']"
      [(ngModel)]="lists.search['username'].value"
      (after)="getLists(true)"
    />
  </nz-input-group>
  <ng-template #nzAddOnAfter>
    <button
      nzSearch
      nz-button
      nzType="primary"
      [bitSearchStart]="lists"
      (after)="getLists(true)"
    >
      <i nz-icon nzType="search"></i>
    </button>
  </ng-template>
</nz-space-item>
```

### 主动触发搜索变动之后

- afterSearch(): Observable< any >
  - Return `Observable<any>`

在不是 `bit-search-change` 与 `bit-search-start` 指令触发的情况下，就需要主动执行：

```typescript
this.lists.afterSearch().subscribe((status) => {
  // ...
});
```

### 主动触发搜索清空之后

- clearSearch(reset: any = {}): Observable< any >
  - reset `any` 重置的数值
  - Return `Observable<any>`

在不是 `bit-search-clear` 指令触发的情况下，就需要主动执行：

```typescript
this.lists
  .clearSearch({
    username: "",
  })
  .subscribe((status) => {
    // ...
  });
```

### 更新分页列表状态

- refreshStatus()

即是否全部选中、不完全选中、是否可执行批量与选中数量，通常列表数据中发生变化需主动执行

```html
<nz-table
  #table
  [nzData]="lists.data"
  [nzLoading]="lists.loading"
  [nzTotal]="lists.totals"
  [nzPageSize]="lists.limit"
  [nzFrontPagination]="false"
  [(nzPageIndex)]="lists.index"
  (nzPageIndexChange)="getLists()"
  nzSize="middle"
>
  <thead>
    <tr>
      <th
        nzShowCheckbox
        nzWidth="65px"
        [(nzChecked)]="lists.checked"
        [nzIndeterminate]="lists.indeterminate"
        (nzCheckedChange)="lists.checkedAll($event)"
      ></th>
      <th>{{bit.l['username']}}</th>
    </tr>
  </thead>
  <tbody>
    <tr *ngFor="let data of table.data">
      <!-- 这里执行 -->
      <td
        nzShowCheckbox
        [(nzChecked)]="data.checked"
        (nzCheckedChange)="lists.refreshStatus()"
      ></td>
      <td>{{data.username}}</td>
    </tr>
  </tbody>
</nz-table>
```

### 更改所有分页列表选中状态

- checkedAll(event: boolean)
  - event `boolean` 选中状态

可附加在主选择器的状态监听中

```html
<nz-table
  #table
  [nzData]="lists.data"
  [nzLoading]="lists.loading"
  [nzTotal]="lists.totals"
  [nzPageSize]="lists.limit"
  [nzFrontPagination]="false"
  [(nzPageIndex)]="lists.index"
  (nzPageIndexChange)="getLists()"
  nzSize="middle"
>
  <thead>
    <tr>
      <!-- 这里执行 -->
      <th
        nzShowCheckbox
        nzWidth="65px"
        [(nzChecked)]="lists.checked"
        [nzIndeterminate]="lists.indeterminate"
        (nzCheckedChange)="lists.checkedAll($event)"
      ></th>
      <th>{{bit.l['username']}}</th>
    </tr>
  </thead>
  <tbody>
    <tr *ngFor="let data of table.data">
      <td
        nzShowCheckbox
        [(nzChecked)]="data.checked"
        (nzCheckedChange)="lists.refreshStatus()"
      ></td>
      <td>{{data.username}}</td>
    </tr>
  </tbody>
</nz-table>
```

### 返回所有被选中的列表

- getChecked(): any[]
  - Return `any[]`

```typescript
const checkedLists = this.lists.getChecked();
```

### 获取当前的页码

- getPage(): Observable<any>
  - Return `Observable<any>`

```typescript
this.lists.getPage().subscribe((index) => {
  // index
});
```

### 执行分页页码的持久化记录

- persistence()

```typescript
this.lists.persistence();
```

### 返回 QuerySchema

- toQuerySchema(): any[]

将 `search` 组合成 Laravel Query 的条件数组（ThinkPHP 同样支持）

```typescript
const schema = this.lists.toQuerySchema();

// console: []
```
