# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

用户登录后发起了一次带 `request_id` 的工作空间创建，合规查询里 actor 却成了 `system`，真实执行人没有留下；真正由后台触发的任务仍应归到 system。请纠正身份上下文写入审计事件的过程，让两类来源可以准确区分。现有测试用例保持原样，断言不要修改；也不要跳过验证或削弱身份归属检查。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/ai-15
- 仓库地址：https://github.com/zhanglei10281852-gif/ai-15.git
- parent SHA：960dcad1171eaa29576bffdf31c751661928974f

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/ai-15.git bug-repro
cd bug-repro
git checkout --detach 960dcad1171eaa29576bffdf31c751661928974f
go test ./internal/service -run ^TestAuditAttributesMutationToAuthenticatedPrincipal$ -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/service -run ^TestAuditAttributesMutationToAuthenticatedPrincipal$ -count=1
--- FAIL: TestAuditAttributesMutationToAuthenticatedPrincipal (1.04s)
    annotation_repo_behavior_test.go:157: workspace audit actor = "system", want "usr_4bf82e01e1dcc642bf71920a"
FAIL
FAIL	github.com/zhanglei10281852-gif/ai/internal/service	1.059s
FAIL

```

stderr：

```text
(empty)
```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/service -run ^TestAuditAttributesMutationToAuthenticatedPrincipal$ -count=1
--- FAIL: TestAuditAttributesMutationToAuthenticatedPrincipal (1.74s)
    annotation_repo_behavior_test.go:157: workspace audit actor = "system", want "usr_47f3a2d32ae3fef5a98fdf06"
FAIL
FAIL	github.com/zhanglei10281852-gif/ai/internal/service	1.971s
FAIL

```

stderr：

```text
(empty)
```

## 通过条件

由已认证用户发起的工作空间创建必须在审计事件中记录真实 principal，关联 request_id 的查询结果不得显示为 system；没有用户身份的后台任务仍应归属 system。定向身份归属用例及相关审计回归须通过，测试与断言保持原样且不得跳过。
