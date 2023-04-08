# 👨‍🍳 요비(YOBEE) - Back-End 👨‍🍳

## 1️⃣ Specification

### 🌐 Back-End

| Title | Content                                                                                                         |
| ------------ | --------------------------------------------------------------- |
| Design Pattern | MVC pattern                |
| DB             | Maria db 15.1, Redis 5.0.7 |
| Dependency Injection | Gradle 7.6 |
| Strategy | Git Flow |
| Third Party Library | Kakao API |
| Infra | AWS Ubuntu 20.04LTS, Jenkins 2.375.3, Docker 23.0.1 <br> Logspout, PaperTrail |
| Tools | IntelliJ |

### 🤖 AI

| Title | Content                                                                                                         |
| ------------ | --------------------------------------------------------------- |
| Model    | SpacyNLP, ko_pipeline                                                                   |
| DB       | Maria db 15.1                                                                           |
| Pipeline | SpacyNLP, ko_pipeline, SpacyTokenizer, SpacyFeaturizer, RegexFeaturizer, DIETClassifier |
| Strategy | Git Flow                                                                                |
| Third Party Library | Openai, RASA |
| Infra | AWS Ubuntu 20.04LTS, Jenkins 2.375.3, Docker 23.0.1 <br> Logspout, PaperTrail |
| Tools | VScode |

## 2️⃣ Package Structure

```
📦 com.example.yobee
 ┣ 📂 configuration
 ┣ 📂 exception
 ┣ 📂 push
 ┃ ┣ 📂 controller
 ┃ ┣ 📂 message
 ┃ ┗ 📂 service
 ┣ 📂 recipe
 ┃ ┣ 📂 controller
 ┃ ┣ 📂 domain
 ┃ ┣ 📂 dto
 ┃ ┣ 📂 repository
 ┃ ┗ 📂 service
 ┣ 📂 response
 ┣ 📂 review
 ┃ ┣ 📂 controller
 ┃ ┣ 📂 domain
 ┃ ┣ 📂 dto
 ┃ ┣ 📂 repository
 ┃ ┗ 📂 service
 ┣ 📂 s3.service
 ┣ 📂 user
 ┃ ┣ 📂 controller
 ┃ ┣ 📂 domain
 ┃ ┣ 📂 dto
 ┃ ┣ 📂 repository
 ┃ ┗ 📂 service
 ┗ 📂 util

```

```jsx
📦 AI
 ┣ 📂 .rasa\cache
 ┣ 📂 actions
 ┃ ┗ 📂 _pyache_
 ┣ 📂 components
 ┣ 📂 data
 ┃ ┣ 📜 nlu.yml
 ┃ ┣ 📜 rules.yml
 ┃ ┗ 📜 stories.yml
 ┣ 📂 Flask
 ┃ ┣ 🐍 actions.py
 ┃ ┣ 🐳 Dockerfile
 ┃ ┣ 📜 requirements.txt
 ┣ 📂 ko_pipeline-0.0.0
 ┣ 📂 models
 ┣ 📂 tests
 ┣ 📂 tmp
 ┣ 🐳 Dockerfile
 ┣ 📜 config.yml
 ┣ 📜 credentials.yml
 ┣ 📜 domain.yml
 ┗ 📜 endpoints.yml
```

## 3️⃣ Architecture

<image style="width: 850px" src="https://github.com/YOBEE-8th/.github/blob/main/profile/project_contents/img/architecture.png">

## 4️⃣ ERD

<image style="width: 700px" src="https://github.com/YOBEE-8th/.github/blob/main/profile/project_contents/img/erd.png">

## 5️⃣ Role

| [강보성](https://github.com/boham97)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | [김나연](https://github.com/NayeonS2) | [김동현](https://github.com/dhLeoKim)                                                                                                                                                                                                                                                                                         | [홍성민](https://github.com/Hurlang)                                                                                                                                                                                                                                                                                                                                                                              |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <image style="width: 200px" src="https://github.com/YOBEE-8th/.github/blob/main/profile/backend_contents/img/boseong.png">                                                                                                                                                                                                                                                                                                                                                                                           |        | <image style="width: 200px" src="https://github.com/YOBEE-8th/.github/blob/main/profile/backend_contents/img/donghyun.png">                                                                                                                                                                    | <image style="width: 200px" src="https://github.com/YOBEE-8th/.github/blob/main/profile/backend_contents/img/seongmin.jpeg">                                                                                                                                                                                                                                                        |
| [검색](https://github.com/YOBEE-8th/.github/blob/main/profile/backend_contents/search.md) <br> [레시피](https://github.com/YOBEE-8th/.github/blob/main/profile/backend_contents/recipe.md) <br> [임시 비번 발급](https://github.com/YOBEE-8th/.github/blob/main/profile/backend_contents/tmp_password.md) <br> [마이페이지](https://github.com/YOBEE-8th/.github/blob/main/profile/backend_contents/mypage.md) <br> [레시피 리뷰](https://github.com/YOBEE-8th/.github/blob/main/profile/backend_contents/review.md) |        | [데이터 수집](https://github.com/YOBEE-8th/.github/blob/main/profile/backend_contents/data_collect.md) <br> [Rasa](https://github.com/YOBEE-8th/.github/blob/main/profile/backend_contents/rasa.md) <br> [NLU](https://github.com/YOBEE-8th/.github/blob/main/profile/backend_contents/nlu.md) | [CI/CD](https://github.com/YOBEE-8th/.github/blob/main/profile/backend_contents/cicd.md) <br> [Rasa](https://github.com/YOBEE-8th/.github/blob/main/profile/backend_contents/rasa.md) <br> [NLU](https://github.com/YOBEE-8th/.github/blob/main/profile/backend_contents/nlu.md) <br> [로그 집중화](https://github.com/YOBEE-8th/.github/blob/main/profile/backend_contents/log.md) |
