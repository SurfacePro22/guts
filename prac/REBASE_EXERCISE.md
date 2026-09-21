# Rebase 실습 가이드 (v2 — 케이스 확장판)

`rebase-demo` 저장소가 준비되어 있습니다. 커밋 메시지 앞에 `[01]`, `[02]`처럼 번호를 붙여놔서,
interactive rebase 화면에서 줄이 이리저리 옮겨져도 "이게 몇 번 커밋이었지"를 바로 알아볼 수 있게 했습니다.
(이 파일은 git으로 추적되지 않으므로, 실습 중 무슨 짓을 해도 사라지지 않습니다.)

## 현재 구조

```
main            [01] ─────────────────────────────── [main] docs: README 사용 예시
                   \
feature/calculator  [02]─[03]─[04]─[05]─[06]─[07]─[08]
                                                  \
feature/power                                     [09]─[10]─fixup![10]
```

브랜치별 커밋 (오래된 순):

| 브랜치 | 번호 | 메시지 | 실습 포인트 |
|---|---|---|---|
| main | 01 | feat: calculator 초기 뼈대 (add 함수) | 공통 조상 |
| feature/calculator | 02 | feat: subtract 함수 추가 | docstring에 오타 "뺀디" |
| feature/calculator | 03 | debug: subtract 함수에 테스트용 print문 추가 | **drop** 후보 |
| feature/calculator | 04 | fix: subtract 함수 docstring 오타 수정 | **fixup/squash** 후보 (→02) |
| feature/calculator | 05 | feat: multiply 함수 추가 | — |
| feature/calculator | 06 | test: calculator 함수 유닛 테스트 추가 | **reorder** 후보 (divide 테스트가 07보다 먼저 옴) |
| feature/calculator | 07 | feat: divide 함수 추가 | 사실 06보다 먼저 와야 함 |
| feature/calculator | 08 | docs: README에 함수 목록 추가 | main의 커밋과 **충돌** 예정 |
| feature/power | 09 | feat: power 함수 추가 | 버그 있음 (`a ** (b+1)`) → **edit** 후보 |
| feature/power | 10 | feat: modulo 함수 추가 | docstring 오타, 바로 다음 커밋에서 수정됨 |
| feature/power | (번호 없음) | `fixup! [10] feat: modulo 함수 추가` | **autosquash** 데모용 실제 fixup 커밋 |
| main | (main) | docs: README에 사용 예시 추가 | feature/calculator의 08과 같은 자리 수정 → **충돌** |

지금 체크아웃되어 있는 브랜치는 `feature/calculator`입니다.

## 실습 1: interactive rebase 기본 (pick / reword / edit / drop)

```
git checkout feature/calculator
git rebase -i main
```

에디터에는 02~08이 **오래된 순서(위→아래)**로 나열됩니다 (5-2 참고, `git log`와 반대 방향).

1. **drop**: `[03] debug: ...print문 추가`를 `drop`으로 바꿔서 통째로 제거
2. **reword**: 아무 커밋이나 `reword`로 바꿔서 메시지를 다시 써보기 (번호 `[..]`는 그대로 유지하는 습관을 들여보세요)

완료 후 `cat calculator.py`로 print문이 사라졌는지 확인하세요.

## 실습 2: squash / fixup으로 커밋 합치기

같은 `rebase -i main`에서:

```
pick ... [02] feat: subtract 함수 추가
fixup ... [04] fix: subtract 함수 docstring 오타 수정
```

`[04]`를 `[02]` 바로 아래에서 `fixup`으로 바꾸면, 오타 수정 내용이 `[02]` 커밋 속으로 조용히 흡수되고
`[04]`라는 커밋 자체는 히스토리에서 사라집니다. (메시지를 합쳐서 직접 편집하고 싶다면 `fixup` 대신 `squash`.)

## 실습 3: reorder — 커밋 순서 바꾸기

`[06] test: ...유닛 테스트`는 `divide()`가 아직 없는 시점에 `divide()`를 테스트하는 코드를 커밋한 것입니다.
`git rebase -i main` 목록에서 `[06]`이 적힌 줄을 `[07] feat: divide 함수 추가` **아래로** 옮겨보세요.

확인하는 방법 (선택): 아래처럼 `--exec`을 붙이면 매 커밋 적용 직후 자동으로 명령을 실행해서,
순서가 잘못됐을 때 바로 어디서 깨지는지 눈으로 볼 수 있습니다.

```
git rebase -i --exec "python3 -c 'import calculator, test_calculator'" main
```

`[06]`이 `[07]`보다 앞에 있으면 `ImportError: cannot import name 'divide'`가 나면서 rebase가 그 자리에서 멈춥니다.
순서를 바로잡으면 끝까지 통과합니다.

## 실습 4: edit — 커밋 내용 직접 고치기

`feature/power` 브랜치의 `[09] feat: power 함수 추가`에는 버그가 있습니다 (지수가 1 더 크게 계산됨).

```
git checkout feature/power
git rebase -i feature/calculator
```

`[09]` 줄을 `edit`으로 바꾸고 저장하면, rebase가 그 커밋을 적용한 직후 멈춥니다. 이때:

```
# calculator.py에서 power 함수를 아래처럼 고친다
#   return a ** (b + 1)   ->   return a ** b
git add calculator.py
git commit --amend --no-edit
git rebase --continue
```

## 실습 5: autosquash — fixup 커밋 자동 정렬

`feature/power`에는 이미 `fixup! [10] feat: modulo 함수 추가`라는 실제 fixup 커밋이 들어 있습니다
(`git commit --fixup=<10의 해시>`로 만든 것 — docstring 오타 "게산" → "계산"을 고치는 내용).

```
git checkout feature/power
git log --oneline   # fixup! 커밋이 보임
git rebase -i --autosquash feature/calculator
```

에디터를 열어보면 손대지 않았는데도 `fixup! [10] ...` 줄이 이미 `[10]` 바로 아래로 옮겨져서
`fixup`으로 표시돼 있을 것입니다. 그대로 저장만 하면 두 커밋이 합쳐집니다.

## 실습 6: rebase --onto — 특정 구간만 다른 브랜치로 옮기기

`feature/power`는 사실 `main`에서 갈라졌어야 하는데, 실수로 `feature/calculator`(아직 정리 안 된 지저분한
히스토리)에서 브랜치를 땄다고 가정해봅시다. `feature/power`만의 커밋(`[09]`, `[10]`, `fixup!`)만 쏙 뽑아서
`main` 위로 옮겨보세요.

```
git rebase --onto main feature/calculator feature/power
```

의미: "`feature/calculator`에는 없고 `feature/power`에만 있는 커밋들을, `main` 위로 옮겨서 재적용해라."
끝나고 `git log --oneline --graph --all`로 `feature/power`가 `feature/calculator`를 거치지 않고
바로 `main` 위에 올라갔는지 확인하세요.

## 실습 7: main 위로 rebase하면서 충돌 해결하기

```
git checkout feature/calculator
git rebase main
```

`README.md`에서 충돌이 납니다. main은 "사용 예시" 절을, feature는 "함수 목록" 절을 같은 위치에
추가했기 때문입니다. `<<<<<<<` / `=======` / `>>>>>>>` 마커를 편집해서 두 섹션을 모두 살린 뒤:

```
git add README.md
git rebase --continue
```

## 실습 8: rebase 취소하고 되돌리기

```
git rebase --abort          # rebase 진행 중일 때 중단
git reflog                  # rebase 전 커밋 해시 찾기 (6-1 참고)
git reset --hard <해시>      # 원하는 시점으로 되돌리기
```

## 참고

- interactive rebase 목록은 `git log`와 반대로 오래된 커밋이 위에 옵니다 (5-2).
- rebase는 이미 push해서 남이 받아간 커밋에는 쓰지 않는 게 원칙입니다 (5-1 황금률).
- merge와 rebase의 히스토리 모양 차이는 `git log --graph --oneline --all`로 비교해보세요 (5-3).

막히면 저장소를 다시 만들어달라고 요청하면 됩니다 — 이 스크립트로 언제든 초기 상태로 재현할 수 있습니다.
