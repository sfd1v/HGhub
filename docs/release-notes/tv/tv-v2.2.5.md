# HGhubX TV tv-v2.2.5

紧急适配上游全新 `dx150-edge-v1` 四作用域加密网关，修复全部业务接口返回 HTTP 503 的问题，并同步带来网络切换静默恢复改进；适用于 Android TV / 电视盒子（`com.hghubx.tv`）。

## 修复
1. **适配上游 `dx150-edge-v1` 四作用域加密网关，修复接口全 503（#255、#256）**：上游于 2026-11-24 下线旧统一入口 `/api/edge/request`（返回 `HTTP 503 {"msg":"接口已停用"}`）并拆分为 `/api/{public,dynamic,playback,user}/request`；TV 版共用底层 Rust 网络层同步切换至 `dx150-edge-v1`（ECDH P-256 + HKDF-SHA256 + AES-256-GCM 单飞会话与 `public` 快速通道），恢复片单、详情、搜索与播放地址解析，同时对未迁移镜像保留旧接口 `404`/`405` 兼容回退。
2. **网络切换与代理探测恢复（#246、#256）**：仅将 HTTP 401 折叠为登录态失效，保留 403 状态供线路探测感知；健康检查与登录前刷新客户端缓存，草稿代理探测同步切至 `/api/public/request`。

## 安装与兼容提示
- Android TV 按 ABI 分包提供 `HGhubX-TV_2.2.5_arm64.apk`、`HGhubX-TV_2.2.5_arm.apk`、`HGhubX-TV_2.2.5_x86_64.apk`（`com.hghubx.tv`，`versionCode=2002005`）。
- 沿用与既有 `com.hghubx.tv` 相同的签名证书与单调递增 `versionCode`，支持直接覆盖升级，保留本机收藏、观看记录、线路偏好与启动 PIN。

## 致谢
- 感谢 @zxsos 完成 #256 上游 `dx150-edge-v1` 加密网关紧急适配与 #246 网络恢复改进。

## 验证
- 本地 `make check`（Rust `124 passed; 0 failed; 5 ignored`、文档链接与版本同步 `release-check(tv): ok 2.2.5, android versionCode=2002005`）、`make test-tv`（TV 版本字段与 `VITE_APP_EDITION=tv pnpm build`）以及 `node scripts/test-tv-remote.mjs`（14/14 场景通过）全部通过。
- 实网四作用域联调（`cargo test --lib edge_crypto::tests::live_dx150_edge_all_four_scopes -- --ignored` 对 `https://huangguoai.com`）全通。
- TV 正式安装包由 `tv-v2.2.5` 标签触发的 GitHub Actions `TV Release` 工作流构建并校验。
