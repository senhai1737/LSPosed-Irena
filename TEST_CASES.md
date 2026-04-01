# Master Log Switch 测试用例

## 测试场景

### 场景1: 初始状态
- 日志总开关: 启用 (默认)
- 详细日志开关: 启用 (默认)
- 预期: 日志服务运行，详细日志启用

### 场景2: 关闭日志总开关
1. 用户关闭日志总开关
2. 预期:
   - 日志服务停止
   - 详细日志自动关闭
   - UI中详细日志开关被禁用
   - `verboseLog()` 返回 false

### 场景3: 尝试在日志总开关关闭时启用详细日志
1. 日志总开关关闭
2. 尝试启用详细日志
3. 预期:
   - 操作被拒绝
   - 详细日志保持关闭状态
   - 显示警告日志

### 场景4: 重新启用日志总开关
1. 日志总开关关闭，详细日志关闭
2. 启用日志总开关
3. 预期:
   - 日志服务启动
   - 详细日志保持关闭状态（需要手动启用）
   - UI中详细日志开关可用

### 场景5: 启用日志总开关后启用详细日志
1. 日志总开关启用
2. 启用详细日志
3. 预期:
   - 详细日志启用
   - `verboseLog()` 返回 true

## 代码验证点

### ConfigManager 验证
1. `setLogEnabled(false)` 应调用 `logcatService.stop()`
2. `setLogEnabled(false)` 应自动调用 `setVerboseLog(false)`
3. `setVerboseLog(true)` 当 `!logEnabled` 时应拒绝并记录警告
4. `verboseLog()` 当 `!logEnabled` 时应返回 false

### LogcatService 验证
1. `start()` 当 `!ConfigManager.getInstance().isLogEnabled()` 时应不启动
2. 构造函数应检查日志总开关并决定是否启动
3. `stop()` 应安全停止线程

### UI 验证
1. SettingsFragment 中详细日志开关应依赖于日志总开关
2. LogsFragment 应正确显示详细日志状态
3. 当日志总开关关闭时，详细日志开关应被禁用

## 手动测试步骤

1. 编译并安装修改后的LSPosed
2. 打开LSPosed管理器
3. 进入设置页面
4. 测试各种开关组合
5. 检查日志输出是否符合预期

## 预期日志输出

### 关闭日志总开关时:
```
LSPosedLogcat: Master log switch is off, not starting logcat
LSPosed: Cannot enable verbose log when master log switch is off
```

### 启用日志总开关时:
```
LSPosedLogcat: start running
```

### 启用详细日志时:
```
LSPosedLogcat: !!start_verbose!!
```

### 关闭详细日志时:
```
LSPosedLogcat: !!stop_verbose!!
```