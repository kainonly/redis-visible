# 指令

## 返回上一级

`[bitBack]`

- @Input() bitTrigger 触发方式 `click|touch`，默认 `click`

添加在按钮中

```html
<button nz-button type="button" bitBack>{{bit.l['cancel']}}</button>
```

## 跨级跳转

`[bitCrossLevel]`

- @Input() bitCrossLevel: string 跨级路由名称
- @Input() bitTrigger = 'click' 触发方式 `click|touch`，默认 `touch`

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
    <ng-container *ngIf="islast;else notLast">{{x.name}}</ng-container>
    <ng-template #notLast>
      <a *ngIf="x.routerlink;else notRouterlink" [bitCrossLevel]="x.routerlink">
        {{x.name}}
      </a>
      <ng-template #notRouterlink>{{x.name}}</ng-template>
    </ng-template>
  </nz-breadcrumb-item>
</nz-breadcrumb>
```

## 表单验证提示

`[bitExplain]`

- @Input() set bitExplain(args: BitExplainArgs) 设置 `formControl` 的表单提醒信息

例如在表单中使用

```html
<nz-form-item>
  <nz-form-label bitFormLabelCol nzRequired>
    {{bit.l['username']}}
  </nz-form-label>
  <nz-form-control bitFormControlCol nzHasFeedback>
    <input
      nz-input
      formControlName="username"
      [placeholder]="bit.l['usernamePlaceholder']"
    />
    <nz-form-explain
      *bitExplain="{
        form:form,
        name:'username',
        async:true,
        explain:{
        required:bit.l['usernameRequire'],
        minlength:bit.l['usernameCorrectly'],
        maxlength:bit.l['usernameCorrectly'],
        duplicated:bit.l['usernameDuplicated'],
        pending:bit.l['validating']
        }
    };let msg"
      >{{msg}}</nz-form-explain
    >
  </nz-form-control>
</nz-form-item>
```

## 表单共用栅格

`[bitFormControlCol]`

在表单中使用

```html
<nz-form-control bitFormControlCol nzHasFeedback>
  <input
    nz-input
    formControlName="email"
    [placeholder]="bit.l['emailPlaceholder']"
  />
  <nz-form-explain
    *bitExplain="{
        form:form,
        name:'email',
        explain:{
        email:bit.l['emailCorrectly']
        }
    };let msg"
    >{{msg}}</nz-form-explain
  >
</nz-form-control>

<nz-form-control bitFormControlCol="submit">
  <button nz-button nzType="primary" [disabled]="!form.valid">
    {{bit.l['submit']}}
  </button>
  <button nz-button type="button" bitBack>{{bit.l['cancel']}}</button>
</nz-form-control>
```

`[bitFormLabelCol]`

在表单中使用

```html
<nz-form-label bitFormLabelCol> {{bit.l['email']}} </nz-form-label>
```

## 表单提交

`[bitFormSubmit]`

- @Output() bitFormSubmit: EventEmitter< any > 监听表单提交

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

## 多语言提示样式

`[bitI18nTipsStyle]`

多语言提示样式统一，与组件 `<bit-i18n-tips #tips name="name"></bit-i18n-tips>` 配合使用

```html
<bit-i18n-tips #tips name="name"></bit-i18n-tips>
<input
  nz-input
  [placeholder]="bit.l['namePlaceholder']"
  [nz-tooltip]="tips.ref"
  bitI18nTipsStyle
  [formControlName]="x"
  (ngModelChange)="bit.i18nUpdateValueAndValidity(form,'name',x)"
/>
```

## 路由跳转

`[bitOpen]`

- @Input() bitOpen: any[] 路由跳转处理，`path[0]` 为基础地址，索引大于 0 则为参数，例如：`['app-edit',1]` 等价于 routerlink 的 `{app-edit}/1`，但包含跨级路由处理
- @Input() bitTrigger = 'click' 触发方式 `click|touch`，默认 `click`

例如替代 `routerlink`

```html
<button nz-button nzType="primary" nzSize="small" [bitOpen]="['sys-add']">
  <span>{{bit.l['add']}}</span>
</button>

<a [bitOpen]="['sys-add',data.id]">
  <i nz-icon type="edit"></i> {{bit.l['edit']}}
</a>
```

## 搜索监听

`[bitSearchChange]`

- @Input() bitSearchChange: string 搜索命名
- @Output() after: EventEmitter< any > 搜索变动之后

监听包含 `NgModelChange` 的组件中

```html
<ng-container *ngIf="bit.hasSearch(0)">
  <nz-select
    [(ngModel)]="bit.search[0].value"
    bitSearchChange="sys-index"
    (after)="getLists(true)"
  >
    <nz-option [nzValue]="x.id" [nzLabel]="x.name"></nz-option>
  </nz-select>
</ng-container>
```

## 搜索触发

`[bitSearchStart]`

- @Input() bitSearchStart: string 搜索命名
- @Output() after: EventEmitter< any > 开始搜索之后

注册搜索字段

```typescript
 this.bit.registerSearch('app-index', {
  field: 'name', op: 'like', value: ''
}).subscribe(() => {
  ...
});
```

同时给组件加入 `click` 与 `enter` 触发搜索

```html
<nz-input-group nzSearch [nzAddOnAfter]="nzAddOnAfter">
  <input
    type="text"
    [(ngModel)]="bit.search[0].value"
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

## 搜索清除

`[bitSearchClear]`

- @Input() bitSearchClear: string 搜索命名
- @Input() reset: any 清除重置的数值
- @Output() after: EventEmitter< any > 清空搜索之后

注册搜索字段

```typescript
this.bit.registerSearch('api-index', {
  field: 'tag', op: '=', value: 0,
}, {
  field: 'name', op: 'like', value: ''
}).subscribe(() => {
 ...
});
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

## 状态切换

`[bitStatusChange]`

- @Input() bitStatusChange: Observable<any> 状态切换请求
- @Input() bitControl = false 是否手动处理返回提示
- @Output() response: EventEmitter< any > 获取请求的响应值

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

## 上传

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
