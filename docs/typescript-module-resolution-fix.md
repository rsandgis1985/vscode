# TypeScript 模块解析错误修复记录

## 问题描述

**错误信息：**
```
TSError: ⨯ Unable to compile TypeScript:
error TS5109: Option 'moduleResolution' must be set to 'NodeNext' (or left unspecified) when option 'module' is set to 'NodeNext'.
```

**触发场景：**
- 执行 `npm run precommit` 时出现编译错误
- husky pre-commit 钩子无法正常运行
- 影响代码提交流程

## 根本原因

TypeScript 配置文件中模块解析选项使用了错误的大小写：
- 使用了小写的 `"nodenext"` 和 `"node16"`
- TypeScript 要求使用正确大小写 `"NodeNext"` 和 `"Node16"`

## 解决方案

### 修改的文件

1. **src/tsconfig.base.json**
   ```json
   // 修改前
   "module": "nodenext",
   "moduleResolution": "nodenext"

   // 修改后
   "module": "NodeNext",
   "moduleResolution": "NodeNext"
   ```

2. **build/tsconfig.json**
   - 同样的大小写修复

3. **extensions/github/tsconfig.json**
   - 同样的大小写修复

4. **build/npm/jsconfig.json**
   ```json
   // 修改前
   "module": "node16"

   // 修改后
   "module": "Node16"
   ```

5. **build/lib/formatter.ts**
   - 修改了 TypeScript 语言服务的编译器选项获取方法
   - 确保使用正确的模块解析设置

6. **tsconfig.json（新建）**
   - 在项目根目录创建配置文件
   - 为 ts-node 提供正确的编译器选项

### 核心修复逻辑

```typescript
// 在 formatter.ts 中添加的修复代码
getCompilationSettings = () => {
    const options = ts.getDefaultCompilerOptions();
    // Fix case sensitivity for module options to prevent TS5109 error
    if (options.module === ts.ModuleKind.NodeNext || (options.module as any) === 'nodenext') {
        options.module = ts.ModuleKind.NodeNext;
        options.moduleResolution = ts.ModuleResolutionKind.NodeNext;
    }
    return options;
};
```

## 验证结果

- ✅ `npm run precommit` 正常执行
- ✅ TypeScript 编译错误消除
- ✅ husky pre-commit 钩子正常工作

## 经验总结

1. **大小写敏感性**：TypeScript 配置选项对大小写敏感，必须使用正确的枚举值
2. **配置文件继承**：修改基础配置文件会影响所有继承的配置
3. **工具链一致性**：确保所有工具（tsc、ts-node、语言服务）使用一致的配置

## 修复日期

2025年6月22日

## 相关分支

- 修复分支：`fix/build-error`
- 涉及提交：TypeScript 模块解析配置大小写修复
