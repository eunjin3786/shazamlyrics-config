# shazamlyrics-config

ShazamLyrics(iOS) 앱의 **원격 설정** 저장소. 앱은 실행 시 `version.json`을 읽어 서비스 종료/강제 업데이트/공지를 제어한다.

- 앱이 읽는 주소(raw): `https://raw.githubusercontent.com/eunjin3786/shazamlyrics-config/master/version.json`
- 파일은 반드시 **하나의 유효한 JSON 객체**. 모든 필드 optional.
- raw CDN 캐시로 수정 후 반영까지 **최대 ~5분 지연** 가능(앱은 로컬 캐시 무시).
- 문구(title/message/label)는 넣은 문자열이 **그대로** 표시됨(앱이 번역/가공 안 함). 원하는 언어로 직접 작성.

## 우선순위
동시에 여러 개 있으면 **blocked > minimumVersion(강제 업데이트) > notice** 중 하나만 표시.

## 스키마
| 필드 | 타입 | 설명 |
|---|---|---|
| `blocked` | object? | 서비스 종료/차단. 전 버전 대상, **닫기 불가**, 업데이트 버튼 없음 |
| `blocked.title` | string? | 제목 (생략 시 앱 기본 "서비스 종료 안내") |
| `blocked.message` | string | 본문 (필수) |
| `blocked.linkURL` | string? | 안내 페이지 링크. 있으면 버튼 표시, 없으면 버튼 없이 완전 차단 |
| `blocked.linkLabel` | string? | 링크 버튼 라벨 (생략 시 "자세히 보기") |
| `minimumVersion` | string? | 이 버전 **미만**이면 강제 업데이트. 세그먼트 비교(`1.2 < 1.10`). 앱 버전은 `CFBundleShortVersionString` |
| `storeURL` | string? | App Store 링크. 강제 업데이트 "업데이트" 버튼이 여는 주소 |
| `updateMessage` | string? | 강제 업데이트 안내 문구 (생략 시 앱 기본 문구) |
| `notice` | object? | 전면 공지 (**닫기 가능**) |
| `notice.id` | string | 공지 식별자. 사용자가 "확인" 누르면 그 id는 다시 안 뜸. **새 공지는 id를 바꿔야 다시 뜬다** |
| `notice.title` | string? | 제목 |
| `notice.message` | string | 본문 (필수) |

## 예시

아무것도 안 띄우기(기본):
```json
{ "minimumVersion": "0.0.0" }
```

서비스 종료(차단):
```json
{ "blocked": { "title": "서비스 종료 안내", "message": "이 앱은 더 이상 지원되지 않습니다.", "linkURL": "https://...", "linkLabel": "자세히 보기" } }
```

강제 업데이트:
```json
{ "minimumVersion": "2.0.0", "storeURL": "https://apps.apple.com/app/idXXXXXXXXX", "updateMessage": "업데이트가 필요해요" }
```

공지 팝업:
```json
{ "notice": { "id": "2026-07-2", "title": "공지", "message": "안내 문구" } }
```

## 앱 구현 위치(참고)
- 모델/체커: `ShazamLyrics/Model/AppUpdate.swift` — `AppUpdateConfig`, `AppUpdateChecker`(`configURL`이 이 파일의 raw 주소)
- 화면: `ShazamLyrics/View/AppUpdateGate.swift` — `.appUpdateGate()`를 `ContentView`에 적용