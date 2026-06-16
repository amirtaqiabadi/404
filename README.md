# 404 — بازی موبایلی

یک بازی سبک Pac‑Man روی یک شهر بی‌نهایتِ تولیدشونده (procedural). شما (آبی) و
شش عاملِ مسیریاب (BFS، DFS، A*، Greedy، Dijkstra، Bi‑BFS) با هم مسابقه می‌دهید
تا زودتر به خانهٔ هدف برسید. این پروژه با **Capacitor** برای **Android** و **iOS**
بسته‌بندی شده است.

## ساختار پروژه

```
.
├── www/                  # کل بازی (وب‌اپ)
│   ├── index.html
│   ├── css/style.css
│   └── js/game.js
├── android/              # پروژهٔ نیتیو اندروید (ساختهٔ Capacitor)
├── ios/                  # پروژهٔ نیتیو iOS (ساختهٔ Capacitor)
├── capacitor.config.json # appId, appName, webDir
├── package.json
└── .github/workflows/    # CI برای ساخت APK و iOS
    ├── android.yml
    ├── ios.yml
    └── ios-release.yml
```

- **appId:** `ir.notfound.game`  (نکته: در اندروید هیچ بخشی از appId نمی‌تواند با عدد شروع شود، برای همین از `404` در شناسه استفاده نشد)
- **appName (نام نمایشی):** `NotFound`
- **webDir:** `www`

برای تغییر نام/شناسهٔ اپ، مقادیر `capacitor.config.json` را عوض کنید و سپس
`npx cap sync` بزنید (برای تغییر کاملِ bundle id بهتر است `android/` و `ios/`
را پاک و دوباره `npx cap add` کنید).

## کنترل‌ها

- **موبایل:** swipe روی صفحه یا دکمه‌های جهت‌دار (D‑pad) گوشهٔ پایین‌چپ.
- **دسکتاپ:** کلیدهای جهت یا `WASD`، و `R` برای شروع دوباره.

## توسعهٔ محلی

```bash
npm install
# اجرای بازی در مرورگر (هر سرور استاتیکی کار می‌کند):
npx serve www      # یا: python3 -m http.server -d www
```

بعد از هر تغییر در `www/` باید همگام‌سازی کنید تا پروژه‌های نیتیو به‌روز شوند:

```bash
npx cap sync
```

## ساخت اندروید (APK)

پیش‌نیاز: JDK 17+ و Android SDK (مثلاً از طریق Android Studio).

```bash
npx cap sync android
cd android
./gradlew assembleDebug        # خروجی دیباگ
# یا برای نسخهٔ منتشرشدنی (نیازمند امضا/keystore):
# ./gradlew assembleRelease
```

خروجی APK اینجاست:

```
android/app/build/outputs/apk/debug/app-debug.apk
```

می‌توانید پروژه را در Android Studio هم باز کنید: `npx cap open android`.

## ساخت iOS

پیش‌نیاز: **macOS** + Xcode + CocoaPods.

```bash
npx cap sync ios
cd ios/App && pod install
npx cap open ios       # باز شدن در Xcode
```

سپس در Xcode با Apple Developer account خود Signing را تنظیم کرده و روی دستگاه/شبیه‌ساز
Run بزنید یا از مسیر **Product → Archive** یک `.ipa` بسازید. ساخت یک `.ipa`
قابل‌توزیع نیازمند حساب توسعه‌دهندهٔ اپل و گواهی امضا است.

## ساخت خودکار با GitHub Actions ✅

دو workflow آماده است و با هر push (یا دستی از تب **Actions → Run workflow**) اجرا می‌شود:

- **Build Android APK** → روی `ubuntu-latest`، یک `app-debug.apk` آمادهٔ نصب می‌سازد
  و به‌صورت artifact آپلود می‌کند.
- **Build iOS App** → روی `macos-14`، یک `.app` امضانشده برای شبیه‌ساز می‌سازد (بدون secret).
- **Build iOS IPA (signed)** → یک `.ipa` امضاشدهٔ قابل‌توزیع می‌سازد. فقط دستی از تب
  **Actions → Run workflow** اجرا می‌شود و به secretهای زیر نیاز دارد.

بعد از اجرای موفق، فایل خروجی را از بخش **Artifacts** همان run دانلود کنید.

### secretهای لازم برای `.ipa` امضاشده

در `Settings → Secrets and variables → Actions` این‌ها را اضافه کنید (نیازمند حساب Apple Developer):

| Secret | توضیح |
|---|---|
| `IOS_CERTIFICATE_BASE64` | گواهی توزیع `.p12` به‌صورت base64 (`base64 -i cert.p12`) |
| `IOS_CERTIFICATE_PASSWORD` | رمز فایل `.p12` |
| `IOS_PROVISIONING_PROFILE_BASE64` | پروفایل `.mobileprovision` به‌صورت base64 |
| `APPLE_TEAM_ID` | شناسهٔ ۱۰ کاراکتری Team در Apple Developer |
| `IOS_EXPORT_METHOD` | یکی از `app-store` / `ad-hoc` / `development` / `enterprise` |

> چرا CI؟ ساخت APK به Android SDK و ساخت iOS به macOS نیاز دارد؛ این محیط‌ها روی
> رانرهای گیت‌هاب فراهم‌اند، پس قابل‌اعتمادترین راهِ گرفتنِ خروجیِ APK و iOS همین است.
