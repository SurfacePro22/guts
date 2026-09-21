# Worktree 실습 가이드

`worktree-demo` 저장소가 준비되어 있습니다. `main`에서 갈라진 `feature-a`, `feature-b` 두 브랜치가 있고,
각 브랜치마다 서로 다른 커밋이 하나씩 추가되어 있습니다 (마치 AI 에이전트 두 개가 각자 다른 기능을 작업 중인 상황을 흉내낸 것).
(이 파일은 git으로 추적되지 않으므로, 실습 중 무슨 짓을 해도 사라지지 않습니다.)

## 현재 상태

```
*  feat: feature_a 함수 추가        (feature-a)
| *  feat: feature_b 함수 추가      (feature-b)
|/
*  초기 커밋: app.py 추가            (main)
```

## 실습 1: worktree로 브랜치 두 개 동시에 열어보기

1. `feature-a`, `feature-b`를 각각 별도 폴더로 체크아웃합니다.

   ```
   cd worktree-demo
   git worktree add ../worktree-demo-feature-a feature-a
   git worktree add ../worktree-demo-feature-b feature-b
   ```

2. 연결된 worktree 목록을 확인합니다.

   ```
   git worktree list
   ```

   `worktree-demo`(main), `worktree-demo-feature-a`(feature-a), `worktree-demo-feature-b`(feature-b) 세 폴더가 각자 다른 브랜치로 동시에 열려있는 걸 볼 수 있습니다.

## 실습 2: stash 없이 원본 작업이 안 건드려지는지 확인

1. 원본 `worktree-demo`(main)에서 커밋하지 않은 변경을 하나 만들어둡니다.

   ```
   echo "# 임시 메모" >> app.py
   git status
   ```

2. 그 상태 그대로 두고, `worktree-demo-feature-a` 폴더로 가서 작업하고 커밋합니다.

   ```
   cd ../worktree-demo-feature-a
   echo "def extra(): pass" >> app.py
   git add -A
   git commit -m "chore: feature-a에서 extra 함수 추가"
   git log --oneline
   ```

3. 원본 폴더로 돌아가서 아까 만든 변경사항이 그대로 있는지 확인합니다.

   ```
   cd ../worktree-demo
   git status        # 여전히 "# 임시 메모" 변경이 unstaged 상태로 남아있음
   cat app.py
   ```

   `stash`나 `checkout`을 한 번도 안 했는데도, 커밋 안 한 작업이 전혀 방해받지 않았습니다.

## 실습 3: 같은 브랜치는 두 곳에서 못 연다

이미 `feature-a`는 다른 worktree에서 체크아웃 중이므로, 같은 브랜치로 worktree를 하나 더 만들려고 하면 git이 막습니다.

```
git worktree add ../worktree-demo-dup feature-a
# fatal: 'feature-a' is already checked out at '.../worktree-demo-feature-a'
```

에이전트마다 별도 브랜치가 필요한 이유가 바로 이것입니다.

## 정리하기

```
git worktree remove ../worktree-demo-feature-a
git worktree remove ../worktree-demo-feature-b
git worktree list   # worktree-demo(main)만 남았는지 확인
```

## 되돌리기

`worktree-demo` 폴더를 지우고 다시 만들어달라고 요청하면 초기 상태(브랜치 3개, worktree 없음)로 재현할 수 있습니다.
