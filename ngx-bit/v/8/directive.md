# 指令

## bitBack 返回上一级

`[bitBack]`

- @Input() bitTrigger `click|touch` 触发方式，默认 `click`

添加在按钮中

```html
<button nz-button type="button" bitBack>{{bit.l['cancel']}}</button>
```

---

## bitCrossLevel 跨级跳转

`[bitCrossLevel]`

- @Input() bitCrossLevel `string` 跨级路由名称
- @Input() bitTrigger `click|touch` 触发方式，默认 `click`

例如使用在面包屑

```html
<nz-breadcrumb [nzSeparator]="breadcrumbIcon">
  <ng-template #breadcrumbIcon>
    <i nz-icon type="right"></i>
  </ng-template>
  <nz-breadcrumb-item>
    <a routerLink="/">{{bit.l['dashboard']}}</a>
  </nz-breadcrumb-item>
  <nz-breadcrumb-item *ngFor="let x of bit.breadcrumb;last as islast">
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

---

## bitCol 栅格标识

栅格标识，由配置决定

`[bitCol]`

在表单中使用

```html
<nz-form-item>
  <nz-form-label bitCol="label" nzRequired> {{bit.l['status']}} </nz-form-label>
  <nz-form-control bitCol="control">
    <nz-switch
      formControlName="status"
      [nzCheckedChildren]="bit.l['on']"
      [nzUnCheckedChildren]="bit.l['off']"
    >
    </nz-switch>
  </nz-form-control>
</nz-form-item>
```

---

## bitFormSubmit 表单提交

`[bitFormSubmit]`

- @Output() bitFormSubmit `EventEmitter<any>` 监听表单提交

在表单中使用

```html
<form nz-form [formGroup]="form" (bitFormSubmit)="submit($event)">....</form>
```

```typescript
form: FormGroup;

constructor(private fb: FormBuilder) {
}

this.form = this.fb.group({...});

submit(data) {...}
```

---

## bitI18nUpdate 多语言联动验证

`[bitI18nUpdate]`

在表单中使用

```html
<nz-form-item formGroupName="name">
  <nz-form-label bitFormLabelCol nzRequired> {{bit.l['name']}} </nz-form-label>
  <ng-container *ngFor="let x of bit.i18nContain">
    <nz-form-control
      *ngIf="bit.equalI18n(x)"
      nzHasFeedback
      bitFormControlCol
      [nzErrorTip]="name.ref"
    >
      <input
        nz-input
        bitI18nUpdate
        [nz-tooltip]="tooltip.ref"
        [formControlName]="x"
        [placeholder]="bit.l['namePlaceholder']"
      />
      <bit-i18n-tooltip #tooltip groupName="name"></bit-i18n-tooltip>
      <bit-error-tip
        #name
        [hasError]="{
            required:bit.l['nameRequire']
        }"
      ></bit-error-tip>
    </nz-form-control>
  </ng-container>
</nz-form-item>
```

---

## bitOpen 路由跳转

`[bitOpen]`

- @Input() bitOpen `any[]` 路由跳转处理，`path[0]` 为基础地址，索引大于 0 则为参数，例如：`['app-edit',1]` 等价于 routerlink 的 `{app-edit}/1`，但包含跨级路由处理
- @Input() extras `NavigationExtras` 修改导航策略的选项
- @Input() bitTrigger `click|touch` 触发方式，默认 `click`

例如替代 `routerlink`

```html
<button nz-button nzType="primary" nzSize="small" [bitOpen]="['sys-add']">
  <span>{{bit.l['add']}}</span>
</button>

<a [bitOpen]="['sys-add',data.id]">
  <i nz-icon type="edit"></i> {{bit.l['edit']}}
</a>
```

---

## bitSearchChange 搜索监听

`[bitSearchChange]`

- @Input() bitSearchChange `string` 搜索命名
- @Input() variable `object` 局部搜索变量
- @Output() after `EventEmitter<any>` 搜索变动之后

监听包含 `NgModelChange` 的组件中

```html
<ng-container *ngIf="bit.hasSearch('name')">
  <nz-select
    [(ngModel)]="bit.search['name'].value"
    bitSearchChange="sys-index"
    (after)="getLists(true)"
  >
    <nz-option [nzValue]="x.id" [nzLabel]="x.name"></nz-option>
  </nz-select>
</ng-container>
```

---

## bitSearchStart 搜索触发

`[bitSearchStart]`

- @Input() bitSearchStart `string` 搜索命名
- @Input() variable `object` 局部搜索变量
- @Output() after `EventEmitter< any >` 开始搜索之后

注册搜索字段

```typescript
this.bit
  .registerSearch("app-index", { field: "name", op: "like", value: "" })
  .subscribe(() => {});
```

同时给组件加入 `click` 与 `enter` 触发搜索

```html
<nz-input-group nzSearch [nzAddOnAfter]="nzAddOnAfter">
  <input
    type="text"
    [(ngModel)]="bit.search['name'].value"
    bitSearchStart="app-index"
    (after)="getLists(true)"
    nz-input
    [placeholder]="bit.l['search']"
  />
</nz-input-group>

<ng-template #nzAddOnAfter>
  <button
    nz-button
    nzType="primary"
    nzSearch
    bitSearchStart="app-index"
    (after)="getLists(true)"
  >
    <i nz-icon type="search"></i>
  </button>
</ng-template>
```

---

## bitSearchClear 搜索清除

`[bitSearchClear]`

- @Input() bitSearchClear `string` 搜索命名
- @Input() variable `object` 局部搜索变量
- @Input() reset `any` 清除重置的数值
- @Output() after `EventEmitter<any>` 清空搜索之后

注册搜索字段

```typescript
this.bit
  .registerSearch(
    "api-index",
    { field: "tag", op: "=", value: 0 },
    { field: "name", op: "like", value: "" }
  )
  .subscribe(() => {});
```

清空搜索绑定在按钮 `click` 事件

```html
<button nz-button bitSearchClear="api-index" (after)="getLists(true)">
  {{bit.l['clearSearch']}}
</button>

<button
  nz-button
  bitSearchClear="api-index"
  [reset]="{tag:0}"
  (after)="getLists(true)"
>
  {{bit.l['clearSearch']}}
</button>
```

---

## bitStatusChange 状态切换

`[bitStatusChange]`

- @Input() bitStatusChange `Observable<any>` 状态切换请求
- @Input() bitControl `boolean` 是否手动处理返回提示，默认 `false`
- @Output() response `EventEmitter<any>` 获取请求的响应值

例如控制管理员的状态

```html
<nz-switch
  [(ngModel)]="data.status"
  [bitStatusChange]="ramService.status(data)"
  [bitControl]="true"
  (response)="status($event)"
>
</nz-switch>
```

自定义返回提示

```typescript
status(res: any) {
    switch (res.msg) {
        case 'error:self':
            this.notification.error(this.bit.l.operateError, this.bit.l.errorStatusSelf);
            break;
        default:
            this.notification.error(this.bit.l.operateError, this.bit.l.statusError);
    }
}
```

---

## bitUpload 上传

`[bitUpload]`

关联辅助配置，在上传组件中加入属性指令

```html
<nz-upload
  nzName="image"
  bitUpload
  nzListType="picture-card"
  (nzChange)="upload($event)"
>
  <ng-container *ngIf="!avatar">
    <i nz-icon type="plus"></i>
    <div class="ant-upload-text">{{bit.l['upload']}}</div>
  </ng-container>
  <img width="120" *ngIf="avatar" [src]="bit.static+avatar" class="avatar" />
</nz-upload>
```
