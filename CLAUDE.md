# project-jelly-marketplace

project-jelly의 **소스 저장소(마켓플레이스)**. 설치 가능한 Claude Code · Codex · OpenCode용
**skill · plugin · MCP 설정**을 여기서 *author* 하고, 실제 사용처(다른 리포)에서 *discovery/install* 한다.

## 레이어 구분 (헷갈리지 말 것)

- **이 리포 = install/manage 레이어** — `marketplace.json`, `plugin.json`, `SKILL.md`, `mcp.json`의 *소스*.
- **startup instruction 레이어와 다르다.** 소비 프로젝트가 부팅 때 읽는 건 그쪽 리포의
  `CLAUDE.md`(Claude Code) / `AGENTS.md`(Codex·OpenCode)다 — 이 리포가 아니다.
- 소비 프로젝트의 instruction 파일엔 **한 줄 방침만** 둔다:
  > 이 프로젝트는 설치된 project-jelly skills와 MCP tools를 우선 활용한다.
  여기 내용을 통째로 복붙해 README 괴물을 만들지 말 것. 나머지는 skill/MCP가 on-demand로 맡는다.

## ⚠️ 이 리포는 PUBLIC — 절대 넣지 말 것

- 시크릿/토큰/키/내부 URL/IP
- 내부 인프라 토폴로지, 비공개 리포명, 크리덴셜 인벤토리, 알려진 약점
- deployment-specific 런북 → **private에 격리** (예: `multi-cluster-handbook`)

공개해도 되는 범위는 *도구 일반 사용법*까지 (예: opsgate MCP를 alias로 호출하는 일반 패턴).
"우리 배포(deployment)"가 아니라 "도구(tool) 자체"만 여기 둔다.

## 구조

```
project-jelly-marketplace/
├── .claude-plugin/
│   └── marketplace.json      # [카탈로그] 플러그인 목록 (아직 없음)
└── plugins/                  # [내용물] 각 플러그인 (아직 없음)
```

> 현재: 기본 세팅만. 플러그인은 아직 추가하지 않음.
