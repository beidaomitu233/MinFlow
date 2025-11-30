"# MinFlow" 
这份文档将为你详细描述这款名为 **“MonoFlow (极简心流)”** 的高效学习工具。

设计核心理念：**“Less is More” (少即是多)**。通过极致的黑白视觉冲击，配合AI智能辅助，将用户的注意力强制聚焦在“当下”的任务上。

-----

# 软件设计文档：MonoFlow (极简心流)

## 1\. 项目概述 (Project Overview)

  * **产品名称**：MonoFlow
  * **定位**：极简主义AI效率工具 / 沉浸式学习伴侣
  * **核心价值**：利用AI自动化拆解任务，通过极简的视觉设计消除噪点，帮助用户快速进入“心流”状态。
  * **技术栈**：
      * **前端**：Vue 3 (Composition API) + Vite + TailwindCSS (用于极简样式) + Canvas/SVG (用于绘图)。
      * **后端**：Node.js (Express/Koa) - 处理API转发及Prompt工程。
      * **AI引擎**：智谱AI (GLM-4.5-flash) - 负责自然语言理解与任务拆解。

-----

## 2\. UI/UX 设计规范 (Design System)

整体风格追求“杂志级”排版，只有黑、白以及少量的高饱和度糖果色（用于点缀任务）。

### 2.1 配色方案 (Color Palette)

支持 **“Day (白昼)”** 和 **“Night (极夜)”** 两种模式，一键切换。

  * **极夜模式**：
      * 背景：`#000000` (纯黑)
      * 文字：`#FFFFFF` (纯白，透明度90%)
      * 装饰线/圆环：`#333333` (深灰)
  * **白昼模式**：
      * 背景：`#FFFFFF` (纯白)
      * 文字：`#000000` (纯黑，透明度90%)
      * 装饰线/圆环：`#E5E5E5` (浅灰)
  * **任务强调色 (Accent Colors)**：
      * 用于待办事项的小色块或文字高亮：`#FF6B6B` (珊瑚红), `#4ECDC4` (薄荷绿), `#FFE66D` (拿坡里黄)。

### 2.2 交互动效

  * **微交互**：点击完成任务时，任务条目不是简单的消失，而是被一条极细的线划掉，然后优雅地淡出。
  * **呼吸感**：计时器运行时，外圈圆环会有极轻微的“呼吸”缩放效果，模拟人的呼吸频率。

-----

## 3\. 功能模块详解 (Functional Specification)

### 模块一：AI 智能待办清单 (AI Smart Todo)

**页面布局**：
页面左侧（或移动端主屏）为清单列表。界面极其干净，没有多余的分割线，仅通过大量的留白（Whitespace）区分条目。

**功能逻辑**：

1.  **输入方式**：
      * **极简输入框**：用户直接打字。
      * **AI 解析入口**：一个明显的“✨”图标。点击后弹出一个大的文本域，用户粘贴一段文字（如会议记录、学习计划文本）。
2.  **AI 处理流程**：
      * 后端接收文本 -\> 调用智谱AI (`glm-4.5-flash`)。
      * **Prompt 策略**：要求AI提取动词开头的可执行事项，并根据任务性质分配一个HEX色值。
3.  **视觉呈现**：
      * 每个任务前有一个极小的圆形色块（AI分配的颜色）。
      * 任务文字采用大号、加粗的无衬线字体（如 Inter 或 Roboto）。

### 模块二：沉浸式圆环计时器 (Immersive Circle Timer)

**页面布局**：
点击某个待办事项后，界面平滑过渡（Hero Animation）到计时页面。此时，屏幕除中央圆环外，全部留白（或留黑）。

**功能逻辑**：

1.  **模式选择**：
      * 圆环下方有两个极简的文字按钮：“专注 (Focus)” / “倒数 (Timer)”。
      * **快捷时间**：点击屏幕出现浮层，提供 `30m` `60m` `90m` 胶囊按钮。
2.  **视觉核心 - “流动的圆”**：
      * **静态**：屏幕中央是一个纤细的圆环。
      * **动态**：
          * **倒计时**：圆环随着时间流逝，线条逐渐闭合或消退（使用 SVG `stroke-dasharray` 实现）。
          * **正计时**：圆环像雷达扫描一样，或者保持呼吸律动。
      * **数字显示**：圆环内部显示巨大的数字 `01:20`，字体极细或极粗（用户可选风格）。
3.  **样式切换**：
      * 提供一个小按钮切换圆环风格：
          * *Style A (极简)*：单线条。
          * *Style B (粒子)*：时间流逝时，圆环边缘散落微小的粒子。
          * *Style C (实心)*：实心圆饼随着时间类似月食般消退。

-----

## 4\. 技术实现方案 (Technical Implementation)

### 4.1 后端实现 (Node.js)

Node.js。我们需要使用 `https` 模块或 `axios` 来请求智谱 API，并支持流式输出（Server-Sent Events, SSE）以获得极快的响应速度。

**核心代码逻辑 (Controller/Service 层):**

```javascript
// backend/services/aiService.js
const axios = require('axios');

const API_KEY = "your-api-key"; // 实际开发请放在环境变量中

// 生成 JWT Token 的逻辑 (智谱AI需要自行封装或使用官方SDK，此处简化示意)
// 建议直接使用官方提供的 Node.js SDK: npm install @zhipuai/zhipuai-sdk-nodejs-v4

async function parseTasksFromText(userText, res) {
    const url = "https://open.bigmodel.cn/api/paas/v4/chat/completions";

    const payload = {
        model: "glm-4.5-flash",
        messages: [
            {
                role: "system",
                content: "你是一个任务拆解专家。请从用户的文本中提取具体的待办事项。返回格式必须是JSON数组，每项包含 'title' (任务名) 和 'color' (根据任务类型推荐一个莫兰迪色系的HEX颜色)。不要任何废话，只返回JSON。"
            },
            {
                role: "user",
                content: userText
            }
        ],
        stream: true, // 开启流式输出
        temperature: 0.7,
        thinking: { type: "disabled" } // 禁用思考模式
    };

    try {
        const response = await axios.post(url, payload, {
            headers: {
                'Authorization': `Bearer ${API_KEY}`, // 需按官方文档生成 Bearer Token
                'Content-Type': 'application/json'
            },
            responseType: 'stream'
        });

        // 将流直接管道转发给前端，或者在后端拼装好再一次性返回
        // 这里演示直接转发流，实现打字机效果
        response.data.pipe(res);

    } catch (error) {
        console.error("AI API Error:", error);
        res.status(500).send("AI processing failed");
    }
}

module.exports = { parseTasksFromText };
```



## 5\. 开发路线图 (Roadmap)

1.  **Phase 1: 原型与基础 (MVP)**
      * 搭建 Vue3 + Vite 框架。
      * 实现黑/白主题切换的基础 CSS 架构。
      * 完成基本的 SVG 圆环倒计时组件。
2.  **Phase 2: 接入 AI 大脑**
      * 搭建 Node.js 后端服务。
      * 对接智谱 AI 接口，编写 Prompt 调试，确保能准确从乱序文本中提取 List。
      * 前端实现“AI生成中”的 Loading 骨架屏动画。
3.  **Phase 3: 美学与细节**
      * 引入字体 (推荐 Google Fonts 的 'Inter' 或 'Space Mono')。
      * 调整 UI 留白比例。
      * 添加音效（可选）：完成任务时的轻微“叮”声，或白噪音背景。

-----
