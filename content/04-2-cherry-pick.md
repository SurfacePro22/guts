# Cherry-pick (개념 + 충돌 해결)

## 한 줄 정의

- **Cherry-pick**: 다른 브랜치에 있는 커밋 하나(또는 몇 개)만 골라서, 그 변경사항을 지금 브랜치 위에 새로운 커밋으로 복사해오는 것.
- 원리는 4-1에서 다룬 patch와 같다 — 그 커밋의 diff를 뽑아서 현재 브랜치 위에 적용하고, 새 해시로 다시 커밋하는 것.

## 표로 비교 — 자주 쓰는 옵션

| 옵션 | 역할 |
|---|---|
| `git cherry-pick <해시>` | 해당 커밋의 변경사항을 새 커밋으로 복사 |
| `-n`, `--no-commit` | 적용만 하고 커밋은 안 함 (여러 커밋을 모아 하나로 묶고 싶을 때) |
| `-x` | 커밋 메시지에 원본 해시 흔적을 남김 (`(cherry picked from commit ...)`) |
| `git cherry-pick A..B` | A 다음부터 B까지 범위의 커밋들을 순서대로 cherry-pick |
| `--continue` / `--abort` / `--skip` | 충돌 발생 시 계속 진행 / 전체 취소 / 이 커밋만 건너뛰기 |

## 충돌 해결 절차

merge 충돌과 해결 방식은 동일하다.

```
git cherry-pick <해시>
# 충돌 발생 시 파일에 <<<<<<< / ======= / >>>>>>> 마커가 표시됨

# 1. 마커를 직접 편집해서 원하는 내용으로 정리
# 2. 해결한 파일을 스테이징
git add <파일>

# 3. cherry-pick 계속 진행 (마치 이 시점에 커밋하듯 동작)
git cherry-pick --continue
```

마음에 안 들면 `git cherry-pick --abort`로 cherry-pick 시작 전 상태로 완전히 되돌릴 수 있다.

## 명령어 치트시트 (4장 — 변경사항 이식하기)

| 명령 | 설명 |
|---|---|
| `git diff` | Workspace의 변경사항 확인 |
| `git diff A..B` | A와 B를 있는 그대로 비교 |
| `git diff A...B` | 공통 조상과 B를 비교 (B에서만 일어난 변경) |
| `git log A..B` / `git log A...B` | 한쪽 방향 커밋 / 양쪽 대칭차집합 |
| `git diff > x.patch` / `git apply x.patch` | 순수 diff 패치 생성 / 적용 |
| `git format-patch -1 HEAD` / `git am x.patch` | 커밋 메타데이터 포함 패치 생성 / 커밋으로 재현 |
| `git apply --check`\|`--reject`\|`--3way` | 적용 전 검증 / 부분 적용 / 3-way 병합 시도 |
| `git cherry-pick <해시>` | 다른 브랜치의 커밋 하나 복사해오기 |
| `git cherry-pick -n` / `-x` / `A..B` | 커밋 안 함 / 원본 해시 흔적 남김 / 범위 지정 |
| `git cherry-pick --continue`\|`--abort`\|`--skip` | 충돌 시 계속 / 취소 / 건너뛰기 |


## 한 줄 요약

Cherry-pick은 커밋 하나의 diff를 patch처럼 뽑아 다른 브랜치 위에 새 커밋으로 복사하는 것이며, 충돌이 나면 merge 충돌과 같은 방식(마커 편집 → `add` → `--continue`)으로 해결한다.
