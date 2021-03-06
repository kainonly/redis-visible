# 操作

## 判断空数组

- emptyArray(array: any[])
- array 数组
- Return `boolean`

判断一个数组是否为空

```typescript
const test = [];

emptyArray(test);
// true
```

## 判断空对象

- emptyObject(object: any)
- object 对象
- Return `boolean`

判断一个对象是否为空

```typescript
const test = {};

emptyObject(test);
// true
```

## 对象转数组

- objectToArray(object: any)
- object 对象
- Return `any[]`
  - key 原主键
  - rows 原键值

将一个对象转为数组

```typescript
const some = {
  car1: "blue",
  car2: "red",
};
objectToArray(some);
// [
//    {key: 'car1', rows: 'blue'},
//    {key: 'car2', rows: 'red'}
// ]
```

## 对象转 Map 对象

- objectToMap(object: any)
- object 对象
- Return `Map<any,any>|false`

将一个对象转为 Map 对象

```typescript
const some = {
  car1: "blue",
  car2: "red",
};
const t = objectToMap(some);
if (t) {
  console.log(t.get("car1"));
  // blue
}
```

## 搜索条件筛选

- whereAndLike(search: any)

注册的搜索条件中存在多种方式，则使用该函数在请求服务内进行筛选

```typescript
// search = this.bit.search
lists(search: any, refresh: boolean): Observable<any> {
    const {where, like} = whereAndLike(search);
    return this.listsService.factory(this.model, where, like, refresh).pipe(map(res => {
        if (!res.error) {
            return res.data.lists;
        } else {
            return [];
        }
    }));
}
```

## 语言包生产

- factoryLocales(packer: any)
- packer 语言包文件

定义一个语言包数据，索引顺序按照 `['zh_cn','en_us']`

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

语言包生产

```typescript
import packer from "./language";

const language = factoryLocales(packer);

console.log(language.zh_cn.username); // 用户名称
console.log(language.en_us.username); // Username
```

## 获取路由名称

- getRouteName(url: string, start = '%7B', end = '%7D'): string
- url 完整路由地址
- start 起始符
- end 结束符
- Return 路由名称

可以在路由事件监听中使用

```typescript
export class DashboardsComponent implements OnInit {
  constructor(private router: Router) {}

  ngOnInit() {
    this.router.events.subscribe((event) => {
      // current url /{admin-edit}/1
      if (event instanceof NavigationEnd) {
        getRouteName(event.url);
        // admin-edit
      }
    });
  }
}
```

## 组件异步验证器

- asyncValidator(req: Observable<any>, field = 'duplicated')
- req 请求对象
- field 自定义返回

在组件内定义异步验证器

```typescript
export class AdminAddComponent implements OnInit {
  constructor(
    private swal: SwalService,
    private fb: FormBuilder,
    public bit: BitService,
    private notification: NzNotificationService,
    private adminService: AdminService
  ) {}

  ngOnInit() {
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
}
```

## 多语言组件初始值

- i18nControlsValue(i18n: string, value?: any)
- i18n 多语言组件标识
- value 多语言组件数值

## 多语言组件同步验证

- i18nControlsValidate(i18n: string, validate?: any)
- i18n 多语言标识
- validate 响应式同步验证数组

## 多语言组件异步验证

- i18nControlsAsyncValidate(i18n: string, asyncValidate?: any)
- i18n 多语言标识
- asyncValidate 响应式异步验证数组
