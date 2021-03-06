## bitBack 导航返回

可以在应用中返回链接

`[bitBack]`

将 `[bitBack]` 设置给 button 按钮，当点击 button 时返回上一级链接

```html
<button nz-button type="button" bitBack>返回上一级</button>
```

## bitHistory 导航历史

加载被历史缓存的参数导航

`[bitHistory]`

| 属性           | 说明         | 类型     | 默认值 |
| -------------- | ------------ | -------- | ------ |
| `[bitHistory]` | 跨级路由名称 | `string` | -      |

假设导航至/team-index/1，再从/team-index/1 导航至/team-index/1/services/T100

```html
<button nz-button type="button" bitHistory="team-index">历史导航</button>
```

此时执行 button 将跳转至/team-index/1

## bitCol 栅格标识

为应用定义统一栅格

`[bitCol]`

| 属性       | 说明         | 类型     | 默认值 |
| ---------- | ------------ | -------- | ------ |
| `[bitCol]` | 统一栅格索引 | `string` | -      |

首先要确保 `src\environments\environment.ts` 中配置完整并将配置加载入模块中，假设统一栅格定义为：

```typescript
const col = {
  label: {
    nzXXl: 4,
    nzXl: 5,
    nzLg: 6,
    nzMd: 7,
    nzSm: 24,
  },
  control: {
    nzXXl: 8,
    nzXl: 9,
    nzLg: 10,
    nzMd: 14,
    nzSm: 24,
  },
  submit: {
    nzXXl: { span: 8, offset: 4 },
    nzXl: { span: 9, offset: 5 },
    nzLg: { span: 10, offset: 6 },
    nzMd: { span: 14, offset: 6 },
    nzSm: { span: 24, offset: 0 },
  },
};
```

那么就可以在组件中使用 `[bitCol]` 分配统一栅格

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

<nz-form-item>
  <nz-form-control bitCol="submit">
    <nz-space>
      <nz-space-item>
        <button nz-button nzType="primary" [disabled]="!form.valid">
          <i nz-icon nzType="check"></i> {{bit.l['submit']}}
        </button>
      </nz-space-item>
      <nz-space-item>
        <button nz-button type="button" bitBack>
          <i nz-icon nzType="close"></i> {{bit.l['cancel']}}
        </button>
      </nz-space-item>
    </nz-space>
  </nz-form-control>
</nz-form-item>
```

## bitFormSubmit 表单提交

协助应用表单提交时自动进行 controls 的 `markAsDirty` `updateValueAndValidity`

`[bitFormSubmit]`

| 属性              | 说明         | 类型                | 默认值 |
| ----------------- | ------------ | ------------------- | ------ |
| `(bitFormSubmit)` | 监听表单提交 | `EventEmitter<any>` | -      |

首先定义组件表单提交

```typescript
import { Component, OnInit } from "@angular/core";
import { FormBuilder, FormGroup } from "@angular/forms";

@Component({
  selector: "app-welcome",
  templateUrl: "./welcome.component.html",
  styleUrls: ["./welcome.component.scss"],
})
export class WelcomeComponent implements OnInit {
  form: FormGroup;

  constructor(private fb: FormBuilder) {}

  ngOnInit(): void {
    this.form = this.fb.group({
      name: [],
    });
  }

  submit(data) {
    // output data
  }
}
```

再为组件定义模板

```html
<form nz-form [formGroup]="form" (bitFormSubmit)="submit($event)">
  <nz-form-item>
    <nz-form-label bitCol="label"> Name </nz-form-label>
    <nz-form-control bitCol="control" nzHasFeedback>
      <input nz-input formControlName="name" />
    </nz-form-control>
  </nz-form-item>
  <nz-form-item>
    <nz-form-control bitCol="submit">
      <nz-space>
        <nz-space-item>
          <button nz-button nzType="primary" [disabled]="!form.valid">
            <i nz-icon nzType="check"></i> Submit
          </button>
        </nz-space-item>
      </nz-space>
    </nz-form-control>
  </nz-form-item>
</form>
```

当表单提交时将触发 `submit($event)`

## bitI18nUpdate 多语言输入更新

协助应用表单的多语言输入组件更新状态，为 `bit-i18n-tooltip` 多语言提示提供支持

`[bitI18nUpdate]`

为 FormGroup 为 name 的多语言输入组件进行状态更新

```html
<nz-card>
  <form nz-form [formGroup]="form" (bitFormSubmit)="submit($event)">
    <!-- 这里使用 -->
    <nz-form-item formGroupName="name" bitI18nUpdate>
      <nz-form-label bitCol="label" nzRequired>
        <span>
          {{bit.l['name']}}
          <i
            nz-icon
            nz-tooltip
            [nzTooltipTitle]="bit.l['i18n']"
            nzType="translation"
            nzTheme="outline"
          >
          </i>
        </span>
      </nz-form-label>
      <ng-container *ngFor="let x of bit.i18nContain">
        <nz-form-control
          *ngIf="bit.equalI18n(x)"
          nzHasFeedback
          bitCol="control"
          [nzValidatingTip]="bit.l['validating']"
          [nzErrorTip]="name.ref"
        >
          <input
            nz-input
            [nz-tooltip]="tooltip.ref"
            [formControlName]="x"
            [placeholder]="bit.l['namePlaceholder']"
          />
          <bit-i18n-tooltip #tooltip groupName="name"></bit-i18n-tooltip>
          <bit-error-tip
            #name
            [hasError]="{
                    required:bit.l['nameRequire'],
                    duplicated:bit.l['nameDuplicated']
                }"
          ></bit-error-tip>
        </nz-form-control>
      </ng-container>
    </nz-form-item>
  </form>
</nz-card>
```

## bitOpen 路由导航

`navigate` 路由导航的扩展，为跨级导航作为基础持久记录其路由参数

`[bitOpen]`

| 属性        | 说明               | 类型             | 默认值 |
| ----------- | ------------------ | ---------------- | ------ |
| `[bitOpen]` | urlTree            | `any[]`          | -      |
| `[extras]`  | 修改导航策略的选项 | NavigationExtras | -      |

`bitOpen` 在使用中与 `routerlink` 基本相同

```html
<button nz-button nzType="primary" nzSize="small" [bitOpen]="['admin-index']">
  导航至管理员列表
</button>

<a [bitOpen]="['admin-edit', 1]">
  <i nz-icon type="edit"></i> 导航至管理员修改
</a>

<button
  nz-button
  nzType="primary"
  nzSize="small"
  [bitOpen]="['team-index']"
  [extras]="{queryParams:{key:'abc'}}"
>
  导航附加 extras
</button>
```

## bitSearchChange 搜索监听

为分页列表提供搜索监听功能

`[bitSearchChange]`

| 属性                | 说明         | 类型                | 默认值 |
| ------------------- | ------------ | ------------------- | ------ |
| `[bitSearchChange]` | 分页列表对象 | `ListByPage`        | -      |
| `(after)`           | 搜索变动之后 | `EventEmitter<any>` | -      |

监听包含 `NgModelChange` 的组件中

```typescript
@Component({
  template: `
    <ng-container *ngIf="lists.hasSearch('type')">
      <select
        [bitSearchChange]="lists"
        [(ngModel)]="lists.search['type'].value"
        (after)="after()"
      >
        <ng-container *ngFor="let option of options">
          <option [label]="option.label" [value]="option.value"></option>
        </ng-container>
      </select>
    </ng-container>
  `,
})
class TestComponent implements OnInit {
  lists: ListByPage;
  options: any[] = [
    { label: "type1", value: 0 },
    { label: "type1", value: 1 },
    { label: "type2", value: 2 },
  ];

  constructor(private bit: BitService) {}

  ngOnInit() {
    this.lists = this.bit.listByPage({
      id: "test",
      query: [{ field: "type", op: "=", value: 0 }],
    });
  }

  after() {}
}
```

## bitSearchStart 搜索触发

为分页列表提供搜索触发功能，触发事件为 `click` 与 `keydown.enter`

`[bitSearchStart]`

| 属性               | 说明         | 类型                | 默认值 |
| ------------------ | ------------ | ------------------- | ------ |
| `[bitSearchStart]` | 分页列表对象 | `ListByPage`        | -      |
| `(after)`          | 搜索变动之后 | `EventEmitter<any>` | -      |

为 `input` 与 `button` 创建分页列表搜索触发功能

```typescript
@Component({
  template: `
    <ng-container *ngIf="lists.hasSearch('username')">
      <input
        [bitSearchStart]="lists"
        [(ngModel)]="lists.search['username'].value"
        (after)="after()"
      />
      <button [bitSearchStart]="lists" (after)="after()">搜索</button>
    </ng-container>
  `,
})
class TestComponent implements OnInit {
  lists: ListByPage;

  constructor(private bit: BitService) {}

  ngOnInit() {
    this.lists = this.bit.listByPage({
      id: "test",
      query: [{ field: "username", op: "=", value: "" }],
    });
  }

  after() {}
}
```

## bitSearchClear 搜索清除

为分页列表提供搜索清除功能

`[bitSearchClear]`

| 属性               | 说明         | 类型                | 默认值 |
| ------------------ | ------------ | ------------------- | ------ |
| `[bitSearchClear]` | 分页列表对象 | `ListByPage`        | -      |
| `[reset]`          | 重置的数值   | `any`               | -      |
| `(after)`          | 搜索变动之后 | `EventEmitter<any>` | -      |

为 `button` 创建分页列表搜索清除功能

```typescript
@Component({
  template: `
    <ng-container *ngIf="lists.hasSearch('type')">
      <select
        [bitSearchChange]="lists"
        [(ngModel)]="lists.search['type'].value"
      >
        <ng-container *ngFor="let option of options">
          <option [label]="option.label" [value]="option.value"></option>
        </ng-container>
      </select>
      <button [bitSearchClear]="lists" (after)="after()">测试清除</button>
    </ng-container>
  `,
})
class TestComponent implements OnInit {
  lists: ListByPage;
  options: any[] = [
    { label: "type1", value: 0 },
    { label: "type1", value: 1 },
    { label: "type2", value: 2 },
  ];

  constructor(private bit: BitService) {}

  ngOnInit() {
    this.lists = this.bit.listByPage({
      id: "test",
      query: [{ field: "type", op: "=", value: 0 }],
    });
  }

  after() {}
}
```

## bitStatusChange 状态切换

为开关选择器提供状态请求切换功能

`[bitStatusChange]`

| 属性                | 说明                 | 类型                | 默认值  |
| ------------------- | -------------------- | ------------------- | ------- |
| `[bitStatusChange]` | 状态切换请求         | `Observable<any>`   | -       |
| `[bitControl]`      | 是否手动处理返回提示 | `boolean`           | `false` |
| `(response)`        | 获取请求的响应值     | `EventEmitter<any>` | -       |

为 `nz-switch` 创建状态请求切换功能

```typescript
@Component({
  template: `
    <nz-switch
      [(ngModel)]="data.status"
      [bitStatusChange]="testService.status(data)"
      [bitControl]="true"
      (response)="statusFeedback($event)"
    >
    </nz-switch>
  `,
})
class TestComponent {
  data: any = {
    id: 1,
    status: true,
  };

  constructor(public testService: TestService) {}

  statusFeedback(event) {}
}
```

## bitUpload 上传

为文件上传组件提供配置绑定

`[bitUpload]`

通常情况 `nz-upload` 需要这样定义

```html
<nz-upload
  nzName="image"
  [nzAction]="bit.uploads"
  [nzWithCredentials]="config.api.withCredentials"
  [nzSize]="5120"
  [nzShowUploadList]="false"
>
</nz-upload>
```

为 `nz-upload` 创建配置绑定后可以更简洁的实现相同定义

```html
<nz-upload nzName="image" bitUpload [nzShowUploadList]="false"> </nz-upload>
```

如果上传为对象存储，则 `nzName` 需要为 `file`

```html
<nz-upload class="upload" bitUpload nzName="file">
  <button nz-button>
    <i nz-icon [nzType]="'upload'"></i><span>{{ bit.l["upload"] }}</span>
  </button>
</nz-upload>
```

此时统一配置需要设置 `api.upload` `api.uploadStorage` `api.uploadFetchSigned` `api.uploadFetchSignedMethod`，`api.upload` 替换为对象存储的请求地址（`nzWithCredentials` 不会继承统一配置并为 `false`），`api.uploadStorage` 是对象存储类型可以设置为 `oss` 阿里云、`cos` 腾讯云、`obs` 华为云等，`api.uploadFetchSigned` `api.uploadFetchSignedMethod` 是后端为对象存储生成的签名请求（不同的服务商返回参数不同），例如：

- 阿里云
  - filename `string` 文件名（不加扩展名）
  - option 配置项
    - policy `string` 对象存储策略
    - access_key_id `string` AccessKeyId
    - signature `string` 签名

```http
GET /system/main/presigned HTTP/1.1
Content-Type: application/json
{
    filename: '...',
    option: {
        policy: '...',
        access_key_id: '...',
        signature: '...'
    }
}
```

- 腾讯云
  - filename `string` 文件名（不加扩展名）
  - option 配置项
    - policy `string` 对象存储策略
    - sign_algorithm `string` 签名算法
    - ak `string` AccessKey
    - key_time `string` 时间范围
    - signature `string` 签名

```http
GET /system/main/presigned HTTP/1.1
Content-Type: application/json
{
    policy: '...',
    sign_algorithm: 'sha1',
    ak: '...',
    key_time: '1610462610;1610463210',
    signature: '...'
}
```

- 华为云
  - filename `string` 文件名（不加扩展名）
  - option 配置项
    - policy `string` 对象存储策略
    - access_key_id `string` AccessKeyId
    - signature `string` 签名

```
GET /system/main/presigned HTTP/1.1
Content-Type: application/json
{
    key: '...',
    policy: '...',
    AccessKeyId: '...',
    signature: '...'
}
```