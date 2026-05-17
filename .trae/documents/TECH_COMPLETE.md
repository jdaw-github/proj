# 技术架构文档（TECH）- MyAiProj 个人工具网站

## 文档信息

- **文档版本**：v1.0
- **更新日期**：2024年
- **文档状态**：已完成

---

## 第一部分：架构设计总览

### 1.1 系统架构

本项目采用纯前端静态页面架构，所有功能通过HTML、CSS和JavaScript原生技术实现，无需后端服务器支持。整体架构遵循分层设计的原则，将页面结构、样式表现和业务逻辑分离到不同的代码区域，便于维护和扩展。

```
┌─────────────────────────────────────────────────────────────┐
│                    表现层 (Presentation Layer)              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐       │
│  │ HTML5语义化   │  │ Tailwind CSS │  │ 自定义CSS    │       │
│  │   标签       │  │   样式框架    │  │   动画效果    │       │
│  └─────────────┘  └─────────────┘  └─────────────┘       │
├─────────────────────────────────────────────────────────────┤
│                    交互层 (Interaction Layer)               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐       │
│  │ DOM操作      │  │ 事件处理     │  │ 表单验证     │       │
│  └─────────────┘  └─────────────┘  └─────────────┘       │
├─────────────────────────────────────────────────────────────┤
│                    业务逻辑层 (Business Logic Layer)         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐       │
│  │ DateCalculator│ │PasswordGenerator│ │ StateManager│       │
│  │ 日期计算      │  │ 密码生成     │  │ 状态管理     │       │
│  └─────────────┘  └─────────────┘  └─────────────┘       │
├─────────────────────────────────────────────────────────────┤
│                    数据层 (Data Layer)                      │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐       │
│  │ LocalStorage │  │ Web Crypto  │  │ 外部API     │       │
│  │ 本地存储      │  │ 随机数生成   │  │ 假期数据    │       │
│  └─────────────┘  └─────────────┘  └─────────────┘       │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 项目组成

本项目包含三个独立的HTML页面，各自承载不同的功能模块。这种设计确保了各工具之间的完全解耦，可以独立开发、测试和部署。

| 页面名称 | 文件路径 | 主要功能 | 技术复杂度 |
|----------|----------|----------|------------|
| 个人博客主页 | index.html | 工具导航、个人介绍 | 低 |
| 安全密码生成器 | tool/HTML_密码生成器/HTML_密码生成器.html | 密码生成、二维码 | 中 |
| 工作日期计算器 | tool/HTML_工作日计算器/HTML_工作日计算器.html | 日期计算、日历展示 | 高 |

---

## 第二部分：个人博客主页技术方案

### 2.1 技术选型

个人博客主页采用最轻量的技术栈实现，优先考虑加载速度和部署便捷性。所有样式通过Tailwind CSS CDN加载，利用浏览器缓存机制减少重复请求。

| 技术项 | 选择 | 说明 |
|--------|------|------|
| 标记语言 | HTML5 | 使用语义化标签组织页面结构 |
| 样式方案 | Tailwind CSS 3 (CDN) | 原子化CSS框架，快速构建响应式布局 |
| 字体 | Noto Sans SC (Google Fonts) | 优化的中文字体显示 |
| 图标 | Emoji + Inline SVG | 无需外部图标库，零额外请求 |
| 构建工具 | 无 | 纯静态文件，直接部署 |

### 2.2 目录结构

```
MyAiProj/
├── index.html                          # 个人博客主页入口
├── images/
│   └── 小猫头像.png                     # 头像图片资源
├── tool/                               # 工具页面目录
│   ├── HTML_密码生成器/
│   │   └── HTML_密码生成器.html         # 密码生成器工具
│   └── HTML_工作日计算器/
│       └── HTML_工作日计算器.html       # 工作日期计算器
└── .trae/
    └── documents/                       # 项目文档目录
        ├── PROJECT_OVERVIEW.md          # 项目总览文档
        ├── PRD_COMPLETE.md              # 产品需求文档
        └── TECH_COMPLETE.md            # 技术架构文档
```

### 2.3 核心组件实现

#### 2.3.1 固定导航栏

导航栏采用`position: sticky`定位方式固定在页面顶部，通过`backdrop-filter: blur()`实现毛玻璃效果。导航栏背景使用`bg-white/95`设置半透明白色底色，确保内容可读性。

```html
<header class="sticky top-0 z-50 bg-white/95 backdrop-blur-sm header-shadow">
    <div class="max-w-5xl mx-auto px-6 py-4">
        <!-- Logo和标题区域 -->
        <div class="flex items-center gap-3">
            <div class="w-10 h-10 rounded-xl overflow-hidden">
                <img src="images/小猫头像.png" alt="头像">
            </div>
            <div>
                <h1 class="text-xl font-bold text-gray-800">我的个人博客</h1>
                <p class="text-xs text-gray-500">记录技术成长，分享编程心得</p>
            </div>
        </div>
        <!-- 导航链接 -->
        <nav class="flex items-center gap-6">
            <a href="#tools" class="text-gray-600 hover:text-purple-600">实用工具</a>
            <a href="#about" class="text-gray-600 hover:text-purple-600">关于我</a>
        </nav>
    </div>
</header>
```

#### 2.3.2 工具卡片组件

工具卡片采用网格布局展示，包含图标容器、标题、描述和操作按钮四个部分。卡片支持悬停交互效果，通过CSS过渡动画实现平滑的视觉反馈。

```html
<article class="bg-white rounded-xl shadow-sm border border-gray-100 p-6 card-hover">
    <div class="flex items-start gap-4">
        <!-- 图标容器：渐变背景配合emoji -->
        <div class="w-12 h-12 bg-gradient-to-br from-purple-100 to-purple-200 rounded-xl flex items-center justify-center">
            <span class="text-2xl">🔐</span>
        </div>
        <!-- 内容区域 -->
        <div class="flex-1">
            <h3 class="text-lg font-semibold text-gray-800 mb-2">工具名称</h3>
            <p class="text-gray-500 text-sm mb-4">工具描述内容...</p>
            <!-- 渐变按钮 -->
            <a href="..." class="gradient-btn inline-flex items-center gap-2 text-white px-4 py-2 rounded-lg text-sm font-medium">
                <span>立即使用</span>
                <svg class="w-4 h-4">...</svg>
            </a>
        </div>
    </div>
</article>
```

#### 2.3.3 欢迎区域

欢迎区域是页面的视觉焦点，包含头像展示和欢迎文案。采用居中布局和充足的留白设计，营造简洁大气的视觉效果。

```html
<section class="text-center mb-16">
    <div class="inline-block mb-6">
        <!-- 带头像的圆形容器 -->
        <div class="w-20 h-20 rounded-2xl overflow-hidden shadow-lg shadow-purple-500/20">
            <img src="images/小猫头像.png" alt="头像">
        </div>
    </div>
    <h2 class="text-3xl font-bold text-gray-800 mb-4">欢迎来到我的技术小站</h2>
    <p class="text-gray-500 max-w-xl mx-auto">这里汇聚了我开发的实用工具...</p>
</section>
```

### 2.4 自定义CSS样式

页面使用内联`<style>`标签定义自定义样式，与Tailwind CSS配合实现完整的视觉效果。

```css
/* 卡片悬停动画：上浮并加深阴影 */
.card-hover {
    transition: all 0.3s ease;
}
.card-hover:hover {
    transform: translateY(-2px);
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08);
}

/* 渐变按钮：紫色渐变背景配合悬停效果 */
.gradient-btn {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    border: none;
    transition: all 0.3s ease;
}
.gradient-btn:hover {
    transform: translateY(-2px);
    box-shadow: 0 6px 20px rgba(102, 126, 234, 0.4);
}

/* 导航栏阴影效果 */
.header-shadow {
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.05);
}
```

### 2.5 样式变量系统

页面使用统一的颜色变量和间距系统，确保视觉风格的一致性。

| 变量类型 | 名称 | 值 | 用途 |
|----------|------|-----|------|
| 主色调 | Primary | #667eea | 按钮、链接、渐变起点 |
| 辅助色 | Accent | #764ba2 | 渐变终点、强调元素 |
| 背景色 | Background | #F8FAFC | 页面背景 |
| 卡片背景 | Card | #FFFFFF | 组件卡片背景 |
| 主文字 | Text Primary | #1F2937 | 主要文字内容 |
| 次文字 | Text Secondary | #6B7280 | 次要说明文字 |
| 边框色 | Border | #E5E7EB | 卡片边框、分割线 |
| 圆角 | Radius | 12px-16px | 卡片、按钮圆角 |
| 阴影 | Shadow | 0 4px 20px rgba(0, 0, 0, 0.06) | 卡片默认阴影 |

### 2.6 响应式布局策略

页面采用Tailwind CSS的响应式前缀实现多端适配，主要调整容器宽度和网格列数。

| 断点 | 宽度范围 | 布局调整 | 容器最大宽度 |
|------|----------|----------|--------------|
| 默认 | < 768px | 单列布局，全宽显示 | 100% |
| md | ≥ 768px | 双列网格，适度内边距 | 768px |
| lg | ≥ 1024px | 最大宽度约束，居中显示 | 1024px |

---

## 第三部分：密码生成器技术方案

### 3.1 技术选型

密码生成器在主页技术基础上增加了二维码生成功能，引入QRCode.js库实现客户端二维码编码。

| 技术项 | 选择 | 说明 |
|--------|------|------|
| 基础框架 | HTML5 + CSS3 + JavaScript | 原生技术栈，无需构建 |
| 样式方案 | Tailwind CSS (CDN) | 响应式样式框架 |
| 随机数生成 | Web Crypto API | crypto.getRandomValues加密安全随机数 |
| 二维码库 | QRCode.js 1.5.1 | 客户端二维码生成 |

### 3.2 核心功能实现

#### 3.2.1 密码字符配置

密码字符集通过JavaScript对象管理，支持灵活的类型组合和相似字符排除。

```javascript
// 字符集配置
const charSets = {
    uppercase: 'ABCDEFGHIJKLMNOPQRSTUVWXYZ',    // 大写字母
    lowercase: 'abcdefghijklmnopqrstuvwxyz',    // 小写字母
    numbers: '0123456789',                       // 数字
    commonsymbols: '_-.',                        // 普通符号
    symbols: '!@#$%^&*()_+-=[]{}|;:,.<>?'       // 特殊符号
};

// 相似字符排除列表
const similarChars = '0Ool1I';
```

#### 3.2.2 密码生成算法

密码生成使用加密安全的随机数生成器，确保每一位字符的随机性。通过Fisher-Yates洗牌算法确保生成的密码字符分布均匀。

```javascript
function generatePassword() {
    // 获取用户配置的密码长度
    const length = parseInt(lengthEl.value);
    
    // 构建可用字符集
    let availableChars = '';
    if (useUppercase) availableChars += charSets.uppercase;
    if (useLowercase) availableChars += charSets.lowercase;
    if (useNumbers) availableChars += charSets.numbers;
    if (useCommonSymbols) availableChars += charSets.commonsymbols;
    if (useSymbols) availableChars += charSets.symbols;
    
    // 排除相似字符
    if (excludeSimilar) {
        availableChars = availableChars.split('')
            .filter(c => !similarChars.includes(c))
            .join('');
    }
    
    // 使用加密随机数生成密码
    const array = new Uint32Array(length);
    crypto.getRandomValues(array);
    
    let password = '';
    for (let i = 0; i < length; i++) {
        password += availableChars[array[i] % availableChars.length];
    }
    
    // 确保每种选中类型至少有一个字符
    let guaranteedChars = '';
    if (useUppercase) guaranteedChars += getRandomChar(charSets.uppercase);
    if (useLowercase) guaranteedChars += getRandomChar(charSets.lowercase);
    // ... 其他字符类型
    
    // 合并并洗牌
    password = guaranteedChars + password.slice(guaranteedChars.length);
    password = shuffleString(password);
    
    return password;
}
```

#### 3.2.3 密码强度计算

密码强度通过综合评分算法计算，考虑长度和字符多样性两个维度。

```javascript
function calculateStrength(password) {
    if (!password) return 0;
    
    let score = 0;
    
    // 长度评分：递进加分
    if (password.length >= 4) score += 1;
    if (password.length >= 8) score += 1;
    if (password.length >= 12) score += 1;
    if (password.length >= 16) score += 1;
    if (password.length >= 24) score += 1;
    
    // 字符多样性评分
    if (/[a-z]/.test(password)) score += 1;   // 小写字母
    if (/[A-Z]/.test(password)) score += 1;   // 大写字母
    if (/[0-9]/.test(password)) score += 1;    // 数字
    if (/[^a-zA-Z0-9]/.test(password)) score += 1;  // 特殊字符
    
    return score;
}
```

#### 3.2.4 二维码生成

二维码生成使用QRCode.js库，支持将密码和附加信息编码为二维码图像。

```javascript
function updateQRCodeWithAccountInfo(password) {
    qrcodeEl.innerHTML = '';
    
    // 构建二维码内容
    let qrContent = password;
    const accountInfo = accountInfoEl.value.trim();
    if (accountInfo) {
        qrContent = password + '\n\n' + accountInfo;
    }
    
    if (!qrContent) return;
    
    // 生成二维码
    QRCode.toCanvas(qrContent, {
        width: 150,
        margin: 1,
        color: {
            dark: '#667eea',    // 前景色：品牌主色
            light: '#ffffff'    // 背景色：白色
        }
    }, function(error, canvas) {
        if (error) {
            console.error(error);
            return;
        }
        qrcodeEl.appendChild(canvas);
    });
}
```

### 3.3 交互事件处理

#### 3.3.1 长度配置联动

密码长度通过滑动条和数字输入框双重控制，两者保持实时同步。

```javascript
// 滑动条变化时同步更新数字输入框
lengthEl.addEventListener('input', () => {
    lengthNumberEl.value = lengthEl.value;
    generatePassword();
});

// 数字输入框变化时同步更新滑动条
lengthNumberEl.addEventListener('input', () => {
    let value = parseInt(lengthNumberEl.value);
    if (isNaN(value)) value = 8;
    value = Math.max(4, Math.min(64, value));  // 限制范围
    lengthNumberEl.value = value;
    lengthEl.value = value;
    generatePassword();
});
```

#### 3.3.2 复制功能实现

复制功能使用Clipboard API实现，同时提供后备的execCommand方案。

```javascript
copyBtn.addEventListener('click', async () => {
    const password = passwordEl.value;
    if (!password) return;
    
    try {
        await navigator.clipboard.writeText(password);
        showMessage('密码已复制到剪贴板！', 'text-green-500');
    } catch (err) {
        // Clipboard API不可用时的后备方案
        passwordEl.select();
        document.execCommand('copy');
        showMessage('密码已复制到剪贴板！', 'text-green-500');
    }
});
```

---

## 第四部分：工作日期计算器技术方案

### 4.1 技术选型

工作日期计算器是技术复杂度最高的模块，涉及日期计算逻辑、状态管理和外部数据获取。

| 技术项 | 选择 | 说明 |
|--------|------|------|
| 基础框架 | HTML5 + CSS3 + JavaScript | 原生技术栈 |
| 样式方案 | Tailwind CSS (CDN) | 响应式UI |
| 状态管理 | 原生JavaScript对象 | 无需状态管理库 |
| 数据持久化 | LocalStorage API | 配置数据本地存储 |
| 日期处理 | 原生Date对象 | 满足所有日期计算需求 |
| 假期数据 | timor.tech API | 法定假期数据来源 |

### 4.2 核心模块设计

#### 4.2.1 状态管理

应用状态通过单一的state对象集中管理，便于调试和状态恢复。

```javascript
let state = {
    scheduleMode: 'double',           // 排班模式：double/single/alternating/custom
    alternatingStartWeek: 1,          // 大小周起始周：1或2
    customRestDays: [],               // 自定义休息日数组
    useLegalHolidays: true,           // 是否使用法定假期
    leaveDays: [],                    // 事假日期数组
    calculationResult: null,          // 计算结果缓存
    calendarStartDate: null,          // 日历显示开始日期
    calendarEndDate: null,            // 日历显示结束日期
    displayStartMonth: new Date(),     // 日历显示起始月份
    currentYear: new Date().getFullYear()  // 当前年份
};
```

#### 4.2.2 配置持久化

使用LocalStorage实现配置的保存和恢复，支持配置的导入导出。

```javascript
const STORAGE_KEY = 'date_calculator_config';

// 保存配置到本地存储
function saveConfig() {
    const config = {
        scheduleMode: state.scheduleMode,
        alternatingStartWeek: state.alternatingStartWeek,
        customRestDays: state.customRestDays,
        useLegalHolidays: state.useLegalHolidays,
        leaveDays: state.leaveDays,
        currentYear: state.currentYear
    };
    localStorage.setItem(STORAGE_KEY, JSON.stringify(config));
}

// 从本地存储加载配置
function loadConfig() {
    const saved = localStorage.getItem(STORAGE_KEY);
    if (saved) {
        try {
            const config = JSON.parse(saved);
            state = { ...state, ...config };
            // 恢复UI状态
            document.querySelector(`input[name="scheduleMode"][value="${state.scheduleMode}"]`).checked = true;
            document.querySelector(`input[name="alternatingStart"][value="${state.alternatingStartWeek}"]`).checked = true;
            document.getElementById('useLegalHolidays').checked = state.useLegalHolidays;
        } catch (e) {
            console.warn('配置加载失败，使用默认值');
        }
    }
}
```

### 4.3 日期计算逻辑

#### 4.3.1 工作日判断

工作日的判断遵循严格的优先级规则，确保计算结果的准确性。

```javascript
// 工作日判断核心函数
function isWorkDay(date) {
    const dateStr = formatDate(date);
    
    // 优先级1：事假判定
    if (isLeaveDay(date)) return false;
    
    // 优先级2：法定假期判定
    const info = HOLIDAY_INFO[dateStr];
    if (info) {
        if (info.type === 1) return false;      // 法定假期，休息
        if (info.type === 3) return true;       // 调休补班，工作
    }
    
    // 优先级3：法定假期数据检查
    if (LEGAL_HOLIDAYS.hasOwnProperty(dateStr)) {
        if (LEGAL_HOLIDAYS[dateStr].includes('调休')) {
            return true;                        // 调休补班，工作
        }
        return false;                           // 法定假期，休息
    }
    
    // 优先级4：排班规则判定
    if (isRestDay(date)) return false;          // 休息日
    
    return true;                                // 普通工作日
}
```

#### 4.3.2 排班规则实现

四种排班模式通过统一接口实现，隐藏底层差异。

```javascript
// 获取指定日期的休息日数组
function getRestDaysOfWeek(date) {
    const scheduleMode = state.scheduleMode;
    
    if (scheduleMode === 'double') {
        return [0, 6];                         // 周六、周日休息
    } else if (scheduleMode === 'single') {
        return [0];                            // 周日休息
    } else if (scheduleMode === 'alternating') {
        // 大小周模式：根据周数判断
        const startDate = new Date('2026-01-01');
        const weekNum = getWeekNumber(date, startDate);
        const isSingleWeek = (weekNum % 2 === state.alternatingStartWeek % 2);
        return isSingleWeek ? [0] : [0, 6];    // 单周休1天，双周休2天
    } else if (scheduleMode === 'custom') {
        return state.customRestDays;           // 自定义休息日
    }
    
    return [0, 6];                             // 默认双休
}

// 判断指定日期是否为休息日
function isRestDay(date) {
    const dayOfWeek = date.getDay();
    const restDays = getRestDaysOfWeek(date);
    return restDays.includes(dayOfWeek);
}
```

#### 4.3.3 日期计算主流程

计算从开始日期起，需要多少个工作日到达的完整流程。

```javascript
async function calculateEndDate() {
    // 1. 解析输入
    const startDateInput = document.getElementById('startDate').value;
    const workDaysInput = document.getElementById('workDays').value;
    const workDays = parseInt(workDaysInput);
    
    // 2. 输入验证
    if (!startDateInput || isNaN(workDays) || workDays < 1) {
        alert('请输入有效的日期和天数');
        return;
    }
    
    // 3. 解析日期
    const startDate = parseDate(startDateInput);
    if (!startDate) {
        alert('日期格式不正确');
        return;
    }
    
    // 4. 加载假期数据
    const year1 = startDate.getFullYear();
    const year2 = year1 + 1;
    if (!LOADED_YEARS.includes(year1) || !LOADED_YEARS.includes(year2)) {
        LEGAL_HOLIDAYS = {};
        HOLIDAY_INFO = {};
        LOADED_YEARS = [];
        await loadHolidaysFromCDN(startDate);
    }
    
    // 5. 执行计算
    let currentDate = new Date(startDate);
    let workDaysCount = 0;
    let totalDays = 0;
    let restDaysCount = 0;
    let holidayDaysCount = 0;
    let leaveDaysCount = 0;
    const calendarDays = [];
    
    while (workDaysCount < workDays) {
        const dayType = getDayType(currentDate);
        const isWork = dayType === 'work' || dayType === 'workday-makeup';
        
        calendarDays.push({
            date: new Date(currentDate),
            type: dayType,
            isWork: isWork
        });
        
        if (isWork) {
            workDaysCount++;
        } else if (dayType === 'rest') {
            restDaysCount++;
        } else if (dayType === 'holiday') {
            holidayDaysCount++;
        } else if (dayType === 'leave') {
            leaveDaysCount++;
        }
        
        totalDays++;
        currentDate.setDate(currentDate.getDate() + 1);
        
        // 防止无限循环
        if (totalDays > 365 * 2) {
            alert('计算范围超过两年');
            return;
        }
    }
    
    // 6. 更新UI
    const endDate = calendarDays[calendarDays.length - 1].date;
    updateResultUI(endDate, calendarDays, {
        totalDays,
        workDaysCount,
        restDaysCount,
        holidayDaysCount,
        leaveDaysCount
    });
    
    // 7. 渲染日历
    renderFullCalendar();
}
```

### 4.4 假期数据管理

#### 4.4.1 API数据获取

通过调用timor.tech的API获取法定假期数据，支持跨域请求。

```javascript
async function loadHolidaysFromAPI(year) {
    const url = `https://timor.tech/api/holiday/year/${year}`;
    
    try {
        const response = await fetch(url, { method: 'GET', mode: 'cors' });
        
        if (response.ok) {
            const data = await response.json();
            const holidays = {};
            const holidayInfo = {};
            const yearStr = year.toString();
            
            if (data.code === 0 && data.holiday) {
                Object.keys(data.holiday).forEach(dateKey => {
                    const item = data.holiday[dateKey];
                    if (item) {
                        let type = 0;
                        let name = item.name || '';
                        
                        // 解析假期类型
                        if (item.holiday === true) type = 1;           // 法定假期
                        else if (item.holiday === false) type = 3;   // 调休补班
                        
                        // 构建完整日期
                        let fullDate = item.date || `${yearStr}-${dateKey}`;
                        
                        holidayInfo[fullDate] = { type, name };
                        if (type === 1) {
                            holidays[fullDate] = name;
                        } else if (type === 3) {
                            holidays[fullDate] = `法定调休(${name})`;
                        }
                    }
                });
            }
            
            return { holidays, holidayInfo };
        }
    } catch (e) {
        console.error(`加载${year}年假期数据失败:`, e);
    }
    return null;
}
```

#### 4.4.2 默认数据配置

当API不可用时，使用内置的默认假期数据作为补充。

```javascript
const DEFAULT_HOLIDAYS_2026 = {
    '2026-01-01': '元旦',
    '2026-01-26': '春节前调休',
    '2026-01-27': '除夕',
    '2026-01-28': '春节',
    '2026-01-29': '正月初二',
    '2026-01-30': '正月初三',
    '2026-01-31': '正月初四',
    '2026-02-01': '正月初五',
    '2026-02-02': '正月初六',
    '2026-04-03': '清明节前调休',
    '2026-04-04': '清明节',
    '2026-04-05': '清明节后调休',
    '2026-05-01': '劳动节',
    '2026-05-02': '劳动节假期',
    '2026-05-03': '劳动节假期',
    '2026-05-04': '劳动节假期',
    '2026-05-05': '劳动节假期',
    '2026-06-19': '端午节前调休',
    '2026-06-20': '端午节',
    '2026-06-21': '端午节后调休',
    '2026-09-25': '中秋节前调休',
    '2026-09-26': '中秋节',
    '2026-09-27': '中秋节后调休',
    '2026-10-01': '国庆节',
    '2026-10-02': '国庆节',
    '2026-10-03': '国庆节',
    '2026-10-04': '国庆节',
    '2026-10-05': '国庆节',
    '2026-10-06': '国庆节',
    '2026-10-07': '国庆节',
    '2026-10-08': '国庆节后调休',
    '2026-10-09': '国庆节后调休'
};

const DEFAULT_WORKDAYS = {
    '2025-01-26': '春节调休补班',
    '2025-02-08': '春节调休补班',
    '2025-04-26': '五一调休补班',
    '2025-05-09': '五一调休补班',
    '2025-09-28': '国庆调休补班',
    '2025-10-11': '国庆调休补班'
};
```

### 4.5 日历渲染实现

#### 4.5.1 日历月份渲染

日历采用双列网格布局，每月一个独立的渲染卡片。

```javascript
function renderCalendarMonths() {
    const container = document.getElementById('calendarPreview');
    container.innerHTML = '';
    
    if (!state.calendarStartDate || !state.calendarEndDate) {
        container.innerHTML = '<p class="text-gray-400 text-center py-8 col-span-2">请先计算以查看日历</p>';
        return;
    }
    
    // 计算需要渲染的月份范围
    const startYear = state.calendarStartDate.getFullYear();
    const startMonth = state.calendarStartDate.getMonth();
    const endYear = state.calendarEndDate.getFullYear();
    const endMonth = state.calendarEndDate.getMonth();
    
    // 遍历渲染每个月
    let currentYear = startYear;
    let currentMonth = startMonth;
    const monthNames = ['一月', '二月', '三月', '四月', '五月', '六月', 
                        '七月', '八月', '九月', '十月', '十一月', '十二月'];
    
    while (currentYear < endYear || (currentYear === endYear && currentMonth <= endMonth)) {
        const monthHTML = renderMonthCard(currentYear, currentMonth, monthNames[currentMonth]);
        container.innerHTML += monthHTML;
        
        currentMonth++;
        if (currentMonth > 11) {
            currentMonth = 0;
            currentYear++;
        }
    }
}

function renderMonthCard(year, month, monthName) {
    // 星期标题行
    let weekDaysHTML = '';
    const weekDays = ['日', '一', '二', '三', '四', '五', '六'];
    weekDays.forEach(day => {
        weekDaysHTML += `<div class="text-center text-xs font-medium text-gray-500 py-1">${day}</div>`;
    });
    
    // 计算本月第一天是星期几
    const firstDayOfMonth = new Date(year, month, 1);
    const startDayOfWeek = firstDayOfMonth.getDay();
    
    // 空单元格填充
    let emptyCellsHTML = '';
    for (let j = 0; j < startDayOfWeek; j++) {
        emptyCellsHTML += `<div class="date-cell"></div>`;
    }
    
    // 日期单元格
    let dateCellsHTML = '';
    const lastDayOfMonth = new Date(year, month + 1, 0);
    for (let day = 1; day <= lastDayOfMonth.getDate(); day++) {
        const currentDate = new Date(year, month, day);
        const dayType = getDayType(currentDate);
        const dayClass = getDayClass(dayType, currentDate);
        
        dateCellsHTML += `
            <div class="date-cell flex flex-col items-center justify-center rounded-lg ${dayClass}" title="${month + 1}月${day}日">
                <span class="text-sm font-medium">${day}</span>
                <span class="text-xs opacity-75">${weekDays[currentDate.getDay()]}</span>
            </div>
        `;
    }
    
    return `
        <div class="calendar-month">
            <h4 class="mb-3 text-center">${year}年 ${monthName}</h4>
            <div class="grid grid-cols-7 gap-1">
                ${weekDaysHTML}
                ${emptyCellsHTML}
                ${dateCellsHTML}
            </div>
        </div>
    `;
}
```

#### 4.5.2 日期类型样式

不同日期类型对应不同的CSS类名，呈现差异化的视觉效果。

```javascript
function getDayClass(dayType, date) {
    const isInRange = state.calendarStartDate && state.calendarEndDate &&
        date >= state.calendarStartDate && date <= state.calendarEndDate;
    
    switch (dayType) {
        case 'work':
            return isInRange ? 'work-day in-range' : 'work-day';
        case 'workday-makeup':
            return 'makeup-work-day';         // 调休补班：橙色
        case 'rest':
            return 'rest-day';               // 休息日：红色
        case 'leave':
            return 'leave-day';              // 事假：黄色
        case 'holiday':
            return 'holiday';                // 法定假期：紫色
        default:
            return 'work-day';
    }
}
```

### 4.6 事假管理功能

#### 4.6.1 添加事假

支持单个日期或日期范围的事假添加。

```javascript
function addLeaveDay() {
    const startDateStr = document.getElementById('leaveStartDate').value;
    const endDateStr = document.getElementById('leaveEndDate').value;
    
    if (!startDateStr) {
        alert('请选择开始日期');
        return;
    }
    
    const datesToAdd = [];
    const startDate = new Date(startDateStr);
    
    if (endDateStr) {
        // 添加日期范围内的所有日期
        const endDate = new Date(endDateStr);
        let currentDate = new Date(startDate);
        while (currentDate <= endDate) {
            datesToAdd.push(formatDate(currentDate));
            currentDate.setDate(currentDate.getDate() + 1);
        }
    } else {
        // 仅添加单个日期
        datesToAdd.push(startDateStr);
    }
    
    // 去重添加
    datesToAdd.forEach(dateStr => {
        if (!state.leaveDays.includes(dateStr)) {
            state.leaveDays.push(dateStr);
        }
    });
    
    state.leaveDays.sort();
    saveConfig();
    updateLeaveDaysList();
    
    // 清空输入
    document.getElementById('leaveStartDate').value = '';
    document.getElementById('leaveEndDate').value = '';
}
```

#### 4.6.2 删除事假

支持单个日期和连续日期范围的删除。

```javascript
// 删除单个日期
function removeLeaveDay(dateStr) {
    state.leaveDays = state.leaveDays.filter(d => d !== dateStr);
    saveConfig();
    updateLeaveDaysList();
}

// 删除日期范围
function removeLeaveRange(startDateStr, endDateStr) {
    const startDate = new Date(startDateStr);
    const endDate = new Date(endDateStr);
    
    let currentDate = new Date(startDate);
    while (currentDate <= endDate) {
        const dateStr = formatDate(currentDate);
        state.leaveDays = state.leaveDays.filter(d => d !== dateStr);
        currentDate.setDate(currentDate.getDate() + 1);
    }
    
    saveConfig();
    updateLeaveDaysList();
}
```

### 4.7 日期工具函数

#### 4.7.1 日期解析

支持多种输入格式的日期解析。

```javascript
function parseDate(input) {
    if (!input) return null;
    
    const trimmed = input.trim();
    
    // 支持的格式列表
    const patterns = [
        /^(\d{4})[-/](\d{1,2})[-/](\d{1,2})$/,      // YYYY-MM-DD 或 YYYY/MM/DD
        /^(\d{4})年(\d{1,2})月(\d{1,2})日?$/,        // YYYY年MM月DD日
        /^(\d{4})年(\d{1,2})月(\d{1,2})$/            // YYYY年MM月DD
    ];
    
    for (const pattern of patterns) {
        const match = trimmed.match(pattern);
        if (match) {
            const year = parseInt(match[1]);
            const month = parseInt(match[2]) - 1;
            const day = parseInt(match[3]);
            if (year >= 2000 && year <= 2100 && month >= 0 && month <= 11 && day >= 1 && day <= 31) {
                return new Date(year, month, day);
            }
        }
    }
    
    // 兜底：尝试JavaScript原生解析
    const date = new Date(trimmed);
    if (!isNaN(date.getTime())) {
        return date;
    }
    
    return null;
}

// 格式化日期为YYYY-MM-DD
function formatDate(date) {
    const year = date.getFullYear();
    const month = String(date.getMonth() + 1).padStart(2, '0');
    const day = String(date.getDate()).padStart(2, '0');
    return `${year}-${month}-${day}`;
}

// 获取星期名称
function getWeekdayName(date) {
    const weekdays = ['周日', '周一', '周二', '周三', '周四', '周五', '周六'];
    return weekdays[date.getDay()];
}
```

---

## 第五部分：通用技术规范

### 5.1 样式变量系统

所有页面共享统一的样式变量系统，确保视觉风格的一致性。

```css
:root {
    /* 主色调 */
    --color-primary: #667eea;
    --color-primary-dark: #764ba2;
    
    /* 背景色 */
    --color-bg: #F8FAFC;
    --color-card: #FFFFFF;
    
    /* 文字色 */
    --color-text: #1F2937;
    --color-text-secondary: #6B7280;
    
    /* 功能色 */
    --color-success: #10B981;
    --color-danger: #EF4444;
    --color-warning: #F59E0B;
    
    /* 间距 */
    --spacing-unit: 8px;
    --radius-sm: 8px;
    --radius-md: 12px;
    --radius-lg: 16px;
    
    /* 阴影 */
    --shadow-sm: 0 1px 3px rgba(0, 0, 0, 0.05);
    --shadow-md: 0 4px 20px rgba(0, 0, 0, 0.06);
    --shadow-lg: 0 10px 30px rgba(0, 0, 0, 0.08);
    
    /* 过渡 */
    --transition-fast: 0.15s ease;
    --transition-normal: 0.3s ease;
}
```

### 5.2 动画效果规范

页面中的动画效果遵循统一的时长和缓动函数规范。

| 动画类型 | 时长 | 缓动函数 | 适用场景 |
|----------|------|----------|----------|
| 快速过渡 | 0.15s | ease | 按钮按下状态 |
| 标准过渡 | 0.3s | ease | 卡片悬停、颜色变化 |
| 滑入动画 | 0.4s | ease-out | 结果显示 |
| 淡入动画 | 0.3s | ease-out | 模态框弹出 |

### 5.3 性能优化策略

#### 5.3.1 CSS优化

- 使用Tailwind CSS CDN，利用浏览器缓存减少重复加载
- 内联关键CSS，减少渲染阻塞
- 避免过深的CSS选择器嵌套
- 使用CSS变量实现主题复用

#### 5.3.2 JavaScript优化

- 事件委托：使用单个事件监听器处理同类元素
- 防抖节流：对高频事件（如滚动、输入）应用防抖
- DOM缓存：避免重复查询同一DOM元素
- 按需加载：二维码库仅在密码生成器页面加载

#### 5.3.3 资源优化

- 图片资源使用适当格式和压缩
- 字体使用font-display: swap确保文字可读
- 外部CDN使用稳定可靠的公共服务
- 避免不必要的第三方依赖

### 5.4 浏览器兼容性

| 浏览器 | 最低版本 | 支持特性 |
|--------|----------|----------|
| Chrome | 80+ | 全部特性 |
| Firefox | 75+ | 全部特性 |
| Safari | 13+ | 全部特性 |
| Edge | 80+ | 全部特性 |
| IE | 不支持 | - |

### 5.5 无障碍支持

| 检查项 | 实现方式 | 说明 |
|--------|----------|------|
| 语义化标签 | 正确使用header、main、nav、section等 | 辅助技术可正确理解页面结构 |
| 焦点管理 | 清晰的焦点样式，支持Tab导航 | 键盘用户可正常使用 |
| 颜色对比 | 主文本对比度≥4.5:1 | 视觉障碍用户可识别 |
| 表单关联 | label与input正确关联 | 屏幕阅读器正确朗读 |
| 替代文本 | 图片有alt属性，图标有aria-label | 非视觉用户可获取信息 |

---

## 第六部分：部署说明

### 6.1 部署方式

项目支持多种静态文件部署方式，根据实际需求选择合适的方案。

| 部署方案 | 适用场景 | 配置复杂度 | 费用 |
|----------|----------|------------|------|
| GitHub Pages | 开源项目、个人网站 | 低 | 免费 |
| Vercel | 快速部署、全球CDN | 低 | 免费额度 |
| Netlify | 静态网站、自动化部署 | 低 | 免费额度 |
| Nginx | 自有服务器 | 中 | 服务器费用 |
| Apache | 自有服务器 | 中 | 服务器费用 |

### 6.2 部署检查清单

在部署前进行以下检查，确保网站正常运行。

- [ ] 所有HTML文件语法正确，无明显错误
- [ ] 外部CDN资源（Tailwind CSS、Google Fonts）可正常访问
- [ ] 工具页面之间的相对路径链接正确
- [ ] 图片资源路径正确，图片可正常加载
- [ ] 移动端响应式布局正常显示
- [ ] 表单交互功能正常工作
- [ ] 浏览器控制台无错误信息

### 6.3 目录结构要求

部署时需要保持以下目录结构不变，确保相对路径引用正常工作。

```
MyAiProj/                          # 项目根目录
├── index.html                     # 主页入口（必须位于根目录）
├── images/                        # 图片资源目录
│   └── 小猫头像.png              # 头像图片
├── tool/                          # 工具页面目录
│   ├── HTML_密码生成器/
│   │   └── HTML_密码生成器.html  # 密码生成器工具
│   └── HTML_工作日计算器/
│       └── HTML_工作日计算器.html # 工作日期计算器工具
└── .trae/                        # 文档目录（可选不上传）
    └── documents/
```

### 6.4 域名配置建议

如果使用自定义域名，建议进行以下配置。

1. **SSL证书**：启用HTTPS，确保数据传输安全
2. **WWW重定向**：将非WWW域名重定向到WWW（或反之）
3. **缓存配置**：为静态资源设置长期缓存策略
4. **Gzip压缩**：启用服务器端压缩，减少传输体积

---

## 第七部分：API接口文档

### 7.1 假期数据API

**接口地址**：`https://timor.tech/api/holiday/year/{year}`

**请求方式**：GET

**请求参数**：

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| year | 整数 | 是 | 年份，如2026 |

**响应格式**：

```json
{
    "code": 0,
    "holiday": {
        "01-01": {
            "holiday": true,
            "name": "元旦",
            "date": "2026-01-01",
            "wage": 3,
            "rest": 1
        }
    },
    "workday": []
}
```

**响应字段说明**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | 整数 | 响应状态码，0表示成功 |
| holiday | 对象 | 法定假期数据，key为月日格式 |
| holiday.holiday | 布尔 | true为假期，false为调休 |
| holiday.name | 字符串 | 假期名称 |
| holiday.date | 字符串 | 完整日期 |
| workday | 数组 | 调休补班数据（可能为空） |

---

## 第八部分：数据模型

### 8.1 配置数据结构

```typescript
interface CalculatorConfig {
    scheduleMode: 'double' | 'single' | 'alternating' | 'custom';
    alternatingStartWeek: 1 | 2;
    customRestDays: number[];     // 0=周日, 1=周一, ..., 6=周六
    useLegalHolidays: boolean;
    leaveDays: string[];          // ['2026-05-20', '2026-05-21']
    currentYear: number;
}
```

### 8.2 LocalStorage存储格式

```json
{
    "scheduleMode": "alternating",
    "alternatingStartWeek": 1,
    "customRestDays": [0, 6],
    "useLegalHolidays": true,
    "leaveDays": ["2026-05-20", "2026-05-21"],
    "currentYear": 2026
}
```

### 8.3 假期信息数据结构

```typescript
interface HolidayInfo {
    [date: string]: {
        type: 1 | 3;    // 1=法定假期, 3=调休补班
        name: string;    // 假期名称
    }
}
```

### 8.4 计算结果数据结构

```typescript
interface CalculationResult {
    startDate: Date;
    endDate: Date;
    totalDays: number;
    workDays: number;
    restDays: number;
    holidayDays: number;
    leaveDays: number;
    calendarDays: CalendarDay[];
}

interface CalendarDay {
    date: Date;
    type: 'work' | 'rest' | 'holiday' | 'leave' | 'workday-makeup';
    isWork: boolean;
}
```

---

## 附录

### A. 文件清单

| 文件路径 | 行数 | 说明 |
|----------|------|------|
| index.html | 158 | 个人博客主页 |
| tool/HTML_密码生成器/HTML_密码生成器.html | 356 | 密码生成器 |
| tool/HTML_工作日计算器/HTML_工作日计算器.html | 1780 | 工作日期计算器 |

### B. 依赖清单

| 依赖名称 | 版本 | CDN地址 | 加载页面 |
|----------|------|---------|----------|
| Tailwind CSS | 3.x | cdn.tailwindcss.com | 全部 |
| Noto Sans SC | - | fonts.googleapis.com | 全部 |
| QRCode.js | 1.5.1 | cdn.jsdelivr.net/npm/qrcode | 密码生成器 |

### C. 浏览器支持矩阵

| 功能 | Chrome 80 | Firefox 75 | Safari 13 | Edge 80 |
|------|-----------|------------|-----------|---------|
| CSS Grid | ✓ | ✓ | ✓ | ✓ |
| CSS Flexbox | ✓ | ✓ | ✓ | ✓ |
| CSS Variables | ✓ | ✓ | ✓ | ✓ |
| LocalStorage | ✓ | ✓ | ✓ | ✓ |
| Web Crypto API | ✓ | ✓ | ✓ | ✓ |
| Fetch API | ✓ | ✓ | ✓ | ✓ |
| async/await | ✓ | ✓ | ✓ | ✓ |
