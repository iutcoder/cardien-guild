# 카르디엔 서부 모험가 길드 — 초안

기존 아카데미 기준본 index(1).html의 고정 메뉴·섹션·카드형 구성을 바탕으로 만든 별도 사이트입니다. 기존 파일과 R2 에셋은 변경하지 않았습니다.

## 먼저 확인하기

public/index.html을 브라우저에서 열면 바로 작동합니다. 별도 빌드나 서버 없이 화면과 파티 선택을 확인할 수 있습니다.

- 세계관 3개 소개
- 서부 지역 6개 소개
- 이름 미정 / 백은의 왕관 / 수월방(獸月幇) 파티별 인물 카드와 펼침 프로필
- 모바일 레이아웃과 키보드 조작

사용자가 제공한 최신 지역·종족·등급·파티·11명 설정을 반영했습니다. 첫 파티만 이름 미정이며, 나머지는 백은의 왕관과 수월방(獸月幇)입니다. 인물 소개는 짧게 요약했습니다.

## 내용 및 이미지 편집

public/index.html에서 `<script id="site-data" type="application/json">`을 찾으세요. 이 JSON에 세계관, 지역, 파티와 인물 정보가 모여 있습니다.

| 항목 | 수정할 곳 |
| --- | --- |
| R2 공개 도메인 | assetBase |
| 상단 배경 이미지 | heroImage |
| 세계관 제목·설명 | world |
| 지역명·설명·이미지 | regions |
| 파티 이름·소개·구성원 | parties |
| 캐릭터 이름·직업·소개·성격·이미지 | members 안의 name, meta, description, tags, image |

assetBase를 실제 R2 공개 도메인으로 입력하고, image에는 버킷의 실제 객체 경로를 입력합니다. 예를 들어 도메인이 https://assets.example.com이고 객체 키가 guild/characters/hero.webp이면 assetBase는 해당 도메인, image는 guild/characters/hero.webp입니다. 예시 주소는 실제 사용 주소가 아닙니다. image에 https 전체 URL을 넣어도 됩니다.

image가 빈 문자열이면 지역은 텍스트 카드, 인물은 이름 첫 글자를 표시합니다. 아직 정해지지 않은 에셋 경로를 임의로 생성하지 않았습니다. 이미지 읽기 실패 시에도 소개는 유지됩니다. 지역 배너는 1216×384, 캐릭터 이미지는 세로형을 권장하며 object-fit:cover로 일부가 잘릴 수 있습니다.

JSON은 쌍따옴표를 사용하고 마지막 항목 뒤에 쉼표를 넣지 마세요. HTML에 포함되는 JSON이므로 소개문에 HTML 태그나 script 종료 태그를 넣지 마세요.

## 구성

페이지 파일은 Workers Static Assets로, 이미지 파일은 R2 공개 도메인으로 제공합니다. 공개 소개 사이트이므로 별도 API 서버, 데이터베이스, R2 접근키 또는 버킷 바인딩이 필요하지 않습니다. 이 구성은 R2 버킷이 공개 에셋용일 때의 방식입니다.

### GitHub 없이 직접 배포

현재 지원되는 Node.js LTS를 설치한 뒤, 압축을 푼 cardien-guild 폴더에서 터미널을 엽니다.

```sh
npx wrangler@latest login
npx wrangler@latest dev
```

브라우저에서 Cloudflare 로그인을 완료합니다. dev가 출력한 로컬 주소에서 확인하고 Ctrl+C로 종료하세요. 실제 공개 배포를 원할 때 실행합니다.

```sh
npx wrangler@latest deploy
```

wrangler.jsonc의 name은 cardien-guild입니다. 같은 이름의 기존 Worker가 있다면 배포 전에 새 이름으로 변경하세요. 성공 시 출력된 URL에서 확인합니다. 이 전달본은 실제 계정에 배포하지 않았습니다.

### R2 연결

1. Cloudflare에서 이미지용 R2 버킷을 선택합니다.
2. 버킷 Settings의 공개 접근 설정에서 Custom Domain을 연결합니다. 기존 공개 에셋 도메인이 있다면 그대로 사용해도 됩니다.
3. 해당 버킷에 이미지를 업로드하고 브라우저에서 이미지 전체 URL이 열리는지 확인합니다.
4. site-data의 assetBase와 각 image 값을 실제 경로로 입력한 뒤 페이지를 재배포합니다.

R2 관리용 S3 API 주소는 이미지 공개 주소가 아닙니다. r2.dev 개발용 주소보다 운영용 사용자 지정 도메인을 사용하세요. 비밀키를 HTML이나 GitHub에 넣지 않습니다. 현재는 img 표시만 하므로 이미지 편집·캔버스용 CORS 설정은 요구하지 않습니다.

### GitHub 자동 배포를 원할 때

1. GitHub에 빈 저장소를 만들고 이 프로젝트의 내용을 올립니다. 저장소 최상위에 wrangler.jsonc와 public 폴더가 오게 합니다.
2. Cloudflare Workers & Pages에서 새 Worker 생성 시 GitHub 저장소를 연결하거나, 기존 Worker의 Settings → Builds에서 연결합니다.
3. 저장소와 운영 브랜치(main)를 선택합니다.
4. 프로젝트 루트는 저장소 최상위, 빌드 명령은 비움, 배포 명령은 `npx wrangler@latest deploy`로 설정합니다. Worker 이름과 wrangler.jsonc의 name을 일치시키세요.
5. 이후 운영 브랜치에 변경을 올리면 배포하도록 설정할 수 있습니다. 연결 완료 후 실제 빌드 로그와 공개 URL을 확인하세요.

GitHub는 필수가 아닙니다. 첫 디자인 검토는 HTML 파일로 하고, 내용·에셋을 정리한 다음 배포해도 됩니다.

## 공식 문서

- [Workers Static Assets](https://developers.cloudflare.com/workers/static-assets/)
- [R2 공개 버킷과 도메인](https://developers.cloudflare.com/r2/buckets/public-buckets/)
- [Workers Builds와 Git 연동](https://developers.cloudflare.com/workers/ci-cd/builds/)

안내 확인일: 2026-09-21. Cloudflare 화면의 메뉴명은 변경될 수 있습니다.

## 전달 전 확인 상태

HTML 안의 JSON과 JavaScript 문법, Wrangler 설정 JSON 구문을 확인했습니다. 브라우저 실행 파일 다운로드가 네트워크에서 시간 초과되어 실제 화면·모바일·클릭 동작 검증과 Cloudflare 배포 검증은 완료하지 못했습니다.

## 펼침 프로필과 육각형 스테이터스

이미지와 짧은 소개를 포함한 카드 전체가 버튼입니다. 클릭하거나 키보드 Enter/Space를 누르면 해당 카드 줄 아래에 전체 너비 패널이 펼쳐집니다. 같은 카드를 다시 누르거나 패널의 접기 버튼을 누르면 닫힙니다. 파티 전환 시 기존 프로필은 닫힙니다. 화면 폭 변경 시 패널은 선택된 카드 줄 아래로 다시 배치됩니다. 동작 감소 설정에서는 애니메이션을 생략합니다.

평가 항목과 수치는 사용자가 제공한 초안을 반영했습니다.

- statAxes: 완력, 기동, 내구, 마력, 기교, 통찰 (정확히 6개, 순서 고정)
- statMax: 5 (공통 5점 척도)
- 각 members의 stats: 해당 순서의 수치 6개

현재 11명 모두 사용자 제공 수치가 입력되어 있습니다. 미입력 값을 0점으로 표시하지 않고 그래프의 눈금과 축만 보여줍니다. 6개 값 모두 유효한 숫자일 때만 실제 능력치 도형을 그립니다. 수치는 종족·등급에서 자동 추론하지 않습니다. 0점은 실제 0이며 null과 다릅니다.

예시 형식: stats: [3, 4, 2, 5, 3, 2] (설명용 예시이며 어느 인물에게도 적용하지 않았습니다.)

다음에 필요한 자료: 인물 프로필 이미지와 실제 R2 경로. 지역 이미지는 연결 완료. 별도 스테이터스 이미지를 제작할 필요는 없습니다. SVG 그래프가 수치로 생성됩니다.

## 지역 에셋 연결 완료

첨부된 PNG 7장을 원본 비율 1216×384 그대로 WebP(품질 90)로 변환했습니다. 생성 메타데이터는 전달용 WebP에 복사하지 않았습니다. 원본 PNG는 변경하지 않았습니다.

| 파일 | 연결 위치 |
| --- | --- |
| 01.webp | 베르겐하임 |
| 02.webp | 해오름골 |
| 03.webp | 헤르반 |
| 04.webp | 부러진 창 벌판 |
| 05.webp | 벨마르 |
| 06.webp | 침묵의 폐허 |
| 07.webp | 길드 내부 · 상단 배너 |

현재 public/index.html의 assetBase는 ./assets/map입니다. 압축을 풀어 public/index.html을 열면 첨부 이미지를 로컬에서 읽습니다. 카드와 상단 배너는 원본 비율을 유지합니다.

R2에 public/assets/map 안의 01.webp~07.webp를 객체 키 24/map/01.webp~24/map/07.webp로 업로드한 다음, public/index.html의 site-data에서 assetBase를 https://cc.iutcoder.com/24/map 으로 변경하세요. r2AssetBase 필드는 전환할 주소를 기록한 참고값이며 자동 전환하지 않습니다. 첫 이미지 전체 주소는 https://cc.iutcoder.com/24/map/01.webp입니다. R2 업로드는 이 작업에서 수행하지 않았습니다.

별도로 전달된 cardien-guild-preview.html은 이미지를 내장한 단일 파일로 인터넷 없이도 지역 이미지를 확인할 수 있습니다. 실제 배포·편집은 ZIP의 public/index.html을 기준으로 하세요. 내장 미리보기는 R2 전환의 영향을 받지 않습니다.

## 캐릭터 기본 이미지 임시 연결

사용자가 지정한 코드 A~K를 기본 이미지에 연결했습니다. 가로형 원본에 맞춘 레이아웃 변경은 보류했습니다. 현재 카드 영역에서는 object-fit:cover에 따라 일부가 잘릴 수 있습니다. 지역 이미지는 테스트용 로컬/내장 방식 그대로이며, 캐릭터 이미지는 인터넷 연결과 실제 R2 객체가 필요합니다.

| 코드 | 인물 | URL |
| --- | --- | --- |
| A | 루루 | https://cc.iutcoder.com/24/A/01.webp |
| B | 에르나 | https://cc.iutcoder.com/24/B/01.webp |
| C | 네라 | https://cc.iutcoder.com/24/C/01.webp |
| D | 카일 | https://cc.iutcoder.com/24/D/01.webp |
| E | 오스왈드 | https://cc.iutcoder.com/24/E/01.webp |
| F | 마리아 | https://cc.iutcoder.com/24/F/01.webp |
| G | 엘리아 | https://cc.iutcoder.com/24/G/01.webp |
| H | 하진 | https://cc.iutcoder.com/24/H/01.webp |
| I | 츠네 | https://cc.iutcoder.com/24/I/01.webp |
| J | 린 | https://cc.iutcoder.com/24/J/01.webp |
| K | 호연 | https://cc.iutcoder.com/24/K/01.webp |
