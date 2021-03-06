# 指令

## 开始搜索

`[bit-search-start]`

- searchSelector 搜索命名
- searchover 开始搜索之后

注册搜索字段

```typescript
this.bit
  .registerSearch("test-index", {
    field: "name",
    value: "",
  })
  .subscribe(() => {
    // after
  });
```

开始搜索绑定在输入框 `enter` 事件

```html
<input
  *ngIf="bit.search[0]!==undefined"
  type="text"
  [(ngModel)]="bit.search[0].value"
  bit-search-start
  searchSelector="test-index"
  (searchover)="getLists(true)"
  nz-input
/>
```

开始搜索绑定在按钮 `click` 事件

```html
<button
  nz-button
  nzType="primary"
  nzSearch
  bit-search-start
  searchSelector="test-index"
  (searchover)="getLists(true)"
>
  <i nz-icon type="search"></i>
</button>
```

## 监听搜索

`[bit-search-change]`

- searchSelector 搜索命名
- searchover 搜索变动之后

监听搜索绑定在 `ngModelChange` 事件

```html
<nz-select
  [(ngModel)]="bit.search[0].value"
  bit-search-change
  searchSelector="test-index"
  (searchover)="getLists(true)"
>
  <nz-option [nzValue]="x.id" [nzLabel]="x.name"></nz-option>
</nz-select>
```

## 清空搜索

`[bit-search-clear]`

- searchSelector 搜索命名
- searchclear 清空搜索之后

清空搜索绑定在按钮 `click` 事件

```html
<button
  nz-button
  bit-search-clear
  searchSelector="test-index"
  (searchclear)="getLists(true)"
>
  清除搜索
</button>
```
