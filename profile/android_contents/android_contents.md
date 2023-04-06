# 안드로이드 개발 컨텐츠

# 👨‍🍳 요비(YOBEE) - Android 👨‍🍳

---

## 1️⃣ Specification

### 📱 Android

| Architecture | CleanArchitecture, MVVM, Single Activity Application |
| --- | --- |
| DI | Hilt |
| Jetpack | Navigation, ViewBinding, DataBinding, ViewModel, LiveData, Lifecycle |
| Network | Retrofit2, OkHttp, Gson |
| Library | Firebase(FCM), Google(Auth), Kakao(Auth), MLKit, Glide, TedPermission, MPAndroidChart, SwipeRefreshLayout, Lottie, PageIndicatorView, SimpleRatingBar, Typer |
| Engine | Google STT Engine, Google TTS Engine |

## 2️⃣ Package Structure

```
📦 com.ssafy.yobee
 ┣ 📂 alarm
 ┣ 📂 ui
 ┃ ┗ 📂 common
 ┃ ┗ 📂 cook
 ┃ ┗ 📂 home
 ┃ ┗ 📂 login
 ┃ ┗ 📂 mypage
 ┃ ┗ 📂 recipe
 ┃ ┗ 📂 recommend
 ┃ ┗ 📂 register
 ┃ ┗ 📂 search
 ┃ ┗ 📂 splash
 ┃ ┗ 📜 MainActivity.kt
 ┃ ┗ 📜 MainViewModel.kt
 ┣ 📂 util
 ┃ ┗ 📂 common
 ┃ ┗ 📂 extension
 ┗ 📜 ApplicationClass.kt
 ┗ 📜 BindingAdapters.kt

📦 com.ssafy.common

📦 com.ssafy.data
 ┣ 📂 local
 ┃ ┗ 📂 local
 ┃ ┃ ┣ 📂 prefs
 ┣ 📂 remote
 ┃ ┗ 📂 common
 ┃ ┗ 📂 datasource
 ┃ ┃ ┗ 📂 dto
 ┃ ┃ ┗ 📂 datasource
 ┃ ┗ 📂 di
 ┃ ┗ 📂 mappers
 ┃ ┗ 📂 repository
 ┃ ┗ 📂 service
 ┗ 📂 util

📦 com.ssafy.domain
 ┣ 📂 model
 ┣ 📂 repository
 ┣ 📂 usecase
 ┗ 📂 utils
```

## 3️⃣ Screenshot

## 4️⃣ Role

| 이민하 | 이수용 | 조수연 |
| --- | --- | --- |
| 홈화면 |  [레시피 목록](https://www.notion.so/14e3e39b97834736bfe7ab88eb212539) | [회원가입](https://github.com/YOBEE-8th/.github/blob/main/profile/android_contents/register.md) |
| 레시피 소개 | 요리 방법 | 로그인
| 완성 사진 분석 | TTS | 회원 정보 관리 |
| 리뷰 | 검색 | 음성 제어 & 질문 |
| FCM | 메뉴 월드컵 | 스플래쉬 |
| | 요리 과정| | 즐겨찾기
| | 카카오 로그인

[민하](https://www.notion.so/67efde060900493f9f19512f70448390)

[수용](https://www.notion.so/14e3e39b97834736bfe7ab88eb212539)

[수연](https://www.notion.so/84daefd7f26f4b5abf33bd0aa06ce3f7)
