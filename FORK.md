# Fork 維護與發版（給 AI agent）

這個 repo 是 `uzairansaruzi/hermex` 的 fork，由 Ian（YI CHENG PAN）以個人開發者帳號
自行編譯、發佈到 TestFlight。`AGENTS.md`（`CLAUDE.md` 是它的 symlink）是上游原作者
寫給貢獻者的規則；其中「maintainer」「App Store」「TestFlight」「push 中繼站」相關段落
指的是原作者的 App，**不適用於這個 fork**。發版、簽署、同步上游以本文件為準；寫程式的
規則（效能、多伺服器、測試）仍照 `AGENTS.md`。

## 最高原則：跟上游的差異降到最低

這個 fork 會長期 merge 上游，任何改到上游檔案的行都可能在之後的 merge 衝突。

- Fork 的設定只放在上游沒有的檔案：`Config/Local.xcconfig`、`FORK.md`、`CLAUDE.local.md`，
  以及 Xcode 建立 Xcode Cloud workflow 時產生的 `HermesMobile.xcodeproj/xcshareddata/xcodecloud/manifest.json`
  （只記錄 product ID，讓 Xcode 認得這個專案已連上 Xcode Cloud）。
- 上游檔案目前只改了 2 行：`HermesMobile.xcodeproj/project.pbxproj` 主程式 target 的
  Debug／Release 各一行 `SUPPORTS_MAC_DESIGNED_FOR_IPHONE_IPAD = YES;`（上游是 `NO`），
  用來在 Apple Silicon Mac 上以「Designed for iPad」執行。
- **不要**改 `Config/Shared.xcconfig`、export options plist、`scripts/`、`.github/`、
  Info.plist 權限文字、App 內的 "Hermex" 字串。想改之前先跟 Ian 確認，並說明會增加哪些衝突風險。
- 確認差異：`git diff upstream/master --stat` 應該只有上面四個 fork 檔案與 pbxproj 2 行。

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
| Git remotes | `origin` = `ianpan119/hermex`（push 這裡）；`upstream` = `uzairansaruzi/hermex`（只 fetch，**永遠不要 push**） |

Team、Bundle ID、App Group 由 `Config/Local.xcconfig` 覆寫：`Shared.xcconfig` 最後一行
`#include? "Local.xcconfig"`。上游把這個檔案列在 `.gitignore`，fork 用 `git add -f` 追蹤它，
所以 Xcode Cloud 的乾淨 clone 也拿得到。`APP_DISPLAY_NAME` 與擴充功能後綴是 project／target
層級設定，優先權高於 xcconfig，無法從 `Local.xcconfig` 改。

## App Store Connect API 工具

API 工具與憑證放在 Ian 的另一個 repo `/Users/ian/Downloads/GPSWarp`：

- `scripts/asc.js`：觸發與查詢 Xcode Cloud、列出 TestFlight build。
- 憑證：`secrets/app-store-connect.enc.json`（SOPS+age），由 `scripts/asc-credentials.js`
  解密到記憶體；需要 `sops` 和 `~/.ssh/id_ed25519`。檢查：`node scripts/asc-credentials.js --check`。
- 這把 API key 屬於個人帳號 `4ZHU768LMQ`。不要把明文憑證寫進任何 repo 或輸出。

以下指令都在 `/Users/ian/Downloads/GPSWarp` 執行。

## 流程一：同步上游

```bash
cd /Users/ian/Downloads/hermex
git status --short            # 必須乾淨；有別人的改動先停下來問
git fetch upstream
git log --oneline master..upstream/master   # 看這次會進來什麼
git merge upstream/master
```

- 衝突最可能出現在 pbxproj 的 `SUPPORTS_MAC_DESIGNED_FOR_IPHONE_IPAD` 附近：保留上游的其他改動，
  只把主程式（`PRODUCT_BUNDLE_IDENTIFIER = "$(APP_BUNDLE_IDENTIFIER)"` 那兩個 configuration）
  的這一行維持 `YES`。擴充功能 target 的這一行維持上游的 `NO`。
- 其他衝突一律以上游為準，fork 不在那些檔案有改動。

Merge 後驗證：

```bash
# 1. 每個 target 仍是個人帳號與 com.pikapan.hermex
xcodebuild -project HermesMobile.xcodeproj -alltargets -configuration Release -showBuildSettings 2>/dev/null \
  | grep -E "^Build settings for|^\s+(PRODUCT_BUNDLE_IDENTIFIER|DEVELOPMENT_TEAM|APP_GROUP_IDENTIFIER|APP_DISPLAY_NAME) ="

# 2. 編譯（只驗證編譯，不要把這個 build 裝到模擬器上，見 AGENTS.md）
xcodebuild -project HermesMobile.xcodeproj -scheme HermesMobile -configuration Debug \
  -destination 'generic/platform=iOS Simulator' CODE_SIGNING_ALLOWED=NO build | tail -3
```

再看上游這次有沒有改到簽署相關的東西，有的話先處理再發版：

```bash
git diff ORIG_HEAD..HEAD --stat -- Config/ '*.entitlements' HermesMobile.xcodeproj/project.pbxproj
```

- **新增 target（新的擴充功能）**：它的 App ID 還沒註冊，Xcode Cloud 會失敗。用本機簽署 archive
  讓 Xcode 自動註冊：
  `xcodebuild -project HermesMobile.xcodeproj -scheme HermesMobile -configuration Release -destination 'generic/platform=iOS' -archivePath <scratch>/Hermex.xcarchive -allowProvisioningUpdates archive`。
  沒有任何 capability 的擴充功能（例如 `.notifications`）Xcode 會用萬用描述檔而不註冊 App ID，
  要用 App Store Connect API `POST /v1/bundleIds` 補註冊。
- **新的 entitlement／capability 或新的 App Group**：同樣用上面的本機簽署 archive 讓 Xcode 開啟。
- **新的 `APP_*` 變數、或 `Shared.xcconfig` 新增身分相關設定**：檢查是否需要在 `Local.xcconfig` 覆寫成 pikapan 的值。

驗證通過後 `git push origin master`。

## 流程二：發佈新的 TestFlight 版本

前提：要發佈的 commit 已經在 `origin/master`（Xcode Cloud 從 GitHub 拉程式碼，不看本機）。

```bash
cd /Users/ian/Downloads/GPSWarp

# 1. 觸發
node scripts/asc.js xcode-cloud start --workflow Default --branch master \
  --repo hermex --bundle-id com.pikapan.hermex
# → 記下輸出的 Build run id

# 2. 追蹤（約 10–15 分鐘；COMPLETE / SUCCEEDED 才算成功）
node scripts/asc.js xcode-cloud status --run-id <run-id>
node scripts/asc.js xcode-cloud actions --run-id <run-id>
node scripts/asc.js xcode-cloud issues --action-id <action-id>   # 失敗時看錯誤；警告很多屬正常

# 3. 確認 TestFlight 處理完成（VALID 才能指派）
node scripts/asc.js testflight list-builds --version <MARKETING_VERSION> --bundle-id com.pikapan.hermex

# 4. 指派給 Internal 群組（必做，否則測試人員在 TestFlight 看不到）
node scripts/asc.js testflight assign-groups --version <MARKETING_VERSION> --build <build-number> \
  --groups Internal --bundle-id com.pikapan.hermex
```

- 確認：`GET /v1/betaGroups/<Internal 群組 id>/builds` 列得到這個 build。測試人員的 `state`
  若是 `NOT_INVITED`，用 `POST /v1/betaTesterInvitations`（app + betaTester）寄出邀請。
  第一次發版時就遇過這兩個問題：群組沒有 build、測試人員未被邀請。

- 觸發後必須確認這次 run 的 source commit 等於 `git rev-parse origin/master`
  （`GET /v1/ciBuildRuns/<run-id>` 的 `sourceCommit.commitSha`）。不符就不要用這個 build。
- 不要只用 `xcode-cloud latest` 判斷結果，以 `start` 回傳的 run id 為準。
- Build number 由 Xcode Cloud 自動遞增；版本號 `MARKETING_VERSION` 沿用上游，不要自己改。
- 外部測試或上架需要另外提交 Beta 審核，先問 Ian。名稱 Hermex 屬於原作者，外部審核可能被拒。

## 不要做的事

- 不要用上游的發版工具：`.github/workflows/release-candidate-testflight.yml`、`scripts/branch-testflight`、
  `scripts/app-store`、`ci/` 底下的 export options。它們指向原作者的 Team `6GYD9C9N6R` 與
  `com.uzairansar.hermesmobile`。這個 fork 的 GitHub Actions 已在 repo 設定中停用。
- 不要碰原作者的 App 或推播中繼站。推播在這個 fork 自動停用（`PushEnvironment.swift` 只接受原作者的
  bundle ID），要用必須自架 `hermex-push` 中繼站。
- 不要 push 到 `upstream`，不要對 `upstream` 開 PR，除非 Ian 明確要求。
- 開始 Xcode Cloud build、上傳、改 App Store Connect 設定、加測試人員之前，先確認 Ian 有要求。

## 已知限制

- 第一個 Xcode Cloud product 只能在 Xcode GUI 建立（API 和網頁都不行）；已建立，之後不需要再做。
- Mac 版是「Designed for iPad」：Live Activity 不顯示，需 Apple Silicon Mac；從 Xcode 直接在這台
  MacBook Pro 跑 debug 版需要它已登記為開發裝置（已登記，UDID `00006000-001A044C1AFA401E`）。
- 系統權限提示與 App 內文字都寫 "Hermex"，與桌面名稱一致；App Store Connect／TestFlight 列表顯示 Hermix。
