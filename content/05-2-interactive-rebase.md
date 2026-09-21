# Interactive Rebase (pick/squash/fixup/drop/reorder)

## 한 줄 정의

- **Interactive Rebase**: `git rebase -i <base>`를 실행하면 에디터가 열리고, 재적용될 커밋 목록을 보여주면서 각 커밋을 어떻게 처리할지 직접 고를 수 있는 방식.
- 일반 rebase가 "그대로 순서대로 재적용"이라면, interactive rebase는 "재적용하면서 합치거나/버리거나/순서를 바꾸는" 것.

## 표로 비교 — 명령어별 동작

| 명령 | 역할 |
|---|---|
| `pick` | 그대로 사용 (기본값) |
| `reword` | 이 커밋을 적용하되, 커밋 메시지만 다시 씀 |
| `edit` | 이 커밋을 적용한 뒤 멈춰서, 내용을 직접 고칠 기회를 줌 |
| `squash` | 바로 위 커밋과 합치고, 두 메시지를 합쳐서 편집할 수 있게 함 |
| `fixup` | 바로 위 커밋과 합치되, 이 커밋의 메시지는 버림 (조용히 합침) |
| `drop` | 이 커밋을 통째로 제거 |
| (줄 순서 바꾸기) | 목록에서 줄 위치를 옮기면 그게 곧 커밋 적용 순서 변경(reorder) |

## 에디터 화면 읽는 법

```
pick a1b2c3 feat: add subtract function
pick d4e5f6 debug: add print statement
pick 789abc fix: correct typo in subtract docstring
pick bcdef1 test: add unit tests
pick 234567 feat: add divide function
```

`git log`는 최신 커밋이 위에 오지만, **interactive rebase 목록은 반대로 오래된 커밋이 위에 온다** (위에서부터 순서대로 재적용되기 때문). 이 방향이 헷갈려서 실수로 순서를 잘못 바꾸는 경우가 많으니 주의.

예를 들어 위 목록을 아래처럼 바꾸면:

```
pick a1b2c3 feat: add subtract function
drop d4e5f6 debug: add print statement
fixup 789abc fix: correct typo in subtract docstring
pick bcdef1 test: add unit tests
pick 234567 feat: add divide function
```

디버그용 커밋은 사라지고, 오타 수정 커밋은 바로 위 커밋에 조용히 합쳐진다.

## 참고: `--autosquash`

커밋 메시지 앞에 `fixup!`이나 `squash!`를 붙여서 커밋해두면(`git commit --fixup=<해시>`), `git rebase -i --autosquash <base>`를 실행할 때 그 커밋들이 자동으로 대상 커밋 바로 아래에 `fixup`/`squash`로 배치된 채로 에디터가 열린다. 직접 줄을 옮길 필요가 없어진다.

## 한 줄 요약

Interactive rebase는 커밋 목록을 열어 `pick`/`reword`/`edit`/`squash`/`fixup`/`drop`과 줄 순서 변경으로 히스토리를 원하는 대로 재구성하는 것이며, 목록이 `git log`와 반대로 오래된 순서(위→아래)로 나열된다는 점을 기억해야 한다.
