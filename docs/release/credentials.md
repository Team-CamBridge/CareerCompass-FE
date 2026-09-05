# 배포 자격 현황

릴리스 파이프라인이 요구하는 자격이 지금 어디까지 채워져 있는지 한자리에 적는다. 워크플로는 [`release-distribution.yml`](../../.github/workflows/release-distribution.yml)(Firebase App Distribution)과 [`release-play-internal.yml`](../../.github/workflows/release-play-internal.yml)(Play 내부 테스트) 둘이다.

기준일은 2026-09-06 이다. 값을 채우거나 지우면 이 표를 함께 고친다.

## 저장소 시크릿

| 시크릿 | 무엇 | 상태 |
|---|---|---|
| `RELEASE_STORE_FILE_B` | release keystore 를 base64 로 담은 값 | 있음 |
| `RELEASE_STORE_PASSWORD` | keystore 비밀번호 | 있음 |
| `RELEASE_KEY_ALIAS` | `careercompass-release` | 있음 |
| `RELEASE_KEY_PASSWORD` | 키 비밀번호 | 있음 |
| `GOOGLE_SERVICES_JSON_B` | 실제 Firebase 프로젝트의 `google-services.json` | 없음 |
| `KAKAO_NATIVE_APP_KEY` | 카카오 네이티브 앱 키 | 없음 |
| `GOOGLE_WEB_CLIENT_ID` | 구글 OAuth 웹 클라이언트 id | 없음 |
| `GCP_WORKLOAD_IDENTITY_PROVIDER` | GitHub Actions 가 GCP 로 인증하는 제공자 | 없음 |
| `GCP_FIREBASE_SERVICE_ACCOUNT` | App Distribution 업로드용 서비스 계정 | 없음 |
| `GCP_PLAY_SERVICE_ACCOUNT` | Play 업로드용 서비스 계정 | 없음 |

저장소의 `app/google-services.json` 은 실제 프로젝트가 아니라 CI 용 스텁(`careercompass-ci-stub`)이다. 실제 Firebase 프로젝트를 만들기 전까지는 이 파일로 빌드만 돌고 Crashlytics·FCM 은 도착하지 않는다.

## 환경

| 환경 | 상태 |
|---|---|
| `play-internal` | 있음. 승인자 1명, `main` 브랜치에서만 배포 |
| `release-distribution` | 있음. 보호 규칙 없음 |

`release-play-internal.yml` 은 `play-internal` 환경을 요구한다. 환경이 없으면 워크플로가 시작조차 하지 못한다.

## 서명 키

2026-09-06 에 만들었다. 파일과 비밀번호는 1Password 개인 볼트에 있고, 지문과 인계 방법은 [`distribution.md`](distribution.md) 의 「운영 자격 인계」 절에 있다.

Play App Signing 에 등록하면 이 키는 업로드 키가 되고, 잃어버려도 구글에 재설정을 요청할 수 있다. 등록 전까지는 이 키가 유일한 서명 수단이므로 잃어버리면 기존 설치본을 업데이트할 방법이 없다.

## 로컬에서 release 빌드를 돌리려면

`local.properties` 에 네 키(`RELEASE_STORE_FILE`·`RELEASE_STORE_PASSWORD`·`RELEASE_KEY_ALIAS`·`RELEASE_KEY_PASSWORD`)를 넣는다. 네 키는 하나의 단위라 모두 채우거나 모두 지운다.

여기에 더해 `KAKAO_NATIVE_APP_KEY` 와 `GOOGLE_WEB_CLIENT_ID` 가 필요하다. 둘 중 하나라도 비면 빌드가 release 태스크에서 스스로 멈춘다. 빈 키로 만든 release APK 는 소셜 로그인이 동작하지 않기 때문이다.

2026-09-06 기준으로 서명 자체는 확인했다. 두 소셜 키에 임시 값을 넣어 `:app:assembleRelease` 를 돌렸더니 R8 축소와 리소스 축소를 거쳐 4.8MB APK 가 나왔고, `apksigner verify` 가 v2 서명을 확인했으며 인증서 지문이 위 키와 같았다. 그 APK 는 소셜 로그인이 되지 않는 임시 값으로 만든 것이라 그 자리에서 지웠다.
