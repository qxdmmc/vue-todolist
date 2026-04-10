# 马卡龙 TodoList

这是一个使用 Vue 3 + Vite + TypeScript 构建的响应式待办事项管理页面，采用马卡龙色系设计。

## 功能

- 待办列表展示
- 新增待办
- 编辑单条待办内容
- 删除待办
- 置顶待办
- 切换完成状态
- 显示最后修改时间
- 空列表提示
- 筛选：全部 / 未完成 / 已完成
- 搜索待办内容
- 排序：置顶优先 / 最新修改 / 最早修改
- `localStorage` 本地持久化

## 安装与运行

```bash
npm install
npm run dev
```

## 构建

```bash
npm run build
```

## 项目结构

- `index.html`：应用入口
- `src/main.ts`：Vue 应用挂载
- `src/App.vue`：核心页面与逻辑
- `vite.config.ts`：Vite 配置
- `tsconfig.json`：TypeScript 配置