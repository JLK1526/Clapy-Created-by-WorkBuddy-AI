# 2026-11-07

- 生成单文件网页游戏 `index.html`（位于工作区根目录）：「拍手模拟器 / 回合制战斗游戏 三鑫特色」。
  - 招式：吸气(+1气，视为0级防御)、1~4级防御(耗 x-1 气)、1~5级攻击(耗 y 气)，5级无防御；攻击力 > 防御/攻击 即判死亡，同级攻击扣气续局，无同归于尽；无血条，只显示气数。
  - 节奏：人机在玩家出招后 50~100ms 出招（此前 500~800ms 需求被后续覆盖）；回合间隔 300~500ms；提示「已进入下一回合」；0气按攻击显示「气数不足」，按其他按钮清除提示。
  - 难度：低/中/高按**玩家胜率** 3/5、2/5、1/5 实现（人机 2/5、3/5、4/5），开局按概率定本场胜负归属，人机据此针对性应招。若要按字面把数值当人机胜率，改 `DIFF` 三行为 2/5、3/5、4/5。
  - 右上角透明圆角边框太阳/月亮主题切换（默认深色，localStorage 记忆）；结果弹窗后自动返回主界面，下方显示对战记录（localStorage 持久化，可清空）。
  - 自检：内联 JS 通过 `node --check`；20000 局模拟验证胜率 0.594/0.398/0.198，无负气、无死循环；ID 引用完整。
- 迭代：出招选择区改为二级菜单。一级仅 3 个两字关键字按钮「吸气 / 防御 / 攻击」（`.cat-btn`），点击展开对应 `.level2` 面板（吸气 1 项 / 防御 4 级 / 攻击 5 级），互斥展开（点其他分类自动收起），再次点同一分类收起；回合结束 `resetButtons()` 会收起全部面板。一级点击也清除「气数不足」提示。规则卡补充「操作」说明。已通过语法与结构自检。
- 迭代：对战界面操作区下方新增「本局回合记录」卡片，`state.log` 每回合结算后追加一条（回合数、双方招式、结算后气数、简短结果），正序显示、自动滚到最新、上限高度 200px 可滚动；致死回合标记为红色结果标签；新对局自动清空。与主界面的「对战记录」（整场结果，localStorage）相互独立。
- 迭代：UI 整体换肤为毛玻璃渐变风。浅色背景 linear-gradient(135deg, #A1C4FD → #C2E9FB)，深色 linear-gradient(135deg, #9795F0 → #FBC8D4)，均 `background-attachment: fixed`；卡片/按钮改为半透明 + backdrop-filter: blur(16px)；新增 --grad-1/2、--accent-rgb、--accent2-rgb、--danger-rgb、--warn-rgb、--overlay、--primary-text、--title-grad 变量，所有强调色 rgba 改为基于 rgb 变量。深色模式为深紫玻璃卡片 + 白字（主按钮文字深紫 #46246b 保证对比度）。游戏逻辑与 HTML 结构未动。已校验 CSS 括号配平、变量无缺失、JS 语法通过。
- 迭代：用户反馈深色模式太亮，把深色渐变在原紫粉色相上整体加深：--grad-1 #9795F0→#41368a，--grad-2 #FBC8D4→#94486d；卡片/按钮/边框透明度调低（更实的深紫玻璃），阴影和遮罩加深。浅色模式未动。
- 迭代：重写人机决策（保留开局 target 抽签精确控制整场胜率）。新策略 `moveScore`(攻击>吸气>防御、同级省气) + `pickMove`(排除无用防御、前两名随机) + `chooseAiMove`：能击杀就出最省气攻击；无胜招时玩家出攻击才允许防御保命（0气遇1级攻击出1级防御挡住，不再无脑吸气）；玩家非攻击回合绝不出防御；放水局仍按玩家招式应对但关键回合失误，超过 16 回合转认真防止无限对局。高难度 playerWin 由 1/5 改为 1/6.5（每 6~7 局赢一局），低/中不变；UI 文案同步。回归验证（直接提取 index.html 真实函数跑 5000 局/难度×2 种玩家）：理性玩家胜率 0.603/0.403/0.153，随机玩家 0.564/0.371/0.141；玩家非攻击时 AI 出防御 0%，攻击率 25~39%，最长回合 ≤31，无负气。
- 迭代：难度区文案精简。高难度按钮 small 与 `DIFF.high.text`（对战界面右上角徽章）改为「玩家胜率约 15%」；难度按钮下方的 hint 只保留「高难度大约每 6~7 局才能赢 1 局。」，删掉低/中难度胜率与人机策略说明。低/中按钮 small 保持 3/5、2/5。
- 迭代：三个难度按钮 small 全部改百分比（60% / 40% / 约 15%），`DIFF.*.text` 同步；难度选择下方说明改为动态区 `#diffHint`（`renderDiffHint()`），点击难度按钮即切换，初始化时渲染默认中难度。每个难度在 `DIFF` 里配 `head`（难度+双方胜率）、`tip`（几局能赢一局）、`desc`（3 条要点：人机打法 / 风险提示 / 建议），新增 `.hint-head/.hint-tip/.hint-line` 样式并用 fade 重播动画。已校验三档数据齐全、JS 语法通过。
- 迭代：右上角新增语言切换按钮。`.corner-controls` 包裹 `.lang-toggle` + `.theme-toggle`，共用 `.corner-btn`（透明背景 + 圆角框），按钮文字显示当前语言「中文 / MC」，`clap-lang` 记忆，`html[data-lang]` 供 CSS 微调（MC 长文案字号/换行）。MC 只改难度按钮与对局招式名：难度「末地(简单)/筛界(普通)/下界(困难)」，攻击统一「横扫之刃」，1~2 防御「伤害吸收」，3~4 防御「抗性提升」，吸气「使用附魔之瓶(吸气)」；一级分类按钮仍保持「吸气/防御/攻击」（遵守此前「只保留两字关键字」要求），面板标题 `<b>` 按语言切换。MC 同名招式靠 small「N 级 · 耗 N 气」区分，cost/level/判定完全不变。
  - 架构：`I18N` + `language` + `moveLabel()/moveSub()`，`MOVES` 的 `label`/`sub` 改为 getter 动态取值；`DIFF` 去掉 label 改用 `diffLabel(key)`，`head` 用 `{D}` 占位；回合记录与历史记录只存 `type:level`（`moveKey/moveFromKey`），旧记录（pLabel/aLabel/detail/中文难度名）保留回退兼容。`applyLanguage()` 统一重渲染难度按钮、二级按钮、面板标题、出招框、回合记录、历史记录、难度徽章、结算弹窗（`lastResult`）、状态栏（`state.detailMoves`）。二级按钮改事件委托绑在 `#actions`，切语言重建 innerHTML 不丢事件。
  - 自检：`vm.Script` 语法通过；41 个 id / 34 个 `$()` 引用无缺失；CSS 括号 159/159；写轻量 DOM 桩实际执行内联脚本，48 项运行时检查全过（含对局中切语言、旧记录兼容、判定回归），胜率模拟 mid=0.371 / high=0.151；Edge headless 截图确认中文与 MC 攻击/防御/吸气面板排版正常、长文案不溢出。
  - 修复：事件委托里误写 `b.dataset.move`（应为 `btn`），静态语法检查查不出，DOM 桩模拟点击时暴露并修复。教训：凡把逐按钮绑定改成事件委托，必须做一次运行时点击验证。

# 2026-11-14

- 迭代：`index.html` 增加第三种语言 English，仍用右上角同一个按钮，循环顺序 中文 → MC → English → 中文；按钮文字显示当前语言名，切到英文时显示 `English`；`clap-lang` 记忆三态，`html[data-lang]` 标记语言，`document.lang` 与 `<title>` 同步切换。英文标题按用户指定为 `Claply Game`，副标题 `Turn-Based Battle Game / Sanxin Special`。
- 架构：把原先只有 name/diff/panel 的 `I18N` 重构为 `DICT`（zh 基准 111 键 / mc 仅 12 个专属键 / en 全量 111 键）+ `T(key)`（当前语言缺键自动回退 zh，保证 MC 除专属词外完全沿用中文）+ `TT(key, vars)` / `fmt()`（`{K}` 占位符替换）。**关键设计：MC 不改任何通用文案，因此中文与 MC 两套既有行为零改动。** 难度 `DIFF` 去掉硬编码 text/head/tip/desc，改存 `headKey/tipKey/descKeys` 指向字典。
- 全界面语言化：给 HTML 静态文本元素补 id（heroTitle / subtitleText / brandText / diffTitle / diffSmall* / rulesTitle / rule1~6 / recordsTitle / playerTag / aiTag / qiCaption / cat* / panel*Name / panel*Note / logTitle / resultMark / resultTip 等），由 `applyStaticText()` 统一刷新；规则 6 行用字典里的 HTML 片段替换 innerHTML（保留 tag/b 结构）。动态部分（describe / briefResult / renderRecords / renderRoundLog / renderDiffHint / renderMove / render / status / warning / toast / resultTitle+resultDetail / diffBadge / roundLabel / logCount）全部改走 T()/TT()。新增 `setStatus(key, vars)` 记录 `state.statusKey/statusVars`，`lastToastKey`/`lastWarningKey`/`lastResult`，使对局中、结算弹窗弹出时切语言也能即时重算。招式 label/sub 仍是 getter，英文为 Breathe / Guard Lv.N / Attack Lv.N / Cost N Qi / +1 Qi。
- 兼容：历史记录仍只存 `type:level` 与难度 key；更早版本存的中文文字记录，在英文模式下用 `legacyMoveLabel()` 反查中文招式名（失败回退 `Unknown move`）并把自由文本 detail 换成英文提示，避免英文界面因旧数据冒出汉字。
- CSS：新增 `html[data-lang="en"]` 下 cat-btn / act-btn / primary / fighter-tag / result-mark 的字号与字距收紧，@media(<560px) 再缩一级，英文单词不溢出。
- 自检：vm 语法通过、33 处 `$()` 引用 id 全部存在、CSS 括号配平、en 字典覆盖 zh 全部 111 键且值不含汉字；DOM 桩实际执行内联脚本，80+ 项全过（三态循环、中文/MC 原样、英文零汉字、对局中切换、气数不足提示、结算弹窗重算、旧记录兼容、判定回归），胜率模拟 mid=0.387 / high=0.162；Edge headless `--dump-dom` + TreeWalker 扫描真实 DOM 文本节点：zh/mc 按预期含中文、en 主界面与 en 对界界面（含展开招式面板）均为 `CLEAN`；截图确认英文主界面/对局/492px 小屏排版正常（420px 那张右侧被裁是无头浏览器最小视口 492px 造成的截图假象，`scrollWidth === innerWidth` 无横向溢出）。
- 环境备注：本机 node 版本目录已由 `22.22.2-3` 变为 `22.22.2-6`，直接用 PATH 里的 `node` 命令更稳妥。
- 迭代：生成站点图标并接入。根目录新增同一图标的三种格式：`favicon.svg`（矢量源，512 viewBox）、`favicon.png`（256×256）、`favicon.ico`（16/32/48/64/256 五帧 PNG）。`index.html` head 增加 3 个 `rel="icon"` + `apple-touch-icon`；`.hero` 内标题上方插入 `.hero-logo` `<img>`（引用 favicon.svg），带 `@keyframes logoIn` 入场动画、深浅色不同阴影、@media(<560px) 缩到 66px。页面内也可见图标，不依赖浏览器是否加载标签页 favicon。
- 图标设计：深紫 `#6a4fd0/#41368a` → 玫粉 `#94486d` 渐变圆角方底（呼应主题），两只白色手掌相对（掌面 + 4 指 + 拇指 + 半透明紫指缝），中间金色 `#ffd9a0` 四芒星 + 两点火花。小于 64px 用简化构图（两个圆角块 + 放大金点）保证 16px 可辨。
- 生成方式（本机无 Pillow/sharp）：Node 生成临时 HTML，在无头 Edge 里用 Canvas 2D 按 512 基准坐标绘制并 `toDataURL()` 导出各尺寸 PNG，再手写 ICO 容器（ICONDIR + 5 个 ICONDIRENTRY + PNG 帧）组装 `favicon.ico`，零第三方依赖；临时脚本与截图用后即删。
- 踩坑：① SVG 的 linearGradient 默认 objectBoundingBox，导致每个手掌 rect 各自渐变、指根出现半圆形色块，改为 `gradientUnits="userSpaceOnUse"` 全局坐标后与 Canvas 版一致；② 中心最初用三条汇聚放射线，看起来像「向下箭头」，改成四芒星路径。
- 自检：13 项静态检查全过（内联 JS 语法、188 个 id、CSS 括号配平、本地资源引用、PNG/ICO/SVG 文件头）；Edge `--dump-dom` 探测 `loaded=1|natural=512|icons=3` 证明 SVG 真实加载；浏览器直接打开 ICO 可解码；截图确认深色/浅色/492px 小屏排版正常，图标不压标题与右上角按钮。
