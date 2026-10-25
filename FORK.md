# Fork 維護與發版（給 AI agent）

這個 repo 是 `uzairansaruzi/hermex` 的 fork，由 Ian（YI CHENG PAN）以個人開發者帳號自行編譯、
發佈到 TestFlight。`AGENTS.md`（`CLAUDE.md` 是它的 symlink）是上游原作者寫給貢獻者的規則；
其中「maintainer」「App Store」「TestFlight」「push 中繼站」相關段落指的是原作者的 App，
**不適用於這個 fork**。發版、簽署、推播、同步上游以本文件為準；寫程式的規則（效能、多伺服器、
測試）仍照 `AGENTS.md`。和 Ian 溝通一律用繁體中文。

## 目前狀態（2026-10-25）

- TestFlight：Hermix 2.0 (1)、2.0 (2) 都在 `Internal` 群組；build 2 起支援推播。
- 下一個 build（3）尚未編譯，會包含 build 2 之後的改動：Hermex 的 MIT 授權聲明（`18c386e`）與上游同步到
  `c6cb3a9` 的 3 個修正（merge `5efa1c6`）。這次合併只驗證了簽署設定與編譯；受影響的 6 個測試類別因 Mac
  記憶體不足（swap 滿）沒在本機跑完，上游合併前已過 CI。要發 build 3 時照「流程二」，只指派給 `Internal`。
- 推播：自架中繼站 `https://hermex-relay.pikapan.me` 已上線，實機驗證過開啟推播、測試通知、
  App 在背景時收到任務完成通知（iPhone 開著 Surge 也正常，不需要額外規則）。
- 平台：iPhone、iPad（上游本來就支援），Apple Silicon Mac（Designed for iPad）。
- **只做內部測試。** build 2 曾通過外部 Beta 審核，但 Ian 決定取消公開測試（2026-10-25）：外部群組 `Beta`
  的公開連結已關閉、群組內沒有 build、沒有測試人員。之後的 build 只指派給 `Internal`，不要送 Beta 審核、
  不要重新開啟公開連結，除非 Ian 明確要求。

## 相關 repo

| Repo | 用途 |
|---|---|
| `/Users/ian/Downloads/hermex`（`ianpan119/hermex`，public） | 本 repo，App 本身 |
| `/Users/ian/Downloads/hermex-push`（`ianpan119/hermex-push`，private） | 自架推播中繼站，部署與憑證見該 repo 的 `FORK.md` |
| `/Users/ian/Downloads/GPSWarp` | App Store Connect API 工具 `scripts/asc.js` 與它的 SOPS 憑證 |
| `/Users/ian/Downloads/flywhere-site` | SOPS + age 憑證管理的參考做法（hermex-push 沿用同一組 recipients） |

## 最高原則：跟上游的差異降到最低

這個 fork 會長期 merge 上游，任何改到上游檔案的行都可能在之後的 merge 衝突。

- Fork 的設定只放在上游沒有的檔案：
  - `Config/Local.xcconfig`：Team、Bundle ID、App Group
  - `FORK.md`、`CLAUDE.local.md`：本文件與讓 Claude Code 自動載入它的入口
  - `HermesMobile.xcodeproj/xcshareddata/xcodecloud/manifest.json`：Xcode 建 workflow 時產生，只記錄 Xcode Cloud product ID
  - `HermesMobile/Resources/ThirdPartyNotices/Hermex-LICENSE.txt`：Hermex 本身的 MIT 授權全文（散布 fork 的授權義務；
    資料夾是 folder reference，放進去就會自動打包，不用改 pbxproj）
- 上游檔案目前只改了三處：
  - `HermesMobile.xcodeproj/project.pbxproj`：主程式 target 的 Debug／Release 各一行
    `SUPPORTS_MAC_DESIGNED_FOR_IPHONE_IPAD = YES;`（上游是 `NO`），讓 Apple Silicon Mac 以「Designed for iPad」執行。
  - `HermesMobile/Push/PushEnvironment.swift`：`PushBundleIdentifier.supported` 加上 `"com.pikapan.hermex"`
    （上一行補逗號），讓這個 build 能向自架中繼站註冊推播。
  - `HermesMobile/Resources/ThirdPartyNotices/NOTICE.txt`：開頭說明段落後加 2 行，註明這是 Hermex 的 fork 並指向
    `Hermex-LICENSE.txt`。衝突時保留上游的套件清單，並確保這 2 行還在。
- **不要**改 `Config/Shared.xcconfig`、export options plist、`scripts/`、`.github/`、Info.plist 權限文字、
  App 內的 "Hermex" 字串，也不要刪除上游檔案。想改之前先跟 Ian 確認，並說明會增加哪些衝突風險。
- 確認差異：`git diff upstream/master --stat` 應該只有上面 5 個 fork 檔案、`project.pbxproj`、`PushEnvironment.swift` 與 `NOTICE.txt`。
- 上游的 `LICENSE` 若有變動（例如年份或作者），同步複製到 `Hermex-LICENSE.txt`。

## 身分與帳號

| 項目 | 值 |
|---|---|
| 開發者帳號 | 個人帳號 YI CHENG PAN，Team `4ZHU768LMQ`（Individual）。**絕不使用公司帳號** TUNG HO STEEL ENTERPRISE CORP.（`G7U89QURDZ`），兩者都登入在這台 Mac 的 Xcode 裡 |
| 主程式 Bundle ID | `com.pikapan.hermex` |
| 擴充功能 | `.shareextension`、`.liveactivitywidget`、`.notifications`（沿用上游後綴，因為後綴寫死在 pbxproj） |
| App Group | `group.com.pikapan.hermex` |
| 桌面顯示名稱 | Hermex（上游原值，`APP_DISPLAY_NAME`） |
| App Store Connect App | 名稱 **Hermix**（"Hermex" 已被原作者使用），App ID `6826159267`，SKU `hermix` |
| Xcode Cloud | product `HermesMobile`，workflow `Default`（只手動觸發、Clean、Archive – iOS、`APP_STORE_ELIGIBLE`），repo `ianpan119/hermex` |
| TestFlight 群組 | `Internal`（內部、允許 Apple Silicon Mac）。雖然開了「可取得所有 build」，Xcode Cloud 的 build 仍不會自動加入，每次都要 `testflight assign-groups`（見流程二） |
| 開發裝置 | 這台 MacBook Pro（UDID `00006000-001A044C1AFA401E`）已登記，可從 Xcode 直接跑 Mac 版 debug |
| Git remotes | `origin` = `ianpan119/hermex`（push 這裡）；`upstream` = `uzairansaruzi/hermex`（只 fetch，**永遠不要 push**） |

Team、Bundle ID、App Group 由 `Config/Local.xcconfig` 覆寫：`Shared.xcconfig` 最後一行
`#include? "Local.xcconfig"`。上游把這個檔案列在 `.gitignore`，fork 用 `git add -f` 追蹤它，
所以 Xcode Cloud 的乾淨 clone 也拿得到。`APP_DISPLAY_NAME` 與擴充功能後綴是 project／target
層級設定，優先權高於 xcconfig，無法從 `Local.xcconfig` 改。

## App Store Connect API 工具

工具與憑證在 `/Users/ian/Downloads/GPSWarp`，以下指令都在那裡執行：

- `scripts/asc.js`：觸發與查詢 Xcode Cloud、列出 TestFlight build、指派測試群組。
- 憑證：`secrets/app-store-connect.enc.json`（SOPS + age），由 `scripts/asc-credentials.js` 解密到記憶體；
  需要 `sops` 和 `~/.ssh/id_ed25519`。檢查：`node scripts/asc-credentials.js --check`。
- 這把 API key 屬於個人帳號 `4ZHU768LMQ`。不要把明文憑證寫進任何 repo 或輸出。

## 流程一：同步上游

```bash
cd /Users/ian/Downloads/hermex
git status --short            # 必須乾淨；有別人的改動先停下來問
git fetch upstream
git log --oneline master..upstream/master   # 看這次會進來什麼
git merge upstream/master
```

- 衝突最可能出現在 `project.pbxproj` 的 `SUPPORTS_MAC_DESIGNED_FOR_IPHONE_IPAD` 附近：保留上游的其他改動，
  只把主程式（`PRODUCT_BUNDLE_IDENTIFIER = "$(APP_BUNDLE_IDENTIFIER)"` 那兩個 configuration）的這一行維持 `YES`，
  擴充功能 target 的這一行維持上游的 `NO`。
- `PushEnvironment.swift` 衝突時保留上游改動，並確保 `supported` 仍含 `"com.pikapan.hermex"`。
- 其他衝突一律以上游為準，fork 不在那些檔案有改動。

Merge 後驗證：

```bash
# 1. 每個 target 仍是個人帳號與 com.pikapan.hermex
xcodebuild -project HermesMobile.xcodeproj -alltargets -configuration Release -showBuildSettings 2>/dev/null \
  | grep -E "^Build settings for|^\s+(PRODUCT_BUNDLE_IDENTIFIER|DEVELOPMENT_TEAM|APP_GROUP_IDENTIFIER|APP_DISPLAY_NAME) ="

# 2. 編譯（只驗證編譯，不要把這個 build 裝到模擬器上，見 AGENTS.md）
xcodebuild -project HermesMobile.xcodeproj -scheme HermesMobile -configuration Debug \
  -destination 'generic/platform=iOS Simulator' CODE_SIGNING_ALLOWED=NO build | tail -3

# 3. 動到推播時，跑推播註冊測試（模擬器 UDID 用 xcrun simctl list devices available 找）
HERMEX_BUNDLE_ID=com.pikapan.hermex scripts/test-sim <udid> --only HermesMobileTests/PushRegistrationTests
```

再看上游這次有沒有改到簽署相關的東西，有的話先處理再發版：

```bash
git diff ORIG_HEAD..HEAD --stat -- Config/ '*.entitlements' HermesMobile.xcodeproj/project.pbxproj
```

- **新增 target（新的擴充功能）**：它的 App ID 還沒註冊，Xcode Cloud 會失敗。用本機簽署 archive 讓 Xcode 自動註冊：
  `xcodebuild -project HermesMobile.xcodeproj -scheme HermesMobile -configuration Release -destination 'generic/platform=iOS' -archivePath <scratch>/Hermex.xcarchive -allowProvisioningUpdates archive`。
  沒有任何 capability 的擴充功能（例如 `.notifications`）Xcode 會用萬用描述檔而不註冊 App ID，
  要用 App Store Connect API `POST /v1/bundleIds` 補註冊。
- **新的 entitlement／capability 或新的 App Group**：同樣用上面的本機簽署 archive 讓 Xcode 開啟。
- **新的 `APP_*` 變數、或 `Shared.xcconfig` 新增身分相關設定**：檢查是否需要在 `Local.xcconfig` 覆寫成 pikapan 的值。
- **上游更新了 `hermex-push` 外掛的最低版本**（`HermexPushPlugin.newestVersion`）：App 會在設定裡提示更新主機的外掛，
  外掛沿用上游版本，不需要 fork 處理。

驗證通過後 `git push origin master`。

## 流程二：發佈新的 TestFlight 版本

前提：要發佈的 commit 已經在 `origin/master`（Xcode Cloud 從 GitHub 拉程式碼，不看本機）。

```bash
cd /Users/ian/Downloads/GPSWarp

# 1. 觸發，記下輸出的 Build run id
node scripts/asc.js xcode-cloud start --workflow Default --branch master \
  --repo hermex --bundle-id com.pikapan.hermex

# 2. 追蹤（約 10 分鐘；COMPLETE / SUCCEEDED 才算成功）
node scripts/asc.js xcode-cloud status --run-id <run-id>
node scripts/asc.js xcode-cloud actions --run-id <run-id>
node scripts/asc.js xcode-cloud issues --action-id <action-id>   # 失敗時看錯誤；警告很多屬正常

# 3. 確認 TestFlight 處理完成（VALID 才能指派；通常編譯完成後幾分鐘內）
node scripts/asc.js testflight list-builds --version <MARKETING_VERSION> --bundle-id com.pikapan.hermex

# 4. 指派給 Internal 群組（必做，否則測試人員在 TestFlight 看不到）
node scripts/asc.js testflight assign-groups --version <MARKETING_VERSION> --build <build-number> \
  --groups Internal --bundle-id com.pikapan.hermex
```

- 觸發後必須確認這次 run 的 source commit 等於 `git rev-parse origin/master`
  （`GET /v1/ciBuildRuns/<run-id>` 的 `sourceCommit.commitSha`）。不符就不要用這個 build。
- 不要只用 `xcode-cloud latest` 判斷結果，以 `start` 回傳的 run id 為準。
- 指派後確認 `GET /v1/betaGroups/<Internal 群組 id>/builds` 列得到這個 build。測試人員的 `state` 若是
  `NOT_INVITED`，用 `POST /v1/betaTesterInvitations`（app + betaTester）寄出邀請。第一次發版就遇過這兩個問題。
- Build number 由 Xcode Cloud 自動遞增；版本號 `MARKETING_VERSION` 沿用上游，不要自己改。
- 目前只做內部測試（見「目前狀態」）。外部測試或上架都要先問 Ian：名稱、圖示與 App 內連結都屬於原作者，
  對外散布有仿冒與授權上的顧慮。

## 推播（自架中繼站）

- 中繼站：私有 repo `ianpan119/hermex-push`，Cloudflare Worker `hermex-relay`，網址 **`https://hermex-relay.pikapan.me`**。
  部署、憑證、同步上游照那個 repo 的 `FORK.md`。
- 外掛沿用上游 `uzairansaruzi/hermex-push`（App 寫死從那裡安裝），只把中繼站換掉。
- Hermes 主機的 `<Hermes 根目錄>/.env`（預設 `~/.hermes/.env`）已設 `HERMEX_PUSH_RELAY_URL=https://hermex-relay.pikapan.me`。
  換主機或重建時，**必須在手機開啟推播之前設定**：主機沒設時，App 會寫入原作者的中繼站
  （`HermexPushPlugin.defaultRelayURL`），而原作者的中繼站不接受 `com.pikapan.hermex`。外掛會自己重新讀取這個值，不需要重啟。
- TestFlight／Release build 的 APNs 環境是 `production`（`APS_ENVIRONMENT`），中繼站兩種環境都支援。
- 驗證：App 的設定 → 通知 →「傳送測試通知」測的是中繼站 → APNs → 手機解密；要測完整鏈路，讓 Agent 跑一個任務並把 App 切到背景。

## 不要做的事

- 不要用上游的發版工具：`.github/workflows/release-candidate-testflight.yml`、`scripts/branch-testflight`、
  `scripts/app-store`、`ci/` 底下的 export options。它們指向原作者的 Team `6GYD9C9N6R` 與
  `com.uzairansar.hermesmobile`。這個 fork 的 GitHub Actions 已在 repo 設定中停用。
- 不要碰原作者的 App 或推播中繼站（`hermex-relay.hermex-relay.workers.dev`）。
- 不要 push 到 `upstream`，不要對 `upstream` 開 PR，除非 Ian 明確要求。
- 開始 Xcode Cloud build、上傳、改 App Store Connect 設定、加測試人員之前，先確認 Ian 有要求。

## 已知限制

- 第一個 Xcode Cloud product 只能在 Xcode GUI 建立（API 和網頁都不行）；已建立，之後不需要再做。
  需要 GUI 操作時，寫 prompt 交給 Codex Desktop 執行。
- Mac 版是「Designed for iPad」：Live Activity 不顯示，需要 Apple Silicon Mac。
- 系統權限提示與 App 內文字都寫 "Hermex"，與桌面名稱一致；App Store Connect／TestFlight 列表顯示 Hermix。
