---
name: discord-channels-setup
description: |
  Claude Code Channels Discord 플러그인 (Anthropic 공식 `discord@claude-plugins-official`) 4단계 인터랙티브 설치 스킬. 마케터·1인 운영자 기준. 4단계 흐름:
  ① Discord 봇 세팅 (공식 README 기준 7단계)
  ② 정상 작동 검증 (Online · 폰 DM · reply · react · edit · fetch 도구 시연)
  ③ 작업 가능 업무 표 정리 (도구 5개 + 시나리오 8개 + 한계 4개 + 부가 기능 4개)
  ④ Gmail + Google Calendar 결합 실습 (폰 DM → Claude 분석 → 답신)

  자동 호출 트리거:
  - **"디스코드 channels 설치하자"** ⭐ 주요 트리거
  - "Discord channels setup"
  - "Claude Channels Discord 설치"
  - "공식 디스코드 봇 설치"
  - "Channels Discord 봇 만들자"
  - "claude-plugins-official discord"

  특이점:
  - 출처: https://github.com/anthropics/claude-plugins-official/blob/main/external_plugins/discord/README.md
  - 사전 요구: Bun + claude.ai 로그인 + Claude Code v2.1.80+ + Pro/Max (Team/Enterprise 는 관리자 채널 활성화 필수)
  - 노출 도구: `reply` · `react` · `edit_message` · `fetch_messages` · `download_attachment` (5개)
  - 한계: 세션 의존 (`--channels` 활성 세션 떠 있어야 작동), 메시지 조회 최대 100개, Discord 검색 API 미지원, 첨부 ≤10개/25MB
  - cron 무인 자동 발송은 Channels 한계. 무인 알림은 Discord Webhook 별도 권장.
  - 호스팅 옵션: caffeinate / 데스크톱 24시간 / Mac mini / Raspberry Pi / VPS (STEP 4.5 참조)
---

# Discord Channels 설치 + Gmail/Calendar 결합 (Claude Code Plugin Skill)

> 본 스킬은 Anthropic 공식 Channels Discord 플러그인 (`discord@claude-plugins-official`) 만 사용하는 4단계 인터랙티브 설치 가이드.
>
> 출처: [공식 README](https://github.com/anthropics/claude-plugins-official/blob/main/external_plugins/discord/README.md) · [Channels 공식 문서](https://code.claude.com/docs/ko/channels)

## 🎬 스킬 시작 시 메시지

본 스킬이 호출되면 Claude 는 다음과 같이 시작 멘트를 출력:

```
🤖 Discord Channels (공식) 단독 노선 설치를 시작합니다.

핵심 한 가지:

  Channels = 실행 중인 Claude Code 세션으로 이벤트를 푸시하는 MCP 서버.
  Discord 메시지가 도착하면 Claude 가 분석하고 답신.
  ⚠️ 세션이 열려 있는 동안에만 이벤트 도착. 항상 켜진 운영은 백그라운드 프로세스 필요.

────────────────────────────────

총 4단계 · 30~50분 (Gmail/Calendar 실습 포함).

  STEP 1 · Discord 봇 세팅 (15~20분 · 사용자 + Claude)
       1.1 사전 점검 (Bun · Claude Code 버전 · claude.ai 로그인)
       1.2 Bot 생성 (Developer Portal)
       1.3 Message Content Intent 활성화
       1.4 OAuth 6 권한 + 서버 초대
       1.5 플러그인 설치 (/plugin install discord@claude-plugins-official)
       1.6 토큰 구성 (/discord:configure <token>)
       1.7 채널 활성화 재시작 (claude --channels plugin:discord@...)
       1.8 페어링 (DM → 코드 → /discord:access pair → policy allowlist)

  STEP 2 · 정상 작동 검증 (5분 · 사용자)
       2.1 Bot Online 표시 확인
       2.2 폰 DM 양방향 검증
       2.3 reply · react · fetch_messages 도구 1회씩 시연

  STEP 3 · 작업 가능 업무 표 (3분 · Claude 안내)
       3.1 노출 도구 5개 표
       3.2 마케터 시나리오 8개 표
       3.3 다른 MCP 결합 패턴 표
       3.4 한계 4가지 명시

  STEP 4 · Gmail + Google Calendar 결합 실습 (10~15분 · 양방향)
       4.1 사전 점검 (Claude.ai 통합 활성 확인)
       4.2 시나리오 A · 오늘 일정 조회 (폰 DM → Calendar → 답신)
       4.3 시나리오 B · CS 메일 분류 (폰 DM → Gmail → 답신)
       4.4 시나리오 C · 일정 추가 (폰 DM → Calendar 쓰기 → 확인)
       4.5 자동화 확장 안내 (launchd + Webhook)

시작할까요? (y/n)
```

사용자 OK 시 STEP 1 진행.

---

## ⚙️ STEP 1 · Discord 봇 세팅 (공식 README 기준 7단계)

### STEP 1.1 · 사전 점검 (자동 5초)

```bash
# 런타임 3개 자동 확인 (Claude 가 실행)
bun --version          # 1.0+ 필수
claude --version       # v2.1.80+ 필수
# claude.ai 로그인 상태 확인 (수동)
```

게이트:
```
사전 점검 결과:
  - Bun: ✅ 1.x.x
  - Claude Code: ✅ v2.1.123
  - claude.ai 로그인: 사용자가 직접 확인 (Pro/Max OK · Team/Enterprise 는 관리자 채널 활성화 필수)

진행하시려면 y, 막힘은 m
```

### STEP 1.2 · Discord Bot 생성 (사용자 직접 · 3분)

```
브라우저에서:

① https://discord.com/developers/applications 접속 → 로그인
② 우상단 "New Application" → 이름 입력 (예: my-claude-bot · 본인이 정한 봇 이름)
③ 좌측 "Bot" 탭
④ Username 설정 → "Reset Token" → "Yes, do it!" → 토큰 복사
   ⚠️ 토큰은 한 번만 표시. 메모장에 즉시 저장.
   ⚠️ Bot A 토큰 (서드파티) 과 헷갈리지 말 것. 본 봇은 Channels 전용.

다음 (y) / 막힘 (m)
```

📸 캡처 포인트: Bot 탭의 Token 복사 직후 화면

### STEP 1.3 · Message Content Intent 활성화 (사용자 · 1분)

```
같은 Bot 페이지 아래로 스크롤 → "Privileged Gateway Intents" 섹션:

⭐ Message Content Intent 만 토글 ON (Presence·Members 는 불필요)

→ 우측 상단 "Save Changes" 클릭 (녹색 버튼)

완료 (y) / 막힘 (m)
```

⚠️ Intent 누락이 가장 자주 발생하는 실수. 페어링 단계에서 봇이 응답 안 함.

### STEP 1.4 · OAuth 6 권한 + 서버 초대 (사용자 · 3분)

```
좌측 메뉴 "OAuth2" → "URL Generator":

① Scopes (1개):
   ✅ bot

② Bot Permissions (6개):
   ✅ View Channels
   ✅ Send Messages
   ✅ Send Messages in Threads
   ✅ Read Message History
   ✅ Attach Files
   ✅ Add Reactions

③ 화면 하단 "Generated URL" 복사
④ 새 탭에서 URL 열기 → 본인 서버 선택 → "Authorize"
⑤ reCAPTCHA 통과 → "✅ Authorized"
⑥ Discord 서버 회원 목록에 봇이 추가됐는지 확인

다음 (y) / 막힘 (m)
```

⚠️ Bot A (서드파티) 권한 7개와 다름. **공식은 6개만**. `applications.commands` scope 와 `Manage Messages`·`Use Slash Commands` 권한 없음.

### STEP 1.5 · 플러그인 설치 (Claude Code 안에서 · 2분)

현재 Claude Code 세션에서 직접 실행:

```
/plugin marketplace add anthropics/claude-plugins-official
/plugin install discord@claude-plugins-official
/reload-plugins
```

→ `/discord:configure`, `/discord:access` 슬래시 명령이 활성화됨

⚠️ 마켓 미등록 에러 시: `marketplace add` 부터 다시 실행

```
완료 (y) / 막힘 (m)
```

### STEP 1.6 · 토큰 구성 (사용자 + 자동 · 1분)

같은 Claude Code 세션에서:

```
/discord:configure <STEP 1.2 에서 복사한 토큰>
```

→ `~/.claude/channels/discord/.env` 에 자동 저장
→ Channels 가 토큰 관리 (사용자가 .env 직접 편집 안 함)

⚠️ 토큰을 노출하지 않도록 슬래시 명령 입력 시 주의 (히스토리·녹화에 노출 위험)

```
완료 (y)
```

### STEP 1.7 · 채널 활성화 모드로 재시작 (사용자 · 2분)

```
⚠️ 중요: 현재 Claude Code 세션 종료 → 새 터미널 열기

새 터미널에서:

  cd "<YOUR_PROJECT_ROOT>"
  claude --channels plugin:discord@claude-plugins-official

→ Bot 이 Discord 에서 Online 으로 표시
→ Claude Code 세션이 메시지 수신 대기 상태

💡 Tip: 본 스킬을 실행 중인 세션은 일반 claude 모드. Channels 세션은 별 터미널 윈도우에서 가동.

새 터미널에서 시작 완료 (y) / 막힘 (m)
```

📸 캡처 포인트: 새 터미널의 `Channels enabled: discord` 메시지 + Discord 회원 목록 봇 Online

### STEP 1.8 · 페어링 (사용자 · 3분)

Channels 세션이 떠 있는 상태에서:

```
① Discord 앱 (PC 또는 폰) 에서 회원 목록의 봇 우클릭/탭 → Send Message (DM)
② "hi" 또는 아무 메시지 전송
③ 봇이 페어링 코드로 회신 (예: ABCD-1234)
   ⚠️ 응답 없으면: Channels 세션 (--channels) 살아있는지 + Message Content Intent ON 확인

④ Channels 세션 (새 터미널) 으로 돌아가서:

  /discord:access pair ABCD-1234

→ 본인 Discord 계정이 자동 허용목록에 추가

⑤ 허용목록 잠그기 (보안):

  /discord:access policy allowlist

→ 페어링 안 된 사람의 메시지는 자동 폐기

페어링 완료 (y) / 막힘 (m)
```

📸 캡처 포인트: 페어링 코드 회신 DM + Claude Code 의 pair 명령 출력

---

## ✅ STEP 2 · 정상 작동 검증 (5분)

### STEP 2.1 · Bot Online 확인 (사용자 · 30초)

폰 또는 PC Discord 앱:
- 본인 Discord 서버 → 우측 회원 목록 → 봇이 **Online (녹색 점)** 으로 표시되는지

📸 캡처: Discord 회원 목록 Online 상태

### STEP 2.2 · 폰 DM 양방향 검증 (사용자 + Claude · 2분)

폰 Discord 앱:
```
1. 봇에게 DM
2. 다음 입력:

  지금 작업 중인 디렉토리 뭐야?

→ Channels 세션이 메시지 수신
→ Claude 가 cwd 분석 + ls 도구 호출
→ reply 도구로 폰 DM 답신
```

성공 조건:
- 10~30초 이내 답신 도착
- 답신에 cwd 경로 또는 디렉토리 구조 포함
- 새 터미널 (Channels 세션) 에 `<channel source="discord">` 이벤트 + reply 호출 로그 표시

📸 캡처: 폰 DM 양방향 + 새 터미널의 reply 호출

### STEP 2.3 · 도구 5개 시연 (사용자 + Claude · 2분)

폰 DM 으로 다음 4가지 차례로 시험:

```
명령 1 (reply 검증) — 이미 STEP 2.2 에서 검증됨
명령 2 (react)    : "이 메시지에 좋아요 이모지 붙여줘"
   → Claude 가 add_reaction (또는 react) 도구 호출
   → 본인이 보낸 메시지에 👍 이모지 자동 부착

명령 3 (edit_message): "방금 답신 메시지 끝에 '검증 완료' 추가해줘"
   → Claude 가 봇이 직전에 보낸 reply 메시지 편집
   → 편집됨 표시 (edited) 와 함께 텍스트 갱신

명령 4 (fetch_messages): "이 DM 의 최근 10개 메시지 요약해줘"
   → Claude 가 fetch_messages 호출 (최대 100개 조회 가능)
   → 메시지 요약 답신
```

성공 시 STEP 2 완료. 5개 도구 중 4개 (reply·react·edit·fetch) 검증. `download_attachment` 는 STEP 4 에서 첨부 파일 다룰 때 시연.

---

## 📋 STEP 3 · 작업 가능 업무 표

### STEP 3.1 · 노출 도구 5개

| 도구 | 기능 | 제약 |
|---|---|---|
| `reply` | 채널 또는 DM 메시지 전송 (text + 선택적 reply_to + 파일) | 첨부 ≤10개, 각 ≤25MB |
| `react` | 메시지에 이모지 반응 (`👍` 또는 `<:name:id>`) | — |
| `edit_message` | 봇이 보낸 메시지 편집 (진행 상황 업데이트용) | 봇 본인 메시지만 가능 |
| `fetch_messages` | 채널 최근 메시지 조회 (시간순) | 최대 100개, Discord 검색 API 미지원 |
| `download_attachment` | 메시지 첨부 파일 다운로드 | `~/.claude/channels/discord/inbox/` 에 저장. 자동 다운로드 X |

### STEP 3.2 · 마케터 시나리오 8개

| # | 시나리오 | 사용 도구 | 트리거 |
|---|---|---|---|
| 1 | 출장 중 폰에서 "지난 광고 어땠어?" 질의 | `reply` | DM |
| 2 | 봇이 보낸 진행 상황 메시지 5초마다 갱신 | `edit_message` | 자동 |
| 3 | 사용자가 보낸 메시지에 ✅ 처리 완료 표시 | `react` | 자동 |
| 4 | 최근 #cs-inbox 100개 메시지 요약 | `fetch_messages` + `reply` | DM |
| 5 | 첨부된 PDF 분석 → 요약 답신 | `download_attachment` + `reply` | DM |
| 6 | 외부 webhook → Claude 가 자동 디버그 (CI 실패 등) | `reply` | 이벤트 |
| 7 | 폰에서 코드 명령 → 도구 사용 권한 채널 통해 승인 | 권한 릴레이 | DM |
| 8 | iMessage·Telegram 도 같이 띄워서 멀티 채널 운영 | `--channels` 멀티 | 통합 |

### STEP 3.3 · 다른 MCP 결합 패턴

| 결합 | 활용 시나리오 |
|---|---|
| **+ Gmail MCP** (Claude.ai 통합) | 폰 DM "어제 CS 메일" → search_threads → 분류 → reply |
| **+ Google Calendar MCP** | 폰 DM "오늘 일정" → list_events → reply |
| **+ Google Sheets MCP** | 폰 DM "광고 시트 분석" → read_sheet → 인사이트 → reply |
| **+ GA4 MCP** | 폰 DM "어제 트래픽" → run_report → reply |
| **+ Meta·Google Ads MCP** | 폰 DM "ROAS 어떤 캠페인 좋아?" → get_insights → reply |
| **+ Notion MCP** | 폰 DM "이 답변 노션에 저장" → notion-create-page → reply 확인 |

### STEP 3.4 · 한계 4가지

| # | 한계 | 우회 방법 |
|---|---|---|
| 1 | **세션 의존** · `--channels` 활성 세션이 떠 있을 때만 작동 | STEP 4.5 호스팅 옵션 표 참조 |
| 2 | **메시지 검색 미지원** · `fetch_messages` 최대 100개 (시간순) | 100개 이상 분석은 외부 도구 또는 Discord 자체 검색 후 ID 전달 |
| 3 | **무인 cron 발송 불가** · `reply` 는 메시지 수신 후에만 호출 가능 | 단순 알림은 Discord Webhook (`curl -X POST $WEBHOOK_URL`) |
| 4 | **채널/역할 자동 생성 불가** · 5개 도구만 노출 | 채널 구조는 사용자가 수동으로 만들기 |

### STEP 3.5 · 부가 기능 4가지 (공식 문서에서 추가 발견 · 마케터 가치)

| # | 기능 | 역할 | 마케터 활용 시나리오 |
|---|---|---|---|
| 1 | **권한 릴레이** (`relay-permission-prompts`) | Claude 가 도구 사용 권한 필요 시 채널 통해 폰으로 승인 요청 푸시 | 폰 DM "광고 예산 200만원 늘려" → Claude 가 Meta Ads 도구 호출 직전 폰에 ✅ 승인 요청 → 본인 ✅ 누르면 실행 |
| 2 | **멀티 채널** (`--channels` 에 공백 나열) | 한 세션이 Discord + Telegram + iMessage 동시 수신 | `claude --channels plugin:discord@... plugin:telegram@...` · 본인이 메신저 여러 개 쓸 때 통합 |
| 3 | **access 정책 3가지** | `pairing`·`allowlist`·`groups` 로 발신자 권한 분리 | • `allowlist` (본인만) — 기본 권장<br>• `groups` (팀 멤버 + 가족) — 팀 마케팅 운영<br>• `pairing` (오픈 페어링) — 초대형 봇 |
| 4 | **`--dangerously-skip-permissions`** | 권한 프롬프트 모두 우회 (무인 자동화용) | ⚠️ 신뢰 환경 + 폐쇄 네트워크에서만. 잘못 쓰면 Claude 가 사용자 확인 없이 광고비 변경 가능 |

⚠️ **부가 기능 안전 가이드**:
- 권한 릴레이는 신뢰하는 발신자만 허용목록 (`allowlist`) 에 추가. 페어링된 사람이 도구 권한 결정함.
- 멀티 채널 사용 시 토큰·.env 위치 분리 확인 (`~/.claude/channels/discord/.env` · `~/.claude/channels/telegram/.env`)
- `--dangerously-skip-permissions` 는 cron 자동화 외에는 쓰지 말기. 광고 예산·콘텐츠 발행 같은 위험 작업은 권한 릴레이 사용 권장.

---

## 🔗 STEP 4 · Gmail + Google Calendar 결합 실습 (10~15분)

### STEP 4.1 · 사전 점검 (자동 30초)

Claude 가 다음 도구 prefix 가 노출돼 있는지 확인:

```
mcp__claude_ai_Gmail__* (search_threads, get_thread, create_draft, label_message, etc.)
mcp__claude_ai_Google_Calendar__* (list_events, create_event, update_event, etc.)
```

⚠️ 없으면 Claude Desktop → Settings → Connectors → Gmail · Google Calendar 연결 필요 (사용자 OAuth · 약 2분)

게이트:
```
Gmail/Calendar 통합 활성 상태 OK (y) / 활성화 필요 (m)
```

### STEP 4.2 · 시나리오 A · 오늘 일정 조회 (3분)

**준비**: Channels 세션 (`claude --channels plugin:discord@...`) 가 떠 있어야 함

폰 Discord 봇 DM:
```
오늘 일정 알려줘
```

기대 흐름:
```
1. Channels 가 메시지 수신
2. Claude 가 mcp__claude_ai_Google_Calendar__list_events 호출
   - calendarId: primary
   - timeMin: today 00:00
   - timeMax: today 23:59
3. 일정 목록 정리 (시간순 · 위치·참석자 포함)
4. reply 도구로 폰 DM 답신:

  📅 오늘 일정 (3건)

  09:00-10:00 · 마케팅 주간 미팅 (회의실 A)
  14:00-15:00 · 광고 캠페인 검토 (Zoom)
  16:30-17:30 · CS 응대 시간 (오피스)
```

성공 조건:
- [ ] 1분 이내 답신
- [ ] 일정 개수·시간·제목 정확
- [ ] reply 텍스트 가독성 (이모지·줄바꿈)

📸 캡처: 폰 DM (사용자 질의 + Claude 일정 답신)

### STEP 4.3 · 시나리오 B · CS 메일 분류 (5분)

폰 Discord 봇 DM:
```
지난 24시간 CS 라벨 메일 분류해줘
```

기대 흐름:
```
1. Channels 메시지 수신
2. Claude 호출:
   - mcp__claude_ai_Gmail__search_threads
     query: "label:CS newer_than:1d"
3. 결과 N개를 4 카테고리 분류:
   🚨 긴급 (배송 분실·결제 오류·VIP 컴플레인)
   💰 환불·교환
   💬 일반 문의 (사이즈·재입고)
   🚫 스팸
4. 긴급 3건 본문 발췌 (제목·발신자·요지)
5. reply 도구로 폰 DM 답신 (embed 형식 권장)
```

기대 답신 형식:
```
📧 CS 메일 N건 (지난 24h)

🚨 긴급 3건 ─ 즉시 처리 권장
  • [홍길동] 배송 후 3일째 못 받음 — 운송장 분실 가능성
  • [김영희] 결제 2중 청구 — 환불 필요
  • [VIP 박철수] 사이즈 교환 5회째 반복 — 응대 톤 점검

💰 환불·교환 5건  → 일괄 처리 권장
💬 일반 문의 8건  → FAQ 자동 응답 적합
🚫 스팸 2건       → 라벨 처리됨
```

성공 조건:
- [ ] 30초 이내 답신
- [ ] 카테고리 합계 = 전체 메일 수
- [ ] 긴급 정확도 ≥ 90%
- [ ] 발신자 이름·요지 정확

📸 캡처: 폰 DM 답신 (긴 메시지)

### STEP 4.4 · 시나리오 C · 일정 추가 (3분)

폰 Discord 봇 DM:
```
내일 14시에 광고 미팅 1시간 추가해줘. 참석자 본인만.
```

기대 흐름:
```
1. Channels 수신
2. Claude 호출:
   - mcp__claude_ai_Google_Calendar__create_event
     calendarId: primary
     summary: "광고 미팅"
     start: 내일 14:00
     end:   내일 15:00
     attendees: [본인 이메일]
3. 생성된 event 의 htmlLink 받기
4. reply 답신:

  ✅ 일정 추가 완료

  📅 광고 미팅
  🗓 2026-05-27 (수) 14:00-15:00
  👤 참석자: your-email@example.com
  🔗 https://calendar.google.com/event?eid=...

  취소하려면 "취소" 라고 답신해주세요.
```

성공 조건:
- [ ] 30초 이내 답신
- [ ] Google Calendar 실제 일정 생성됨
- [ ] htmlLink 클릭 시 이벤트 페이지 열림

⚠️ 안전: 일정 생성 같은 쓰기 작업은 사용자 확인 게이트 권장. Claude Code 의 도구 권한 정책에서 `mcp__claude_ai_Google_Calendar__create_event` 권한 프롬프트가 채널 통해 사용자 폰으로 릴레이될 수 있음.

📸 캡처: 폰 DM 답신 + Google Calendar 실제 일정 화면

### STEP 4.5 · 호스팅 옵션 + 자동화 확장 안내 (3분)

#### 🏠 STEP 4.5.1 · 호스팅 옵션 5종 (노트북 닫으면 종료 문제 해결)

⚠️ **핵심 문제**: Channels 는 `--channels` 활성 세션이 떠 있어야 메시지 도착. **노트북 닫으면 sleep → 세션 종료 → DM 보내도 답신 없음.**

| # | 옵션 | 1회 비용 | 월 비용 | 복잡도 | 적합 마케터 |
|---|---|---|---|---|---|
| 1 | **`caffeinate -dis`** (노트북 깨어있게) | 0 | 0 (외부 전원 필요) | ⭐ | 임시·테스트·간헐 사용 |
| 2 | **데스크톱 PC 24시간** + caffeinate | 0 (있다면) | 전기료 5~10$ | ⭐ | 책상에 PC 있는 사람 ⭐ 권장 1 |
| 3 | **Mac mini 홈 서버** | 600~1,400$ | 전기료 2~5$ | ⭐⭐ | 본격 운영 ⭐ 권장 2 |
| 4 | **Raspberry Pi 5 (Linux + Claude Code)** | 80~150$ | 전기료 1$ | ⭐⭐⭐ | 기술 관심 있는 사람 (Bun + claude.ai 로그인 까다로움) |
| 5 | **Linux VPS** (DigitalOcean·Hetzner 등) | 0 | 6~12$ | ⭐⭐⭐⭐ | 진입장벽 큼 — claude.ai OAuth 가 VPS 브라우저 없이 어려움 |

#### 💻 권장 1 · 노트북·데스크톱 macOS 에서 caffeinate (가장 간단)

`claude --channels` 세션 시작 전 또는 별 터미널에서:

```bash
# 옵션 A · 가장 강력 (모니터·아이들·시스템 sleep 모두 방지)
caffeinate -dis &

# 옵션 B · 노트북 닫아도 작동 (외부 전원 필수)
caffeinate -dis -t 28800 &   # 8시간 (-t 초 단위)

# Channels 세션 시작
cd "$YOUR_PROJECT_ROOT"
claude --channels plugin:discord@claude-plugins-official
```

⚠️ 노트북 닫기 + caffeinate 사용 시 주의:
- **외부 전원 연결 필수** (배터리만으로는 5분 후 강제 sleep)
- 발열 주의 (가방 안 노트북 절대 금지)
- 환기 잘 되는 곳에 거치
- 종료는 `pkill caffeinate` 또는 터미널 종료

#### 🖥 권장 2 · Mac mini 홈 서버 + launchd 상시 가동

장기 운영에 가장 안정적. launchd plist 골격:

```xml
<!-- ~/Library/LaunchAgents/com.example.claude-discord-channels.plist -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.example.claude-discord-channels</string>

    <key>ProgramArguments</key>
    <array>
        <string>/opt/homebrew/bin/claude</string>
        <string>--channels</string>
        <string>plugin:discord@claude-plugins-official</string>
    </array>

    <key>WorkingDirectory</key>
    <string>/Users/YOUR_USERNAME/YOUR_PROJECT</string>

    <key>RunAtLoad</key>  <true/>
    <key>KeepAlive</key>  <true/>

    <key>StandardOutPath</key>
    <string>/tmp/discord-channels.out.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/discord-channels.err.log</string>

    <key>EnvironmentVariables</key>
    <dict>
        <key>PATH</key>
        <string>/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin</string>
    </dict>
</dict>
</plist>
```

활성화 명령:
```bash
launchctl load ~/Library/LaunchAgents/com.example.claude-discord-channels.plist
launchctl start com.example.claude-discord-channels

# 상태 확인
launchctl list | grep discord-channels

# 종료
launchctl unload ~/Library/LaunchAgents/com.example.claude-discord-channels.plist
```

⚠️ launchd 로 띄운 세션은 백그라운드라 stdin 입력 불가. 폰 DM 통해서만 상호작용.

#### 🎉 STEP 4.5.2 · 본 스킬 마무리

```
Channels + Gmail + Calendar + 호스팅 옵션 + 부가 기능 모두 정리 완료.

이걸 더 강력하게:

  A. 무인 cron 알림 (Discord Webhook · Channels 와 별개):
     크론에서 curl -X POST $WEBHOOK_URL
     → 매주 월요일 09:00 광고 리포트 자동 발송
     → reply 와 달리 세션 의존 없음 (PC 꺼져 있어도 OK)

  B. 멀티 채널 (Telegram + iMessage):
     claude --channels plugin:discord@... plugin:telegram@...
     → 한 세션이 여러 채널 동시 수신

  C. 권한 릴레이 활성화 (광고 예산 변경 같은 위험 작업 대비):
     도구 권한 프롬프트가 폰 DM 으로 푸시되어 ✅/❌ 클릭으로 승인

본 스킬 끝. Part 10 AX 시스템에서 더 깊은 활용:
  - cs-responder · 메일 도착 즉시 자동 응답 + 분류 후 디스코드 통보
  - daily-briefing · 매일 09시 자동 브리핑 (Calendar + Gmail + 광고)
```

---

## 트러블슈팅 (공식 Channels 노선 한정)

| 증상 | 원인 | 해결 |
|---|---|---|
| `/plugin install` 실패 | 마켓플레이스 미등록 | `/plugin marketplace add anthropics/claude-plugins-official` 먼저 |
| `Bun: command not found` | Bun 미설치 | `curl -fsSL https://bun.sh/install \| bash` |
| 봇 Offline 표시 | `--channels` 플래그 누락 또는 Channels 세션 종료됨 | 새 터미널 + `claude --channels plugin:discord@claude-plugins-official` |
| 페어링 코드 안 옴 | Channels 세션 꺼져 있거나 Message Content Intent OFF | STEP 1.3 + STEP 1.7 재확인 |
| `Channels disabled` 시작 경고 | Team/Enterprise 인데 관리자 미활성화 | claude.ai Admin → Claude Code → Channels 활성화 (Pro/Max 는 무관) |
| `Plugin not in allowed list` | 조직이 `allowedChannelPlugins` 제한 설정 | 관리자에게 `discord@claude-plugins-official` 추가 요청 |
| reply 텍스트 안 보임 | 답신은 Discord 에 도착 (터미널엔 도구 호출만 표시) | 폰/PC Discord 앱에서 확인 |
| 권한 프롬프트 응답 없이 일시중지 | 도구 권한 확인 대기 | 채널 권한 릴레이로 폰 승인 또는 `--dangerously-skip-permissions` (신뢰 환경만) |
| 첨부 25MB 초과 발송 실패 | 공식 제한 | 파일 분할 또는 Google Drive 링크 첨부 |
| fetch_messages 100개 이상 못 가져옴 | Discord 검색 API 미지원 | 100개씩 페이지네이션 또는 별 도구 |

## 사전 검증된 설정값

| 항목 | 값 |
|---|---|
| 플러그인 패키지 | `discord@claude-plugins-official` |
| 출처 | https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/discord |
| 런타임 | Bun 1.0+ |
| Claude Code 최소 | v2.1.80 |
| 인증 요구 | claude.ai 로그인 (API Key 인증 불가) |
| Intent 필요 | Message Content Intent 만 (1개) |
| OAuth Scope | `bot` 만 (1개) |
| Bot Permissions (6개) | View Channels · Send Messages · Send Messages in Threads · Read Message History · Attach Files · Add Reactions |
| 토큰 저장 위치 | `~/.claude/channels/discord/.env` (자동 관리) |
| 페어링 정책 기본 | `pairing` (누구나 페어링 코드 받을 수 있음) |
| 페어링 정책 권장 | `allowlist` (`/discord:access policy allowlist`) |
| 노출 도구 | 5개 (`reply`·`react`·`edit_message`·`fetch_messages`·`download_attachment`) |
| 첨부 한도 | 메시지당 ≤10개, 각 ≤25MB |
| 메시지 조회 한도 | 최대 100개 (Discord 검색 API 미지원) |
| 다중 인스턴스 환경변수 | `DISCORD_STATE_DIR` (상태 디렉토리 분리) |
| 다른 채널 플러그인 | `telegram@claude-plugins-official` · `imessage@claude-plugins-official` · `fakechat@claude-plugins-official` (멀티 가능) |

## 보안 가이드

- **Bot Token 은 `~/.claude/channels/discord/.env` 에만 저장**. git commit 절대 금지 (`.gitignore` 에 등록 확인).
- `--channels` 플래그가 활성화된 세션에서는 페어링된 발신자가 도구 권한도 결정할 수 있음. 신뢰하는 본인 계정만 허용목록 (`/discord:access policy allowlist`) 에 추가.
- 토큰 노출 시 Developer Portal → Bot → Reset Token 으로 재발급 + `/discord:configure` 재실행.
- 광고 예산·콘텐츠 발행 같은 위험 작업은 권한 릴레이로 폰 원격 승인 권장. `--dangerously-skip-permissions` 는 신뢰 환경 무인 자동화 외 비추.

## 다음 단계 (스킬 종료 후 활용)

- **무인 cron 알림** : Discord Webhook (`curl -X POST $WEBHOOK_URL`) — Channels 세션 의존 없이 정기 알림
- **상시 가동** : STEP 4.5 호스팅 옵션 표 (caffeinate / Mac mini / launchd plist)
- **멀티 채널** : `--channels plugin:discord@... plugin:telegram@...` — Discord + Telegram + iMessage 동시 운영
- **Gmail/Calendar 외 다른 MCP 결합** : Google Sheets · GA4 · Meta Ads · Notion 등 사용자의 워크플로에 맞게 확장

## 관련 자료

- [Claude Code Channels 공식 문서 (ko)](https://code.claude.com/docs/ko/channels)
- [Discord 플러그인 공식 README](https://github.com/anthropics/claude-plugins-official/blob/main/external_plugins/discord/README.md)
- [Telegram 플러그인](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/telegram)
- [iMessage 플러그인](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/imessage)
- [fakechat 데모 채널 (로컬 테스트용)](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/fakechat)
