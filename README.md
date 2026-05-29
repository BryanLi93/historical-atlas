# 历史图谱 · History Lineage

在同一条时间轴上对比不同文明的演变脉络,并显式表达**分裂**(一国裂为多国)与**合并**(多国归一)的谱系关系。

普通 timeline 是线性的(秦→汉→唐),但历史的真实结构是**时间轴上的有向无环图 (DAG)**:罗马帝国分裂为东西罗马、汉末分裂为三国、法兰克三分为后来的法德意雏形。本项目以此为核心建模目标。

## 特性

- **共享时间轴 + 多泳道 (swimlane)**:多个文明并排,纵向切一刀即可对比同期状态(秦汉时罗马在干什么)。
- **分裂 / 合并可视化**:用分叉连线表达谱系,而非孤立的横条。
- **并存政权自动错行**:同期并立的政权(三国、东西罗马)由贪心算法分配到不同行,不重叠。
- **零构建**:单个 HTML 文件,双击即开。无需 Node、打包器或本地服务器。
- **数据与渲染解耦**:维护者只改一段 YAML,不碰渲染代码。

## 快速开始

直接双击 `history-graph.html`。需联网加载 CDN 资源(D3、js-yaml、字体)。

交互:滚轮缩放时间轴 · 拖拽平移 · 悬停看详情 · 鼠标移动时竖直参考线显示当前年份。

## 数据模型

数据内嵌在 HTML 的 `<script type="application/yaml">` 块中。两类对象:

**tracks**(泳道,每个文明一条):

```yaml
- id: china
  name: 中国
  name_en: China
  color: "#a8341f"
  bg: "#e8cfc4"
```

**entities**(朝代 / 政权):

```yaml
- { id: han, name: 汉, track: china, start: -206, end: 220, successors: [wei, shu, wu] }
```

| 字段 | 说明 |
| --- | --- |
| `id` | 唯一标识(英文),被 `successors` 引用 |
| `name` | 显示名 |
| `track` | 所属泳道 id |
| `start` / `end` | 年份,**负数 = 公元前** |
| `successors` | 演变去向(可省略)。多个 = 分裂;多个 entity 指向同一 id = 合并 |

关系语义全部由 `successors` 一个字段承载:

- 更替:`秦.successors = [han]`
- 分裂:`汉.successors = [wei, shu, wu]`
- 合并:`魏.successors = [jin]` 且 `吴.successors = [jin]`

## 项目结构

```
history-graph.html    # 全部:渲染逻辑 + 内嵌 YAML 数据
README.md
CLAUDE.md             # Claude Code 工作约定
```

## 技术栈与设计取舍

- **D3 v7** 渲染,**js-yaml** 解析,均走 CDN。
- **为什么零构建**:MVP 阶段双击可开、易分享的价值,高于类型安全和工程化。代价是放弃了 TypeScript 体验——schema 稳定后再迁移到 Vite + TS 不迟。
- **为什么 YAML 内嵌而非独立文件**:浏览器 `file://` 协议禁止 fetch 本地文件 (CORS)。内嵌让"双击即开"和"手写 YAML 维护"两个诉求同时成立。拆分为独立 `data.yaml` 需配合本地服务器。

## Roadmap

- [ ] 数据拆分为独立 `data.yaml` + fetch 加载(需 local server)
- [ ] 现代映射:`modern: [...]` 字段标注古国对应今日哪些国家(奥匈帝国 → 奥地利 / 匈牙利 / 捷克 …),与历史继承链分层展示
- [ ] 年份校准(当前为简化值,历史分期本有争议)
- [ ] 多文明扩展(中亚、印度、伊斯兰世界)
- [ ] 工程化迁移(Vite + TypeScript),引入 schema 校验

## License

待定。
