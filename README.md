# 물풀 정원 · 안드로이드 앱

이 폴더는 '우리 물풀 정원'을 폰에 설치하는 앱(APK)으로 만드는 프로젝트예요.
아래 두 방법 중 하나만 하면 돼요. PC에 아무것도 안 깔아도 되는 **A 방법**을 추천해요.

---

## A. GitHub로 만들기 (설치 프로그램 없음, 무료)

1. github.com 에 가입하고 로그인해요.
2. 오른쪽 위 **+** → **New repository**
   - 이름: `mulpul-garden`
   - **Private** 선택 (나만 볼 수 있게)
   - **Create repository**
3. 새 저장소 화면에서 **uploading an existing file** 을 눌러요.
   압축을 푼 폴더 **안의 내용물 전부**를 끌어다 놓아요.
   (`www`, `assets`, `keystore`, `.github`, `package.json`, `capacitor.config.json`, `README.md`)
   → 아래 **Commit changes**
   - 맥에서는 `.github` 폴더가 숨겨져 안 올라갈 수 있어요. 그럴 땐 맨 아래
     "`.github` 폴더가 안 올라갔을 때"를 따라 해 주세요.
4. 위쪽 **Actions** 탭으로 가면 **Build APK** 가 자동으로 돌아가요. 5~10분 걸려요.
   (안 돌아가면 왼쪽 **Build APK** → 오른쪽 **Run workflow**)
5. 초록 체크가 뜨면 그 실행을 눌러요. 맨 아래 **Artifacts** 의
   **mulpul-garden-apk** 를 받으면 zip 파일이 와요. 풀면 `app-debug.apk` 가 들어 있어요.
   - 폰 브라우저로 GitHub에 로그인해서 이 화면에서 바로 받아도 돼요.
6. 폰에서 `app-debug.apk` 를 열어 설치해요.
   - "출처를 알 수 없는 앱" 허용을 물으면 허용해 주세요.
   - Play 프로텍트 경고가 뜨면 **무시하고 설치**를 눌러요. 직접 만든 앱이라 뜨는 경고예요.

## B. PC의 Android Studio로 만들기

1. Node.js(LTS)와 Android Studio를 설치해요.
2. 이 폴더에서 터미널을 열고:
   ```
   npm install
   npx cap add android
   npx capacitor-assets generate --android
   npx cap sync android
   npx cap open android
   ```
3. 폰을 USB로 연결하고(개발자 옵션 → USB 디버깅 켜기) Android Studio에서 ▶ Run.
- A 방법으로 만든 앱 위에 B 방법으로 덮어 설치하려면, 먼저 `keystore/debug.keystore` 를
  내 사용자 폴더의 `.android/debug.keystore` 로 복사해 주세요(원래 파일은 따로 보관).
  서명이 같아야 덮어 설치가 돼요.

---

## 지금 수조·정원 옮기기

1. claude.ai 링크 버전에서 **보관함 → 모두 내보내기** → 파일 저장
2. 앱에서 **보관함 → 백업 불러오기** → 그 파일 선택
3. 목록에 추가된 수조를 **열기**

## 앱 업데이트하기

1. 새 `index.html` 을 받아요.
2. GitHub 저장소의 `www` 폴더 → **Add file → Upload files** 로 덮어써요.
3. 자동으로 새 APK가 만들어져요. 받아서 그대로 덮어 설치하면 데이터는 그대로 남아요.

**주의**
- `keystore` 폴더는 지우거나 바꾸지 마세요. 서명이 바뀌면 덮어 설치가 안 돼서
  앱을 지우고 다시 깔아야 하고, 그러면 데이터가 사라져요.
- 업데이트 전에는 **모두 내보내기**로 백업해 두면 안전해요.
- 앱을 삭제하면 안의 수조·정원도 같이 지워져요.

## 알아둘 점

- 인터넷 없이 돌아가요. 저장은 폰 안에만 돼요(계정 연동 없음).
- 글꼴(도현체)은 인터넷이 될 때 불러와요. 오프라인에선 기본 글꼴로 보여요.
- 백업 내보내기는 공유 창으로 열려요. 드라이브, 파일 앱, 카톡 나에게 보내기 등에 저장하면 돼요.

---

### `.github` 폴더가 안 올라갔을 때

저장소에서 **Add file → Create new file**, 이름 칸에 `.github/workflows/build.yml` 을
그대로 입력하고, 아래 내용을 붙여 넣은 뒤 **Commit changes**.

```yaml
name: Build APK

on:
  workflow_dispatch:
  push:
    branches: [main, master]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 17
      - name: Install
        run: npm install
      - name: Add Android platform
        run: npx cap add android
      - name: App icon
        run: npx capacitor-assets generate --android
        continue-on-error: true
      - name: Sync web files
        run: npx cap sync android
      - name: Fixed signing key (so updates keep your data)
        run: |
          mkdir -p ~/.android
          cp keystore/debug.keystore ~/.android/debug.keystore
      - name: Build APK
        run: |
          cd android
          chmod +x gradlew
          ./gradlew assembleDebug
      - uses: actions/upload-artifact@v4
        with:
          name: mulpul-garden-apk
          path: android/app/build/outputs/apk/debug/app-debug.apk
```
