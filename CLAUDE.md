# project-jelly-marketplace

설치 가능한 **skill · plugin · MCP 설정**의 소스 저장소(마켓플레이스).
Claude Code · Codex · OpenCode에서 discovery/install 해서 쓴다.

## 레이어 구분

- **이 리포 = install/manage 레이어** — `marketplace.json`, `plugin.json`, `SKILL.md`, `mcp.json`의 *소스*.
- 소비 프로젝트가 부팅 때 읽는 startup instruction(`CLAUDE.md`/`AGENTS.md`)과는 **다른 레이어**다.
- 소비 프로젝트의 instruction 파일엔 한 줄 방침만 두고, 나머지는 skill/MCP가 on-demand로 맡는다.

## 구조

```
project-jelly-marketplace/
├── .claude-plugin/
│   └── marketplace.json      # [카탈로그] 플러그인 목록
└── plugins/                  # [내용물] 각 플러그인
```

> 현재: 기본 세팅 + 빈 카탈로그. 플러그인은 아직 없음.

공개 저장소이니 시크릿·키는 커밋하지 말 것.
