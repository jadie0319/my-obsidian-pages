# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 이 저장소의 역할

Obsidian 노트를 GitHub Pages 블로그(`https://jadie0319.github.io/my-obsidian-pages`)로 발행하는 **발행 전용 저장소**다. 소스 코드는 없고, 정적 사이트 생성기 `my-obsidian`(별도 npm 패키지, 소스는 `~/my-obsidian`)에 넘길 마크다운과 설정만 있다.

- `vault/` — 발행할 노트와 첨부(`vault/attachments/`). **원본 볼트가 아니다.** 원본 볼트의 `/publish` 스킬(2026-09-29부터; 그 전에는 markdown-export 플러그인)이 내보낸 사본이다.
- `obsidian.config.json` — `my-obsidian build`의 설정. 사이트 메타, `basePath`, 링크 검사 옵션.
- `dist/` — 빌드 산출물. gitignore 되어 있고 CI가 매번 새로 만든다. 로컬 미리보기 용도로만 존재.
- `.github/workflows/deploy.yml` — `main` push → `npm install -g my-obsidian@latest` → build → Pages 배포.

원본 Obsidian 볼트는 iCloud에 있다: `~/Library/Mobile Documents/iCloud~md~obsidian/Documents/Jadie icloud/`. 볼트 쪽 CLAUDE.md와 발행 작업 로그(`95.Vault/work_result_blog.md`), 발행 절차 노트(`98.Resource/obsidian/옵시디언 노트를 블로그로 발행하는 절차.md`)가 파이프라인의 결정 이력을 담고 있으니, 발행 규칙을 바꾸기 전에 참고한다. 사용자 홈 디렉터리가 기기마다 다르다(`jdragon` / `choejaeyong`). README의 절대경로는 개인 Mac 기준이므로 그대로 쓰지 말고 현재 위치에서 유도한다.

## 명령어

테스트·린트는 없다. 빌드와 미리보기만 있다.

```bash
# 빌드 (전역 설치된 my-obsidian 사용; npm 최신 1.4.0, 2026-09-29)
my-obsidian build --config obsidian.config.json

# 로컬 미리보기
npx http-server dist -p 8000
```

로컬 미리보기 시 `obsidian.config.json`의 `basePath`를 `/`로 바꿔야 정적 리소스를 찾는다. **배포 전에 반드시 `/my-obsidian-pages/`로 되돌린다.** 이 값은 커밋 대상이므로 미리보기용 변경을 커밋하지 않도록 주의한다.

발행은 `main`에 push 하는 것으로 끝난다. 별도 배포 명령 없음.

## 발행 파이프라인

**발행은 원본 볼트의 `/publish <제목>` 스킬로 한다(2026-09-29부터). 플러그인은 비상용.** 스킬은 볼트의 Claude 세션에서 실행되며(`<볼트>/.claude/skills/publish/publish.py`), 노트와 이미지를 플러그인과 같은 규칙으로 `vault/`에 복제하고 이 저장소에서 commit·push까지 한다. 관문: `(No Upload)` 경로는 무조건 거부, `origin: mine`이 아니면 거부, 미발행 노트 링크는 `--unlink-missing`으로 사본에서만 텍스트화. 기존 발행본이 있으면 그 프론트매터를 보존하고 본문만 갱신한다. 설계: 볼트 `95.Vault/publish 스킬 설계와 구현 계획.md`.

```
원본 볼트 노트 ──(/publish: 복제 + 관문 + commit/push)──> vault/*.md
   ──> GitHub Actions ──(my-obsidian build)──> GitHub Pages
```

아래 플러그인 규칙은 스크립트가 그대로 재현하므로 여전히 유효하다.

### markdown-export 플러그인이 만드는 산출물 규칙

`vault/`의 파일이 왜 그런 모양인지 이해하려면 이 규칙을 알아야 한다.

- **조상 경로가 사라진다.** 폴더를 내보내면 `vault/<폴더명>/`, 노트 하나만 내보내면 `vault/<노트>.md`로 떨어진다. 원본 볼트의 `02.Zettelkasten/002_Notes/Refactoring Study/`가 여기서는 `vault/Refactoring Study/`다.
- **첨부파일 이름은 `md5(원본 파일명)`으로 바뀐다.** `Pasted image 20260227125407.png` → `72b254334dc680739df4629a77c09718.png`. 본문의 `![[…]]`는 `![](attachments/<해시>.png)`로 치환된다.
- **덮어쓰기만 하고 지우지 않는다.** 원본에서 이름을 바꾸거나 삭제한 노트는 `vault/`에 그대로 남는다. 정리는 이 저장소에서 직접 한다.
- 플러그인 설정(`output` 절대경로)은 iCloud로 동기화되므로 다른 기기에서 내보내면 경로가 어긋날 수 있다.

### 발행본 프론트매터 손질

플러그인도 `/publish`도 프론트매터를 변환하지 않고 그대로 복사한다. 지금까지 발행한 글 대부분은 발행 뒤 사람이 `title`, `description`, `permalink`, `redirectFrom`(시리즈는 `series`, `seriesOrder`)을 손으로 넣었다. **`/publish`는 재발행 때 이 손질을 보존한다**(기존 발행본의 프론트매터 유지, 본문만 갱신). 플러그인으로 다시 내보내면 덮어써지니 플러그인은 쓰지 않는다.

## 빌드가 읽는 프론트매터 계약

`my-obsidian`이 해석하는 키. 볼트 전용 키(`origin`, `aliases`, `related`, `source`, `tool`)는 무시된다.

| 키 | 동작 |
|---|---|
| `created`, `modified` | 홈·전체 목록 정렬 기준(`created` 우선, 없으면 `modified`) |
| `title`, `description`, `tags` | 페이지 메타, 태그 페이지(`dist/tags/<tag>/`) 생성 |
| `status: draft` / `draft: true` / `published: false` | 배포 제외. 공개 글은 `status: budding` 사용 중 |
| `series`, `seriesOrder` | 시리즈 목차와 이전/다음 링크 |
| `permalink`, `redirectFrom` | 주소 고정과 옛 주소 리다이렉트. 파일명 변경 시 필수 |

`publish: false`는 **생성기가 무시한다**(과거 git 훅 전용 표식이었고 훅은 제거됨). 발행 제외는 `status: draft`로만 한다.

## 링크 검사 (`strictLinks: true`)

`obsidian.config.json`의 `publishing.strictLinks`가 `true`라서 깨진 링크가 있으면 **CI 빌드가 실패한다.** 가장 흔한 원인:

- 본문의 `[[…]]`가 발행하지 않은 노트를 가리킴. 프론트매터 `related:`는 검사 대상이 아니고 본문만 본다.
- `[[원문]](자막)` 같은 표기가 마크다운 링크 `[텍스트](자막)`로 읽혀 "자막" 파일을 찾는다.

`/publish`가 push 전에 미발행 링크를 잡아 준다(파일 존재 확인). CI가 최종 검사. `false`로 바꾸지 않는다.

## 발행 기준 (사용자 결정, 2026-09-23)

- 블로그에는 **`origin: mine`인 글만** 올린다. AI 요약(`origin: library`, `tool: claude` 등)은 올리지 않는다.
- 테스트 글과 AI 요약 발행본은 2026-09-29에 저장소에서 제거했다(11개 + 첨부 4개). `image_test`만 남아 있다.

## 파일명 주의

노트 파일명은 한글·공백·쉼표를 그대로 쓴다. 쉘에서는 항상 경로를 따옴표로 감싼다. 한글 파일명은 NFC/NFD 정규화 차이로 `find -name`이 오탐하므로 존재 확인은 `[ -e "경로" ]`로 한다. `…에이전틱 .md`처럼 끝에 공백이 있는 파일이 있다.
