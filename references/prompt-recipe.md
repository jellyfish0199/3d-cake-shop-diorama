# Prompt recipe — reproduce or iterate the cake shop

## One-shot master prompt (build from scratch)

> 用 Three.js 给我做一个模块化的甜品店立体模型网页游戏，主题叫
> **Wonder Half Sugar Dreamland（四季甜点铺）**。
>
> 场景：一座 18×18 的悬浮底座，中间是粉色奶油风的甜品店（圆角建筑、草莓/马卡龙/甜甜圈屋顶装饰、
> 红白格纹遮阳篷、玻璃圆窗、粉色大门），屋顶上有旋转小丑帽水母摆件、戴伞的龙猫；
> 店外有遮阳伞下的藤编桌椅、冰淇淋车、巨型草莓奶油蛋糕、路灯、菜单板；全部用柔和卡通渲染。
>
> 玩法：玩家是像素小人（霜糖，四方向行走动画 6 帧），出生在**左侧藤椅正前方**
> (−0.25, 0.05, 4.95)，面向镜头；WASD/方向键/触屏摇杆走到大门口即可进店，
> 靠近柜台出现店员"薄荷喵呜"，点单对话：
> 「欢迎光临Wonder Half Sugar Dreamland！今天您要点什么甜品？」
> 三个甜品按钮（焦糖布丁/蜜糖小圆饼/薄荷糖霜冻），选完打包，
> 页面会话只允许完成一次点单，打包后玩家回到椅子前出生点。
>
> 相机：默认店外斜视角（约 position (14.8, 15.4, 29.6) 看向 (−0.5, 2, −0.4)），
> 手动 OrbitControls（旋转/缩放/平移，无自动旋转无阻尼），另有店内视角。
>
> 环境与时光面板（右上、粉色半透明圆角）：白天日光 / 黑夜静谧切换 +
> 春夏秋冬四季（春樱花瓣、夏烈日、秋枫叶、冬雪 380 片+屋顶地面积雪）。
> 夜晚：深蓝天空、星星、月亮，店内暖黄灯光。太阳和月亮**固定在世界坐标
> (−14, 9, −12)**（屋顶后上方），只做面向相机的公告牌旋转，大小固定
> （太阳 1.0 / 月亮 1.05），禁止按屏幕位置追随视角。
>
> 右侧竖排三个独立工具按钮：音乐开关（Jelly Sea Dreams，尝试自动播放，
> 首次交互兜底，记忆偏好）、门/店内视角切换、恢复视角。与面板顶部对齐，
> 移动端和桌面一致。页面标题用手写体 Allura 单行
> 「Wonder Half Sugar Dreamland」。
>
> 加载页：天空蓝渐变底 + 戴礼帽的小水母贴图（宽 clamp(38px, 11.5vw, 50px)）
> 轻轻漂浮、不可拖拽不可交互，场景与贴图全部就绪后淡出移除（10 秒兜底）。
>
> 工程要求：原生 ES 模块 + 本地 vendored Three.js，无打包器，所有资源相对路径；
> 按关注点拆分模块（shop/props/interior/pixel-world/navigation/season/ui/music/main）；
> html/body 锁 100dvh，canvas 和 UI 层钉在 visualViewport 上，
> 防手机地址栏收起时整页上移；像素图用 nearest 采样的直立 Mesh 平面；
> 点单状态机 idle→choosing→packed→completed；无反向描边轮廓。

## Per-feature iteration prompts (how the user actually drove it)

| Want | Prompt pattern |
| --- | --- |
| Move spawn | 「人物初始出生位置改到 XX 的前面，不要被遮挡」→ 改 `pixel-world.js` 初始位 + `main.js` 结账归位点，两处必须同步 |
| Celestial behavior | 「太阳/月亮任何时候都要在屋顶上方/固定在世界坐标」→ 改 `season.js` 的 `frameSky`，同步更新 `main.js` 调用签名 |
| Panel/tools layout | 「XX 按钮单独竖排放右侧 / 面板和按钮对齐 / 别挡住建模」→ 改 `index.html` DOM + `style.css` 媒体查询，桌面与手机分开校验 |
| Loading veil | 「加一个和主体配色一致的加载页，贴图 XX 漂浮、不可拖拽」→ `#loader` 结构 + `.loader-jelly` 尺寸/动画 + 帧就绪后移除 |
| Mobile shift bug | 「手机进去整页上移」→ 锁 `100dvh` + `position:fixed` body + visualViewport 驱动的 `--vv-*` 变量 |
| Scope guard | Every prompt ends with 「其他我没叫你改的不要改」 |

## Verification prompt (after any change)

> 改完跑：`node build-static.mjs`，起本地静态服务，用无头 Chrome 分别在
> 1280×900 / 390×844 / 844×390 截图，确认：本次改动生效、面板与工具列顶边
> 对齐、出生点无遮挡、无 console error、无资源加载失败。把截图存到
> `qa/` 下再汇报。
