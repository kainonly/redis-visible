# 组件

## bit-i18n-switch 多语言切换器

`<bit-i18n-switch (i18nChange)="change($event)"></bit-i18n-switch>`

- @Output() i18nChange `EventEmitter<string>` 监听 bit.i18n 值的变化

控制表单内所有多语言输入组件，例如

```html
<ng-template #nzExtra>
  <bit-i18n-switch></bit-i18n-switch>
</ng-template>

<form nz-form [formGroup]="form" (bitFormSubmit)="submit($event)">
  <nz-form-item formGroupName="name">
    <nz-form-label bitFormLabelCol nzRequired>
      {{bit.l['name']}}
    </nz-form-label>
    <ng-container *ngFor="let x of bit.i18nContain">
      <nz-form-control *ngIf="bit.equalI18n(x)" bitFormControlCol nzHasFeedback>
        <input
          nz-input
          [placeholder]="bit.l['namePlaceholder']"
          [formControlName]="x"
        />
      </nz-form-control>
    </ng-container>
  </nz-form-item>
</form>
```

---

## bit-i18n-tooltip 多语言提示

`<bit-i18n-tips #tips name="name"></bit-i18n-tips>`

- ref `TemplateRef< any >`
- groupName `string` FormGroupName

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

## bit-error-tip 验证错误提示

`<bit-error-tip #username [hasError]="{}"></bit-error-tip>`

- @Input() hasError `any` errors 错误信息

```html
<nz-form-control
  nzHasFeedback
  [nzValidatingTip]="'正在验证'"
  [nzErrorTip]="username.ref"
>
  <input nz-input formControlName="username" />
  <bit-error-tip
    #username
    [hasError]="{
        required:'请填写用户名称',
    }"
  ></bit-error-tip>
</nz-form-control>
```
