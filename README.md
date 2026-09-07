# my-obsidian-pages

`my-obsidian`으로 Obsidian 노트를 정적 사이트로 빌드해 배포하는 저장소다.

## 로컬 실행

`my-obsidian`을 로컬에서 수정하면서 이 저장소를 함께 테스트하려면 아래 순서로 진행한다.

1. `my-obsidian` 저장소를 받거나 수정한다.
2. `my-obsidian` 저장소에서 `npm link`를 실행한다.
3. 이 저장소(`my-obsidian-pages`)로 이동한다.
4. 전역 설치를 갱신한다.

```bash
npm install -g my-obsidian
```

5. 초기 설정 파일이 없으면 프로젝트를 초기화한다.

```bash
my-obsidian init
```

6. 로컬 실행용으로 [`obsidian.config.json`](/Users/choejaeyong/my-obsidian-pages/obsidian.config.json)의 `basePath`를 `/`로 맞춘다.

```json
{
  "basePath": "/"
}
```

7. 사이트를 빌드한다.

```bash
my-obsidian build --config obsidian.config.json
```

8. 정적 파일을 로컬 서버로 확인한다.

```bash
npx http-server dist -p 8000
```

브라우저에서 `http://localhost:8000`으로 접속하면 된다.

## `basePath` 설정

로컬 실행과 GitHub Pages 배포에서는 `basePath`를 다르게 써야 한다.

- 로컬에서는 `basePath`를 `/`로 둬야 정적 리소스 경로를 정상적으로 찾는다.
- GitHub Pages로 배포할 때는 저장소 이름 기준 경로를 사용한다.

예시:

```json
{
  "basePath": "/my-obsidian-pages/"
}
```

즉, 로컬 테스트할 때는 `/`, 실제 배포 전에는 `/my-obsidian-pages/`로 바꾼 뒤 빌드하고 push 하면 된다.

## 글 메타데이터

공개 글은 `created`, `modified`, `description`, `tags`, `status`를 frontmatter에 기록한다. `created` 기준 최신순으로 홈과 전체 글 목록에 표시되며, 없거나 유효하지 않으면 `modified`를 사용한다. `status: draft`, `draft: true`, `published: false`인 글은 배포에서 제외된다.

리팩터링처럼 순서가 있는 글에는 `series`와 `seriesOrder`를 추가하면 시리즈 목차와 이전 글·다음 글 링크가 생성된다. 파일명을 바꾸거나 주소 체계를 변경할 때는 새 주소를 `permalink`로 지정하고, 기존 주소를 `redirectFrom` 배열에 남긴다.

```yaml
---
title: 글 제목
created: "2026-09-07T10:00:00+09:00"
modified: "2026-09-07T11:00:00+09:00"
description: 글의 한 줄 요약
tags: [topic]
status: budding
permalink: /notes/my-post/
redirectFrom:
  - /old/my-post.html
---
```

배포 설정은 깨진 내부 링크·이미지·스크립트·스타일·제목 앵커를 검사하고, `strictLinks: true`일 때 문제가 있으면 배포를 중단한다. 테스트 글은 `status: draft`로 두어 공개 목록에서 제외한다.

## 문제 해결

로컬에서 `basePath`를 `/`로 설정했는데도 정적 리소스를 못 찾는다면 `dist/assets` 디렉터리를 상위 폴더로 옮겨서 경로를 다시 확인해본다.
