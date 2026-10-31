# GitHub 업로드 git 명령어 정리

로컬 프로젝트를 GitHub에 처음 올릴 때 쓰는 명령어들을 순서대로 정리했습니다.

## 1. `git init`
현재 폴더를 git으로 관리하기 시작합니다. 폴더 안에 숨겨진 `.git` 폴더가 생기는데, 여기에 모든 버전 기록이 저장됩니다. 이게 있어야 이 폴더가 "git 저장소"가 됩니다.

## 2. `git add <파일명...>`
지정한 파일들을 **staging area(커밋 대기 목록)**에 올립니다. "이번 커밋에 이 파일들의 현재 상태를 포함시킬 거야"라고 표시하는 단계입니다. 아직 저장(커밋)된 건 아니고, 커밋 전 준비 단계입니다.

```bash
git add cv_intro_project.ipynb cv_project_study_notes.html
```

## 3. `git commit -m "메시지"`
staging area에 올려둔 내용을 실제로 **하나의 저장 지점(커밋)**으로 기록합니다. `-m` 뒤의 메시지는 "이 커밋에서 뭘 했는지"를 남기는 설명입니다. 이 시점부터 이 변경 내역은 git 역사에 영구히 남습니다.

```bash
git commit -m "Initial commit: CNN vs Transfer Learning on Cats vs Dogs"
```

## 4. `git branch -M main`
현재 브랜치(작업 줄기)의 이름을 `main`으로 강제 지정(rename)합니다. git을 처음 `init`하면 기본 브랜치 이름이 환경에 따라 `master`이거나 `main`일 수 있는데, GitHub는 기본적으로 `main`을 표준으로 쓰기 때문에 확실히 맞춰주는 명령입니다. `-M`은 이미 그 이름의 브랜치가 있어도 강제로 덮어쓴다는 뜻입니다.

## 5. `git remote add origin <URL>`
로컬 git 저장소에 "원격 저장소(remote)"를 등록합니다. `origin`은 그 원격 저장소를 부르는 별칭(관례적으로 항상 `origin`을 씀)이고, `<URL>`은 GitHub에 만든 저장소 주소입니다. 이 단계는 "어디로 올릴지"만 알려주는 것이지, 아직 실제로 업로드하지는 않습니다.

```bash
git remote add origin https://github.com/<your-username>/cv-intro-cnn-transfer-learning.git
```

## 6. `git push -u origin main`
로컬의 `main` 브랜치 커밋들을 `origin`(방금 등록한 GitHub 저장소)으로 실제 업로드합니다.

- `-u` (`--set-upstream`)는 로컬 `main` 브랜치와 원격 `origin/main` 브랜치를 연결(추적)해두는 옵션입니다. 이후부턴 그냥 `git push`만 쳐도 어디로 보낼지 자동으로 기억합니다.

## 전체 흐름

```
init(시작) → add(대기 등록) → commit(로컬에 저장)
→ branch -M(브랜치 이름 정리) → remote add(업로드할 곳 등록) → push(실제 업로드)
```

---

# 실험용 브랜치 만들고 main에 병합하기

`main`은 항상 "잘 동작하는 상태"로 유지하고, 새로운 실험은 별도 브랜치에서 진행한 뒤 검증되면 `main`에 합치는 흐름입니다.

## 1. `git branch` (현재 브랜치 확인)
지금 어떤 브랜치들이 있고, 내가 어디에 있는지 확인합니다. `*`가 붙은 게 현재 브랜치입니다.

```bash
git branch
```

## 2. `git checkout -b <브랜치명>` (실험 브랜치 생성 + 이동)
현재 브랜치(보통 `main`)에서 새 브랜치를 만들고 바로 그 브랜치로 이동합니다. `-b`가 "새로 만들기"를 의미합니다.

```bash
git checkout -b test-augmentation
```

최신 git에서는 같은 동작을 `git switch -c <브랜치명>`으로도 할 수 있습니다(`-c` = create).

```bash
git switch -c test-augmentation
```

## 3. 실험 진행 (add → commit 반복)
이 브랜치 안에서는 `main`에 영향을 주지 않고 자유롭게 코드를 수정하고 커밋을 쌓을 수 있습니다.

```bash
git add .
git commit -m "Try data augmentation to reduce overfitting"
```

필요하면 이 브랜치를 GitHub에도 올려서 백업하거나 공유할 수 있습니다(선택사항).

```bash
git push -u origin test-augmentation
```

## 4. 기존 브랜치로 다시 이동하기
브랜치 간 이동은 `checkout` 또는 `switch`로 합니다.

```bash
git checkout main
# 또는
git switch main
```

## 5. 실험이 잘 됐다면 → `main`에 병합
먼저 병합받을 브랜치(`main`)로 이동한 뒤, 실험 브랜치를 합칩니다.

```bash
git switch main
git merge test-augmentation
```

- 충돌(conflict)이 없으면 자동으로 합쳐집니다.
- 충돌이 나면 git이 표시해주는 부분을 직접 수정하고, `git add <파일>` → `git commit`으로 마무리합니다.

병합 후 원격에도 반영하려면:

```bash
git push
```

## 6. 다 쓴 실험 브랜치 정리
병합이 끝나 더 이상 필요 없는 브랜치는 삭제해도 됩니다.

```bash
git branch -d test-augmentation          # 로컬 브랜치 삭제 (병합 안 됐으면 경고 후 거부)
git push origin --delete test-augmentation  # 원격에 올렸던 브랜치도 삭제
```

> `-d`는 안전 삭제(병합되지 않았으면 막아줌)이고, 정말 강제로 지우고 싶으면 `-D`를 씁니다.

## 실험이 잘 안 됐다면?
그냥 `main`으로 돌아가고 실험 브랜치를 무시하거나 삭제하면 됩니다. `main`은 전혀 영향받지 않습니다.

```bash
git switch main
git branch -D test-augmentation   # 실패한 실험 브랜치 강제 삭제
```

## 전체 흐름 (실험 브랜치 워크플로우)

```
main에서 시작
→ checkout -b test-xxx (실험 브랜치 생성)
→ add → commit (여러 번 반복하며 실험)
→ (성공) switch main → merge test-xxx → push
→ (실패) switch main → branch -D test-xxx
→ branch -d test-xxx (병합 후 정리, 선택)
```
