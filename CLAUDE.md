# CLAUDE.md

历史图谱项目。在共享时间轴上对比多个文明的演变,并用分叉连线表达分裂/合并的谱系关系。数据本质是**时间轴上的 DAG**,不是线性 timeline——任何改动都要尊重这个模型。

## 运行

无构建。浏览器打开 `history-graph.html` 即可。如需改为独立数据文件加载,必须配合本地服务器(`python3 -m http.server`),因为 `file://` 禁止 fetch。

## 架构

全部在单个 `history-graph.html`:

- 数据:`<script type="application/yaml">` 块,js-yaml 解析。
- 渲染:D3 v7,分层 `g`:`gLaneBg`(泳道底) → `gGrid` → `gLink`(连线) → `gEntity`(横条) → `gAxis` → `gCross`(参考线)。层级顺序即 z 序,勿随意调换。
- 入口流程:解析 YAML → 行分配 → 计算布局常量 → `render()`。`resize()` 与 zoom 回调都只重算 x 标尺后调 `render()`。

## 数据契约(改数据只动 YAML 块)

entity:`{ id, name, track, start, end, successors? }`。

- `id` 唯一,英文,被 `successors` 引用。
- `start`/`end` 为整数年,**负数 = 公元前**。
- 关系全部由 `successors` 表达:多个 = 分裂;多个 entity 指向同一 id = 合并。
- 新增文明:在 `tracks` 加一条,entity 的 `track` 指过去,泳道自动生成。
- 加 entity 后务必确认 `successors` 引用的 id 真实存在(代码会 `console.warn` 缺失项)。

## 不变量(勿破坏)

- **行分配**:`trackRows` 用贪心算法把同泳道内时间重叠的 entity 排到不同行。改布局时不能让同期并立政权(三国、东西罗马)重叠。
- **连线**:从父条右端中点连到子条左端中点,`d3.linkHorizontal` 贝塞尔。分裂(父有多个 successors)的线加 `.is-split` 加粗。
- **缩放仅作用于 x(时间)**。`zoom` 回调用 `rescaleX` 重算时间标尺;y(泳道/行)固定。不要引入 y 方向缩放。

## 禁忌

- **不引入构建工具 / 包管理 / 框架**,除非明确要求。本项目核心卖点是双击即开、零依赖。
- **不使用 localStorage / sessionStorage**。
- **不把年份当字符串处理**,始终是 number,负数表 BC。
- 现存的"历史继承链"(`successors`)与未来的"现代国家映射"(`modern`)是两种不同关系,**不要混入同一字段**。

## 风格

- 当前为 vanilla JS,保持无依赖(除 D3 / js-yaml CDN)。
- 若未来迁移工程化:用 TypeScript,先为 entity/track 定义 type 与 schema 校验,再动渲染。
- 中文环境:UI 文案中文,标识符 / 代码注释可中英混用;中文显示依赖 Noto Serif SC,改字体需保留中文 fallback。
