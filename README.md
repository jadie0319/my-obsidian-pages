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

## 문제 해결

로컬에서 `basePath`를 `/`로 설정했는데도 정적 리소스를 못 찾는다면 `dist/assets` 디렉터리를 상위 폴더로 옮겨서 경로를 다시 확인해본다.
