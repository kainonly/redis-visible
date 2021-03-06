# 更新日志

## 11.4.0

### New Features

- 重新启用 `bit.module.ts` 精简配置初始化

例如：原方式需要引入所有服务的依赖

```typescript
const bitConfig = () => {
  const env = environment.bit;
  const service = new BitConfigService();
  return merge.recursive(service, env);
};

@NgModule({
    ...
  providers: [
    NzIconService,
    BitService,
    BitHttpService,
    BitEventsService,
    BitSupportService,
    BitSwalService,
    { provide: BitConfigService, useFactory: bitConfig },
  ],
    ...
})
export class AppModule {
  constructor(config: BitConfigService, nzIconService: NzIconService) {
    if (config.url.icon) {
      nzIconService.changeAssetsSource(config.url.icon);
    }
  }
}
```

现在可以重新引入 `BitModule.forRoot(config: BitConfig)` 做初始化达到相同效果

```typescript
const bitConfig = () => {
  const env = environment.bit;
  const service = new BitConfigService();
  return merge.recursive(service, env);
};

@NgModule({
    ...
  imports: [
    BitModule.forRoot(environment.bit),
  ],
    ...
})
export class AppModule {
}
```

- 新增组件 `bit-transport` 上传状态提示，详细使用可查看 van-skeleton/cms [Media 组件](https://github.com/van-skeleton/cms/tree/main/media)

### Breaking Changes

- 废弃 `ngx-bit/types` 并迁移至 `ngx-bit` 下导出
- 废弃 `ngx-bit/factory` 并迁移至 `ngx-bit` 下导出
- 废弃 `ngx-bit/operates` 中的 `getQuerySchema(options: SearchOption[]): any[]` ，将其并入 `ListByPage` 类的静态函数

## 11.3.0

新增 `bit-transport` 批量上传按钮

## 11.2.0

优化路由定位

### Breaking Changes

- 操作库弃用 `getSelectorFormUrl(url: string, match: any[])`
- `bit-cross-level` 与 `crossLevel(selector: string): void` 分别变更为 `bit-history` 与 `history(key: string): void`
- Routes `path` 标准化，由 `/{any}` 变更为 `/any`

## 11.1.3

SearchOption 增加 Format，例如：

```typescript
{ field: 'time', op: 'between', value: [], format: 'unixtime' }
```

## 11.1.2

解除 `<bit-search-start>` 限制

## 11.1.1

完善 `BitSupportService` 下 `unsubscribe`，修复退出登录后面包屑未清零的问题

## 11.1.0

### Breaking Changes

- 废弃模块 `NgxBitModule` ，按功能需要导入服务提供商
- 废弃管道 `UndefinedPipe` `EmptyArrayPipe` `EmptyObjectPipe`，替代为符合业务场景的 `EmptyPipe` 是否为空
- 废弃管道 `LocalePipe` `JSONParsePipe`，合并为 `ObjectPipe` 转化为对象（或指定语言显示）
- 废弃操作库函数 `emptyArray` `emptyObject`，合并为 `empty` 是否为空
- 废弃操作库函数 `factoryBitConfig`，配置生产替换为服务提供商工厂模式
- 标准化公共语言包，部分组件语言包索引变更（库组件语言包索引统一为 `PascalCase` ，项目组件建议使用 `camelCase`）
- 移除 `ajv` `sweetalert2` `@ngx-pwa/local-storage` 等第三方库，同时废弃操作库 `validate`
- 提示确认 `BitSwalService` 作为可选项，使用需安装 `sweetalert2`
- 功能支持 `BitSupportService` 作为可选项，使用需要安装 `@ngx-pwa/local-storage`
- `nzIconService.changeAssetsSource` 需要自行调用
- `SearchOption` 废弃 `must`，替换为 `exclude` 作为排除值，默认为 `['', 0, null]`

### New Features

- 丰富统一配置，增加更多配置项定义
- 丰富上传指令 `nz-upload[bitUpload]` 业务功能，支持：阿里云 OSS、腾讯云 COS、华为云 OBS 直传对象存储
- 增加语言包模板函数 `print` 、管道 `PrintPipe` 与组件 `<bit-print></bit-print>`
- 增加页头填充 `<bit-header></bit-header>` 顶部公告 `nz-alert[bitBanner]`

## 11.0.1

增加指令选择器，精确监听事件，例如：`input[bitSearchStart]` 下 `click` 事件不在触发

- `selector: '[bitSearchClear]'` 变更为 `selector: 'button[bitSearchClear]'`
- `selector: '[bitSearchStart]'` 变更为 `selector: 'input[bitSearchStart],button[bitSearchStart]'`
- `selector: '[bitStatusChange]'` 变更为 `selector: 'nz-switch[bitStatusChange]'`
- `selector: '[bitUpload]'` 变更为 `selector: 'nzUpload[bitUpload]'`

## 11.0.0

升级所有组件至 Angular 11
