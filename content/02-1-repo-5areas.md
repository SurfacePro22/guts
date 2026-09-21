# 리포지터리의 5가지 영역 (Workspace / Index / Local Repo / Upstream / Stash)

## 한 줄 정의

- **Workspace(작업 디렉토리)**: 실제 파일을 수정하는 공간
- **Index(Staging Area)**: `add`한 파일이 다음 커밋을 기다리는 대기 공간
- **Local Repo(로컬 저장소)**: `commit`한 히스토리가 쌓이는 `.git` 내부 저장소
- **Upstream(원격 저장소)**: GitHub 등 서버에 있는 저장소, 협업의 기준점
- **Stash**: 커밋하지 않고 변경사항을 잠깐 치워두는 임시 서랍

## 표로 비교

| 영역 | 위치 | 담는 명령 | 꺼내는/되돌리는 명령 | 히스토리 보존 | 네트워크 |
|---|---|---|---|---|---|
| Workspace | 로컬 파일시스템 | 직접 수정 | `checkout`, `restore` | 없음 | 불필요 |
| Index | `.git/index` | `git add` | `git reset` | 없음 | 불필요 |
| Local Repo | `.git` 내부 | `git commit` | `git log`, `git checkout` | 있음 | 불필요 |
| Upstream | 원격 서버 | `git push` | `git fetch`, `git pull` | 있음(서버) | 필요 |
| Stash | `.git/refs/stash` | `git stash` | `git stash pop/apply` | 임시 스택 | 불필요 |

## 한 줄 요약

수정(Workspace) → 스테이징(Index) → 커밋(Local Repo) → 푸시(Upstream)가 기본 흐름이고, Stash는 그 흐름 중간에 잠깐 치워두는 임시 서랍이다.
