# 血压心率记录 App（Android）

把现有的单文件网页应用 `index.html` 封装成的原生 Android 应用。数据存储在手机本地
（WebView 的 localStorage），完全离线可用；CSV/JSON 导出会通过系统分享面板保存或发送。

- 应用名：血压心率记录
- 包名：com.example.bphr
- minSdk 26（Android 8.0）/ targetSdk 34

## 目录结构

```
android-app/
├── settings.gradle
├── build.gradle
├── gradle.properties
├── gradle/wrapper/gradle-wrapper.properties   # gradlew 由首次构建自动生成
├── .github/workflows/build-apk.yml      # GitHub Actions 云端自动打包
└── app/
    └── src/main/
        ├── AndroidManifest.xml
        ├── assets/index.html            # 网页应用本体（由 ../index.html 同步）
        ├── java/com/example/bphr/MainActivity.java
        └── res/                         # 图标 / 主题 / 字符串 / FileProvider
```

## 一、Android Studio 一键打包（推荐，最省心）

1. 用 Android Studio（Hedgehog 或更新版，自带 JDK 17 与 SDK）打开本目录。
2. 首次打开等待 Gradle Sync 完成（会自动补齐 `gradle/wrapper/gradle-wrapper.jar`）。
3. 菜单 **Build → Build Bundle(s) / APK(s) → Build APK(s)**。
4. 完成后点击提示里的 **locate**，得到：
   `app/build/outputs/apk/debug/app-debug.apk`
5. 传到手机点按安装即可（需允许「安装未知来源应用」）。

命令行打包（可选）：本机装有 Gradle 时，先在项目根目录执行
`gradle wrapper --gradle-version 8.2` 生成 `gradlew`，再执行
`./gradlew assembleDebug`（Windows 用 `gradlew.bat assembleDebug`）。
用 Android Studio 打开会自动生成 wrapper，无需手动执行。

## 二、GitHub Actions 云端打包（本机不用装任何东西）

适合本机不想装 Android Studio 的情况。工作流已配置好：自动安装 JDK 17、Android SDK，
并用 `gradle wrapper` 生成构建脚本，最后产出 debug APK。

> ⚠️ **最关键的一点**：仓库**根目录**必须直接是 `build.gradle`、`settings.gradle`、
> `.github`、`app` 这些内容，**不要再套一层 `android-app` 文件夹**。
> 否则 GitHub 找不到 `.github/workflows`（工作流必须位于仓库根目录的 `.github/workflows`）。

**1. 建仓库**
- 登录 GitHub（github.com，免费）→ 右上角 **+ → New repository**，
  名字随意（如 `bp-hr-app`），Public/Private 均可 → **Create**。

**2. 上传代码（二选一）**
- 方式 A · 命令行（需先装 Git）：
  ```bash
  cd android-app                 # 进入工程目录
  git init
  git add .
  git commit -m "bp-hr app"
  git branch -M main
  git remote add origin https://github.com/你的用户名/bp-hr-app.git
  git push -u origin main
  ```
  这样推上去的仓库根目录就是这些文件本身（符合上面的关键要求）。
- 方式 B · 图形界面：安装 GitHub Desktop → **Add an Existing Repository** 选
  `android-app` 目录 → **Publish**。或在网页 **Add file → Upload files**，
  把工程**内容**（含隐藏的 `.github` 文件夹）拖进去提交。

**3. 触发打包**
- 仓库页切到 **Actions** 标签 → 左侧点 **Build APK** → 右侧
  **Run workflow** → 再点一次 **Run workflow**。
- 等约 3–8 分钟，出现绿色 ✓ 即成功。（每次 push 也会自动触发一次。）

**4. 下载 APK**
- 点进该次运行记录，页面最下方 **Artifacts** 区下载
  `bphr-recorder-debug-apk`（zip），解压得到 `app-debug.apk`，
- 传到手机点按安装（允许「安装未知来源」即可）。

## 三、自定义

- 改应用名：`app/src/main/res/values/strings.xml` 的 `app_name`
- 改配色/主题：`app/src/main/res/values/styles.xml`
- 改图标：`app/src/main/res/drawable/ic_launcher_foreground.xml`
- 改完网页：更新 `index.html` 后，同步到 `app/src/main/assets/index.html` 再重新打包

## 四、关于签名

上面产物是 **debug 签名** APK，可直接安装使用。
若要长期发布到应用商店，请在 Android Studio 里
**Build → Generate Signed Bundle / APK** 生成自己的 keystore 并打出 release 包。
