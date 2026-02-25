# seonggi.github.io 블로그 사용법

이 저장소는 Jekyll GitBook 테마 기반의 GitHub Pages 블로그입니다.
글은 마크다운(.md) 파일을 `_posts/` 폴더에 넣고 git push 하면 자동으로 사이트에 반영됩니다.

사이트 주소: https://seonggi.github.io

---

## 글 쓰는 법

### 1단계: 마크다운 파일 만들기

`_posts/` 폴더 안에 아래 규칙에 맞는 파일을 만듭니다.

파일 이름 규칙:
```
YYYY-MM-DD-영문제목.md
```

예시:
```
2026-02-25-keycloak-install.md
2026-03-01-gcp-cloud-run.md
```

### 2단계: 글 맨 위에 front matter 작성

파일 맨 위에 아래와 같이 `---`로 감싼 설정 블록을 넣어야 합니다.
이게 없으면 Jekyll이 글로 인식하지 못합니다.

```yaml
---
title: 여기에 글 제목을 한글로 적으세요
author: Seong Gi
date: 2026-02-25
category: GCP
layout: post
---
```

각 항목 설명:
- title: 사이드바와 페이지에 표시될 글 제목
- author: 작성자 이름
- date: 작성 날짜 (YYYY-MM-DD)
- category: 사이드바 대분류 그룹 (아래 참고)
- layout: 항상 `post` 로 고정

### 3단계: 본문 작성

front matter 아래에 일반 마크다운 문법으로 본문을 작성합니다.

```markdown
---
title: Keycloak 설치 및 구성 매뉴얼
author: Seong Gi
date: 2026-02-25
category: 시스템
layout: post
---

## 개요
Keycloak은 오픈소스 IAM 솔루션입니다.

## 설치 방법
1. Docker로 실행
2. 초기 설정 진행

코드 블록도 사용 가능:
```

### 4단계: git push

```bash
git add .
git commit -m "새 글 추가: Keycloak 설치 매뉴얼"
git push
```

push 하고 1~2분 정도 기다리면 https://seonggi.github.io 에 자동 반영됩니다.

---

## 현재 카테고리 목록

사이드바에 표시되는 대분류 카테고리입니다.
글의 front matter에서 `category` 값을 아래 중 하나로 맞추면 해당 그룹 아래에 표시됩니다.

| category 값 | 용도 |
|---|---|
| GCP | Google Cloud Platform 관련 |
| 시스템 | 시스템/인프라 관련 |
| 달빛궁전 | 달빛궁전 관련 |

### 새 카테고리 추가하는 법

별도의 설정 파일 수정 없이, 새 글의 front matter에 새 category 값을 넣으면 자동으로 생깁니다.

예를 들어 "보안" 카테고리를 새로 만들고 싶으면:

```yaml
---
title: 방화벽 설정 가이드
author: Seong Gi
date: 2026-03-15
category: 보안
layout: post
---
```

이렇게 글을 하나 push 하면 사이드바에 "보안" 카테고리가 자동 생성됩니다.

---

## 이미지 넣는 법

### 방법 1: 로컬 이미지 파일 사용

1. `assets/images/` 폴더에 이미지 파일을 넣습니다
2. 글 안에서 아래처럼 참조합니다

```markdown
![설명문구](/assets/images/파일이름.png)
```

### 방법 2: 외부 이미지 URL

```markdown
![설명문구](https://example.com/image.png)
```

---

## 글 삭제하는 법

`_posts/` 폴더에서 해당 .md 파일을 삭제하고 git push 하면 됩니다.

```bash
rm _posts/2026-02-25-gcp-start.md
git add .
git commit -m "글 삭제"
git push
```

---

## 파일 구조 요약

```
seonggi.github.io/
├── _posts/          <- 블로그 글을 넣는 곳 (핵심!)
├── _pages/          <- About, Contact 등 고정 페이지
├── _config.yml      <- 사이트 전체 설정 (제목, 테마 등)
├── assets/          <- 이미지, CSS, JS 등 정적 파일
└── README.md        <- 이 파일
```

---

## 자주 하는 실수

- 파일 이름에 한글을 쓰면 안 됩니다 (영문과 숫자, 하이픈만 사용)
- front matter (`---` 블록)가 빠지면 글이 표시되지 않습니다
- date 형식이 틀리면 빌드 에러가 납니다 (YYYY-MM-DD 형식 지켜주세요)
- category 값은 대소문자를 구분합니다 (GCP와 gcp는 별개 카테고리)
