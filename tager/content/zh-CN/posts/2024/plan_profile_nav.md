# 个人信息与导航栏容器开发计划

## 任务概述

设计一个可展开的个人资料导航容器，包含头像、名字、联系方式和导航链接。

## 更改记录

### 2026-04-04 更新

1. **流媒体按钮简化**：去掉悬浮弹出的提示框，只保留颜色填充效果
2. **默认状态样式**：使用液态玻璃风格作为默认状态
3. **点击逻辑**：
   - 容器未完全展开前：点击识别为拖拽
   - 容器完全展开后：才能点击按钮
4. **布局要求**：按钮不要错位，背景保持一致

## 参考动画效果

- 容器展开动画参考：`x:\works\code\MyBlog\example\容器展开动画.vue`
  - 悬浮时展开动画：0.5s ease-in-out
  - 边框旋转效果：transform rotate(10deg) -> rotate(0)
  - 缩放效果：scale(1.05)
  - 边框内收：inset: 0px -> inset: 15px

## 配置扩展

### 1. author.config.ts 扩展

```typescript
export default {
  // 原有配置...
  name: 'Author Name',
  avatar: '/content/images/avatar.png',
  bio: 'A passionate developer',

  // 新增：联系方式配置
  contacts: [
    {
      id: 'github',
      name: 'GitHub',
      icon: '/icons/github.svg',
      value: 'https://github.com/username',
      action: 'open', // "open" | "copy"
      enabled: true,
    },
    {
      id: 'email',
      name: 'Email',
      icon: '/icons/email.svg',
      value: 'example@email.com',
      action: 'copy',
      enabled: true,
    },
  ],
};
```

### 2. navigation.config.ts 新建

```typescript
export default {
  // 导航类别配置
  categories: [
    {
      id: 'posts',
      title: '文章',
      component: 'PostsWindow',
      icon: '/icons/posts.svg',
      enabled: true,
    },
    {
      id: 'about',
      title: '关于',
      component: 'AboutWindow',
      icon: '/icons/about.svg',
      enabled: true,
    },
    {
      id: 'friends',
      title: '♡友链',
      component: 'FriendsWindow',
      icon: '/icons/heart.svg',
      enabled: true,
    },
  ],
};
```

## 组件结构

### ProfileNavWindow.vue

```
ProfileNavWindow/
├── ProfileNavWindow.vue    # 主组件
└── ProfileNavWindow.css    # 样式文件
```

### 组件状态

- **收起状态（默认）**：
  - 液态玻璃背景
  - 显示头像（小尺寸）
  - 分隔线
  - 导航类别（文章/关于/友链）

- **悬浮展开状态**：
  - 头像放大
  - 头像下方显示名字
  - 名字下方显示联系方式按钮（液态玻璃风格，悬浮变色）
  - 分隔线上方显示联系方式
  - 容器边框动画展开

### 点击逻辑

```
用户点击
  ↓
检查容器是否完全展开
  ↓ 未展开
    触发拖拽
  ↓ 已展开
    检查点击目标
      ↓ 联系方式按钮
        执行复制/跳转
      ↓ 导航类别
        打开对应窗口
      ↓ 其他区域
        触发拖拽
```

## 样式规范

### 液态玻璃风格（默认状态）

- 背景：`var(--window-bg)`
- 边框：1px solid rgba(255,255,255,0.3)
- 圆角：`var(--border-radius)`
- 阴影：多层阴影营造层次感
- 模糊：`backdrop-filter: blur(20px)`

### 联系方式按钮（简化版）

```css
.contact-btn {
  /* 液态玻璃基础 */
  background: var(--window-bg);
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 50%;

  /* 悬浮变色 */
  transition: all 0.3s ease;
}

.contact-btn:hover {
  background: var(--primary-color);
  border-color: var(--primary-color);
  color: white;
}
```

### 动画参数

```css
/* 容器展开动画 */
.profile-window {
  transition: all 0.5s ease-in-out;
}

.profile-window.expanded {
  transform: scale(1.05);
}

/* 边框动画 */
.border-animation {
  transition: all 0.5s ease-in-out;
  inset: 0px;
  opacity: 0;
  transform: rotate(10deg);
}

.profile-window.expanded .border-animation {
  inset: 10px;
  opacity: 1;
  transform: rotate(0);
}

/* 头像放大 */
.avatar {
  transition: all 0.5s ease-in-out;
  width: 60px;
  height: 60px;
}

.profile-window.expanded .avatar {
  width: 80px;
  height: 80px;
}

/* 内容渐显 */
.expand-content {
  opacity: 0;
  max-height: 0;
  overflow: hidden;
  transition: all 0.5s ease-in-out;
}

.profile-window.expanded .expand-content {
  opacity: 1;
  max-height: 200px;
}
```

## 交互逻辑

### 展开状态管理

```typescript
const isExpanded = ref(false); // 是否悬浮
const isFullyExpanded = ref(false); // 是否完全展开（可点击）

// 悬浮时展开
function handleMouseEnter() {
  isExpanded.value = true;
  // 延迟标记完全展开
  setTimeout(() => {
    isFullyExpanded.value = true;
  }, 500); // 与动画时间一致
}

// 离开时收起
function handleMouseLeave() {
  isExpanded.value = false;
  isFullyExpanded.value = false;
}

// 点击处理
function handleClick(e: MouseEvent) {
  if (!isFullyExpanded.value) {
    // 未完全展开，触发拖拽
    elasticDrag.startDrag(e);
    return;
  }

  // 检查点击目标
  const target = e.target as HTMLElement;
  if (target.closest('.contact-btn')) {
    // 处理联系方式点击
    return;
  }
  if (target.closest('.nav-item')) {
    // 处理导航点击
    return;
  }

  // 其他区域触发拖拽
  elasticDrag.startDrag(e);
}
```

### 联系方式按钮

```typescript
interface ContactAction {
  id: string;
  name: string;
  icon: string;
  value: string;
  action: 'open' | 'copy';
}

async function handleContactClick(contact: ContactAction) {
  if (contact.action === 'copy') {
    await navigator.clipboard.writeText(contact.value);
    // 可选：显示复制成功提示
  } else {
    window.open(contact.value, '_blank');
  }
}
```

### 导航点击

```typescript
function handleNavClick(category: NavCategory) {
  if (!isFullyExpanded.value) return;
  // 打开对应的窗口组件
  openWindow(category.component);
}
```

## 实现步骤

1. **扩展配置文件**
   - 更新 author.config.ts 添加 contacts
   - 创建 navigation.config.ts

2. **创建组件**
   - ProfileNavWindow.vue
   - ProfileNavWindow.css

3. **实现动画效果**
   - 悬浮展开动画
   - 边框旋转动画
   - 头像放大动画
   - 内容渐显动画

4. **实现交互功能**
   - 展开状态管理（悬浮 -> 完全展开）
   - 点击逻辑（未展开=拖拽，已展开=可点击）
   - 联系方式按钮（液态玻璃+悬浮变色）
   - 导航类别点击

5. **集成到主页面**
   - 在 App.vue 中添加 ProfileNavWindow
   - 设置初始位置

## 检查清单

- [ ] 配置文件扩展完成
- [ ] 组件样式分离
- [ ] CSS 使用变量而非硬编码
- [ ] 动画效果与参考一致
- [ ] 联系方式支持复制/跳转
- [ ] 导航类别可配置启用/禁用
- [ ] 未展开时点击=拖拽
- [ ] 展开后才能点击按钮
- [ ] 按钮布局不错位
- [ ] 背景使用液态玻璃
- [ ] 运行 lint 通过
- [ ] 运行 fmt:check 通过
