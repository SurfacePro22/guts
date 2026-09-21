# Git 심화 강의 목차

진행 순서: 컨텐츠(md) 작성 → 실습 환경 구성(필요한 항목만, 지시 하에) → 마지막에 PPT/HTML 변환. 형식/디자인은 나중 문제.

## 1. Git과 GitHub의 차이
- 1-1. Git과 GitHub의 차이 — `01-git-vs-github.md`

## 2. 저장소의 구조
- 2-1. 리포지터리의 5가지 영역 (Workspace/Index/Local Repo/Upstream/Stash) — `02-1-repo-5areas.md`
- 2-2. HEAD란? (HEAD, 브랜치, 커밋의 관계) — `02-2-head.md`

## 3. 원격 저장소
- 3-1. 원격과 프로토콜, Bare Repository란? — `03-1-remote-protocol-bare.md`
- 3-2. Git 설정 (git config) — `03-2-git-config.md`
- 3-3. Fetch vs Pull 차이 (origin/main vs origin main) — `03-3-fetch-vs-pull.md`

## 4. 변경사항 이식하기
- 4-1. Diff와 Hunk (`..` vs `...` 차이), Patch 개념 — `04-1-diff-patch.md`
- 4-2. Cherry-pick (개념 + 충돌 해결) — `04-2-cherry-pick.md`

## 5. Rebase
- 5-1. Rebase 개념과 규칙 — `05-1-rebase-basics.md`
- 5-2. Interactive Rebase (pick/squash/fixup/drop/reorder) — `05-2-interactive-rebase.md`
- 5-3. Merge vs Rebase 비교 — `05-3-merge-vs-rebase.md`

## 6. 되돌리기 전략
- 6-1. Reset(soft/mixed/hard) vs Revert, Reflog로 복구하기 — `06-1-reset-revert-reflog.md`

## 7. 작업 공간 확장
- 7-1. Stash 대신 Worktree를 써야 하는 이유 — `07-1-worktree.md`
- 7-2. Submodule이란? — `07-2-submodule.md`
- 7-3. 모노레포 vs 멀티레포 — `07-3-monorepo-vs-multirepo.md`

## 8. 마무리
- 8-1. 참고 링크 — `08-1-references.md`
- 8-2. 전체 명령어 치트시트 — `08-2-cheat-sheet.md`

각 항목은 별도 md로 작성하고, 파일명은 `번호-주제.md` 형식으로 통일.
실습 환경(prac/)은 필요한 항목에서만, 사용자 지시를 받아가며 구성.
