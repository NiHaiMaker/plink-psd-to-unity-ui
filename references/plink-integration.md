# PLink 接入与生成边界

执行前遵守 [两处审核与结果验收](../SKILL.md#当前迭代阶段的两处审核) 和 [文件与 MD 一致性](../SKILL.md#文件与-md-一致性)：PSD 整理方案确认并完成必要结果验收后，直接调用现成工具导出；只有完整 Prefab 模式在工具返回正式 Prefab 后，再基于实际节点提交减法与适配清单。两处审核处理不同资产和动作，不得互相替代；任一阶段提出修改意见时，完成实际产物后必须等该阶段结果验收再继续。发现未确认的文件与记录差异，先展示并等用户确认，再补录基线或修改资产。

源 PSD 更新须把 [源更新差异](psd-preparation.md#源更新先审核差异) 纳入同一份 PSD 整理方案审核，用户同意后才更新 `_UI` 并执行本阶段；它不是额外确认门，且仅用于源 PSD 更新；首次生成、整理稿续作及纯适配等入口按各自授权执行。图片模式导出后直接进入第三份 MD；完整 Prefab 模式生成后按当前说明对比、恢复仍适用减法并更新变化项，再生成代码。业务仅修复受影响旧逻辑或补全此前已实现功能的差异接入；此前未实现的功能不自动补全。

仅在目标项目实际使用 PLink、PSUIResolver 或 XUGUI 且任务命中时读取。本文是个人 Skill 的接入导航。其他项目 Skill 的流程要求与本 Skill 冲突时，按用户指定的 [Skill 优先级](../SKILL.md#与其他-skill-的优先级) 采用本 Skill；仍读取项目资料以核实 API、可选包和工具入口，不因此修改 `.aiworkflow/` 或工具。

## 读取路由

下列路径相对于本次目标项目根，不是本 Skill 目录：

| 当前任务 | 目标项目资料 |
|---|---|
| 执行现有 PSD 工具导入与资源导出 | 由本 Skill 当前资料直接约束，不额外读取工具开发 Skill |
| 修复、扩展、迁移或替换 PSD 导入/生成工具 | `.aiworkflow/skills/plink-psd-importer-development/SKILL.md` 及其直接资料；必须由用户另行明确委托 |
| Prefab、节点、代码生成、界面逻辑 | `.aiworkflow/skills/plink-ui-panels/SKILL.md`、`reference.md`、`examples.md` |
| 编写业务层 PLink 手写实体（不修改框架或编辑器工具） | `.aiworkflow/skills/plink-framework/SKILL.md` 及其要求的资料 |
| 资源加载与生命周期 | 项目当前 `plink-assetmiddleware`；启用 YooAsset 时再按路由读 `plink-yooasset` |
| Unity Editor 操作 | `.aiworkflow/rules/ai-workflow.mdc` 与 `.aiworkflow/docs/conventions/unity-mcp-unity-playmode.md` |

缺少某份资料时不能假设已安装对应能力，应检查目标项目的替代入口及工具源码。若目标项目要求 Coplay MCP，所有 Unity Editor 交互只通过 Coplay；不得改用 batchmode、另一自动化通道或直接编辑资源 YAML 绕过。保持 Enter Play Mode Options、Reload Domain、Reload Scene 不变，除非用户明确授权修改。

### Coplay 连接故障处理

按 [主规则的连接故障处理](../SKILL.md#coplay-连接故障处理) 默认由 Agent 自行发现、加载、连接和只读验证 Coplay；异常时自行排查与恢复，不向用户追加连接不可用提醒或要求接入。连接诊断记录到当前阶段 MD，保持真实执行状态；继续提交 PSD 整理方案，并在有效确认和 PSD 编辑能力具备时完成 `_UI` 整理与自检，不能在源稿盘点后就结束完整制作。恢复后从实际未完成阶段续作，不要求重复确认已批准方案。

请求超时或返回异常时，读取当前 MCP 能力、目标 Editor 实例、Editor 状态和本次请求的真实产物，再恢复连接、改用 Coplay 的等价直接工具或缩小请求，每次诊断保留状态证据。导出写入请求只能进行只读状态检查或重连诊断，不再调用导出入口。超时不等于未执行，写入请求不得盲目重发；不无限空等，不虚构调用，不使用外部脚本或其他 Unity 通道代替。

每个新的 Coplay MCP 会话先读取当前服务能力、可用 Editor 实例和 Editor 状态，再选择本轮可用的调用入口；不得复用上次会话的工具名、参数或实例标识。生成或后处理前确认目标 Editor 处于 EditMode、未编译且没有待处理的阻断状态；调用超时后先读取真实产物和 Editor 状态；导出写入只续记原事件，不重试。

### 简单属性修改优先使用专用工具

通过 Coplay 修改单项属性或 RectTransform 时，先核对当前专用工具（如 `set_property`、`set_rect_transform`）的参数、Prefab 目标支持及保存行为；只能使用 Coplay 提供的直接工具或请求载荷，不生成、保存、读取或执行外部脚本文件。Coplay 没有完成本次属性编辑的等价能力时停止并报告阻断。此选择仅适用于属性编辑，不替代 PSD 生成和 UI 代码生成的现成入口。原调用超时时，先按已有超时规则核验实际状态，不能同时换工具重复写入。

## 资源与导入

使用目标项目现有的 PSUIResolver；旧 PSD2UI 只用于明确的兼容需求，不因本 Skill 自动安装或替换导入工具。本 Skill 禁止修改框架、PSD 导入器、UI 代码生成器及共享运行基类源码；手写代码仅限本次业务/UI 逻辑，正常资源配置与现成工具生成不受此限制。发现工具缺陷只核实并准确报告，工具开发必须作为另行明确委托的独立任务，不能从本流程顺势扩展。

输出位置由现成工具当前配置及已确认的必需参数确定，其他 Skill 的目录规范不构成重定向依据，本 Skill 不另立固定目录。生成前读取实际 Importer 的 Module Name 与工程 PS Importer 设置，由现成工具的路径规则解析最终位置：

| 产物 | 当前工具配置来源 |
|---|---|
| Sprite / Texture 切图 | `imageOutputRoot` / `textureOutputRoot`、Texture 开关与尺寸阈值、模块层级 |
| SpriteAtlas | `atlasOutputRoot` 与图集命名模板 |
| UI Prefab | `prefabOutputRoot` 与 Prefab 命名模板 |
| 面板 / Item 代码 | 按项目 UI 规范在现成代码生成工具中指定的最终目录 |

不要为匹配旧示例目录克隆或覆盖导出配置，也不要先错误导出再搬目录冒充现成工具输出。用户明确要求的资源迁移另按对应范围执行，并与工具新写出、已有资源复用分别记录。要求 Coplay MCP 的项目只使用 Coplay 直接工具或请求载荷；不新增导出器、代码生成器、脚本文件或永久菜单。

PSD 源路径沿用项目明确约定及用户指定位置，不从旧样例推导统一存储目录。盘点已有图片及已确认的 Common 资源，再决定复用或导出：

- 同名、同路径或文件存在不足以证明可复用。核对实际切图内容（含尺寸、透明区域）及影响显示的导入设置（如 Sprite / Texture 类型、切片与九宫格边界、PPU、压缩）；能证明与当前 PSD 的预期输出一致才复用。缺失、已变化或暂不能证明一致的图片由现成工具按当前正式输出配置处理，核对本次写入范围与实际结果，不自制切图。
- “普通”“常规”“默认”等用途描述不表示通用资源，禁止翻译、映射或导出为 `common`，也不得仅因名称匹配移动到 Common 目录。仅当项目已有该目录、存在跨界面复用证据且用户明确确认时，才将资源放入或命名为 `common`；否则保留页面专用输出路径。
- 导出后归纳普通重复图标：内容、透明边缘和影响显示的导入设置均相同而仅尺寸不同时，按第三份 MD 的已确认清单保留较大资源。九宫格图片另按可拉伸区域判断，不以宽高或宽高比一致为必要条件：核对四角、非拉伸边缘、可拉伸中心、Sprite Border、PPU、导入设置及各节点的 `Sliced` 显示；目标 RectTransform 必须容得下 Border，在各目标尺寸预览边角、透明边缘、描边、渐变和纹理不失真。能用一个候选源还原各处视觉时，按已确认清单及覆盖尺寸、清晰度选源，不机械选面积较大的图片；需要改 Prefab Image Type 等字段时另按第二份 MD 审核。两类复用均先通过 Coplay 切换全部 Prefab、Sprite 与共享引用并回读显示和代码字段，再在确认无其他引用后清理旧资源及其 `.meta`；证据不足、九宫拉伸失真或共享引用未核清时保持原资源。
- 已有 UI 先保留正式资源的可恢复版本、GUID/引用和当前阶段及关联字段所需 MD 中的适配基线，然后默认只导出图片并走第三份 MD；批准重生成时才走新建流程及 Prefab 减法与适配审核、恢复，不默认新建候选模块。正式图片更新、导入设置及 Prefab 引用变化只有已经明确列入已确认的 Prefab 或第三份资源清单时才不额外确认。共享图片仅当前页面需要变化时，优先通过工具现有能力生成页面专用资源；确需更新共享资源时先查清引用范围，超出原授权的影响单独说明。
- 更新原资源时保留其 `.meta` / GUID；新建独立资源使用新身份，不复制旧 GUID。Sprite 切片变化还需核对实际子资源引用，不能只看主资源 GUID。不同层同名、图片复用、Sprite 与 RawImage 分流均检查最终输出及引用。

生成图集时核对 `Include In Build`；项目使用 YooAsset SBP 时按项目资源流程核对 `TrackSpriteAtlasDependencies`。UI 制作不隐含授权发布或重建整个资源包，涉及打包配置变更按任务边界另行处理。

### 现有导出入口

调用前执行 [导出前整理稿检查](psd-preparation.md#导出前整理稿检查)，确保实际输入是已核验版本；直接调用导出工具、临时导出与重导入验证均适用。

核对 `client/Assets/A_PLink_Public/Scripts/Editor/PSUIResolver/Editor/Importer/` 下当前实现，确认本次 `_UI.psd / _UI.psb` 已使用现成 `PsScriptedImporter`，并读取其实际设置。已有界面维护默认使用 PSD Inspector 的 `Export Images` 或同一入口 `PS2UIResolver.PsImporterEditor.ExecuteExportNow(importer, false)`；该模式导出图片并跳过 Prefab 生成，不触发 XUGUI 绑定。首次制作或用户批准重生成时使用 `Generate Prefab` / `ExecuteExportNow(importer, true)`，由同一工具完成图片及 Prefab 输出。调用前核对目标项目当前实现支持这些入口，不另拼底层服务。

该入口当前为私有方法，可通过 Coplay 的最小调用代码反射执行，传入实际 importer；这只替代点击操作，不替代工具实现。不得另行串联 Parse、BuildTree、ExportTextures、GeneratePrefab 等底层服务，不创建任务专用生成器，也不复制工程设置以重定向正式输出。图片复用与覆盖沿用该工具的现有处理，不能为了避开弹窗切换到自写流程。

图片复用/覆盖窗口由用户手动确认。`Export Images` 只处理图片，不等待 Prefab 窗口；当前 `Generate Prefab` 入口先执行图片导出，再执行 Prefab 生成；图片窗口确认后，同一次请求可能继续出现已有 Prefab 的确认窗口。调用前按规范化的 Importer 路径、实际导入的 `_UI` 指纹、导出模式、正式输出目标及影响输出的关键配置检查排重键，再在对应 MD 建立事件（Prefab 模式第二份、图片模式第三份），同时检查第二、第三份中同一图片输出范围的未终结事件；已有同键未终结事件只续记，事件编号不改变排重键。同一图片输出范围或正式输出目标有旧请求未终结时，即使输入或配置变化，也不并行发起新请求。首次调用前须把原事件持久写为“准备调用/可能已发出”并读回成功；调用发出后只续记该事件，不能因回包中断或旧记录未更新而重发。图片模式确认后核验图片产物并记录现有 Prefab 引用/显示；Prefab 模式须继续记录 Prefab 确认及产物核验。窗口等待、超时、Domain Reload 或调用返回不明时，只读核对弹窗、Editor、可归属本次调用的日志及导出前后产物版本；旧产物存在不能证明本次成功。有限核验后仍结果不明时保持待人工裁决并停止自动导出；只有用户知悉风险后明确结束原事件、重新请求且无在途弹窗或任务，才可建立新事件。同一输入正常终结后再次导出仍需记录新的用户请求依据；实际导入的 `_UI` 指纹变化时重新核对授权和输出范围。没有出现窗口时不增加额外人工确认，也不因事件记录增加第三次方案确认。

检查工具的取消/失败状态、实际日志和保存后的资源，记录真正使用的入口、配置来源与输出路径；新写出、迁移、复用分别说明。当前约定 `_UI` 是实际导入输入，先确认工具覆盖规则能够保留正式身份及恢复适配。用户另行要求的隔离也只能用现成工具支持的方式；不得为实现任何输出方式重写或修改导出链。工具仍无法执行时报告该阶段阻断，不用自制产物补齐流程。

## 节点与代码

以目标 XUGUI 生成器的扫描逻辑为准；目前标准协议是名称的最后一个下划线分段：

| 节点名示例 | 生成用途 |
|---|---|
| `Content_g` | GameObject 引用 |
| `Confirm_b` | UIEventTriggerListener；不是 Unity Button 字段 |
| `Progress_c` | 当前节点的受支持 UI 组件 |
| `Cell_@` | 列表项生成边界及独立 Item 代码 |

`_c` 当前常见识别顺序为 ScrollRect、InputField、Text、Image、RawImage、TMP_InputField、TextMeshProUGUI、Slider；必须核对当前源码。Button、Toggle、Dropdown 等不能因为加了 `_c` 就视为受支持。

成组品质切图的 `Quality_1`、`Quality_2` 等名称在固定 `_UI` 阶段确定并用于图片输出，`_c` 只标记需要生成字段的运行时控件。当前图片导出器按非文字层 `layer.Name` 构造 PNG 文件名，Prefab 生成器也按同一名称创建初始节点；不能宣称一次导出自动把 `Quality_1` 图片层变成 `Quality_c` 控件。完整 Prefab 模式在实际节点审核及后处理中建立最终 Image 控件和 Sprite 对应关系；只图片模式不重命名既有控件，按第三份 MD 核验 Sprite 引用。不得为品质颜色分别生成多套同职责绑定字段，除非已确认的运行结构确实需要。

绑定由运行需求决定：静态装饰和纯容器无需后缀；已有 `_c` 或 `_b` 引用可以访问自身 GameObject，不必另套 `_g` 只为控制同一对象。HorizontalLayoutGroup、VerticalLayoutGroup、LayoutElement、ContentSizeFitter 不属于当前 `_c` 支持列表；仅需 Inspector 配置时不生成引用。确有功能需求需要代码访问时，遵循目标项目允许的手写 Mono partial/组件扩展方式，不能用 `_c` 假装绑定成功。

同一生成作用域内名称唯一且能组成合法 C# 标识符；未标记的父节点不会阻断子节点扫描。使用 `Cell_@`，不要套用旧资料中的 `@Item` 示例。移除全部绑定标记可能触发生成器删除现有 Mono/Auto 文件，执行前核对范围。

## 通用 Item 判定

分类时比较重复控件的结构、字段类型和独立生命周期。只有图片、文字、数量或冷却值不同，且结构能由同一组字段表达时，优先考虑一个可复用 Item；装饰分组或结构明显不同的控件不强行合并。

- `_@` 标记单个 Item 模板根，不标记整个列表容器。选择已有字段完整的样本，内部采用 `Icon_c`、`Count_c`、`Cooldown_c` 等通用字段名，并保留实际的 Image/Text 类型；不要为凑结构伪造文字层或业务字段。
- 同一面板内不能复制多个同名 `Skill_@` 并声称它们自动复用。当前生成器逐个生成父字段和克隆方法，Item 类型名由面板名与 Item 根名称组成，不包含父层级；重复根名可能造成重复成员和类型文件覆盖。
- `SkillRed_@`、`SkillBlue_@` 会生成两个不同 Item 类型；颜色变化本身不应成为拆分类型的理由。确实具有不同结构和职责时，可以使用不同名称分别生成。
- 只做 PSD 分类且要保留全部外观时，可以保留一个 `Skill_@` 模板和其余无绑定标记的参考组。参考组后代同样会被扫描，因此不能在它们里面继续放重复的 `Icon_c` 等标记；使用有区分度、无绑定后缀的资源名称，兼顾切图路径唯一性。命名校验同时检查父面板作用域、每个 Item 的子字段作用域和整个面板的 Item 类型名。

```text
Skills
  Skill_@
    Icon_c
    Cooldown_c
  SkillBlue_Reference
    SkillBlueIcon
    SkillBlueCooldown
```

参考组保留画面，但不是已完成绑定的同类型实例。完整流程在固定 `_UI` 阶段先确定模板结构、显示数量、差异数据来源和参考组用途；正式 Prefab 导出后，按第一份 MD 逐组核验模板、切图、文本、状态、遮罩、局部几何与引用，再决定参考根在 Prefab 中的最终去留。若发现模板必要内容在 `_UI` 遗漏，先回 PSD 阶段确认并整理，不在 Prefab 中首次设计模板。减法核销及绑定前字段审计通过后，才通过权威生成、编译、刷新得到模板类型；运行验证再从模板克隆并填入图标/文本，避免参考与运行实例重复显示。不要将任意参考 GameObject 直接传给克隆帮助方法；当前方法要求其已具有正确的 Item Mono 与序列化引用。

核对当前 Clone/Push 实现：常见 Clone 方法直接 Instantiate 模板并沿用模板父节点；隐藏模板的克隆需要显式激活，并确认它进入正确的布局容器。`UIPanelBaseItem.Push()` 只回收包装对象及引用时，克隆 GameObject 的隐藏、销毁或池化须由明确的所有者处理，不能把 Push 当作 GameObject 回收。模板可作为首个实例，也可隐藏后只用于克隆，须选定一种方式，避免多显示一项或重复回收。

## 生成与刷新

完整制作流程在 Prefab 减法和布局完成后执行本节；已有指定阶段边界时按用户范围处理。必须使用现成 XUGUI 代码生成与刷新工具，面板、Item、Mono、Auto 及初始骨架由工具创建；不自行实现另一套模板写出、生成扫描或绑定赋值逻辑，外部调用代码也不例外。随后仅维护已有业务及新增视觉节点接入；完全未实现的功能交给另一个 Skill。

维护任务按 [已有链条维护](existing-ui-maintenance.md) 区分：每次 PSD 重新生成正式 Prefab 并完成减法与适配审核、恢复适配后都调用本节工具流程；用户仅调整适配时检查显示、维护原说明，不运行生成或刷新。[仅修改手写功能代码](existing-ui-maintenance.md#仅修改手写功能代码) 且节点、绑定组件与序列化引用需求均未变化时，不运行生成或刷新。已确认的 Prefab 清单范围内，适配恢复、绑定及已有业务功能代码不再二次确认。

当前公开静态入口位于 `client/Assets/A_PLink_Public/Scripts/Editor/XUGUI/UIPrefabOperationEditor.CreateCS.cs`：

`PLink.Editor.UI.UIPrefabOperationEditor.RefreshUIPrefab(prefabAssetPath, finalPanelCsDirectory, isJustCodeGenerate)`。

先传 `true` 生成，依据新生成的字段与类型完成已确认清单逐项写明的手写引用迁移、删除引用处理或类型适配。随后通过 Coplay 的最小 Editor 调用执行 `AssetDatabase.Refresh(ImportAssetOptions.ForceSynchronousImport)` 和 `CompilationPipeline.RequestScriptCompilation()`，等待 `EditorApplication.isCompiling` 结束，检查本轮编译错误并确认生成 Mono/Item 类型可加载，再传 `false` 刷新。没有引用变化则跳过手写修复，但仍须主动触发并核实生成文件导入/编译；不能要求用户手动点击 Unity，也不能因旧手写代码编译失败而先刷新绑定。绑定通过后只完成清单已明确的已有功能差异接入，不二次确认；新发现的代码差异、业务行为或作用域必须补清单并取得确认。功能代码修改后再次请求编译，确认类型加载和 Console 无本轮错误，再进入最终审计与运行验证。`finalPanelCsDirectory` 是必须已存在的最终面板代码目录，静态方法不会追加面板名；UI 操作窗口选择的是代码父目录，由窗口追加面板名。当前生成器私有 `RequestScriptCompilation()` 内的 `CompilationPipeline.RequestScriptCompilation()` 已被注释，不能依赖方法名或一次生成调用自动开始/等待编译。

`RefreshUIPrefab(..., false)` 的当前实现会在根层调用 `SetMonoParamVals`，预期把根 RectTransform 归一化为 `position=0`、`rotation=identity`、`localScale=(1,1,1)`、`sizeDelta=(0,0)`、`anchorMin=(0,0)`、`anchorMax=(1,1)`，再保存 Prefab。刷新后必须从保存后的 Prefab 重新读回这些值，并在目标 Canvas 下核验全屏；不能只检查引用数量或 `compiling=false`。根缩放为零、中心固定锚点或设计尺寸仍留在最终根上时，说明刷新、类型加载、保存或后续恢复步骤未正确完成，当前阶段失败。先查实际执行顺序和日志；在已确认 Prefab 调整范围内可恢复根配置，但不得修改 XUGUI/PSUIResolver 源码或让用户手动补点完成流程。

首次生成时使用上述静态入口或 `Tools/Client/UI/UI操作窗口` 明确指定正确目录。仅挂 `UIBasePanelMono` 基类时，Inspector 按类型名定位代码目录会指向基类所在位置；面板专属 Mono 生成并刷新后，再按实际类型使用 Inspector 的按钮。每次执行前核对当前源码，保留目标已有手写文件。

代码生成及必要手写引用修复后，确认 Unity 编译通过，再刷新 Prefab。检查实际 Mono 类型、序列化引用、根 Transform 和 Canvas；不能将“已有生成文件”或“刷新调用没有异常”当作绑定通过。根缩放为零、引用为空或 Prefab 没保存都应使该阶段验收失败。

- `*_Mono.cs`、`*Entity_Auto.cs` 及 Item 的生成文件均按实际覆盖行为保护，不因缺少 `_Auto` 后缀而手改。
- 业务与生命周期写在不会被覆盖的 `*Entity.cs` / Item partial。新增序列化字段使用生成器不会命中的独立手写 Mono partial 或专用组件。
- 可生成引用的节点使用生成字段；不要通过 `Find`、`GetComponentsInChildren` 或按名称遍历来补绑定，也不要手写生成结果。
- 按当前生成器核对命名空间、Prefab 路径与字段名，不复用旧面板的类型名、资源 GUID 或缓存字段。

## 手写行为和适配

面板遵循当前 UIBaseEntity 生命周期及 `[UIPanel]` 要求，保留框架 `isSetSefaArea` 的实际拼写；安全区是否启用按页面布局和需求决定，不能继承某个加载页的关闭设置。

OnShow/OnHide 的基类调用、事件绑定/解绑和回收成对；自定义字段先清理，`base.Dispose()` / `base.Push()` 最后执行。PLink 实体按 EntityFactory/GetNeed 系列创建获取，业务状态通过业务实体或管理器接口变更，业务日志用 `PLink.Core.Log`。

项目同时安装 XUGUIManager.Extension 与 DOTween 时遵循当前 Skill 的 AddClickListener 注册规则，未启用时不引入该依赖。本流程只维护业务逻辑及界面资源，不修改框架或编辑器工具；查源码用于确定调用和行为，不代表获得修改权限。

750×1334 是个人流程的设计基准，不意味着在每个面板任意新增 CanvasScaler 或改动全局 Canvas。核对运行时管理器如何设置 Canvas、相机、缩放与安全区，再选择适配落点。进度条 Filled 类型可以由明确的 Prefab/手写初始化设置，但不得只改表现而漏掉生成引用校验。

至少分开报告 PSD 结构、静态 Prefab、Editor 生成/刷新、编译和 PlayMode 的结果。外部 C# 编译或脚本盘点不能替代 Unity 中的加载、适配和重开验证。
