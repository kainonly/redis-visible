# 服务

---

## BitService 助手工具

辅助架构中的助手工具，以下示例中 `bit` 为 `BitService` 服务的注入命名

### 基本属性

| 属性          | 说明                 | 类型                | 默认值 |
| ------------- | -------------------- | ------------------- | ------ |
| `static`      | 静态资源地址         | `string`            | `''`   |
| `uploads`     | 组合后的统一上传路径 | `string`            | `''`   |
| `locale`      | 多语言标识           | `string`            | `''`   |
| `l`           | 语言包索引           | `object`            | `{}`   |
| `i18n`        | 多语言输入标识       | `string`            | `''`   |
| `i18nTooltip` | 多语言输入提示       | `I18nTooltipOption` | `{}`   |
| `i18nContain` | 多语言输入类型集合   | `string[]`          | `[]`   |

- `uploads` 为组合后的统一上传路径（即接口地址+统一上传路径），通常可以直接使用在 `nz-upload` 组件中

```html
<!-- 使用在nzAction -->
<nz-upload
  nzName="image"
  [nzAction]="bit.uploads"
  [nzWithCredentials]="config.api.withCredentials"
  [nzSize]="5120"
  [nzShowUploadList]="false"
>
</nz-upload>
<!-- 使用更简洁的方式实现相同定义 -->
<nz-upload nzName="image" bitUpload [nzShowUploadList]="false"> </nz-upload>
```

- `locale` 为当前多语言标识状态通常使用在 `ObjectPipe`，例如：

```html
{{name|object:bit.locale}}
```

- `l` 为语言包索引，在组件完成语言包注册后可在模板中使用，例如：

```html
{{bit.l['dashboards']}}
```

- `i18nContain` 为多语言输入类型集合，通常可以用来筛选多语言输入的 ID，例如：

```html
<nz-form-item formGroupName="name">
  <nz-form-label bitFormLabelCol nzRequired> {{bit.l['name']}} </nz-form-label>
  <ng-container *ngFor="let x of bit.i18nContain">
    <nz-form-control *ngIf="bit.equalI18n(x)" bitFormControlCol nzHasFeedback>
      <input nz-input [formControlName]="x" />
    </nz-form-control>
  </ng-container>
</nz-form-item>
```

### 路由导航

- open(urlTree: any[], extras?: NavigationExtras)
  - urlTree `any[]` urlTree
  - extras `NavigationExtras` 修改导航策略的选项

`navigate` 路由导航的扩展，为跨级导航作为基础持久记录其路由参数

```typescript
bit.open(["admin-edit", 1]);
```

### 路由历史

- history(key: string)
  - selector `string` 路由标签

路由历史导航，加载被历史缓存的参数导航，例如：

```typescript
// 假设导航至/{team-index}/1
bit.open(["team-index", 1]);
// 再从/{team-index}/1导航至/{team-index}/1/services/T100
bit.open(["team-index", 1, "services", "T100"]);
// 此时使用history，可导航至/{team-index}/1
bit.history("team-index");
```

### 返回上一级

- back()

返回上一级，并重置 `i18n` 多语言输入标识

```typescript
// 导航至/{admin-index}
bit.open(["admin-index"]);
// 再从/{admin-index}导航至/{admin-edit}/1
bit.open(["admin-edit", 1]);
// 返回至/{admin-index}
bit.back();
```

### 局部注册语言包

- registerLocales(packer: Promise<any>)

将局部语言包加工与公共语言包合并，再提供给 `bit.l`，通常文件定义在组件内 `language.ts`

- packer `Promise< any >` 导入的语言包文件

```typescript
import { Component, OnInit } from "@angular/core";
import { BitService } from "ngx-bit";

@Component({
  selector: "app-welcome",
  templateUrl: "./welcome.component.html",
  styleUrls: ["./welcome.component.scss"],
})
export class WelcomeComponent implements OnInit {
  constructor(public bit: BitService) {}

  ngOnInit(): void {
    this.bit.registerLocales(import("./language"));
  }
}
```

### 设置多语言标识

- setLocale(locale: string)
  - locale `string` 语言包标识

```typescript
import { Component, OnInit } from "@angular/core";
import { BitService } from "ngx-bit";

@Component({
  selector: "app-welcome",
  templateUrl: "./welcome.component.html",
  styleUrls: ["./welcome.component.scss"],
})
export class WelcomeComponent implements OnInit {
  constructor(public bit: BitService) {}

  ngOnInit(): void {}

  switchToEnglish() {
    this.bit.setLocale("en_us");
  }
}
```

### 生成分页列表对象

- listByPage(option: ListByPageOption): ListByPage
  - option `ListByPageOption` 分页列表参数

```typescript
import { Component, OnInit } from "@angular/core";
import { BitService } from "ngx-bit";
import { ListByPage } from "ngx-bit/factory";

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
      id: "welcome",
      query: [{ field: "type", op: "=", value: 0 }],
    });
  }
}
```

### 判断多语言输入标识是否相等

- equalI18n(i18n: string): boolean
  - i18n `string` 多语言输入标识

```html
<nz-form-item formGroupName="name">
  <nz-form-label bitFormLabelCol nzRequired> {{bit.l['name']}} </nz-form-label>
  <ng-container *ngFor="let ID of bit.i18nContain">
    <nz-form-control *ngIf="bit.equalI18n(ID)" bitFormControlCol nzHasFeedback>
      <input
        nz-input
        [placeholder]="bit.l['namePlaceholder']"
        [formControlName]="ID"
      />
    </nz-form-control>
  </ng-container>
</nz-form-item>
```

### 重置多语言输入标识

- resetI18n()

```typescript
this.bit.resetI18n();
```

### 多语言输入 FormGroup 初始化

- i18nGroup(options: I18nGroupOption): any
  - options `I18nGroupOptions` 多语言组件参数
    - value `object` 默认值
      - ${ID} `array` 属于某个多语言标识的数值
    - validate `object` 同步验证器数组
      - ${ID} `array` 属于某个多语言标识的同步验证器
    - asyncValidate `object` 异步验证器数组
      - ${ID} `array` 属于某个多语言标识的异步验证器

```typescript
import { Component, OnInit } from "@angular/core";
import { BitService } from "ngx-bit";
import {
  AbstractControl,
  AsyncValidatorFn,
  FormBuilder,
  FormGroup,
  Validators,
} from "@angular/forms";
import { of } from "rxjs";

@Component({
  selector: "app-welcome",
  templateUrl: "./welcome.component.html",
  styleUrls: ["./welcome.component.scss"],
})
export class WelcomeComponent implements OnInit {
  form: FormGroup;

  constructor(public bit: BitService, private fb: FormBuilder) {}

  ngOnInit(): void {
    this.form = this.fb.group({
      name: this.fb.group(
        this.bit.i18nGroup({
          value: {
            zh_cn: "测试",
            en_us: "TEST",
          },
          validate: {
            zh_cn: [Validators.required],
            en_us: [],
          },
          asyncValidate: {
            en_us: [this.fun1],
          },
        })
      ),
    });
  }

  fun1: AsyncValidatorFn = (control: AbstractControl) => {
    return of({ error: true, duplicated: true });
  };
}
```

---

## BitEventsService 组件通讯

BitEventsService 为组件提供事件通讯功能，以下示例中 `events` 为 `BitEventsService` 服务的注入命名，例如多语言的切换基于：

```typescript
events.on("locale").subscribe((args) => {
  console.log(args);
  // zh_cn or en_us
});
```

### 判断组件通讯事件是否存在

- exists(topic: string): boolean
  - topic `string` 主题名称

```typescript
events.exists("test");
```

### 发布组件通讯事件

- publish(topic: string, args?: any)
  - topic `string` 主题名称
  - args `args` 发送参数

```typescript
events.publish("any", {
  name: "kain",
});
```

### 订阅组件通讯事件

- on(topic: string): Observable<any>
  - topic `string` 主题名称
  - Return `Observable<any>`

```typescript
events.on("any").subscribe((args) => {
  console.log(args);
});
```

### 取消订阅的组件通讯事件

- off(topic: string)
  - topic `string` 主题名称

```typescript
events.off("any");
```

在每次路由组件 `OnDestory` 时，都需要将自定义事件取消订阅

---

## BitHttpService 请求处理

BitHttpService 请求处理是对 `HttpClient` 的封装，可以通过它快捷对接后端 CURD 接口，以下示例中 `http` 为 `BitHttpService` 服务的注入命名

### 创建请求对象

- req(url: string, body: any = {}, method = 'post'): Observable<any>
  - url `string` 请求路由
  - body `any` 发送数据
  - method `string` 请求类型, 默认为 `post` 请求
  - Return `Observable<any>`

```typescript
// 例如：请求资源接口
http.req("main/resource").subscribe((res) => {
  console.log(res);
});
```

### 创建获取单条数据的请求

- get(model: string, condition: number | string | SearchOption[], order?: OrderOption, path?: string): Observable<any>
  - model `string` 模块名称
  - condition `number | string | SearchOptions[]` 查询条件，当类型为 `number` 或 `string` 是将作为主键返回后端，若为 `SearchOptions[]` 则会组合成 Laravel Query 的条件数组以 `where` 返回后端（ThinkPHP 同样支持）
  - order `OrderOption` 排序条件
  - path `string` 自定义路径

```typescript
// 主键查询
http.get(this.model, id);
// 条件查询
http.get(this.model, [{ field: "username", op: "=", value: "kain" }]);
```

### 创建分页列表数据的请求

- lists(model: string, factory: ListByPage, option: ListsOption, path?: string): Observable<any>
  - model `string` 模块名称
  - factory `ListByPage` 分页列表对象
  - option `ListsOption`
    - refresh `boolean` 刷新，即重置分页
    - persistence `boolean` 持久存储，即记录分页历史
  - path `string` 自定义路径

```typescript
const search = bit.listByPage({
  id: "admin-index",
  query: [{ field: "sex", op: "=", value: 1 }],
});

http.lists("admin", search, {
  refresh: true,
  persistence: true,
});
```

### 创建列表数据的处理

- originLists(model: string, condition: SearchOption[] = [], order?: OrderOption, path?: string): Observable<any>
  - model `string` 模块名称
  - condition `SearchOptions[]` 条件数组，组合成 Laravel Query 的条件数组以 `where` 返回后端（ThinkPHP 同样支持）
  - order `OrderOption` 排序条件
  - path `string` 自定义路径

```typescript
http.originLists("admin");
```

### 创建一个新增的处理

- add(model: string, data: any, path?: string): Observable<any>
  - model `string` 模块名称
  - data `any` body 数据
  - path `string` 自定义路径

```typescript
const data = {
  username: "kain",
  email: "zhangtqx@vip.qq.com",
};

http.add("admin", data);
```

### 创建一个编辑的处理

- edit(model: string, data: any, condition?: SearchOptions[], path?: string): Observable<any>
  - model `string` 模块名称
  - data `any` body 数据
  - condition `SearchOptions[]` 条件数组，组合成 Laravel Query 的条件数组以 `where` 返回后端（ThinkPHP 同样支持）
  - path `string` 自定义路径

```typescript
const data = {
  id: 1,
  username: "kain",
  email: "kainonly@qq.com",
};

http.edit("admin", data);

// 当没有主键时，需要设置 condition
const condition = [{ field: "username", op: "=", value: "kain" }];
const data = {
  email: "kainonly@qq.com",
};

http.edit("admin", data, condition);
```

### 创建状态切换的请求

- status(model: string, data: any, field = 'status', extra?: any, path?: string): Observable<any>
  - model `string` 模块名称
  - data `any` body 数据
  - field `string` 状态字段，默认 `status`
  - extra `any` 扩展字段
  - path `string` 自定义路径

状态将以相反的数值提交给后端

```typescript
const data = {
  id: 1,
  status: true,
};
// 此时将为后端传递的 status 为 false
http.status("admin", data);

// 如果状态字段为线上 online 可以这样设定
const data = {
  id: 1,
  online: true,
};

http.status("admin", data, "online");

// 假设它必须要增加一段关键的字段，则可以
const data = {
  id: 1,
  online: true,
};
// 此时 {key:'a123'} 将合并入 body 内提交后端
http.status("admin", data, "online", {
  key: "a123",
});
```

### 创建删除的请求

- delete(model: string, id?: any[], condition?: SearchOption[], path?: string): Observable<any>
  - model `string` 模块名称
  - id `any[]` 主键数组
  - condition `SearchOptions[]` 条件数组，组合成 Laravel Query 的条件数组以 `where` 返回后端（ThinkPHP 同样支持）
  - path `string` 自定义路径

`id` 与 `condition` 两者必须选一种

```typescript
// 使用主键数组删除
http.delete("admin", [1]);
// 使用条件数组删除
http.delete("admin", undefined, [
  { field: "username", op: "=", value: "kain" },
]);
```

---

## BitSupportService 功能支持

BitSupportService 作为附加的功能支持，以下示例中 `support` 为 `BitSupportService` 服务的注入命名

### 基本属性

`BitSupportService` 服务所包含以下属性：

| 属性            | 说明             | 类型                 | 默认值 |
| --------------- | ---------------- | -------------------- | ------ |
| `title`         | 当前路由的标题   | `any`                | `''`   |
| `breadcrumb`    | 面包屑数组       | `BreadcrumbOption[]` | `[]`   |
| `breadcrumbTop` | 面包屑默认最高级 | `any`                | `0`    |
| `navActive`     | 被激活的导航     | `any[]`              | `[]`   |

- `title` 可以使用在组件模板的页头中，显示当前路由的标题

```html
<nz-page-header
  [nzTitle]="bit.support.title|Locale:bit.locale"
></nz-page-header>
```

- `breadcrumb` 为仪表盘面包屑提供数据

```html
<nz-breadcrumb class="app-breadcrumb" [nzSeparator]="breadcrumbIcon">
  <ng-template #breadcrumbIcon>
    <i nz-icon nzType="right"></i>
  </ng-template>
  <nz-breadcrumb-item>
    <a routerLink="/">{{bit.l['dashboard']}}</a>
  </nz-breadcrumb-item>
  <nz-breadcrumb-item *ngFor="let x of support.breadcrumb;last as islast">
    <ng-container *ngIf="islast;else notLast"
      >{{x.name|Locale:bit.locale}}</ng-container
    >
    <ng-template #notLast>
      <a *ngIf="x.router;else notRouterlink" [bitCrossLevel]="x.key">
        {{x.name|Locale:bit.locale}}
      </a>
      <ng-template #notRouterlink>{{x.name|Locale:bit.locale}}</ng-template>
    </ng-template>
  </nz-breadcrumb-item>
</nz-breadcrumb>
```

- `breadcrumbTop` 用于定义面包屑默认最高级，这取决于面包屑数据对父节点的定义
- `navActive` 为仪表盘导航菜单提供激活状态

```html
<ul
  nz-menu
  [nzInlineCollapsed]="collapsed"
  [nzMode]="collapsed?'vertical':'inline'"
>
  <ng-container
    *ngTemplateOutlet="navTpl; context: {$implicit: navLists}"
  ></ng-container>
  <ng-template #navTpl let-navs>
    <ng-container *ngFor="let x of navs">
      <ng-container *ngIf="x.router;else notRouter">
        <li
          nz-menu-item
          [nzSelected]="support.navActive.indexOf(x.key)!==-1"
          [bitOpen]="[x.key]"
        >
          <i nz-icon [nzType]="x.icon"></i>
          <span class="nav-text">{{x.name|Locale:bit.locale}}</span>
        </li>
      </ng-container>
      <ng-template #notRouter>
        <li nz-submenu [nzOpen]="support.navActive.indexOf(x.key)!==-1">
          <span title
            ><i nz-icon [nzType]="x.icon"></i
            ><span>{{x.name|Locale:bit.locale}}</span></span
          >
          <ul>
            <ng-container
              *ngTemplateOutlet="navTpl; context: {$implicit: x.children}"
            ></ng-container>
          </ul>
        </li>
      </ng-template>
    </ng-container>
  </ng-template>
</ul>
```

### 手动设置面包屑

- setBreadcrumb(...breadcrumb: BreadcrumbOption[])
  - breadcrumb `BreadcrumbOption[]` 面包屑参数
    - $.name `any` 面包屑名称
    - $.key `string` 路由标签
    - $.router `0 | 1` 是否为路由

```typescript
const data: BreadcrumbOption[] = [
  {
    name: {
      zh_cn: "测试1",
    },
    key: "test2",
    router: 1,
  },
  {
    name: {
      zh_cn: "测试2",
    },
    key: "test2",
    router: 1,
  },
];
support.setBreadcrumb(...data);
```

### 设置资源

- setResource(resource: Map< string, any >, router: Map< string, any >)
  - resource `Map<string, any>` 资源数据
  - router `Map<string, any>` 路由数据

为辅助框架提供路由解析基础，后端数据可查看 https://github.com/kainonly/ngx-bit/blob/master/projects/ngx-bit/mock/resource.json

```typescript
const resourceData: Map<string, any> = new Map<string, any>();
const routerData: Map<string, any> = new Map<string, any>();
const nav: any = [];

if (!res.error) {
for (const x of res.data) {
    resourceData.set(x.key, x);
    if (x.router === 1) {
        routerData.set(x.key, x);
    }
}
for (const x of res.data) {
    if (!x.nav) {
        continue;
    }

    if (x.parent === 'origin') {
        nav.push(x);
    } else {
        const parent = x.parent;
        if (resourceData.has(parent)) {
            const rows = resourceData.get(parent);
            if (!rows.hasOwnProperty('children')) {
                rows.children = [];
            }
            rows.children.push(x);
        }
    }
}
support.setResource(resourceData, routerData);
```

### 安装 support 支持

- setup(router: Router, match: string[] = ['%7B', '%7D'])
  - router `Router` 应用 `Router` 对象
  - match `string[]` 路由标签获取符，默认`['%7B', '%7D']`

```typescript
support.setup(this.router);
```

### 清除辅助框架相关本地存储

- clearStorage()

```typescript
support.clearStorage();
```

### 取消 support 订阅

- unsubscribe()

```typescript
support.unsubscribe();
```

---

## BitSwalService 提示确认

BitSwalService 是基于 sweetalert2 的提交反馈栏服务

### 创建提示确认框

- create(option: AlertOption): Observable<any>
  - title `string` 标题
  - content `string` 内容
  - type `SweetAlertIcon` 图标类型
  - width `number` 宽度
  - okText `string` 确认按钮文字
  - okDanger `boolean` 确认按钮是否为危险按钮
  - okShow `boolean` 是否显示确认按钮
  - cancelText `string` 取消按钮文字
  - cancelShow `boolean` 是否显示取消按钮

```typescript
export class AclIndexComponent implements OnInit {
  constructor(public bit: BitService, private swal: BitSwalService) {}

  ngOnInit(): void {
    this.swal
      .create({
        title: "This is a success message",
        content: "some messages...some messages...",
        type: "success",
      })
      .subscribe(() => {});
  }
}
```

### 新增返回反馈栏

- addAlert(res: any, form: FormGroup, reset?: any): Observable<any>
  - res `any` 请求响应结果
  - form `FormGroup` 表单对象
  - reset `any` FormGroup 重置值

例如, 在新增操作下组件表单提交中使用, `status` 为 `true` 表示确认提示框

```typescript
export class AclAddComponent implements OnInit {
  constructor(
    public bit: BitService,
    private swal: BitSwalService,
    private aclService: AclService
  ) {}

  submit(data): void {
    this.aclService
      .add(data)
      .pipe(
        switchMap((res) =>
          this.swal.addAlert(res, this.form, {
            status: true,
          })
        )
      )
      .subscribe(() => {});
  }
}
```

### 修改返回反馈栏

- editAlert(res: any): Observable<any>
  - res `any` 请求响应结果

例如, 在修改操作下组件表单提交中使用, `status` 为 `true` 表示确认提示框

```typescript
export class AclEditComponent implements OnInit {
  private id: any;

  constructor(
    public bit: BitService,
    private swal: BitSwalService,
    private aclService: AclService
  ) {}

  submit(data): void {
    Reflect.set(data, "id", this.id);
    this.aclService
      .edit(data)
      .pipe(switchMap((res) => this.swal.editAlert(res)))
      .subscribe((status) => {
        if (status) {
          this.getData();
        }
      });
  }
}
```

### 删除返回反馈栏

- deleteAlert(observable: Observable<any>): Observable<any>
  - observable `Observable< any >` 异步处理

例如, 在删除操作下使用, 订阅返回删除请求对象的响应值

```typescript
export class AclIndexComponent implements OnInit {
  constructor(
    public bit: BitService,
    private swal: BitSwalService,
    private message: NzMessageService,
    public aclService: AclService
  ) {}

  deleteData(id: any[]): void {
    this.swal.deleteAlert(this.aclService.delete(id)).subscribe((res) => {
      if (!res.error) {
        this.message.success(this.bit.l.deleteSuccess);
        this.getLists(true);
      } else {
        this.message.error(this.bit.l.deleteError);
      }
    });
  }
}
```
