# 重试逻辑实现说明 / Retry Logic Implementation

[English version below]

## 中文说明

### 问题
原始的 Upptime 代码存在以下问题：
1. 重试逻辑有 bug：缺少 `await` 关键字，导致重试立即执行而不是延迟
2. 第一次重试延迟只有 1 秒
3. 只有 2 次重试（总共 3 次尝试）

### 解决方案
本 PR 实现了需求："当检测到网站错误时，延迟 10s，重试3次，避免网络波动导致误报"

**具体实现：**
- 修复了缺少 `await` 的 bug
- 将所有重试延迟改为 10 秒
- 添加了第 3 次重试（总共 4 次尝试：初始 + 3 次重试）

**重试流程：**
1. 第 1 次检测到网站故障
2. 等待 10 秒
3. 第 2 次尝试
4. 如果仍然失败，等待 10 秒
5. 第 3 次尝试
6. 如果仍然失败，等待 10 秒
7. 第 4 次尝试（最后一次）
8. 只有在所有 4 次尝试都失败时才报告为宕机

这样可以在 30 秒内（3 × 10秒）完成所有重试，既能快速检测到真实的故障，又能有效避免网络波动导致的误报。

### 技术细节
由于 Upptime 使用预构建的 GitHub Action，我们采用了以下方案：
1. 创建补丁文件 (`.github/retry-fix.patch`) 包含代码修复
2. 创建自定义组合 Action (`.github/actions/run-uptime-monitor`) 来：
   - 克隆 upptime/uptime-monitor v1.41.0
   - 应用补丁
   - 构建并缓存修改后的版本
3. 更新所有工作流以使用自定义 Action
4. 删除 `update-template.yml` 以防止自动更新覆盖我们的修改

### 维护说明
- **不要**重新启用 `update-template.yml` 工作流
- 升级 Upptime 版本时，可能需要更新补丁文件
- 详细文档请参阅 `.github/CUSTOM_UPPTIME.md`

---

## English Version

### Problem
The original Upptime code had the following issues:
1. Retry logic had bugs: missing `await` keywords caused retries to execute immediately
2. First retry delay was only 1 second
3. Only 2 retries (3 total attempts)

### Solution
This PR implements the requirement: "When website errors are detected, delay 10s and retry 3 times to avoid false positives caused by network fluctuations"

**Implementation:**
- Fixed missing `await` bugs
- Changed all retry delays to 10 seconds
- Added 3rd retry (total of 4 attempts: initial + 3 retries)

**Retry Flow:**
1. 1st detection of website failure
2. Wait 10 seconds
3. 2nd attempt
4. If still failing, wait 10 seconds
5. 3rd attempt
6. If still failing, wait 10 seconds
7. 4th attempt (final)
8. Only report as down if all 4 attempts fail

This allows all retries to complete within 30 seconds (3 × 10s), enabling quick detection of real outages while effectively avoiding false positives from network fluctuations.

### Technical Details
Since Upptime uses a pre-built GitHub Action, we implemented:
1. Created patch file (`.github/retry-fix.patch`) with code fixes
2. Created custom composite action (`.github/actions/run-uptime-monitor`) to:
   - Clone upptime/uptime-monitor v1.41.0
   - Apply the patch
   - Build and cache the modified version
3. Updated all workflows to use the custom action
4. Removed `update-template.yml` to prevent auto-updates from overwriting our changes

### Maintenance Notes
- **DO NOT** re-enable the `update-template.yml` workflow
- When upgrading Upptime versions, the patch file may need updates
- See `.github/CUSTOM_UPPTIME.md` for detailed documentation
