---
hide:
  - navigation
  - toc
---

<style>
/* 隐藏自动生成的页面标题 */
.md-content__inner > h1:first-child {
  display: none;
}

/* Hero 区域 */
.lei-hero {
  display: flex;
  align-items: center;
  gap: 1.2rem;
  padding: 0.6rem 0 1.4rem 0;
  border-bottom: 1px solid var(--md-default-fg-color--lightest);
  margin-bottom: 1.6rem;
}
.lei-hero-bar {
  width: 4px;
  height: 3.4rem;
  background: var(--md-primary-fg-color);
  border-radius: 2px;
  flex-shrink: 0;
}
.lei-hero-title {
  font-size: 1.75rem;
  font-weight: 700;
  letter-spacing: 0.01em;
  color: var(--md-default-fg-color);
  line-height: 1.2;
}
.lei-hero-subtitle {
  font-size: 0.88rem;
  color: var(--md-default-fg-color--light);
  margin-top: 0.4rem;
  letter-spacing: 0.04em;
  white-space: nowrap;
}

/* 卡片：只做悬停效果，不加色条 */
.md-typeset .grid.cards > ul > li,
.md-typeset .grid.cards > ol > li {
  transition: transform 0.18s ease, box-shadow 0.18s ease;
  position: relative;
  overflow: hidden;
}

/* 悬停左侧细线（从无到有） */
.md-typeset .grid.cards > ul > li::after,
.md-typeset .grid.cards > ol > li::after {
  content: '';
  position: absolute;
  left: 0;
  top: 0;
  bottom: 0;
  width: 3px;
  background: var(--md-primary-fg-color);
  border-radius: 0;
  transform: scaleY(0);
  transition: transform 0.18s ease;
  transform-origin: center;
}

.md-typeset .grid.cards > ul > li:hover,
.md-typeset .grid.cards > ol > li:hover {
  transform: translateY(-3px);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.08) !important;
}

.md-typeset .grid.cards > ul > li:hover::after,
.md-typeset .grid.cards > ol > li:hover::after {
  transform: scaleY(1);
}
</style>

<div class="lei-hero">
  <div class="lei-hero-bar"></div>
  <div>
    <div class="lei-hero-title">Lei-Lab Knowledge Platform</div>
    <div class="lei-hero-subtitle">课题组知识共享平台</div>
  </div>
</div>

<div class="grid cards" markdown>

-   :material-book-open-variant:{ .lg .middle } **课程**

    ---

    生物统计学、深度学习等核心课程，配合可视化工具辅助理解。

    [:octicons-arrow-right-24: 进入课程](courses/index.md)

-   :material-tools:{ .lg .middle } **演示工具**

    ---

    交互式统计演示工具，直观感受统计原理。

    [:octicons-arrow-right-24: 查看工具](tools/index.md)

-   :material-code-braces:{ .lg .middle } **代码库**

    ---

    课题组 Python 代码库文档与 API 参考，包含 `histox` 完整文档。

    [:octicons-arrow-right-24: 浏览代码库](api/index.md)

-   :material-file-document-outline:{ .lg .middle } **文献抄读**

    ---

    课题组文献阅读记录与精读笔记。

    [:octicons-arrow-right-24: 查看文献](papers/index.md)

-   :material-pencil:{ .lg .middle } **知识博客**

    ---

    课题组成员的学习心得、技术分享与研究随笔。

    [:octicons-arrow-right-24: 阅读博客](blog/index.md)

-   :material-account-group:{ .lg .middle } **课题组**

    ---

    了解我们的团队成员与研究方向。

    [:octicons-arrow-right-24: 认识我们](team/index.md)

</div>