# 원격과 프로토콜, Bare Repository란?

## 한 줄 정의

- **Remote(원격)**: 내 로컬 저장소와 연결된, 다른 위치에 있는 Git 저장소를 가리키는 이름표. 기본 이름은 `origin`, 원본 프로젝트를 따로 가리킬 땐 관례상 `upstream`을 쓴다.
- **프로토콜**: 원격 저장소에 접속하는 방식. HTTPS, SSH, 로컬 파일시스템을 쓰는 File(`file://`), 그리고 Git 고유의 Git(`git://`) 프로토콜이 있다.
- **Bare Repository**: 작업 디렉토리(워킹 트리) 없이 `.git` 내부 내용만 있는 저장소. 사람이 직접 파일을 수정하는 곳이 아니라, 서로 주고받는 용도의 서버 저장소.

## 표로 비교 — HTTPS vs SSH vs File vs Git

| | HTTPS | SSH | File | Git |
|---|---|---|---|---|
| 주소 형태 | `https://github.com/user/repo.git` | `git@github.com:user/repo.git` | `file:///Users/me/repo.git` 또는 로컬 경로 | `git://example.com/repo.git` |
| 인증 방식 | 아이디/비밀번호 또는 토큰(PAT) | SSH 키 쌍 (공개키 등록) | 없음 — 로컬 파일 권한만 있으면 접근 가능 | 없음 — 누구나 접근 가능 (읽기 전용) |
| 암호화 | O | O | 해당 없음(로컬) | X (평문 전송) |
| 최초 설정 | 간단 (토큰만 발급) | SSH 키 생성·등록 필요, 번거로움 | 불필요 (경로만 있으면 됨) | 불필요, `git daemon` 서버만 떠 있으면 됨 |
| 네트워크 | 필요 | 필요 | 불필요 (같은 컴퓨터/마운트된 디스크) | 필요 (기본 포트 9418) |
| 방화벽/사내망 | 대체로 잘 통과 (80/443 포트) | 막혀 있는 경우 있음 (22 포트) | 해당 없음 | 막혀 있는 경우 많음 (9418 포트) |
| 현재 상태 | 표준, 가장 널리 사용 | 표준, 널리 사용 | 실습·로컬 테스트용 | 인증·암호화 부재로 GitHub 등 주요 호스팅에서 지원 종료, 사실상 폐기 |

## 표로 비교 — 일반 저장소 vs Bare Repository

| | 일반 저장소 | Bare Repository |
|---|---|---|
| 워킹 트리 | 있음 (파일을 보고 수정 가능) | 없음 |
| 용도 | 개발자가 직접 작업 | 원격 서버처럼 push/pull을 주고받는 중계용 |
| 생성 명령 | `git init` | `git init --bare` |
| 폴더 구조 | `.git/`이 하위 폴더로 존재 | `.git` 내부 내용이 최상위에 그대로 펼쳐짐 |
| 직접 커밋 | 가능 | 불가능 (워킹 트리가 없어서 파일 수정 자체가 안 됨) |

## Bare Repository의 폴더 구조

일반 저장소의 `.git/` 폴더 안에 있던 내용물이, bare 저장소에서는 껍데기(워킹 트리) 없이 최상위 폴더에 그대로 펼쳐져 있다.

```
my-repo.git/          # bare 저장소 (자체가 곧 .git 내용물)
├── HEAD
├── config
├── description
├── hooks/
├── info/
├── objects/
└── refs/
```

일반 저장소라면 이 목록이 `my-repo/.git/` 안에 숨어 있고, 그 옆에 사람이 보는 파일들(워킹 트리)이 있는 것과 대조된다. `config`/`description`/`hooks/`/`info/`는 각각 설정·메모·자동화 스크립트 보관용 부가 요소이고, 핵심은 아래 세 가지다.

### HEAD
파일을 열어보면 `ref: refs/heads/main` 한 줄이 전부다. clone하는 사람이 아무 옵션 없이 `git clone`만 치면 기본으로 받게 될 브랜치를 정하는 역할. 이 값을 바꾸는 명령이 `git symbolic-ref HEAD refs/heads/develop`이고, GitHub의 "Default branch" 설정도 결국 이걸 바꾸는 것이다.

### objects/
세 종류의 객체가 섞여 저장된다 — **blob**(파일 내용), **tree**(디렉터리 구조, 파일명↔blob 매핑), **commit**(트리 스냅샷 + 부모 커밋 + 메시지). `git cat-file -t <해시>`로 종류를, `-p`로 내용을 볼 수 있다. 저장 경로는 `objects/ab/cdef1234...` (해시 앞 2자리 폴더 + 나머지 38자리)이며, 이런 낱개 파일(loose object)이 쌓이면 `git gc`가 여러 개를 `.pack` 파일 하나로 압축해 `objects/pack/`에 넣는다.

### refs/
`refs/heads/main`, `refs/tags/v1.0`처럼 파일 하나가 커밋 해시 한 줄만 담은 것뿐이다 (`cat refs/heads/main`으로 직접 확인 가능). `heads/*`는 브랜치, `tags/*`는 태그, `remotes/*`는 원격 추적 브랜치를 담는데, bare repo는 그 자체가 다른 저장소들의 origin 역할이라 보통 `remotes/`는 비어 있고 `heads/*`만 채워진다. ref 개수가 많아지면 낱개 파일 대신 `packed-refs` 파일 하나로 압축되기도 한다.

정리하면 `objects/`가 데이터 창고, `refs/`와 `HEAD`는 그 데이터를 가리키는 이름표다.

## 한 줄 요약

Remote는 다른 위치의 저장소를 가리키는 이름표, 프로토콜은 그곳에 접속하는 방법(HTTPS/SSH/File/Git)이고, Bare Repository는 `.git` 내용물이 워킹 트리 없이 최상위에 그대로 펼쳐진, 사람이 직접 작업하지 않는 서버용 저장소다.
