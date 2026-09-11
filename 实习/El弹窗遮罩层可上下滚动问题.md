背景：第一个section是字段展示，第二个section是表格展示

el-dialog的DOM结构：
1. **底层 `v-modal`**：它只是一个单纯的、铺满全屏的**半透明暗色背景层**（纯视觉效果，负责把底下的页面变暗）。
2. **中间 `el-dialog__wrapper`**：它是一个**铺满全屏、透明的实际容器**。
3. **顶层 `el-dialog`**：它是放在 `wrapper` 里的**白色卡片子元素**。

出现可上下滚动问题的原因：
1. **Element UI 默认给这个 wrapper 加了 `overflow: auto`**，当弹窗自带的margin-top高度+表格高度超过屏幕高度，这个全屏容器就会**产生滚动条**。
2. 弹窗外面的空白区域滚动鼠标时，鼠标事件触发在 `.el-dialog__wrapper` 上，滚轮触发了wrapper容器的滚动，白色弹窗卡片作为它内部的一个子元素，自然就跟着被推着滑动。

解决方案：
1. **把外层全屏容器的滚动锁死（`overflow: hidden`）**
2. **弹窗居中钉死，只让表格内部的数据行带滚动条，表头与外框永远纹丝不动。**

---

## 限制弹窗内部滚动（博客笔记）

> 来源：http://120.77.152.123:8088/posts/el%e5%bc%b9%e7%aa%97%e5%ae%b9%e5%99%a8debug ｜ 原发布日期：2026-08-13

实现“弹窗整体高度固定，内容过多时在弹窗内部滚动”的效果，**最简单有效的做法是：把限制高度和滚动的 CSS 样式加在弹窗内部的内容容器上。**

如`<el-row>`是实际显示内容的容器，外层包上一层`div` ,内联样式加上`max-height: 60vh; overflow-y: auto;`

---

## el-dialog 撑开全屏问题（博客笔记）

> 来源：http://120.77.152.123:8088/posts/el%e5%bc%b9%e7%aa%97%e7%bb%84%e4%bb%b6%e9%97%ae%e9%a2%98%e4%bf%ae%e5%a4%8d ｜ 原发布日期：2026-08-11

写组件过程中遇到el-dialog组件内容过大时会撑开到全屏，其滚动条几乎和浏览器界面滚动条重合，从而在滚动浏览器滚动条的同时，会出现提示滚动el-dialog的情况。

但我想要其弹窗是居中固定的，内容只在弹窗内部滚动：

做法：

1. 给弹窗内容加一个最大高度max-height，这样高度过大就不撑大到全屏
2. 超出部分：overflow-y:auto，让超出最大高度部分内容在弹窗内滚动。

---

## 三个滚动条排错（博客笔记）

> 来源：http://120.77.152.123:8088/posts/%e4%b8%89%e4%b8%aa%e6%bb%9a%e5%8a%a8%e6%9d%a1%e7%9a%84%e9%97%ae%e9%a2%98%e6%8e%92%e9%94%99 ｜ 原发布日期：2026-08-13

### Element UI 弹窗多重滚动条排错总结

在开发 Vue + Element UI 项目时，经常会在带有大量表单字段的弹窗（`el-dialog`）中遇到**同时出现多个滚动条（如浏览器全局垂直滚动条、弹窗内部垂直滚动条、内部横向滚动条）**的样式问题。本文档总结了该现象的产生原因及标准解决思路。

#### 1. 现象描述

在使用 `<el-dialog>` 嵌套一个表单组件（如高度超过屏幕的大量字段）并给内部容器添加了 `max-height` 和 `overflow-y: auto` 后，页面上同时出现了三个滚动条：

1. **全局垂直滚动条**：属于浏览器窗口或底层 `el-dialog__wrapper` 的滚动条。
2. **横向滚动条**：出现在表单容器底部的水平滚动条。
3. **内部垂直滚动条**：预期中加在内部内容容器上的垂直滚动条。

---

#### 2. 根因分析与相关知识点

##### 2.1 全局垂直滚动条

> [!NOTE] **知识点：** `el-dialog` 默认拥有 `margin-top: 15vh`（距离视口顶部 15% 的高度）。`el-dialog__wrapper` 默认是 `position: fixed` 占满全屏且 `overflow: auto`。

**产生原因**： 当弹窗内部通过 `max-height: 60vh` 限制了表单区域高度，但弹窗整体高度还包括了**弹窗 Header、Footer、额外提示文本（rp-tip）**等。 如果：`15vh (margin-top) + Header高度 + 60vh (内部最大高度) + Footer高度 > 100vh`。 整个弹窗的高度就超过了当前屏幕的可用视口高度，从而触发了外层 `el-dialog__wrapper` 或 `body` 的全局滚动条。

##### 2.2 横向滚动条

> [!NOTE] **知识点：** Element UI 的栅格系统（`el-row` / `el-col`）中，为了使列之间的间距（`gutter`）对齐，`el-row` 会自动在左右两侧添加**负外边距（Negative Margin）**（例如 `margin-left: -10px; margin-right: -10px;`）。

**产生原因**： 当你使用一个普通的 `<div>` 包裹了一层包含栅格的表单，且该 `<div>` 宽度为 100% 时，内部 `el-row` 的负边距会将内容撑出 `<div>` 的实际宽度。如果没有限制横向溢出，浏览器就会渲染出水平滚动条。

##### 2.3 内部垂直滚动条

这是由于开发者手动添加 `overflow-y: auto; max-height: 60vh;` 产生，属于**预期内的正确滚动条**。

---

#### 3. 关键排错点与修改方案

解决此类问题的核心思路是：**消除不必要的溢出，保留预期的内部滚动。**

##### 修改步骤 1：压降整体弹窗高度，消除全局滚动条

针对 `el-dialog` 的默认下压空间过大，导致内容超出屏幕的问题，可以直接修改 `top` 属性，缩减顶部留白。

```diff
  <el-dialog
    title="新增检查记录"
    :visible.sync="visible"
-   width="560px"
+   width="680px"
+   top="5vh"
    append-to-body
  >
```

*提示：如果表单变为双列布局（如 `el-col :span="12"`），适当增加 `width`（如 `680px`）可以防止横向过于拥挤。*

##### 修改步骤 2：处理栅格负边距，消除横向滚动条

针对包裹表单的 `div` 容器，强制隐藏 X 轴溢出，并补偿一定的右侧内边距，防止出现的右侧垂直滚动条遮挡表单控件。

```diff
-   <div style="max-height: 60vh; overflow-y: auto;">
+   <div style="max-height: 60vh; overflow-y: auto; overflow-x: hidden; padding-right: 10px;">
      <EastForm ... />
    </div>
```

---

#### 4. 最佳实践总结

> [!TIP] 以后在 Element UI 中开发“长表单弹窗”时，建议形成以下肌肉记忆模板：

```html
<el-dialog
  title="标题"
  :visible.sync="visible"
  width="650px"
  top="5vh" <!-- 【关键】减少顶部留白避免超高 -->
>
  <!-- 【关键】独立的内容滚动层，禁用横向溢出，并给右侧留出滚动条呼吸空间 -->
  <div style="max-height: 65vh; overflow-y: auto; overflow-x: hidden; padding-right: 10px;">
    <el-form>
      <el-row :gutter="20">
        <el-col :span="12">...</el-col>
      </el-row>
    </el-form>
  </div>

  <div slot="footer">
    <el-button>取消</el-button>
    <el-button type="primary">保存</el-button>
  </div>
</el-dialog>
```
