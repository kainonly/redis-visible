# 操作

## 组件异步验证器

- asyncValidator(req: Observable<any>, field = 'duplicated')
- req 请求对象
- field 自定义返回

在组件内定义异步验证器

``` typescript
import {operates} from 'ngx-bit';

export class AdminAddComponent implements OnInit {

  constructor(private swal: SwalService,
              private fb: FormBuilder,
              public bit: BitService,
              private notification: NzNotificationService,
              private adminService: AdminService) {
  }

  ngOnInit() {
    this.bit.form = this.fb.group({
      username: [null, [Validators.required,Validators.minLength(4),Validators.maxLength(20)],[this.validedUsername]],
    });
  }

  validedUsername = (control: AbstractControl) =>
    operates.asyncValidator(this.adminService.validedUsername(control.value));
}
```

## 判断空数组

- emptyArray(array: any[])
- array 数组
- Return `boolean`

判断一个数组是否为空

``` typescript
const test = [];

emptyArray(test); 
// true
```

## 判断空对象

- emptyObject(object: any)
- object 对象
- Return `boolean`

判断一个对象是否为空

``` typescript
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
  car1: 'blue',
  car2: 'red'
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

将一个对象转为Map对象

```typescript
const some = {
  car1: 'blue',
  car2: 'red'
};
const t = objectToMap(some);
if(t) {
  console.log(t.get('car1'));
  // blue
}
```
