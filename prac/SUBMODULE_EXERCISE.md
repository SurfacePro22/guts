# Submodule 실습 가이드

`submodule-lib`(독립 저장소)와 `submodule-demo`(이걸 `lib` 서브모듈로 포함한 부모 저장소),
그리고 `submodule-demo`를 미리 일반 `clone`해 둔 `submodule-demo-clone`이 준비되어 있습니다.
(이 파일은 git으로 추적되지 않으므로, 실습 중 무슨 짓을 해도 사라지지 않습니다.)

> 참고: 로컬 폴더 경로를 서브모듈 url로 쓰기 때문에, git이 보안상 기본으로 막아둔
> `file://` 전송을 허용해야 합니다. 이 실습 환경에서는 이미 `git config --global protocol.file.allow always`가
> 설정되어 있습니다 — 실제 회사/개인 프로젝트에서는 서브모듈 url이 대부분 https/ssh라 이 설정이 필요 없습니다.

## 현재 상태

```
submodule-lib (독립 저장소)
  85243d7  초기 커밋: greet 함수 추가

submodule-demo (부모 저장소)
  c7e7888  chore: submodule-lib을 lib 서브모듈로 추가
  1fdfc17  초기 커밋: app.py 추가

submodule-demo/.gitmodules
  [submodule "lib"]
      path = lib
      url = ../submodule-lib

submodule-demo-clone
  submodule-demo를 "git clone" (recurse-submodules 옵션 없이)로 그대로 복제해 둔 상태.
  lib/ 폴더가 비어있습니다.
```

## 실습 1: clone 직후 서브모듈 폴더가 비어있는 이유 확인하기

1. 미리 clone해 둔 `submodule-demo-clone`을 열어봅니다.

   ```
   cd submodule-demo-clone
   ls lib          # 비어있음
   cat .gitmodules # lib이 어디를(../submodule-lib) 가리키는지 기록되어 있음
   git submodule status
   # -85243d7ddb7814b30811632511a93cbef4f89326 lib
   #   ↑ 맨 앞의 '-'는 "아직 초기화 안 됨"이라는 뜻
   ```

   `git clone`만으로는 부모 저장소의 커밋(`.gitmodules`, gitlink)만 받아오고,
   서브모듈의 실제 코드는 받아오지 않는다는 걸 직접 확인하는 단계입니다 (07-2 참고).

2. 비어있는 서브모듈을 채워 넣습니다.

   ```
   git submodule update --init --recursive
   git submodule status
   # 앞의 '-'가 사라지고 공백으로 바뀌면 초기화 완료
   cat lib/greeting.py
   ```

   앞으로는 처음부터 `git clone --recurse-submodules <url>`로 clone하면 이 두 단계를 한 번에 할 수 있습니다.

## 실습 2: 서브모듈은 완전히 독립된 저장소 — 안에서 직접 커밋하기

1. `submodule-demo`(clone 말고 원본) 안의 `lib`로 들어가서 직접 커밋해봅니다.

   ```
   cd ../submodule-demo
   git status                 # 여기서는 lib에 대해 별다른 변화 없음
   cd lib
   git branch                 # * main  (등록된 커밋이 마침 main의 최신이라 브랜치에 붙어있는 상태)
   echo "def stub(): pass" >> greeting.py
   git add -A
   git commit -m "feat: stub 함수 추가 (임시)"
   ```

2. 상위 폴더로 돌아가서 부모 저장소 입장에서는 뭐가 바뀐 걸로 보이는지 확인합니다.

   ```
   cd ..
   git status --short          #  M lib
   git diff --submodule=log    # Submodule lib 85243d7..xxxxxxx: > feat: stub 함수 추가 (임시)
   ```

   부모 저장소는 `lib` 폴더 안의 파일 하나하나가 바뀐 게 아니라, "lib이 가리키는 커밋 해시가 바뀌었다"는 것만 봅니다.

3. 이 포인터 변화를 부모 저장소에 커밋해서 반영합니다.

   ```
   git add lib
   git commit -m "chore: lib 서브모듈 포인터 갱신"
   git log --oneline -3
   ```

   여기서 중요한 점: 방금 `lib` 안에서 만든 커밋은 `lib` 자신의 로컬 브랜치에만 있을 뿐,
   `lib`의 원격(`submodule-lib`)에는 아직 없습니다. 서브모듈은 완전히 독립된 저장소라서,
   이 커밋을 팀과 공유하려면 **`lib` 폴더 안에서 따로 `git push`**를 해줘야 합니다 (부모 저장소를 push해도 같이 안 올라감).

## 실습 3: 서브모듈이 detached HEAD가 되는 순간 직접 겪어보기

이번엔 반대 상황 — "팀원이 라이브러리 자체를 업데이트했고, 나는 그걸 받아와야 하는" 시나리오입니다.

1. `submodule-lib`(원본, `lib` 폴더 말고 진짜 `submodule-lib`)에 새 커밋을 추가합니다.
   실무에서는 다른 사람이 이 저장소에 push한 상황이라고 생각하면 됩니다.

   ```
   cd ../submodule-lib
   echo "def farewell(n): return f'Goodbye, {n}!'" >> greeting.py
   git add -A
   git commit -m "feat: farewell 함수 추가"
   git log --oneline
   ```

2. `submodule-demo`로 돌아가서 서브모듈을 원격 최신 커밋으로 갱신합니다.

   ```
   cd ../submodule-demo
   git submodule update --remote lib
   git submodule status
   #  +xxxxxxx lib (remotes/origin/HEAD)   <- '+'는 "인덱스에 기록된 커밋과 실제 체크아웃이 다르다"는 뜻
   cat lib/greeting.py       # farewell 함수가 들어와 있음
   git diff --submodule=log
   # Submodule lib xxxxxxx...yyyyyyy:
   #   < feat: stub 함수 추가 (임시)     <- 실습 2에서 만든 커밋이 사라진 것처럼 보임!
   #   > feat: farewell 함수 추가
   ```

3. 실습 2에서 만든 `stub` 커밋이 진짜 사라진 건지 확인합니다.

   ```
   cd lib
   git status          # HEAD detached at yyyyyyy
   git log --oneline --all
   git reflog          # stub 커밋 해시가 남아있는지 확인 (6-1 reflog 참고)
   cd ..
   ```

   `git submodule update --remote`는 **원격의 특정 커밋을 그대로 checkout**하는 방식이라,
   방금 전까지 `main` 브랜치 위에 있던 `lib`이 이 순간 **detached HEAD**로 바뀝니다.
   실습 2의 `stub` 커밋은 어떤 브랜치에서도 가리키지 않게 되어 `git log`에는 안 보이지만,
   `reflog`에는 남아있어서 필요하면 복구할 수 있습니다. 서브모듈을 다룰 때 원격 push를 안 하고
   방치한 로컬 커밋이 이렇게 쉽게 "붕 뜰" 수 있다는 걸 보여주는 실습입니다.

4. 갱신된 포인터를 부모 저장소에 커밋해서 마무리합니다.

   ```
   git add lib
   git commit -m "chore: lib 최신 버전(farewell 함수)으로 갱신"
   ```

## 추가로 해볼 것

- `git submodule foreach 'git status'`: 서브모듈이 여러 개일 때 한 번에 상태 확인하기
- `git submodule deinit lib` → `git submodule update --init lib`: 서브모듈을 완전히 비웠다가 다시 채워보기
- `.gitmodules`(부모 저장소가 커밋하는 설정 파일)와 `.git/modules/lib/config`(로컬 전용, 커밋 안 됨)의 내용을 비교해서 서브모듈 설정이 실제로 어디에 나뉘어 저장되는지 확인하기
- `git clone --recurse-submodules ../submodule-demo <새 경로>`로 clone해서, 실습 1의 두 단계(clone → update --init)가 한 번에 되는지 확인하기

## 되돌리기

`submodule-lib`, `submodule-demo`, `submodule-demo-clone` 세 폴더를 지우고 다시 만들어달라고 요청하면
초기 상태(각각 커밋 1개/2개, clone은 미초기화 상태)로 재현할 수 있습니다.
