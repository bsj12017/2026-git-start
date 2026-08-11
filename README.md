# 2026-git-start
hi
melong
GitHub 웹에서 추가한 내용입니다.

GitHub 웹에서 추가한 내용입니다.

오늘의 학습 목표: 작업자 A·B의 Git 협업 및 Merge 충돌 해결

# Git Merge 충돌 해결 실습 시퀀스 다이어그램

> 대상 자료  
> - `4-2_1단계__GitHub와_로컬_저장소의_Merge_충돌_해결_실습.pdf`
> - `4-2_2단계_작업자_AB의_Fetch_Merge_및_충돌_해결.pdf`

## 1단계: GitHub 웹과 로컬 저장소의 충돌 해결

GitHub 웹과 로컬에서 `README.md`의 같은 위치를 서로 다르게 수정한 뒤, 로컬에서 원격 변경을 가져와 충돌을 해결하는 과정이다.

```mermaid
sequenceDiagram
    autonumber
    actor U as 사용자
    participant G as GitHub 원격 저장소<br/>(origin/main)
    participant L as 로컬 저장소<br/>(main)
    participant F as 로컬 파일<br/>(README.md)

    U->>G: 저장소 생성<br/>(README.md 포함)
    U->>L: git clone 저장소_URL
    G-->>L: 파일과 커밋 이력 복제<br/>origin 등록 및 main 연결

    rect rgb(238, 247, 255)
        Note over U,G: GitHub 웹에서 원격 변경 생성
        U->>G: README.md 수정 후 Commit<br/>"GitHub에서 README 수정"
        Note over G,L: 원격에는 새 커밋이 있지만<br/>로컬 main은 아직 이전 커밋 상태
    end

    rect rgb(246, 241, 255)
        Note over U,F: 로컬에서 같은 위치를 다르게 수정
        U->>F: README.md 수정
        U->>L: git add README.md
        U->>L: git commit -m<br/>"로컬에서 README 수정"
        L->>G: git push
        G-->>L: Push 거절<br/>(fetch first)
        Note over G,L: 원격과 로컬에 서로 없는 커밋이 있어<br/>브랜치가 분기(diverge)됨
    end

    rect rgb(255, 247, 230)
        Note over U,L: 원격 변경 확인 및 병합
        U->>L: git fetch origin
        G-->>L: 원격 커밋 다운로드<br/>origin/main 업데이트
        Note over L,F: fetch만으로는 main과 작업 파일이<br/>바뀌지 않음
        U->>L: git merge origin/main
        L->>F: 같은 위치의 변경 병합 시도
        F-->>L: README.md 콘텐츠 충돌
        L-->>U: CONFLICT 및 main|MERGING 표시
    end

    rect rgb(235, 250, 239)
        Note over U,F: 충돌 해결 및 Merge 완료
        U->>F: 충돌 표시 확인<br/>(HEAD / origin/main)
        U->>F: 두 변경을 모두 반영하고<br/>충돌 표시 삭제 후 저장
        U->>L: git add README.md
        Note over L: 충돌 해결 완료로 표시되지만<br/>Merge는 아직 진행 중
        U->>L: git commit -m<br/>"README 충돌 해결"
        Note over L: Merge Commit 생성<br/>MERGING 상태 종료
        L->>G: git push
        G-->>U: 병합된 README.md와<br/>Merge Commit 반영 완료
    end
```

## 2단계: 작업자 A·B 협업 환경 구성

같은 GitHub 저장소를 두 폴더에 각각 Clone하여 서로 독립적인 작업자 A와 B의 로컬 저장소를 구성한다.

```mermaid
sequenceDiagram
    autonumber
    actor U as 실습자
    participant A as 작업자 A 로컬<br/>(/c/2026-git-start)
    participant G as GitHub 원격 저장소<br/>(origin/main)
    participant B as 작업자 B 로컬<br/>(/c/2026-git-start-b)

    U->>A: git pull origin main<br/>시작 상태 최신화
    U->>B: git clone 저장소_URL<br/>2026-git-start-b
    G-->>B: 동일 저장소를 별도 폴더에 복제
    U->>A: user.name / user.email을<br/>Worker A로 로컬 설정
    U->>B: user.name / user.email을<br/>Worker B로 로컬 설정
    Note over A,B: 작업 파일, main 브랜치, Staging Area,<br/>로컬 커밋과 Git 설정은 서로 독립적
```

## 2단계 - 1차: 충돌 없는 협업

두 작업자가 서로 다른 파일을 수정하면 Git이 변경을 자동으로 병합할 수 있다.

```mermaid
sequenceDiagram
    autonumber
    participant A as 작업자 A 로컬
    participant G as GitHub 원격 저장소<br/>(origin/main)
    participant B as 작업자 B 로컬

    A->>A: worker-a.md 생성<br/>add → commit
    A->>G: git push
    Note over G,B: B의 로컬은 자동으로 바뀌지 않음

    B->>G: git fetch origin
    G-->>B: A의 커밋을 가져와<br/>origin/main 업데이트
    B->>B: git merge origin/main
    Note over B: worker-a.md가 로컬 main에 반영됨

    B->>B: worker-b.md 생성<br/>add → commit
    B->>G: git push
    G-->>B: Push 성공

    A->>G: git fetch origin
    G-->>A: B의 커밋을 가져와<br/>origin/main 업데이트
    A->>A: git merge origin/main
    Note over A,G: A, B, GitHub에 README.md,<br/>worker-a.md, worker-b.md가 모두 존재
```

## 2단계 - 2차: 같은 파일 수정과 충돌 해결

A와 B가 동일한 공통 커밋에서 `README.md`의 같은 문장을 다르게 수정한다. A가 먼저 Push한 뒤 B가 원격 변경을 병합하면서 충돌을 해결한다.

```mermaid
sequenceDiagram
    autonumber
    participant A as 작업자 A 로컬<br/>(main)
    participant G as GitHub 원격 저장소<br/>(origin/main)
    participant B as 작업자 B 로컬<br/>(main)

    rect rgb(238, 247, 255)
        Note over A,B: 충돌 실습 전 동일한 커밋으로 동기화
        A->>G: git fetch origin
        G-->>A: 원격 커밋 정보 전달
        A->>A: git merge origin/main
        B->>G: git fetch origin
        G-->>B: 원격 커밋 정보 전달
        B->>B: git merge origin/main
        Note over A,B: 공통 문장<br/>"오늘의 학습 목표: Git 협업 이해"
    end

    rect rgb(246, 241, 255)
        Note over A,G: A가 먼저 같은 문장을 수정
        A->>A: README.md 수정<br/>add → commit
        A->>G: git push
        G-->>A: A의 커밋 반영

        Note over G,B: B는 아직 A의 커밋을 받지 않은 상태
        B->>B: 같은 문장을 다른 내용으로 수정<br/>add → commit
        B->>G: git push
        G-->>B: Push 거절<br/>(fetch first)
        Note over A,B: A와 B의 main이 서로 다른<br/>커밋을 가진 분기 상태
    end

    rect rgb(255, 247, 230)
        Note over G,B: B가 원격 변경을 가져와 병합
        B->>G: git fetch origin
        G-->>B: A의 커밋 다운로드<br/>origin/main 업데이트
        B->>B: git merge origin/main
        B-->>B: README.md 충돌 발생<br/>main|MERGING
        Note over B: HEAD = B의 로컬 변경<br/>origin/main = A가 Push한 원격 변경
    end

    rect rgb(235, 250, 239)
        Note over B: B가 충돌 해결 담당
        B->>B: 두 작업자의 의도를 합친<br/>최종 문장으로 수정
        B->>B: 충돌 표시 삭제 후 저장
        B->>B: git add README.md
        B->>B: git commit -m<br/>"B: A의 변경과 README 충돌 해결"
        Note over B: Merge Commit 생성 및<br/>MERGING 상태 종료
        B->>G: git push
        G-->>B: 충돌 해결 결과 반영
    end

    rect rgb(238, 247, 255)
        Note over A,G: A가 최종 Merge 결과 동기화
        A->>G: git fetch origin
        G-->>A: B의 Merge Commit 다운로드
        A->>A: git merge origin/main
        Note over A,B: 작업자 A, 작업자 B, GitHub가<br/>모두 같은 최신 커밋을 보유
    end
```

## 핵심 명령 흐름

```text
작업 시작: git fetch origin → git merge origin/main
작업 저장: 파일 수정 → git add → git commit → git push
충돌 해결: 충돌 파일 수정 → git add → git commit → git push
```

- `git fetch origin`: 원격 커밋 정보를 가져와 `origin/main`을 갱신하지만, 현재 `main`과 작업 파일은 바로 바꾸지 않는다.
- `git merge origin/main`: 원격 추적 브랜치의 변경을 현재 로컬 `main`에 병합한다.
- 충돌은 Merge를 실행한 작업자의 로컬 저장소에서 발생하며, 해당 작업자가 해결한다.
- 해결 결과를 Push한 후 다른 작업자도 `fetch → merge`로 최종 결과를 동기화한다.
