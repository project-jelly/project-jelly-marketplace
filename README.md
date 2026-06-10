# project-jelly-marketplace

project-jelly의 설치 가능한 플러그인 마켓플레이스(카탈로그)입니다. **Claude Code와
Codex CLI 양쪽**에서 같은 repo로 설치됩니다 — 카탈로그 파일만 도구별로 둡니다
(`.claude-plugin/marketplace.json`, `.agents/plugins/marketplace.json`).

## 플러그인

| 플러그인 | 설명 |
|---|---|
| [`opsgate`](./plugins/opsgate) | opsgate MCP(runtime + admin 서피스)를 연결하고 사용/자격증명 관리 스킬을 제공. opsgate는 LLM이 secret을 직접 보지 않고 alias로 HTTP/REST API 호출과 Postgres 조회를 수행하는 policy-gated credential broker. |

## 설치

**Claude Code**
```text
/plugin marketplace add project-jelly/project-jelly-marketplace
/plugin install opsgate@project-jelly
/plugin enable opsgate@project-jelly
/mcp        # opsgate OAuth 로그인 (CIMD, 그대로 동작)
```

**Codex CLI**
```sh
codex plugin marketplace add project-jelly/project-jelly-marketplace
codex plugin add opsgate@project-jelly
```

설치 이름은 `<plugin>@<marketplace-name>` 형식이며, `project-jelly`는 카탈로그의
`name` 필드입니다(repo명이 아님).

## 인증

자격증명/토큰은 저장소에 포함되지 않습니다 — 각 사용자가 기기별로 1회 인증합니다.
Claude Code는 `/mcp` 브라우저 로그인이 그대로 동작합니다(opsgate는 CIMD 방식).
**Codex는 `codex mcp login`이 DCR을 요구해 기본 배포에선 실패**할 수 있어, bearer token
또는 authgate DCR 활성화가 필요합니다 — 자세한 내용은
[`plugins/opsgate/README.md`](./plugins/opsgate/README.md)의 *Authentication caveat* 참고.

## 팀 자동 설치 (선택)

소비 프로젝트의 `.claude/settings.json`에 박아두면 그 repo를 여는 사람에게 자동 적용됩니다(신뢰 승인 1회).

```json
{
  "extraKnownMarketplaces": {
    "project-jelly": {
      "source": { "source": "github", "repo": "project-jelly/project-jelly-marketplace" }
    }
  },
  "enabledPlugins": { "opsgate@project-jelly": true }
}
```

## 문서

- 마켓플레이스/플러그인 구조와 동작: [`docs/architecture.md`](./docs/architecture.md)
- opsgate 플러그인 상세: [`plugins/opsgate/README.md`](./plugins/opsgate/README.md)

## 검증

```sh
claude plugin validate .                    # marketplace.json 스키마
claude plugin validate ./plugins/opsgate    # 플러그인 manifest + 컴포넌트
```
