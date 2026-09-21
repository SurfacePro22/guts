# diff / format-patch 실습 가이드

`calculator-a`(원본)와 `calculator-b`(A를 클론해서 만든 별개의 저장소)가 준비되어 있습니다.
두 저장소 모두 커밋 작성자는 `Linus Torvalds <torvalds@linux-foundation.org>`로 설정되어 있습니다.
(이 파일은 git으로 추적되지 않으므로, 실습 중 무슨 짓을 해도 사라지지 않습니다.)

## 현재 상태

```
calculator-a: 3e8f705 초기 커밋: add, subtract 함수 추가
calculator-b: 3e8f705 초기 커밋: add, subtract 함수 추가   (clone으로 생성)
```

`calculator.py`에는 `add`, `subtract` 두 함수만 있습니다. 이번 실습은 버그를 고치는 게 아니라,
**새 기능(`multiply` 함수)을 추가하는 변경**을 patch로 만들어 다른 저장소에 옮기는 흐름입니다.

## 실습 1: diff로 확인하고 패치 파일 만들어서 다른 저장소에 적용하기

1. `calculator-a`에서 새 함수를 추가합니다.

   ```
   cd calculator-a
   ```

   `calculator.py` 맨 아래에 추가:

   ```python
   def multiply(a, b):
       """두 수를 곱한다"""
       return a * b
   ```

2. 변경사항을 diff로 확인하고 패치 파일로 저장합니다.

   ```
   git diff
   git diff > ../add-multiply.patch
   cat ../add-multiply.patch
   ```

3. 이 패치를 `calculator-b`에 적용합니다.

   ```
   cd ../calculator-b
   git apply --check ../add-multiply.patch   # 미리 검증
   git apply ../add-multiply.patch           # 실제 적용
   git diff                                  # calculator-a와 똑같이 바뀌었는지 확인
   git status                                # 커밋은 안 됨 — 워킹 디렉토리만 바뀐 상태
   ```

## 실습 2: 커밋 → format-patch → 다른 저장소에 커밋으로 재현

1. `calculator-a`에서 새 함수 추가를 커밋합니다.

   ```
   cd ../calculator-a
   git add calculator.py
   git commit -m "추가: multiply 함수 구현"
   ```

2. 이 커밋을 format-patch로 뽑아봅니다.

   ```
   git format-patch -1 HEAD -o ../
   ls ../0001-*.patch
   cat ../0001-*.patch
   ```

   `From:` / `Date:` / `Subject:` 헤더와 diffstat이 들어있는 걸 확인하세요.
   (커밋 메시지가 한글이라 `Subject:` 줄이 `=?UTF-8?q?...`처럼 인코딩되어 보이는데, 정상입니다 — `git am`으로 적용하면 다시 한글로 복원됩니다.)

3. `calculator-b`는 실습 1에서 워킹 디렉토리만 바뀐 상태이니, 먼저 되돌립니다.

   ```
   cd ../calculator-b
   git checkout .
   git status   # 다시 깨끗한 상태
   ```

4. format-patch로 만든 패치를 커밋으로 적용합니다.

   ```
   git am ../0001-*.patch
   git log --oneline                       # calculator-a와 동일한 메시지의 새 커밋 확인
   git log -1 --format="%an <%ae>"         # 작성자도 Linus Torvalds로 그대로 유지되는지 확인
   ```

## 되돌리기

각 저장소 안에서:

```
git log --oneline               # 되돌릴 커밋 해시 확인
git reset --hard <커밋 해시>
```

완전히 초기화하고 싶으면 `calculator-a`, `calculator-b` 폴더를 지우고 다시 만들어달라고 요청하면 됩니다.
