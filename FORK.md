# Fork 说明

本仓库是 [mattpocock/skills](https://github.com/mattpocock/skills) 的 fork,唯一目的:
把"TDD 默认关闭"的个人测试策略固化在构建路径技能里,并长期低成本跟随上游。

- marketplace 名:`phgors-skills`(定义在 `.claude-plugin/marketplace.json`)
- 插件 id:`mattpocock-skills@phgors-skills`,技能前缀仍为 `mattpocock-skills:*`
- 本地维护 clone:`~/repos/marketplaceskills`(origin=本仓库,upstream=上游)

## 与上游的差异(改了什么)

策略(为什么改):

- 默认不使用 TDD,也不主动调用 tdd 技能:直接实现代码,然后运行现有测试验证。
- 仅在以下情况使用 TDD:用户明确要求;或修改属于高风险/复杂逻辑(协议解析与
  编解码、加密/认证、路由与负载均衡决策、并发与状态机、配置迁移)。
- 非 TDD 场景下,实现后必须运行相关现有测试;未被现有测试覆盖的改动,在回复中说明。

改动清单:

| 文件 | 改动 |
| --- | --- |
| `skills/engineering/tdd/SKILL.md` | description 改为 `Use ONLY when …`(门控);正文新增 `## Not the default` 一节 |
| `skills/engineering/implement/SKILL.md` | 原无条件 `Call the Skill tool with "tdd"` -> 默认直建 + 现有测试验证 + 说明未覆盖改动 |
| `skills/engineering/implement-spec/SKILL.md` | implementer 子代理的 tdd 委托 -> 默认直建 + 现有测试验证 |
| `skills/engineering/ask-matt/SKILL.md` | 主流程描述改为直接实现 + 现有测试验证;TDD 标注为 the exception, not the default |
| `.claude-plugin/marketplace.json` | marketplace 名 `mattpocock` -> `phgors-skills`;description 注明 fork 用途 |
| `package.json` + `.claude-plugin/plugin.json` | 版本号改为 `<上游版本>-phgors.<n>`(两处必须同步) |

明确不动:`diagnosing-bugs`(其回归测试属于 bug 诊断回路,不算 TDD 构建流程)、
`docs/` 镜像、`README.md`。

查看当前完整差异:

```bash
git fetch upstream 2>/dev/null; git diff upstream/main --stat
```

## 上游有更新时:同步流程

```bash
cd ~/repos/marketplaceskills
git fetch upstream
git merge upstream/main
```

### 1. 解冲突(只可能落在 4 个策略文件)

- 保持语义不变:默认直建 + 运行现有测试验证 + 说明未覆盖改动;
  tdd 仅在 `用户明确要求 || 高风险类目` 时使用。
- 高风险类目在 4 处必须是同一句英文:
  `protocol parsing & encoding/decoding, crypto/authentication, routing & load-balancing decisions, concurrency & state machines, configuration migration`
- 上游若新增了无条件的 tdd 委托/推荐,同样加门控;上游的其他改动一律保留。
- 逐文件抽查门控关键词是否仍在:

```bash
grep -l "Use ONLY when" skills/engineering/tdd/SKILL.md
grep -l "Do not default to TDD" skills/engineering/implement/SKILL.md
grep -l "builds the ticket directly" skills/engineering/implement-spec/SKILL.md
grep -l "TDD is the exception, not the default" skills/engineering/ask-matt/SKILL.md
```

### 2. 版本 bump

`package.json` 与 `.claude-plugin/plugin.json` 两处同步改为
`<merge 后的上游版本>-phgors.<n>`(上游用 changesets 同步这两处,fork 手动同步)。
不 bump 则 ZCode 侧看不到更新。纯文档类 commit(如只改本文件)不需要 bump。

### 3. push + ZCode 更新

```bash
git push origin main
node ~/Applications/zcode/resources/glm/zcode.cjs plugins update mattpocock-skills@phgors-skills
```

`update` 拿不到新 commit 时,兜底重挂渠道:

```bash
ZC="node ~/Applications/zcode/resources/glm/zcode.cjs"
$ZC plugins uninstall mattpocock-skills@phgors-skills --force
$ZC plugins marketplace remove phgors-skills
$ZC plugins marketplace add https://github.com/phgors/marketplaceskills.git
$ZC plugins install mattpocock-skills@phgors-skills
```

## 首次在新机器上准备维护环境

```bash
git clone https://github.com/phgors/marketplaceskills.git ~/repos/marketplaceskills
git -C ~/repos/marketplaceskills remote add upstream https://github.com/mattpocock/skills.git
```

仅恢复安装(不需要维护环境)时,两条命令即可:

```bash
node ~/Applications/zcode/resources/glm/zcode.cjs plugins marketplace add https://github.com/phgors/marketplaceskills.git
node ~/Applications/zcode/resources/glm/zcode.cjs plugins install mattpocock-skills@phgors-skills
```

## 注意

- 不要同时从 `claude-plugins-official` 装 mattpocock-skills:同名技能会遮蔽
  (仅 discovery 序列第一个生效)。
- 策略本身的持久兜底在用户全局 `~/.zcode/AGENTS.md`(优先级高于技能指令);
  本 fork 的作用是让技能措辞与策略一致,避免指令互相打架。
- 机器相关的完整坐标(缓存路径、安装记录位置、验证命令)见
  `~/.zcode/mattpocock-skills-tdd-override.md`;本文件随仓库走,那份随机器走。
