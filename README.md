# OpenMAIC 速查表 | OpenMAIC Cheatsheet | Guía rápida de OpenMAIC

> 一页看懂 OpenMAIC（清华 MAIC 团队开源）：给一个主题或一份文档，几分钟生成一整间「会讲课的教室」——AI 老师语音讲课、白板上画图推导，AI 同学陪你讨论、辩论，幻灯片、测验、互动模拟、项目制学习（PBL）全齐，还能把整堂课导出成可编辑的 PPTX 或一个能离线播放的 HTML 文件。备课从几小时压到几分钟。

- 上游项目：[THU-MAIC/OpenMAIC](https://github.com/THU-MAIC/OpenMAIC)（2026-11-02 实测 42,448 ★ / 6,538 forks，24 个 release，MIT）
- 平台：自托管 Web 服务（常驻 Node.js 进程 + PostgreSQL）；或先用官方在线 Demo 零安装体验
- 本页结构：中文主文 → English → Español → Português

---

Apoyo y más: [GitHub Sponsors](https://github.com/sponsors/Ezra-Zhao) · [Gumroad — plantillas y cuadernos que uso](https://ezraflow28.gumroad.com) · [Ko-fi](https://ko-fi.com/ezra777zhao)

## 它是什么、不是什么

| 是 | 不是 |
|---|---|
| 一键生成课堂的开源平台：输入主题或文档，两阶段流水线（先大纲、后逐场景生成）产出整堂课 | 不是录播课工具：课堂里有 AI 老师和 AI 同学实时互动、讨论、圆桌辩论、白板讲解 |
| 模型你自己配：服务端统一在 `openmaic.yml` 里声明服务商与每个槽位用哪个模型 | 不是某个厂商的 SaaS：没有官方托管订阅，key 在你自己的 `.env.local` 里 |
| 课堂存在服务端：关掉浏览器、重启服务，生成不中断、课程不丢 | 不是纯前端玩具：1.2.0 起是服务端优先架构，必须有常驻进程和数据库才跑得起来 |
| 界面支持 11 种语言（含西班牙文、葡萄牙文） | 上手文档目前只有英文与简体中文 README——这正是本页存在的原因 |

## 两条起步路，先选一条

| | 路 A：先玩 Demo（零安装） | 路 B：自托管（给学生长期用） |
|---|---|---|
| 入口 | 打开 [open.maic.chat](https://open.maic.chat/) 在线体验 | 自己机器/服务器部署，见下方五步卡 |
| 适合 | 先验证「生成一节课」是不是你要的效果 | 机构培训、教会课程、自有学员长期使用 |
| 代价 | 不用装任何东西；深度与数据都在官方托管侧 | 要备 Node、pnpm、PostgreSQL 和模型 API key |
| 提醒 | 玩明白了再决定要不要自托管，别一上来先啃配置 | 1.2.0 起不再支持 Vercel 等 serverless 部署（只支持到 1.1.x） |

## 自托管五步卡

环境要求：**Node.js ≥ 22.19、pnpm ≥ 10、PostgreSQL 16**（本地开发可用 Docker 一条命令拉起，见第 3 步）。

1. **克隆装依赖**：`git clone https://github.com/THU-MAIC/OpenMAIC.git` → `cd OpenMAIC` → `pnpm install`
2. **复制两份配置**：`cp .env.example .env.local`（放 API key 与服务端选项）；`cp openmaic.example.yml openmaic.yml`（声明服务端能调哪些模型）。示例开箱只需在 `.env.local` 里填一个 `OPENAI_API_KEY`，也可以换成你有 key 的任何一家。
3. **起数据库**：`pnpm db:up` 用 Docker 在 127.0.0.1:5432 起一个开发用 PostgreSQL，然后把 `.env.local` 里 `DATABASE_URL=postgres://openmaic:openmaic-dev@127.0.0.1:5432/openmaic` 那行取消注释。没有 `DATABASE_URL` 服务会直接拒绝启动——这是设计，不是 bug。
4. **启动**：`pnpm dev`，打开 http://localhost:3000 。生产环境用 `pnpm build && pnpm start`，并让它常驻运行（进程管理器或容器）：生成在服务端跑，浏览器关了也不中断。
5. **或整套 Docker**：填好 `.env.local` 后 `docker compose up --build`（应用 + PostgreSQL 两容器）。默认是个人安装口径：单用户模式、只监听 127.0.0.1:3000；要对外开放，先看诚实标注区的 ACCESS_CODE 那条。

## 模型槽位（slot）一页对照

`openmaic.yml` 的核心概念只有三个：**providers**（服务端能用的服务商账号，key 用 `${VAR}` 从 `.env.local` 引用）、**slots**（每一处用到 AI 的地方默认用哪个模型）、**lock / allowUserKeys**（谁能改）。写进文件的槽位是默认值；没被 lock 的，用户可以在网页模型设置里自己改。

| 槽位 | 管什么 | 备注 |
|---|---|---|
| `llm` | 默认对话模型 | 其他聊天类槽位没单独指定时都跟它 |
| `course.outline` | 课堂大纲生成 | 两阶段流水线的第一阶段 |
| `course.content` | 场景正文生成 | 有子槽位 `course.content.slide / .quiz / .interactive / .pbl` 可分开配 |
| `course.actions`、`course.agents` | 课堂动作与智能体行为 | 老师/同学在课上怎么动、怎么说 |
| `course.research` | 生成前的调研 | 联网查资料再备课 |
| `agent` | Pro 工作台 agent | 需要支持工具调用（tool calling）的模型 |
| `image` / `video` | 配图 / 视频片段生成 | 写 `video: null` 即默认关掉这项能力 |
| `tts` / `asr` | 语音合成 / 语音识别 | 老师讲课的嗓子和听学生说话的耳朵 |
| `webSearch` / `webFetch` | 联网搜索 / 网页抓取 | 抓取槽位 1.2.1 起有，需手动开启 |
| `document` | 文档解析 | 可接 MinerU 增强表格、公式、OCR 解析 |

- **锁住不让改**：把槽位名写进 `lock:`（连同它的子槽位一起锁），如 `lock: [video]`；`lock: all` 全锁。
- **禁止用户自带 key**：设 `allowUserKeys: false`，用户只能在你声明的服务商里选。
- **支持的服务商**（上游 README 口径）：OpenAI、Azure OpenAI、Anthropic、Amazon Bedrock、Google Gemini、DeepSeek、通义千问、Kimi、MiniMax、Grok (xAI)、OpenRouter、TokenDance、豆包、腾讯混元 / TokenHub、小米 MiMo、智谱 GLM、Ollama，以及本地的 Lemonade（本地 AI）与 FunASR（本地 ASR），外加任何兼容 OpenAI API 的服务。套餐型服务商（如 MiniMax Token Plan）可用一个 key 同时覆盖对话、图像、视频、TTS 与联网搜索。
- **排障提示**：`openmaic.yml` 写错时服务启动会直接报错并指出出错字段（YAML 语法错误会指出行号），先看启动日志再查别的。

## 五种互动形态怎么选

深度交互模式在标准幻灯片/测验之外，给每节课加一种可动手探索的界面：

| 形态 | 适合什么内容 | 一句话判断 |
|---|---|---|
| 3D 可视化 | 分子结构、人体解剖、天体运行等空间型知识 | 抽象结构「看一眼立体的」胜过十张平面图 |
| 模拟实验 | 物理、化学、电路、流程类动态过程 | 要让学生自己调参数看结果，就选它 |
| 游戏 | 语言、历史、规则类需要反复记的内容 | 靠挑战和即时反馈把记忆钉住 |
| 思维导图 | 概念多、关系密的总览型主题 | 先建整体框架，再往细节里钻 |
| 在线编程 | 编程与数据类课程 | 浏览器里直接写、直接跑，边学边改 |

AI 老师在互动界面里不是旁观者：它能高亮关键区域、设置实验条件、给提示，把学生的注意力在恰当时机引到该看的地方。

## 导出三路对照（另有资源包）

| 格式 | 得到什么 | 适合 |
|---|---|---|
| PPTX | 可编辑幻灯片（含图片、图表、LaTeX 公式）；互动/测验/PBL 场景保留为原位占位页，带「在线打开」按钮与二维码 | 还要二次加工、进自己课件体系的课 |
| 离线 HTML | 单个文件离线播放整堂课：带字幕旁白、聚光灯与激光笔、幻灯片视频、测验与互动场景（旁白可选含/不含） | 发给学生自学、内网/无网环境 |
| 课堂 ZIP | 完整课堂（课程结构 + 媒体文件），`.maic.zip` | 备份、分享、在另一处部署间迁移 |
| （附）教学资源包 | PPTX + 每个互动场景的独立网页 | 想把互动页拆出来单独用的场景 |

补充：导出时会把互动场景引用的外部资源内联进 HTML，导出的课断网也能播；「导出视频（MP4）」是可选的独立渲染服务（需 Chromium + FFmpeg），用 Docker 的 `video-export` profile 单独起。

## 诚实标注区

- **1.3.0 升级有硬条件**：2026-10-29 发布的 v1.3.0 升级前必须先停掉所有旧实例，且升级后不能回滚到 1.3.0 以下；动手前先读上游 CHANGELOG（原仓动态栏自述）。
- **版本分界线在 1.2.0**：1.2.0 起服务端优先（生成跑在服务端、需要 PostgreSQL 与常驻 Node），Vercel 等 serverless 只支持到 1.1.x；还想在 Vercel 上跑就得用 `release/1.1.x` 分支，别拿主分支硬部署。
- **不设 ACCESS_CODE 等于没门**：上游明确警告——共享部署不在 `.env.local` 里设足够长的随机 `ACCESS_CODE`（至少 16 个字符）时，任何能访问该端口的人都拥有整个课程库，可以编辑甚至删除。要对外开放，先设码，再谈发布地址。
- **课程归属模式先想好**：单用户（Compose 默认，所有访客共用一个库）、共享团队（访问码背后共用）、匿名（每个浏览器一个库）、自有账号四种模式，首次启动 1.2.0 前就要定，因为旧版课堂会导入给当时配置的所有者。
- **效果取决于你配的模型组合**：官方推荐口径是「打开全部模态效果最好」——配图、语音、视频、联网检索、网页抓取都参与生成；只配一个文本模型，课能生成，但不是完整体验。成本也随之走，别忽略 token 账。
- **界面有西/葡语，文档没有**：界面支持 11 种语言、12 个区域设置（含西班牙文-墨西哥、葡萄牙文-巴西），但官方 README 只有英文与简体中文，上手与配置文档以英文/中文为准；本页西/葡语部分为学习译稿、待母语校对。

---

## English (summary)

OpenMAIC (by THU-MAIC, Tsinghua's MAIC team; ~42.4k stars and ~6.5k forks as of 2026-11-01, MIT) turns a topic or a document into a full interactive classroom in minutes: an AI teacher lectures with voice and a whiteboard, AI classmates discuss and debate with you, and lessons combine slides, quizzes, interactive simulations and project-based learning (PBL). Finished classes export as editable PPTX, a single offline-playable HTML file, or a classroom ZIP for backup and migration.

Two ways in: the zero-install live demo at open.maic.chat, or self-hosting. Self-hosting (the 1.2+ architecture is server-first) needs Node.js ≥ 22.19, pnpm ≥ 10 and PostgreSQL 16: clone, `pnpm install`, copy `.env.example` to `.env.local` (one API key is enough to start) and `openmaic.example.yml` to `openmaic.yml`, start the dev database with `pnpm db:up`, then `pnpm dev` and open localhost:3000 — or run the whole stack with `docker compose up --build`.

Models are configured server-side in `openmaic.yml`: providers hold the accounts (keys referenced as `${VAR}` from `.env.local`), slots assign a model to each AI touchpoint (`llm`, `course.outline`, `course.content` and its slide/quiz/interactive/pbl sub-slots, `image`, `video`, `tts`, `asr`, `webSearch`, `webFetch`, `document`, …), `lock` pins slots for everyone, and `allowUserKeys: false` stops users from bringing their own keys. 20+ providers are supported, including OpenAI, Anthropic, Gemini, DeepSeek, Qwen, Kimi, GLM, Ollama and any OpenAI-compatible endpoint.

Honest notes: since 1.2.0, generation runs server-side, so Vercel-style serverless is only supported up to 1.1.x; upgrading to v1.3.0 requires stopping all old instances first and cannot be rolled back below 1.3.0; and a shared deployment without a long random `ACCESS_CODE` leaves the whole course library open to anyone who can reach the port. The UI ships in 11 languages (including Spanish and Brazilian Portuguese), but the docs are English/Chinese only.

---

## Español (guía rápida — traducción de estudio, pendiente de revisión por un hablante nativo)

OpenMAIC (del equipo MAIC de Tsinghua, ~42,4 mil estrellas y ~6,5 mil forks a noviembre de 2026, MIT) convierte un tema o un documento en un aula interactiva completa en minutos: un profesor de IA explica con voz y pizarra, compañeros de IA discuten y debaten contigo, y la clase combina diapositivas, cuestionarios, simulaciones interactivas y aprendizaje basado en proyectos (PBL). Al terminar, exportas la clase como PPTX editable, como un único archivo HTML que se reproduce sin conexión, o como ZIP del aula para respaldo y migración.

**Dos caminos para empezar**: la demo en vivo sin instalar nada en open.maic.chat, o el autoalojamiento. Desde la versión 1.2 la arquitectura es «el servidor primero»: necesitas Node.js ≥ 22.19, pnpm ≥ 10 y PostgreSQL 16. Los pasos: clona el repositorio, `pnpm install`, copia `.env.example` a `.env.local` (basta una clave de API para arrancar) y `openmaic.example.yml` a `openmaic.yml`, levanta la base de datos de desarrollo con `pnpm db:up`, y luego `pnpm dev` y abre localhost:3000. O todo junto con `docker compose up --build`.

**Los modelos se configuran en el servidor**, en `openmaic.yml`: los *providers* guardan las cuentas (las claves se referencian como `${VAR}` desde `.env.local`), los *slots* asignan un modelo a cada punto donde se usa IA (`llm`, `course.outline`, `course.content` con sus sub-slots de diapositivas/cuestionarios/interactivo/PBL, `image`, `video`, `tts`, `asr`, `webSearch`, `webFetch`, `document`…), `lock` fija slots para todos y `allowUserKeys: false` impide que los usuarios traigan sus propias claves. Hay más de 20 proveedores compatibles: OpenAI, Anthropic, Gemini, DeepSeek, Qwen, Kimi, GLM, Ollama y cualquier endpoint compatible con OpenAI.

**Cinco formatos interactivos** además de diapositivas y cuestionarios: visualización 3D (estructuras espaciales), simulación (procesos que el alumno ajusta y observa), juegos (memoria con desafío), mapas mentales (marcos conceptuales) y programación en el navegador (escribir y ejecutar al momento).

**Notas honestas**: desde 1.2 la generación corre en el servidor, así que el despliegue sin servidor tipo Vercel solo se admite hasta 1.1.x; actualizar a la v1.3.0 exige detener antes todas las instancias antiguas y no permite volver a versiones anteriores a 1.3.0; y un despliegue compartido sin un `ACCESS_CODE` largo y aleatorio deja toda la biblioteca de cursos abierta a cualquiera que alcance el puerto. La interfaz está en 11 idiomas (incluido español), pero la documentación oficial solo está en inglés y chino.

---

## Português (guia rápido — tradução de estudo, aguardando revisão de um falante nativo)

O OpenMAIC (da equipe MAIC de Tsinghua, ~42,4 mil estrelas e ~6,5 mil forks em novembro de 2026, MIT) transforma um tema ou um documento em uma sala de aula interativa completa em minutos: um professor de IA explica com voz e quadro branco, colegas de IA discutem e debatem com você, e a aula combina slides, questionários, simulações interativas e aprendizagem baseada em projetos (PBL). Ao final, você exporta a aula como PPTX editável, como um único arquivo HTML que roda offline, ou como ZIP da aula para backup e migração.

**Dois caminhos para começar**: a demo ao vivo sem instalar nada em open.maic.chat, ou a hospedagem própria. Desde a versão 1.2 a arquitetura é «servidor primeiro»: você precisa de Node.js ≥ 22.19, pnpm ≥ 10 e PostgreSQL 16. Os passos: clone o repositório, `pnpm install`, copie `.env.example` para `.env.local` (uma chave de API já basta para começar) e `openmaic.example.yml` para `openmaic.yml`, suba o banco de dados de desenvolvimento com `pnpm db:up`, depois `pnpm dev` e abra localhost:3000. Ou tudo de uma vez com `docker compose up --build`.

**Os modelos são configurados no servidor**, no `openmaic.yml`: os *providers* guardam as contas (as chaves são referenciadas como `${VAR}` a partir do `.env.local`), os *slots* atribuem um modelo a cada ponto onde a IA é usada (`llm`, `course.outline`, `course.content` com seus sub-slots de slides/questionários/interativo/PBL, `image`, `video`, `tts`, `asr`, `webSearch`, `webFetch`, `document`…), `lock` fixa slots para todos e `allowUserKeys: false` impede que usuários tragam as próprias chaves. São mais de 20 provedores compatíveis: OpenAI, Anthropic, Gemini, DeepSeek, Qwen, Kimi, GLM, Ollama e qualquer endpoint compatível com OpenAI.

**Cinco formatos interativos** além de slides e questionários: visualização 3D (estruturas espaciais), simulação (processos que o aluno ajusta e observa), jogos (memória com desafio), mapas mentais (estruturas conceituais) e programação no navegador (escrever e executar na hora).

**Notas honestas**: desde a 1.2, a geração roda no servidor, então implantações serverless tipo Vercel só são suportadas até a 1.1.x; atualizar para a v1.3.0 exige parar antes todas as instâncias antigas e não permite voltar a versões anteriores à 1.3.0; e uma implantação compartilhada sem um `ACCESS_CODE` longo e aleatório deixa toda a biblioteca de cursos aberta a qualquer um que alcance a porta. A interface está em 11 idiomas (incluindo português brasileiro), mas a documentação oficial só existe em inglês e chinês.

---

## 来源与声明

- 灵感来源 / Source / Fuente / Fonte: [THU-MAIC/OpenMAIC](https://github.com/THU-MAIC/OpenMAIC)（在线体验 open.maic.chat）
- 本仓是个人学习整理：内容用我自己的话总结自上游公开文档，不搬运原文；环境要求、配置槽位与导出格式以 2026-11-02 的上游 README、示例配置与配置文档为准，上游迭代很快，请以官方文档为最终依据。
- 西语、葡语部分为学习译稿，待母语校对。Las secciones en español y portugués son traducciones de estudio pendientes de revisión nativa.
- License: MIT © 2026 Guangyi Zhao
