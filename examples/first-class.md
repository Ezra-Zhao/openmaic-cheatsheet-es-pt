# 第一课演练 | First-class drill: from topic to an offline HTML handout

把「生成一节课并交到学生手里」这条链路一次走通，顺序别乱：

1. **先在 Demo 里验效果**：打开 open.maic.chat，用你真要讲的主题（如「零基础 30 分钟学会 Python 变量」）生成一节课，完整听一遍老师讲解、答一次测验、点开一个互动场景——先确认产出质量，再谈部署。
2. **自托管起服务**：按速查表五步卡走完（Node ≥ 22.19、pnpm ≥ 10、`pnpm db:up` 起库、`pnpm dev`），浏览器打开 localhost:3000 确认能进。
3. **配最小可用模型组合**：`.env.local` 先放一个 key，`openmaic.yml` 里至少把 `llm`、`course.outline`、`course.content`、`tts` 指到可用模型；`video` 先写 `null` 省钱，跑通后再逐项打开 image、webSearch。
4. **生成并人工过一遍**：用同一个主题在自托管里再生成一次，对照 Demo 版检查大纲、测验判分、白板讲解；发现模型差距，先换槽位模型，别先怀疑平台。
5. **导出离线 HTML 发给学生**：导出单个 HTML 文件，断网打开验证能播完整堂课（旁白、字幕、测验、互动都在），再发给学生——这一步过了，这条链才算真正打通。
