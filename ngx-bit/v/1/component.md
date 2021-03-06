# 组件

## 表单实例

定义组件：需要注入 `BitService`、`FormBuilder`, 需要在 `ngOnInit` 周期下定义好 `FormGroup` 对象的 `this.bit.form`

```typescript
export class AdminAddComponent implements OnInit {
  // 注入相关依赖
  constructor(
    private swal: SwalService,
    private fb: FormBuilder,
    public bit: BitService,
    private notification: NzNotificationService,
    private adminService: AdminService
  ) {}

  ngOnInit() {
    // 这里定义好 this.bit.form
    this.bit.form = this.fb.group({
      username: [
        null,
        [
          Validators.required,
          Validators.minLength(4),
          Validators.maxLength(20),
        ],
        [this.validedUsername],
      ],
    });
  }

  validedUsername = (control: AbstractControl) =>
    asyncValidator(this.adminService.validedUsername(control.value));

  submit = (data) => {
    // 这里定义提交，data是表单提交的结果
  };
}
```

在模版中使用通用处理属性, `bit.submit(event, callback)` 中 `$event` 一定要赋值，这样是为了防止默认表单提交与前端验证处理, `callback` 则是在组件中的 `submit` 函数

```html
<form nz-form [formGroup]="bit.form" (submit)="bit.submit($event,submit)">
  <!-- 这里是写表单内组件 -->
  <nz-form-item>
    <nz-form-label [nzSpan]="7" nzRequired>
      {{bit.l['username']}}
    </nz-form-label>
    <nz-form-control [nzSpan]="12" nzHasFeedback>
      <input
        nz-input
        formControlName="username"
        [placeholder]="bit.l['username_placeholder']"
      />
      <nz-form-explain *ngIf="bit.formExplain('username',true)">
        <ng-container *ngIf="bit.explain('username','required')">
          {{bit.l['username_require']}}
        </ng-container>
        <ng-container
          *ngIf="bit.explain('username','minlength')||bit.explain('username','maxlength')"
        >
          {{bit.l['username_correctly']}}
        </ng-container>
        <ng-container *ngIf="bit.explain('username','duplicated')">
          {{bit.l['username_duplicated']}}
        </ng-container>
        <ng-container *ngIf="bit.explain('username','pending')">
          {{bit.l['validating']}}
        </ng-container>
      </nz-form-explain>
    </nz-form-control>
  </nz-form-item>
</form>
```

### formControlName="username"

对应组件中 `this.form.get('username')` 对象，如获取这个 `FormContol` 的数值，需要 `this.form.get('username').value`

### bit.formExplain('username',true)

其中 `true` 代表对这个 `FormContol` 是包含异步验证的，如果仅有同步验证可以写成 `bit.formExplain('username')`。如果包含异步验证条件设为同步，这里将会不能正常工作

### bit.explain('username','required')

代表这个 `FormContol` 有 `required` 错误

## 表格实例

```html
<nz-table
  #table
  [nzData]="lists"
  [nzLoading]="bit.lists_loading"
  [nzNoResult]="bit.l['no_result']"
  [nzTotal]="bit.lists_totals"
  [nzPageSize]="bit.page_limit"
  [nzFrontPagination]="false"
  [(nzPageIndex)]="bit.lists_page_index"
  (nzPageIndexChange)="getLists()"
  nzSize="middle"
>
  <thead>
    <tr>
      <th
        nzShowCheckbox
        [(nzChecked)]="bit.lists_all_checked"
        [nzIndeterminate]="bit.lists_indeterminate"
        (nzCheckedChange)="bit.listsCheckAll($event,lists)"
      ></th>
      <th>{{bit.l['username']}}</th>
      <th>{{bit.l['role']}}</th>
      <th>{{bit.l['email']}}</th>
      <th>{{bit.l['phone']}}</th>
      <th>{{bit.l['call']}}</th>
      <th>{{bit.l['status']}}</th>
      <th>{{bit.l['action']}}</th>
    </tr>
  </thead>
  <tbody>
    <tr *ngFor="let data of table.data">
      <td
        nzShowCheckbox
        [(nzChecked)]="data.checked"
        (nzCheckedChange)="bit.listsRefreshStatus(lists)"
      ></td>
      <td>{{data.username}}</td>
      <td>{{role_lists[data.role]|JSONParse:bit.locale}}</td>
      <td>{{data.email}}</td>
      <td>{{data.phone}}</td>
      <td>{{data.call}}</td>
      <td>
        <nz-switch
          [nzCheckedChildren]="bit.l['on']"
          [nzUnCheckedChildren]="bit.l['off']"
          [(ngModel)]="data.status"
          [nzControl]="true"
          (click)="bit.statusChange(adminService.status(data),statusCallback)"
        >
        </nz-switch>
      </td>
      <td>
        <a routerLink="/{admin-edit}/{{data.id}}">
          <i nz-icon type="edit"></i> {{bit.l['edit']}}
        </a>
        <nz-divider nzType="vertical"></nz-divider>
        <a (click)="deleteData(data.id)">
          <i nz-icon type="delete"></i> {{bit.l['delete']}}
        </a>
      </td>
    </tr>
  </tbody>
</nz-table>
```

## 上传方式

在组件中定义上传相关函数与属性，因为使用 FormGroup 会缺乏一些灵活性，所以可以直接将上传属性定义在组件中，最后提交前再去整合提交数据

```typescript
export class AdminAddComponent implements OnInit {
  avatar = "";

  constructor(
    private swal: SwalService,
    private fb: FormBuilder,
    public bit: BitService,
    private notification: NzNotificationService,
    private adminService: AdminService,
    private roleService: RoleService
  ) {}

  upload(info) {
    if (info.type === "success") {
      this.avatar = info.file.response.data.save_name;
      this.notification.success(
        this.bit.l["success"],
        this.bit.l["upload_success"]
      );
    }
    if (info.type === "error") {
      this.notification.error(this.bit.l["notice"], this.bit.l["upload_error"]);
    }
  }

  submit = (data) => {
    if (this.avatar) {
      data.avatar = this.avatar;
    }
  };
}
```

在模版中使用

```html
<nz-form-item>
  <nz-form-label [nzSpan]="7" nzRequired> {{bit.l['avatar']}} </nz-form-label>
  <nz-form-control [nzSpan]="12">
    <p>{{bit.l['avatar_tips']}}</p>
    <nz-upload
      nzName="image"
      [nzWithCredentials]="true"
      [nzAction]="bit.uploads"
      [nzSize]="5120"
      nzListType="picture-card"
      [nzShowUploadList]="false"
      (nzChange)="upload($event)"
    >
      <ng-container *ngIf="!avatar">
        <i nz-icon type="plus"></i>
        <div class="ant-upload-text">{{bit.l['upload']}}</div>
      </ng-container>
      <img
        width="120"
        *ngIf="avatar"
        [src]="bit.static+avatar"
        class="avatar"
      />
    </nz-upload>
  </nz-form-control>
</nz-form-item>
```

### nzWithCredentials

如果存在携带 `Cookie` 必须将其设为 `true`，通常前后端分离会将 `Token` 存在服务端的 `Cookie` 中

### nzAction

则是上传请求地址，可以直接调用 `bit.uploads`

### nzChange

中函数就是组件中的上传

## 语言包定义

每个模块下语言包会以公用 + 独自的形式存在，定义语言包 `language.ts`

```typescript
export default {
  username: ["用户名称", "Username"],
  username_placeholder: ["请输入用户名称", "Please Enter Username"],
  username_require: [
    "当前用户名称不能为空",
    "Current user name cannot be empty",
  ],
  username_correctly: [
    "请输入正确的用户名称",
    "Please enter the correct username",
  ],
  username_duplicated: [
    "当前用户名称已存在",
    "Current user name already exists",
  ],
  role: ["权限组", "Role Group"],
  role_placeholder: ["请选择权限组", "Please Enter Role Group"],
  password: ["用户密码", "Password"],
  password_placeholder: ["请输入用户密码", "Please Enter Password"],
  password_require: ["请输入用户密码", "Please Enter Password!"],
  password_correctly: [
    "请输入正确的用户密码",
    "Please enter the correct user password",
  ],
  call: ["称呼", "Call Name"],
  call_placeholder: ["请输入称呼", "Please Enter Call Name"],
  email: ["电子邮件", "Email"],
  email_placeholder: ["请输入电子邮件", "Please Enter Email"],
  email_correctly: ["请输入正确的电子邮件", "Please enter the correct email"],
  phone: ["联系电话", "Phone"],
  phone_placeholder: ["请输入手机号码", "Please Enter Phone"],
  avatar: ["头像", "Avatar"],
};
```

在组件内引入语言包，并注册到 `bit.l` 属性中

```typescript
// 移入语言包
import packer from "./language";

export class AdminAddComponent implements OnInit {
  constructor(public bit: BitService, private adminService: AdminService) {}

  ngOnInit() {
    // 注册语言包
    this.bit.registerLocales(packer);
  }
}
```

### this.bit.registerLocales(packer)

将语言包注册至 `bit.l`，这样模版下就能直接使用 `bit.l['any']`

## 多语言设置

组件均可以使用 Ng-Zorro，设定多语言输入模式需要设定一个统一切换按钮, 在 `card` 的头部加入 `<i18n-switch></i18n-switch>`

```html
<ng-template #tools>
  <i18n-switch [(i18n)]="i18n"></i18n-switch>
</ng-template>
<nz-card [nzExtra]="tools"> </nz-card>
```

- `i18n` 是双向绑定属性，在对应的组件内设定

在表单中通过判断去切换多语言输入

```html
<nz-form-item formGroupName="name">
  <nz-form-label [nzSpan]="7" nzRequired> {{bit.l['name']}} </nz-form-label>
  <ng-template ngFor let-x [ngForOf]="bit.i18ns">
    <nz-form-control [nzSpan]="12" nzHasFeedback *ngIf="i18n===x">
      <input
        nz-input
        [formControlName]="x"
        [placeholder]="bit.l['name_placeholder']"
      />
      <nz-form-explain *ngIf="bit.formExplain('name.'+x)">
        {{bit.l['name_require']}}
      </nz-form-explain>
    </nz-form-control>
  </ng-template>
</nz-form-item>
```
