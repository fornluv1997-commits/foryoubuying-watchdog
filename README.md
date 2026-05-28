# foryoubuying-watchdog

Oracle Cloud SMS Gateway 외부 watchdog. GitHub Actions cron으로 5분마다 `/health` 폴링하고, 다운 또는 30분+ heartbeat 정체 시 텔레그램으로 즉시 알림.

## 왜 외부에 있나

In-server watchdog (`/opt/sms_gateway/server/watchdog.py`)는 자기 자신이 죽었을 때 알림을 못 보냄. 2026-05-25~28 사고에서 3일간 무알림으로 이 패턴이 확인됨. 이 워크플로우는 GitHub 인프라에서 돌아서 Oracle 인스턴스 전체가 hang/down일 때도 정상 동작함.

## Secrets

- `TELEGRAM_BOT_TOKEN`
- `TELEGRAM_CHAT_ID`
- `SERVER_URL` (예: `https://foryoubuying.duckdns.org`)

## 동작

- 5분마다 `/health` 호출
- HTTP 200 아니면 10초 간격 3회 재시도
- 다운 확정 시 텔레그램으로 owner alert
- `last_heartbeat`가 30분 이상 stale이면 맥북 alert
- spam 방지는 cron 5분 간격 자체 + 텔레그램 사일런스로 처리
