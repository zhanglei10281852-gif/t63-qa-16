# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

数据库健康探测在调用方已经取消时仍继续访问连接池，关闭流程必须等待探测完成。请修复 Ping 的取消传播，使已取消请求立即返回。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/t63-qa-16
- 仓库地址：https://github.com/zhanglei10281852-gif/t63-qa-16.git
- parent SHA：ed3304f2646c81fd531a4e1453f7c946a2316ac1

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/t63-qa-16.git bug-repro
cd bug-repro
git checkout --detach ed3304f2646c81fd531a4e1453f7c946a2316ac1
go test ./internal/storage/sqlite -run TestPingHonorsContextCancellation -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/storage/sqlite -run TestPingHonorsContextCancellation -count=1
--- FAIL: TestPingHonorsContextCancellation (0.66s)
    integration_test.go:220: cancelled ping succeeded
FAIL
FAIL	sanitation-operations/internal/storage/sqlite	0.660s
FAIL

```

stderr：

```text
warning: internal/storage/sqlite/integration_test.go has type 100755, expected 100644
warning: internal/storage/sqlite/integration_test.go has type 100755, expected 100644

```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/storage/sqlite -run TestPingHonorsContextCancellation -count=1
--- FAIL: TestPingHonorsContextCancellation (0.86s)
    integration_test.go:220: cancelled ping succeeded
FAIL
FAIL	sanitation-operations/internal/storage/sqlite	1.097s
FAIL

```

stderr：

```text
warning: internal/storage/sqlite/integration_test.go has type 100755, expected 100644
warning: internal/storage/sqlite/integration_test.go has type 100755, expected 100644

```

## 通过条件

在触发条件下，定向测试 TestPingHonorsContextCancellation 应通过，相关包、全量测试、竞态测试和构建检查均通过；回退 gold 唯一修复后定向测试重新失败。
