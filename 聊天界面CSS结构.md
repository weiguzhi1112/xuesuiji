# 雪穗机 · 聊天界面 CSS 结构说明（给 AI 读，用于改写自定义 CSS）

自定义 CSS 写在「外观 → 自定义 CSS」，会作为最后一个 <style id="ccss"> 注入，覆盖默认样式。
用法：把本说明和你想要的效果一起交给 AI，让它只输出一段可直接粘贴的 CSS。

## 0. 通用规则
- 整个手机是 `#phone`（390 × 844 逻辑像素，会被整体缩放）。微信在 `#wx` 里，聊天页是 `#vk`。
- 所有聊天样式都以 `#vk` 开头写，避免影响短信（`#sm`）等其他页面；短信的结构几乎一样（`#sm`）。
- 默认样式里有不少 `!important`（顶栏按钮背景、头像等），自定义时需要用更高优先级：`#phone #vk .xxx{...!important}`。
- 颜色请优先改 CSS 变量，而不是逐个选择器改；深色模式由 `:root[data-theme="dark"]` 切换变量，写自定义色时请同时考虑两种模式。
- 不要改 `position / inset / transform / z-index`（会破坏滚动与键盘适配）。

## 1. CSS 变量（改这些最省事）
写在 `:root{}` 或 `#phone{}` 里：

| 变量 | 作用 |
|---|---|
| `--bub-me` / `--bub-me-t` | 我发的气泡背景 / 文字色 |
| `--bub-ot` / `--bub-ot-t` | 对方气泡背景 / 文字色 |
| `--c-text` | 次要文字（时间戳、已读、昵称） |
| `--fade` | 顶部/底部渐隐遮罩的 RGB（写法 `250,250,250`，无 rgb()） |
| `--tailm` | 气泡小尾巴的 SVG mask（一般不要动） |
| `--app` | 页面背景 |
| `--card` / `--soft` | 卡片 / 浅灰填充（输入框、按钮底） |
| `--ink` | 主文字色 |
| `--wc` | 毛玻璃白底的 RGB（`255,255,255`；深色为 `44,44,46`） |
| `--hk` | 高光强度 0–1（深色为 .2） |
| `--bfs` | 气泡字号（由「聊天气泡」设置写入，默认 16px） |
| `--brd` | 气泡圆角（默认 18px） |
| `--bps` | 气泡内边距倍率（默认 1） |
| `--bmw` | 气泡最大宽度 |
| `--kg` | 键盘弹出时输入栏距底部间距（px 数字） |

## 2. DOM 结构（聊天页 `#vk`）
```
#vk                                   聊天页根（绝对定位铺满，::before/::after 是上下渐隐遮罩）
├─ .hd26                              顶栏容器（::before 是模糊遮罩带）
│  ├─ button.gz.bk[data-a=home]       返回按钮（里面有 svg.ico，可含 i.bn 未读角标）
│  ├─ .cn[data-a=cmore]               中间：头像 + 名字
│  │   ├─ .av                         头像（56px）
│  │   └─ .np.gz > b + svg            名字胶囊（b 是昵称）
│  └─ button.gz.rt[data-a=cmore]      右上角「更多」
├─ .ms#ml                             消息滚动区（padding-top 避开顶栏、padding-bottom 避开输入栏）
│  ├─ button.sy.mlmore                「查看更早的消息」
│  ├─ .sy.tm                          时间分隔（内含 small“iMessage” + 时间文字）
│  ├─ .sy[data-i]                     系统提示（居中小字）
│  ├─ .sy.pat                         拍一拍提示
│  └─ .mr[.me][.gf][.ge][.hq][.off]   一行消息（见下）
└─ .ib#ib                             底部输入区
   ├─ .pmn.gz#pmn                     「+」弹出的功能面板（按钮 button[data-a=pi]，含 svg + span）
   ├─ (表情面板 emoPanel)
   ├─ .qbar#qbar[.on]                 「回复 xxx」引用条（.on 时显示）
   └─ .ibr                            输入行
      ├─ button.gz.pl                 「+」按钮
      └─ .tw.gz                       输入框外壳（圆角胶囊）
         ├─ textarea#mi               输入框
         ├─ button.emb                表情按钮
         └─ button.sd                 发送按钮（有内容时才显示）
```

### 2.1 一行消息 `.mr`
```
.mr  (+ .me 我发的 / 无 .me 对方发的)
     (+ .gf 一组连发的第一条 / .ge 一组的最后一条 → 只有 .ge 显示气泡小尾巴)
     (+ .hq 带引用 / .off 线下长文)
└─ .mc                                消息列（相对定位，引用胶囊绝对定位在它上面）
   ├─ .qp.qm | .qp.qo                 引用胶囊：.qm 引用的是“我”的话，.qo 引用的是“对方”的话
   ├─ .nl                             对方昵称（只在一组第一条且对方发时出现）
   ├─ .bb[.gf][.ge]                   气泡本体
   │    修饰类：.sk 表情包  .im 图片  .tfx 转账/位置/红包(.hbk 红包)  .vo 语音  .ca 通话记录  .offb 线下剧情  .old 旧消息  .typ 对方正在输入
   │    内含：文字 / img / .rx 表情回应(i)
   ├─ (.xs 额外卡片，如工具调用结果)
   └─ .dl                             “已送达 / 已读”（只在我最后一条消息下面）
```
- 我的气泡：`.mr.me .bb`；对方气泡：`.mr:not(.me) .bb`。
- 小尾巴：`.bb.ge::before`（对方在左下，`.mr.me .bb.ge::before` 在右下），用 `background` 取气泡色、`mask: var(--tailm)` 裁形状。改气泡颜色时尾巴会自动跟随变量；如果直接给 `.bb` 写 background，要同时给 `::before` 写同色。
- 正在输入：`.bb.typ > i × 3`（三个小圆点，带动画）。

### 2.2 引用（回复某条消息）
- 引用胶囊 `.qp` 是**绝对定位**在 `.mc` 上方的小气泡（高 44px，字号 10.5px，宽 80–112px，用 SVG mask 做出带小尾的胶囊形），右侧/左侧各有一条曲线连接线 `.mc:has(>.qp)::after`。
- 带引用的 `.mr.hq` 会在上方留出 58px 的 margin-top 给胶囊。
- 想改引用颜色：`.qp.qm{background:…;color:…}`、`.qp.qo{…}`；想改大小：改 `.qp` 的 `height / min-width / max-width / font-size / padding`（保持 `width:max-content`）。
- 输入栏上方的「回复」条是 `.qbar`（pill 形，`span` 是文字，`button` 是关闭）。

### 2.3 特殊消息的内部类（需要时再改）
- 语音：`.bb.vo`（内含波形和秒数）；通话：`.bb.ca`；转账/红包/位置：`.bb.tfx`。
- 图片：`.bb.im img`；表情包：`.bb.sk img.stk`；截图：`img.shot`。
- 线下剧情：`.mr.off .bb.offb`（卡片样式，`.oh` 头部、`.ot` 正文、`.oa` 旁白）。

## 3. 顶栏样式的两种模式
- 默认「模糊遮罩」：`.hd26::before` 是渐变 + `backdrop-filter` 模糊带，按钮和名字胶囊无背景。
- 「悬浮按钮」（外观里切换）：`#phone.hdfl` 生效，按钮/名字胶囊恢复毛玻璃圆形（`.gz` 的样式）。
- 自定义时可以判断 `#phone.hdfl #vk .hd26 …` 区分两种模式。

## 4. 输入栏要点
- `.ib` 的 `padding-bottom` 受 `--kg` 控制，键盘弹出时 `#phone.kbup .ib` 会覆盖，不要写死。
- 输入框外壳 `.tw.gz`：`border-radius:21px`；textarea `#mi` 自适应高度（rows=1，JS 控制高度）。
- 发送按钮 `.sd` 默认隐藏（`transform/opacity`），有内容时显示；不要改它的 transform。

## 5. 示例（可直接粘贴）
```css
/* 我的气泡渐变，对方气泡带描边 */
#vk .mr.me .bb{background:linear-gradient(135deg,#6a8dff,#8f6aff);color:#fff}
#vk .mr.me .bb.ge::before{background:#8f6aff}
#vk .mr:not(.me) .bb{box-shadow:0 0 0 1px rgba(0,0,0,.08)}

/* 气泡更圆、字更小 */
#phone{--brd:22px;--bfs:15px}

/* 顶栏名字胶囊换成彩色 */
#phone #vk .hd26 .np{background:#ffe3ec!important;color:#c2185b!important;border-radius:15px!important}

/* 输入框外壳改成方圆角 */
#vk .tw{border-radius:14px}
```

## 6. 给 AI 的提示词模板
「下面是我的聊天界面 CSS 结构说明。请根据它，只输出一段可粘贴的 CSS（不要解释），实现：<你想要的效果>。要求：选择器以 `#vk` 开头；深色模式也要好看；不改定位和 transform。」
