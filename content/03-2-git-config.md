# Git 설정 (git config)

## 한 줄 정의

- **git config**: Git의 동작 방식을 정하는 키-값 설정 저장소.
- 설정은 **System / Global / Local** 세 범위로 나뉘고, 더 좁은 범위가 넓은 범위를 덮어쓴다 (Local > Global > System).

## 표로 비교 — System vs Global vs Local

| | System | Global | Local |
|---|---|---|---|
| 저장 위치 | `/etc/gitconfig` | `~/.gitconfig` | `<저장소>/.git/config` |
| 적용 범위 | 이 컴퓨터의 모든 사용자·모든 저장소 | 이 계정의 모든 저장소 | 이 저장소 하나만 |
| 설정 명령 | `git config --system` | `git config --global` | `git config` (기본값) 또는 `--local` |
| 우선순위 | 가장 낮음 | 중간 | 가장 높음 |
| 확인 명령 | `git config --list --show-origin`으로 어느 파일에서 왔는지 확인 가능 | | |

## 자주 쓰는 설정

| 키 | 역할 | 예시 |
|---|---|---|
| `user.name`, `user.email` | 커밋에 남는 작성자 정보 | `git config --global user.name "jihee"` |
| `core.editor` | `commit -m` 없이 커밋할 때 열리는 에디터 | `git config --global core.editor "vim"` |
| `init.defaultBranch` | `git init` 시 기본 브랜치명 | `git config --global init.defaultBranch main` |
| `pull.rebase` | `git pull`이 merge 대신 rebase로 동작하게 함 | `git config --global pull.rebase true` |
| `alias.*` | 명령어 단축키 | `git config --global alias.co checkout` |
| `credential.helper` | HTTPS 인증 정보 캐싱 방식 | `git config --global credential.helper cache` |

## 원격 설정도 결국 여기에 저장된다

`git remote add origin <url>` 같은 명령을 실행하면, 그 결과는 새 명령이 아니라 **Local 설정(`.git/config`)에 기록**되는 것뿐이다.

```
[remote "origin"]
    url = git@github.com:me/repo.git
    fetch = +refs/heads/*:refs/remotes/origin/*
[branch "main"]
    remote = origin
    merge = refs/heads/main
```

`branch.main.remote`/`branch.main.merge`가 바로 "내 `main` 브랜치는 `origin`의 `main`을 추적한다"는 설정이며, 다음 항목(Fetch vs Pull)에서 나오는 추적 브랜치 동작의 근거가 된다.

## 한 줄 요약

git config는 System → Global → Local 순으로 좁은 범위가 우선하는 설정 저장소이고, 사용자 정보부터 원격 주소·브랜치 추적 정보까지 전부 이 안에 저장된다.
