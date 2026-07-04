# oter Vibe Coding 채용 과제

## 한 줄 요약

레거시 PHP RealWorld 앱을 NHN Cloud NCS(Container Service)로 마이그레이션하고, 자기 선택 스택으로 재구현한 뒤, RealWorld 공식 E2E 테스트로 두 결과물의 동등성을 보증한다. 공터영어(oter)가 실제로 마주한 상황(레거시 PHP + 재구축 + 검증 안전망)을 축소한 실무 시뮬레이션이며, 레벨과 무관하게 **AI 네이티브 개발자**를 대상으로 한다.

## 이 과제로 보려는 것

바이브 코딩 시대의 개발자는 "얼마나 빨리 코드를 뽑아내는가"가 아니라 **AI를 도구로 활용하면서도 설계 의도를 스스로 통제하고, 검증 가능한 방식으로 결과물을 보증하며, 과정에서 성장하는가**로 평가받아야 한다고 믿습니다. 이 과제는 완성도 그 자체보다 **설계 의도와 성장 궤적**을 봅니다.

## 평가 역량

| 영역 | 배점 |
|---|---|
| 목표 완수 (마이그레이션 10 / 재구현 15 / 배포 10) | 35 |
| 아키텍처 · ADR | 20 |
| E2E 안전망 (동등성 검증) | 15 |
| 성장 · 학습 궤적 | 20 |
| 토큰 효율 | 10 |
| **합계** | **100** |

세부 배점, Must-have/Stretch 범위, 불합격 기준은 [docs/PRD.md](docs/PRD.md)를 참고하세요.

## 수행 작업과 평가 포인트

아래 M1~M6이 이번 과제의 실제 작업입니다. 각 작업의 **"포인트"는 평가자가 무엇을 보는지 / 어디서 실력이 갈리는지**입니다. 완성도보다 **판단의 근거와 검증**을 우선합니다.

### M1. 레거시 앱을 NCS에 배포 (마이그레이션)
제공된 **동작하는 레거시 PHP RealWorld 앱**(`app-legacy/`)을 컨테이너화해 NHN Cloud **NCS**(NHN Container Service, AWS Fargate류 관리형 컨테이너)에 배포한다.
- **포인트**: 앱을 실제로 기동시키는 컨테이너화·배포의 정확성. 코드 변경 범위는 재량이되 **왜 그렇게 했는지 ADR로 설명**. "일단 뜨게" 만드는 것과 "재현 가능하게" 만드는 것의 차이.

### M2. E2E 안전망 세우기 (baseline 확보)
RealWorld **공식 API 테스트 스위트**를 M1 배포본에 실행해 통과/실패 프로파일을 확보한다.
- **포인트**: 이 레거시 앱(v1 세대)은 최신 v2 스위트에서 **완전 그린이 아니다(약 388/456 통과)**. 미통과 건은 `e2e/README.md`에 문서화돼 있다. **"완전 그린"을 억지로 만들려 하지 말고, 지금 통과하는 것이 baseline임을 파악해 회귀 감시 기준선으로 삼는 것**이 핵심. 안전망을 "왜 이게 기준인가"까지 이해했는지 본다.

### M3. 선택 스택으로 재구현 (동등성 증명)
자기가 고른 언어·프레임워크로 같은 API를 재구현하고, **M2와 동일한 스위트에서 baseline 프로파일을 회귀 없이** 통과시킨다.
- **포인트**: 재구현이 레거시와 **동등하게 동작함을 E2E로 증명**하는 것. baseline 대비 회귀는 감점, 문서화된 미통과(68건) 해소는 가점. 스택 선택은 자유지만 **선택 이유를 ADR로** 방어할 수 있어야 한다.

### M4. 아키텍처 결정 기록(ADR) + IaC
재구현 스택 선택 근거를 **ADR**로 남기고, 인프라를 **Terraform**으로 코드화한다.
- **포인트**: NHN Terraform provider에는 **NCS·NCR 네이티브 리소스가 없다**(NKS만 지원). 이 갭을 발견하고, NCS/NCR 프로비저닝을 콘솔/NHN API/CLI/`local-exec` 중 무엇으로 할지 **직접 선택해 ADR로 정당화**하는지가 실력 지점. 지원 인프라(네트워크·LB 등)는 Terraform으로 코드화.

### M5. CI 파이프라인 (GitHub Actions)
`빌드 → NCR(이미지 레지스트리) → NCS 배포` 흐름을 **GitHub Actions**로 자동화한다.
- **포인트**: baseline이 완전 그린이 아니므로 **E2E 실패 시 빌드를 막을지 말지(CI 정책)를 스스로 정하고 ADR로 근거**를 남기는 것. 시크릿은 `secrets.*`로만 다루고 하드코딩하지 않는다.

### M6. 과정 드러내기 (데일리 push · 회고 · 토큰 효율)
매일 최소 1회 push하고, push 시 자동 생성되는 **Retrobot 회고(KPT)**와 **tokenhabit 토큰 리포트**를 남긴다.
- **포인트**: **AI를 숨기지 말고 드러내라.** 프롬프트·시행착오·복구 궤적이 성장 평가의 1순위 사료다. 데일리 push는 필수(미준수 감점). 토큰 낭비를 줄여가는 추세도 본다.

> 권장 순서: M1 배포 → M2 안전망 → M3 재구현 → (M4 ADR·IaC / M5 CI는 병행) → M6은 전 구간 상시. 상세 정의·산출물은 [docs/PRD.md](docs/PRD.md), 실행 방법은 템플릿 저장소 README를 따르세요.

## 기간 및 티어

- **기간**: 1주
- **Must 티어**: PRD의 M1~M6 필수 항목
- **Stretch 티어**: 관측성, 무중단 배포·오토스케일, 비용 최적화, 프론트 연동, 부하 테스트 중 선택

## 도구

- **Claude Code 필수** — 바이브 코딩 과정 자체(프롬프트, 커밋 이력, 회고)가 평가 대상입니다.

## 제출 및 발표

- 제출 방법과 일정: [docs/SUBMISSION.md](docs/SUBMISSION.md)
- 온라인 발표·Q&A 진행 방식: [docs/INTERVIEW.md](docs/INTERVIEW.md)

## 시작하기

[![Use this template](https://img.shields.io/badge/Use%20this%20template-2ea44f?style=for-the-badge&logo=github&logoColor=white)](https://github.com/roboco-io/oter-vibe-challenge-template/generate)

위 버튼을 누르면 템플릿에서 새 저장소를 만드는 화면으로 이동합니다. **Owner를 본인 계정으로, 저장소를 Private으로** 설정해 복제한 뒤 시작하세요. (버튼이 안 보이면 아래 링크를 직접 여세요.)

- 템플릿에서 새 저장소 만들기: https://github.com/roboco-io/oter-vibe-challenge-template/generate
- 템플릿 저장소: https://github.com/roboco-io/oter-vibe-challenge-template
