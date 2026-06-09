# discord-channels-setup

> **Claude Code Channels Discord 플러그인 + Gmail/Google Calendar 결합 4단계 인터랙티브 설치 스킬.**
> 마케터·1인 운영자가 폰 DM 한 줄로 Claude 호출 + Gmail·Calendar 자동 처리 환경을 30분 안에 구축.

## 무엇을 해주나

이 스킬을 설치하면 Claude Code 안에서 `"디스코드 channels 설치하자"` 같은 자연어 한 줄로 **4단계 인터랙티브 가이드**가 자동 시작됩니다:

| STEP | 내용 | 누가 |
|---|---|---|
| **1** | Discord 봇 발급 → 플러그인 설치 → 페어링 (8단계 게이트형) | 사용자 직접 + Claude 안내 |
| **2** | Bot Online · 폰 DM 양방향 · 도구 4종 시연 검증 | 사용자 + Claude |
| **3** | 노출 도구 5개 + 마케터 시나리오 8개 + 한계 4개 + 부가 기능 4개 표 | Claude |
| **4** | Gmail · Calendar 결합 실습 3 시나리오 (오늘 일정·CS 메일 분류·일정 추가) | 양방향 |

각 단계마다 `다음(y) / 막힘(m)` 게이트로 진행. 막히면 트러블슈팅 11종 자동 안내.

## 설치 (한 줄)

Claude Code 세션에서:

```
/plugin marketplace add steveaimkt/discord-channels-setup-plugin
/plugin install discord-channels-setup@discord-channels-setup
/reload-plugins
```

설치 후 자연어로 호출:

```
디스코드 channels 설치하자
```

## 사전 요구사항

- **Claude Code v2.1.80+** · `claude --version`
- **Bun 1.0+** · `bun --version`
- **claude.ai 로그인** · Pro/Max 권장 (Team/Enterprise 는 관리자 채널 활성화 필수)
- **Discord 무료 계정** + 본인 서버 1개
- **Discord Developer Portal** 접근 (Bot Token 발급용)
- (선택) **Gmail · Google Calendar** Claude.ai 통합 활성 (STEP 4 결합 실습용)

## 폴더 구조

```
discord-channels-setup-plugin/
├── .claude-plugin/
│   └── marketplace.json
└── plugins/
    └── discord-channels-setup/
        ├── .claude-plugin/
        │   └── plugin.json
        └── skills/
            └── discord-channels-setup/
                └── SKILL.md
```

## 주요 기능

### 노출 도구 5개 (공식 Channels Discord 플러그인)

| 도구 | 기능 |
|---|---|
| `reply` | 채널·DM 메시지 전송 (파일 ≤10개/25MB) |
| `react` | 이모지 반응 |
| `edit_message` | 봇이 보낸 메시지 편집 |
| `fetch_messages` | 채널 최근 메시지 (최대 100개) |
| `download_attachment` | 첨부 다운로드 |

### Gmail + Calendar 결합 시나리오 (STEP 4)

- 폰 DM `"오늘 일정 알려줘"` → Calendar 조회 → 답신
- 폰 DM `"지난 24시간 CS 메일 분류"` → Gmail 분류 → embed 답신
- 폰 DM `"내일 14시 광고 미팅 추가"` → Calendar 생성 → htmlLink 답신

### 호스팅 옵션 (노트북 닫으면 종료 문제 해결)

| 옵션 | 비용 | 권장 |
|---|---|---|
| `caffeinate -dis` (노트북 깨어있게) | 0 | 임시·테스트 |
| 데스크톱 PC 24시간 + caffeinate | 전기료 | ⭐ 권장 1 |
| Mac mini 홈 서버 + launchd plist | 600~1,400$ | ⭐ 권장 2 (장기) |
| Raspberry Pi 5 | 80~150$ | 기술 관심자 |
| Linux VPS | 6~12$/월 | 진입장벽 (claude.ai OAuth) |

## 한계

- **세션 의존** — `--channels` 활성 세션이 떠 있을 때만 작동 (호스팅 옵션 표 참조)
- **메시지 검색 미지원** — 최대 100개, 시간순만
- **무인 cron 발송 불가** — `reply` 는 메시지 수신 후에만 호출 가능 (대체: Discord Webhook + `curl`)
- **채널/역할 자동 생성 불가** — 5개 도구만 노출

## 보안

- Bot Token 은 `~/.claude/channels/discord/.env` (Channels 자동 관리) 에만 저장. `.gitignore` 필수.
- `/discord:access policy allowlist` 로 본인 외 발신자 차단 권장.
- `--dangerously-skip-permissions` 는 신뢰 환경 무인 자동화 외 사용 금지.
- 광고 예산·콘텐츠 발행 같은 위험 작업은 권한 릴레이로 폰 원격 승인 권장.

## 라이선스

MIT License — 자유롭게 fork·수정·공유 가능. 출처 표기만 부탁드립니다.

## 출처 & 크레딧

- [Claude Code Channels 공식 문서 (ko)](https://code.claude.com/docs/ko/channels)
- [Discord 플러그인 공식 README](https://github.com/anthropics/claude-plugins-official/blob/main/external_plugins/discord/README.md)
- [anthropics/claude-plugins-official 마켓플레이스](https://github.com/anthropics/claude-plugins-official)

## 기여

이슈·PR 환영합니다. 사용 중 막힌 부분이나 트러블슈팅 보강 제안 받아요.
