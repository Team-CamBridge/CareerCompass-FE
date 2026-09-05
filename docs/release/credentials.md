# 배포 자격 현황

릴리스 파이프라인이 요구하는 자격이 지금 어디까지 채워져 있는지 한자리에 적는다. 워크플로는 [`release-distribution.yml`](../../.github/workflows/release-distribution.yml)(Firebase App Distribution)과 [`release-play-internal.yml`](../../.github/workflows/release-play-internal.yml)(Play 내부 테스트) 둘이다.

기준일은 2026-09-06 이다. 값을 채우거나 지우면 이 표를 함께 고친다.

## 저장소 시크릿

| 시크릿 | 무엇 | 상태 |
|---|---|---|
| `RELEASE_STORE_FILE_B64` | release keystore 를 base64 로 담은 값 | 있음 |
| `RELEASE_STORE_PASSWORD` | keystore 비밀번호 | 있음 |
| `RELEASE_KEY_ALIAS` | `careercompass-release` | 있음 |
| `RELEASE_KEY_PASSWORD` | 키 비밀번호 | 있음 |
| `GOOGLE_SERVICES_JSON_B64` | Firebase 프로젝트 `careercompass-fe` 의 설정 | 있음 |
| `GOOGLE_WEB_CLIENT_ID` | 구글 로그인용 OAuth 웹 클라이언트 id | 있음 |
| `GCP_WORKLOAD_IDENTITY_PROVIDER` | GitHub Actions 가 GCP 로 인증하는 제공자 | 있음 |
| `GCP_FIREBASE_SERVICE_ACCOUNT` | App Distribution 업로드용 서비스 계정 | 있음 |
| `GCP_PLAY_SERVICE_ACCOUNT` | Play 업로드용 서비스 계정 | 있음 |
| `KAKAO_NATIVE_APP_KEY` | 카카오 네이티브 앱 키 | 있음 |

이름의 `_B64` 접미사에 주의한다. `_B` 로 넣으면 워크플로가 빈 값을 읽고, 그 빈 값으로 만든 APK 는 서명이 없거나 설정이 비어 있다.

## 카카오

| 항목 | 값 |
|---|---|
| 앱 이름 | CareerCompass (앱 ID 1568218) |
| 등록한 패키지명 | `com.cambridge.careercompass_fe` |
| 등록한 키 해시 | release 키, 그리고 이 저장소를 처음 세팅한 기기의 debug 키 |
| 카카오 로그인 | 사용 설정됨 |
| 동의항목 | 닉네임 필수 동의. 나머지는 사용 안 함 |

이메일을 필수 동의로 받으려면 비즈 앱 전환이 필요하고, 그건 사업자 정보나 본인인증을 요구한다. 지금은 닉네임만 받는다. 새 기기에서 debug 빌드로 카카오 로그인을 시험하려면 그 기기의 debug 키 해시를 콘솔에 더 등록해야 한다.

## Firebase 프로젝트

| 항목 | 값 |
|---|---|
| 프로젝트 id | `careercompass-fe` (프로젝트 번호 571807312588) |
| Android 앱 id | `1:571807312588:android:c28a6ccd4b442c14e7f069` |
| 패키지명 | `com.cambridge.careercompass_fe` |
| 등록된 SHA-1 | release 키, 그리고 이 저장소를 처음 세팅한 기기의 기본 debug 키 |
| 로그인 제공업체 | 구글 사용 설정됨. 카카오는 Firebase 밖이라 여기 없다 |

기기를 옮기거나 새 팀원이 debug 빌드로 구글 로그인을 시험하려면 그 기기의 debug 키 SHA-1 을 Firebase 콘솔에 더 등록해야 한다. `./gradlew :app:signingReport` 가 그 값을 준다.

구글 로그인을 켜면서 OAuth 웹 클라이언트가 자동으로 만들어졌다. 그 클라이언트의 id 가 `GOOGLE_WEB_CLIENT_ID` 이고 앱이 쓰는 값이다. 같이 발급된 클라이언트 시크릿은 앱에 필요 없으므로 저장소에도 시크릿에도 넣지 않았다.

## GCP

| 항목 | 값 |
|---|---|
| 워크로드 아이덴티티 풀 | `github` (global) |
| 제공자 | `github`. 발급자는 GitHub Actions OIDC, `repository_owner == 'Team-CareerCompass'` 조건이 걸려 있다 |
| 서비스 계정 | `fad-uploader@careercompass-fe.iam.gserviceaccount.com` (App Distribution 관리자), `play-uploader@careercompass-fe.iam.gserviceaccount.com` |
| 임시 자격 위임 | 두 계정 모두 이 저장소(`Team-CareerCompass/CareerCompass-FE`)의 토큰만 위임받을 수 있다 |

서비스 계정 키 파일은 만들지 않았다. 워크플로가 워크로드 아이덴티티로 단기 토큰을 받으므로 저장소에 키를 둘 이유가 없다.

`play-uploader` 는 만들어 뒀지만 Play Console 에서 그 계정을 앱에 연결해야 실제로 업로드할 수 있다. 그 연결은 계정 등록 이슈가 맡는다.

## App Distribution

테스터 그룹 `careercompass`(표시 이름 CareerCompass QA)를 2026-09-06 에 만들었다. Gradle 설정이 이 별칭을 가리킨다. 지금 이 그룹에는 아무도 없다. 프로젝트 테스터로는 `dnfjddk2@gmail.com` 하나가 등록돼 있고, 그룹에 넣는 것은 콘솔에서 한다.

## 환경

| 환경 | 상태 |
|---|---|
| `play-internal` | 있음. 승인자 1명, `main` 브랜치에서만 배포 |
| `release-distribution` | 있음. 보호 규칙 없음 |

`release-play-internal.yml` 은 `play-internal` 환경을 요구한다. 환경이 없으면 워크플로가 시작조차 하지 못한다.

## 서명 키

2026-09-06 에 만들었다. 파일과 비밀번호는 1Password 개인 볼트에 있고, 지문과 인계 방법은 [`distribution.md`](distribution.md) 의 「운영 자격 인계」 절에 있다.

Play App Signing 에 등록하면 이 키는 업로드 키가 되고, 잃어버려도 구글에 재설정을 요청할 수 있다. 등록 전까지는 이 키가 유일한 서명 수단이므로 잃어버리면 기존 설치본을 업데이트할 방법이 없다.

## 로컬에서 빌드하려면

`local.properties` 에 서명 네 키(`RELEASE_STORE_FILE`·`RELEASE_STORE_PASSWORD`·`RELEASE_KEY_ALIAS`·`RELEASE_KEY_PASSWORD`)를 넣는다. 네 키는 하나의 단위라 모두 채우거나 모두 지운다. release 빌드에는 `KAKAO_NATIVE_APP_KEY` 와 `GOOGLE_WEB_CLIENT_ID` 도 필요하고, 둘 중 하나라도 비면 빌드가 스스로 멈춘다. 빈 키로 만든 release APK 는 소셜 로그인이 동작하지 않기 때문이다.

2026-09-06 기준 실측이다. 자격을 다 채운 뒤 release 빌드가 통과한다. R8 축소와 리소스 축소를 거쳐 4.8MB APK 가 나왔고, `apksigner verify` 가 v2 서명과 인증서 지문을 확인했으며, 매니페스트에 실제 카카오 OAuth 스킴(`kakao<네이티브키>`)이 실렸다. 그 APK 를 App Distribution 에 올려 테스터 그룹까지 배포되는 것도 확인했다.

다만 그 업로드는 이 기기의 firebase CLI 자격으로 한 것이다. GitHub Actions 의 워크로드 아이덴티티 경로는 아직 실행해 보지 못했다. 확인용 카나리 워크플로가 GitHub 에 등록되지 않아 실행할 수 없기 때문이고, 그 문제는 별도로 추적한다.
