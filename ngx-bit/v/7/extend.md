# 扩展

## 本地存储

### clear()

清除所有本地存储

```typescript
this.storage.clear();
```

### setMenu(menu: any, router: any)

设置本地存储菜单数据

- menu 菜单数据
- router 路由数据

```typescript
this.storage.setMenu(data.menu, data.router);
```

### autoBreadcrumb(router: Router, match = ['%7B', '%7D'])

自动计算面包屑

- router 应用 `Router` 对象
- match 路由标签获取符，默认`{}`

```typescript
this.storage.autoBreadcrumb(this.router);
```

### destoryBreadcrumb()

销毁面包屑自动计算

```typescript
this.storage.destoryBreadcrumb();
```

## 提示框

### static native

sweetalert2 对象

### addAlert(res: any, form: FormGroup, reset?: any, customize?: AlertCustomize): Observable<any>

新增返回反馈栏

- res 请求响应结果
- form 表单对象
- reset FormGroup 重置值
- customize 自定义文本
  - text 提示文本
  - error_text 返回错误提示文本
  - confirmButtonText 确认按钮文本
  - cancelButtonText 取消按钮文本
- Return `Observable<any>`

例如, 在新增操作下组件表单提交中使用, `status` 为 `true` 表示确认提示框

```typescript

export class AdminAddComponent implements OnInit {

    ...

    submit(data) {
        this.adminService.add(data).pipe(
            switchMap(res => this.swal.addAlert(res, this.form, {
                status: true
            }))
        ).subscribe((status) => {
            // status => true or false
        });
    }
}
```

### editAlert(res: any, customize?: AlertCustomize): Observable<any>

修改返回反馈栏

- res 请求响应结果
- customize 自定义文本
  - text 提示文本
  - error_text 返回错误提示文本
  - confirmButtonText 确认按钮文本
  - cancelButtonText 取消按钮文本
- Return `Observable<any>`

例如, 在修改操作下组件表单提交中使用, `status` 为 `true` 表示确认提示框

```typescript
export class AdminEditComponent implements OnInit {
    private id: any;

    ...

    submit(data) {
        data.id = this.id;
        this.adminService.edit(data).pipe(
            switchMap(res => this.swal.editAlert(res))
        ).subscribe((status) => {
            // status => true or false
        });
    }
}
```

### deleteAlert(service: Observable<any>, customize?: AlertCustomize)

删除返回反馈栏

- service 删除请求对象
- customize 自定义文本
  - text 提示文本
  - confirmButtonText 确认按钮文本
  - cancelButtonText 取消按钮文本
- Return `Observable<any>`

例如, 在删除操作下使用, 订阅返回删除请求对象的响应值

```typescript
export class AdminIndexComponent implements OnInit {
    ...

  deleteData(id: any) {
    this.swal.deleteAlert(this.adminService.delete(id)).subscribe(res => {
      if (!res.error) {
        this.notification.success(this.bit.l['operate_success'], this.bit.l['delete_success']);
        this.getLists(true);
      } else {
        switch (res.msg) {
          case 'error:self':
            this.notification.error(this.bit.l['operate_error'], this.bit.l['error_delete_self']);
            break;
          default:
            this.notification.error(this.bit.l['operate_error'], this.bit.l['delete_error']);
        }
      }
    });
  }
}
```
