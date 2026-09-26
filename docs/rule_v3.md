# rule_v2.md — 수상권 코드 부정검증 실행 사양서

> **2026 인하 인공지능 챌린지(World Model)** 상위 7팀(트랙별) 코드 심사용.
> 적대적 검증자(Critic)가 **판정의 유일한 근거**로 삼는다.
> **v3.1** · 2026-08-25 · v1.0=`rule.md`, v2.0=`rule_v2.md` 별도 보존 (변경점은 §14)
> **킷 원본 실물 확보됨** — 부록 A·B 는 배포본 직접 판독 결과다(추정 아님).
> **실행 환경 고정**: bash(globstar off) · ripgrep 14.1.0(PCRE2) · python3. 이 환경에서 §6 명령 전량 실측.

---

## 1. 역할과 범위

검증자는 제출 아티팩트 `A = {전처리·학습·추론 코드, weight, 환경기재}` 를 **규범 R1~R9** 에 대해 판정한다.
**성능을 평가하지 않는다**(점수는 조사 우선순위에만). **선의를 가정하지 않으나, 증거 없이 유죄를 선언하지 않는다.** 출력은 **심사 자료**이지 처분이 아니다.
**범위 한정**: 코드 아티팩트만. 참가 자격(팀 구성·재학 요건)은 주최측 소관.

---

## 2. 근거 위계

**T1 > T2 > T3 > T4.** 하위는 상위를 뒤집지 못한다. 동일 티어는 後法優先.

| 티어 | 근거 | 효력 |
|---|---|---|
| T1 | 대회 [규칙] 탭 1)~7), [평가] 탭 | Violation 근거 |
| T2 | 운영진 공식 토크 공지 | Violation 근거 (T1과 동등) |
| T3 | 배포 README, 킷 README, 베이스라인 | 단독 Violation 불가 |
| T4 | **비공개 구두 추가 설명**(2026-08-07, 개별 청취) | **Violation 근거 불가.** 최대 Gray |

**T2 출처 대응표**

| ID | 공지명 | 조문 도출 |
|---|---|---|
| 417076 | Submission Kit 사용 기준 명확화 + 리더보드 초기화 (2026.08.03) | R7-a~e |
| 417050 | [리마인드] 제출킷은 제출 CSV 생성 목적으로만 | R7 보강 |
| 417168 | Private 리더보드 공개 및 코드 제출 안내 | R8-c·d |
| 417014·417015·417016 | 데이터 구조 / FAQ / 대회 주요 규칙 | 참고 — 조문 미도출 |

> ※ 위계는 **"무엇이 규칙인가"** 의 순서이며 **"무엇이 사실인가"** 의 증거력과 무관하다.
> T3 문서(README 실행 순서 등)는 팀의 사실관계 입증 증거로 제한 없이 쓴다.

### 2.1 T4 격하 — Gray 부여 전 필수 선(先)포섭검사

전체 참가자에게 공지되지 않은 해석으로 팀을 탈락시킬 수 없다. **다만 Gray 로 내리기 전 반드시 아래를 검사한다.**

```
Q-a. R7-0 의 "및 이를 통해 추출한 정보" 에 포섭되는가?
     (킷 실행 점수 / 킷 전처리 규격 재현값 / 킷 checkpoint 추출 임베딩 → 전부 포섭)
Q-b. 417076 의 제한 사례 1~5 중 하나에 해당하는가?
Q-c. R4 제2문("출력값이 달라지도록 수정")에 해당하는가?

→ 하나라도 YES: T1/T2 근거로 Violation. T4 격하 규정 적용 안 됨.
→ 전부 NO 인 잔여 영역(실무상 "킷 구조만 참고해 백지에서 자체 설계")만 Gray.
→ 이 3검사 수행 사실을 보고서에 기록한다.
```

> T4는 위반 성립 근거로 못 쓰나, T1/T2로 Violation 이 확정된 사안에 한해 처분란에
> "해당 팀은 2026-08-07 추가 설명을 직접 청취한 것으로 기록됨" 을 **사실만** 병기할 수 있다.

---

## 3. 규범 조문

> 인용부호 안은 **원문**. 화살표(→)는 검증자의 해석.

**R1. 사전학습모델 범위** 〔T1 규칙 1)〕
> "공식적으로 가중치가 공개된 사전학습모델 중, 상업적 이용 또는 비상업적 이용이 허용된 라이선스로 배포된 모델만 사용 가능" (MIT, Apache 2.0, CC BY, CC BY-NC 등)

→ **가중치 공개** + **라이선스 허용**, 두 요건 모두 필요.

**R2. API·외부 서버 의존 제한** 〔T1 규칙 2)〕
> "원격 서버 기반의 API 형태로만 접근 가능한 모델(OpenAI API, Gemini API 등)은 사용이 불가합니다. 모든 모델은 로컬 환경에서 직접 실행 가능해야 하며, **외부 서버에 의존하는 방식은 제한됩니다.**"

→ 기준선은 **"로컬에서 직접 실행 가능한가"**. 제2문 후단은 모델 호출에 한정되지 않는다.

**R3. 데이터 사용 제한** 〔T1 규칙 3) + 유의사항〕
> "본 경진대회에서 제공하는 데이터만 사용 가능합니다. 단, 제공하는 평가 데이터(eval)는 **어떠한 형태로도** 모델 학습에 활용할 수 없습니다."
> 〔유의사항〕 "**모델 학습에서** 평가 데이터(eval) 정보 활용(Data Leakage)시 수상 제외 **(평가 데이터 Pseudo Labeling 기법 포함)**"

- **R3-a** 평가 데이터 정보 활용 → D1 / **R3-b** 외부 데이터 → D3(대회 중)·D4(코드 검증)
- **R3-c** 〔해석 확정〕 "학습" = **파라미터·통계량·선택기준이 평가 집합 정보로 갱신되는 모든 절차**(갱신 대상 불문). **개별 샘플 자기 자신만을 입력으로 하는 변환은 제외.** 조문화된 규범이므로 P6 대상 아님.
→ **추론 시점의 선택·보정**은 R3 이 아니라 **R7-0·R7-d** 로 귀속. R3만으로 다투면 "학습엔 안 썼다"는 항변에 무너진다.

**R4. Submission Kit 변경 금지** 〔T1 규칙 4)〕
> "참가자는 `submission_kit`에 포함된 코드, 평가용 모델, checkpoint 및 가중치를 **어떠한 경우에도** 변경할 수 없습니다."
> "`submission_kit`을 변경하여 제출 파일을 생성하거나, **평가용 모델의 출력값이 달라지도록 수정하는 행위는 규칙 위반에 해당합니다.**"

→ 금지 대상은 **파일 변경**과 **출력값 변경** 두 갈래(원문 2문장). 해시가 동일해도 **런타임 개입**(monkey-patch, forward hook, 전처리 후킹, 라이브러리 셰도잉, 환경 주입)은 제2문의 직접 적용 대상이며 P6 의 확대해석이 아니다. `action_extractor.ckpt` 재학습·파인튜닝·덮어쓰기 금지.

**R5. 시간·하드웨어 제한** 〔T1 규칙 5) + T2 417168〕
> **[학습]** GPU: RTX PRO 6000 (96GB VRAM) **1대 기준** / 학습 시간 제한: 최대 4일
> **[추론]** GPU: RTX PRO 6000 (96GB VRAM) **1대 기준** / 추론 시간 제한: 평가 데이터(eval)에 대해 1시간 이내
> 〔417168〕 "학습은 학습 데이터(train)에 대하여 최대 4일 내, **단일** RTX PRO 6000 (VRAM 96GB)의 컴퓨팅 자원에서 작동 가능" / "추론은 평가 데이터(eval)에 대하여 1시간 내"
→ **추론 GPU 도 원문상 1대 기준이다.** "추론은 다중 GPU 허용"이라는 항변은 성립하지 않는다.

→ 〔해석 확정〕 "4일"은 **제출 weight 를 만들어낸 모든 학습 단계의 합**이다. 학습 코드가 로드하는 초기화 가중치는 (i) 제3자가 **대회 시작(2026-07-16) 이전 공개 배포**했거나 (ii) **제출 학습 코드가 4일 예산 안에서 스스로 생성**한 것이어야 한다.

**R6. CSV 생성 규칙** 〔T1 규칙 6)〕
> "리더보드 제출용 CSV는 반드시 대회에서 제공하는 `submission_kit` 폴더 내 `make_submission_csv.py`를 실행하여 생성해야 합니다."
> "`make_submission_csv.py` 실행으로 생성된 CSV 파일을 **임의로 조작, 보정 또는 수정**하는 행위는 규칙 위반에 해당합니다."

→ 금지 행위는 원문 그대로 **조작·보정·수정** 3종. "후처리"라는 조문 외 표현은 쓰지 않는다.

**R7. Submission Kit 사용 범위**

**R7-0 (T1 규칙 7) — 최상위 근거)**
> "Submission Kit은 참가자가 추론을 완료한 최종 영상 파일을 리더보드 제출용 CSV로 변환하는 용도로만 사용할 수 있습니다."
> "Submission Kit에 포함된 코드, 평가용 모델, checkpoint, 가중치 **및 이를 통해 추출한 정보**는 모델의 학습, 추론, **영상 선택·보정·수정** 등 최종 결과 생성 과정에 활용할 수 없습니다."
> "모델 학습과 추론은 Submission Kit 실행 전에 모두 완료되어야 하며, Submission Kit은 **최종 mp4 파일이 확정된 이후** 수정 없이 실행해야 합니다."

→ 금지 객체 = 4종(코드·모델·checkpoint·가중치) + **"이를 통해 추출한 정보"** = 킷을 매개로 얻은 일체의 2차 파생물(실행 점수, 전처리 규격 재현값, 추출 특징·임베딩, **그 출력으로 만든 라벨**).
→ 금지 용도 = 학습·추론 + **"영상 선택·보정·수정"**. **시점 한정 없음 — 대회 전 기간 적용.**

**R7-a~b (T2 417076)** — 금지 사례 5종 **(원문 그대로)**
> 1. "Submission Kit에 포함된 모델, 코드 또는 checkpoint를 활용하여 **학습용 특징값이나 정보를 추출하는 행위**"
> 2. "**Test 데이터의 특징값을 추출하여 예측 결과를 선택하거나 보정하는 행위**"
> 3. "Submission Kit의 **실행 결과를 활용하여 최종 mp4 파일을 생성·선택·수정하는 행위**"
> 4. "Submission Kit 내부의 **모델·코드·정보를 학습, 추론, 후처리 또는 앙상블 과정에 활용하는 행위**"
> 5. "그 밖에 Submission Kit을 **제출 CSV 생성 외의 목적으로 사용하여 모델 성능에 영향을 주는 행위**"

**R7-c (실행 순서)**
> ① 학습·추론 코드만으로 최종 mp4 생성 → ② 최종 mp4 확정 → ③ 킷을 수정 없이 실행 → ④ CSV 생성·제출
> "AI 모델의 학습과 추론, **후보 영상 생성 및 최종 영상 선택** 과정은 Submission Kit 실행 이전에 모두 완료되어야 합니다."

**R7-d (판단 기준)**
> "특히 **Test 데이터에서 추출한 정보를 활용하여 예측 결과를 보정하거나 선택하는 방식**은 평가 데이터 정보 활용으로 판단될 수 있으므로 유의"

**R7-e (적용 시점)**
> 〔417076〕 "본 공지 이후 제출되는 코드에서 … 확인될 경우 … 평가 제외 또는 수상 제외 **처리될 수 있습니다.**"
> ⇒ R7-0(T1)에는 시점 한정이 없고 417050 이 이미 동일 취지를 공지했다. **소급 항변은 어느 시점에도 성립하지 않는다.** 다만 행위 시점이 공지 이전임이 로그로 확인되면 처분란에 정상참작으로 기록.

**R8. 제출물 형식·검증 요건**

- **R8-a 형식**: 상대 경로 / `.py`·`.ipynb` / **코드·주석 UTF-8** / 오류 없이 실행(라이브러리 로딩 포함) / **개발 환경(OS)·라이브러리 버전 기재**
- **R8-b 필수 제출물**: Private Score 복원 가능한 **전처리·학습·추론 코드** / **모델 weight** / **추론 코드**(필수, 중요)
- **R8-c 검증 5종** 〔417168 원문〕
  ① "제출 코드(학습, 추론)으로부터 Private Score 재현 가능 여부"
  ② "Random Seed, Hyperparameter 등 재현에 필요한 설정값을 코드에 반드시 기재하고, **학습 로그를 함께 제출**해야 합니다."
  ③ "**학습 로그가 없는 경우 실제 학습한 Epoch 수 등 주요 학습 정보를 반드시 명시**해 주세요."
  ④ "규칙 위반 관련 (Data Leakage, 기타 치팅 요소 등)"
  ⑤ "코드 동작 여부와 **리소스 범위 내에서의 추론 여부**"
  → ②와 ③은 **성립 조건이 다른 독립 의무**다(로그 유무). 병합하면 "로그가 없으면 대체 의무도 없다"는 구멍이 생긴다.
  → ③ 위반(로그·Epoch 모두 미제공) 시 대조 불능은 Insufficient-Evidence 에 그치지 않고 **R8-c-③ 의무 위반으로 D2**, 예산 초과 정황 병존 시 R5 근거 **D4**.
- **R8-d 기한**: 2026.08.21(금) 14:00, `aix-server@inha.ac.kr`

**R9. 절차 규정** 〔T1 유의사항〕
- 언어 Python / 1일 제출 10회 (2026.08.03 11:00~, 이전 3회)
- "모든 csv 형식의 데이터와 제출 파일은 **UTF-8 인코딩**을 적용합니다."
- "Private Score는 **선택된 파일 중에서** 채점" → **V1 의 재현 대상은 팀이 최종 선택한 제출 파일 1개**
- "데이콘 대회 부정 제출 이력이 있는 경우 평가가 제한됩니다."
- "**대회 진행 중** 규칙 위반 의심 시 코드 제출 요청 … 요청 2일 이내 코드 미제출 **혹은 외부 데이터 사용이 확인되었을 경우** 리더보드 기록이 삭제됩니다."
  → **"대회 진행 중" 한정.** 대회 종료 후 코드 검증 단계에는 평가 탭의 "시상 취소 → 후순위 승계"가 적용된다.
- (T3 참고) 부정행위 세부 기준 https://dacon.io/notice/notice/13

---

## 4. 판정 원칙

| ID | 원칙 | 내용 |
|---|---|---|
| P1 | 증거 우선 | 모든 판정에 `파일:라인` 또는 함수명 인용. 인용 불가 → Violation 아님 (예외: P8-①) |
| P2 | 조문 귀속 | 모든 지적은 R1~R9 중 하나에 귀속. 귀속 불가 = 지적 불가 |
| P3 | 실행 경로 우선 | 도달 여부를 따진다. **단 아래 3개 예외** |
| P4 | 의도 불문 | "실수였다"는 성립을 막지 못한다. 처분란에 정상참작 기록 |
| P5 | 성능은 증거 아님 | 절대 점수는 근거 불가. **단 개입 전후 차분(differential)은 메커니즘 증거로 사용 가능** |
| P6 | 회색은 회색으로 | 조문 미명시 영역을 취지상 금지로 확대하지 않는다. **조문화된 해석은 대상 아님** — R3-c(transductive), R5 해석(학습 단계 합산), **R3-b 의 데이터-담지 파라미터 해석〔C7-b〕**, **R7-b-5 의 산식-표적 프록시 해석〔부록 A(ii)〕** |
| P7 | 재현성은 독립 축 | 부정 없어도 재현 불가는 그 자체로 수상 취소. 섹션을 분리한다 |
| P8 | 침묵도 결과다 | ① **R8-b 필수 제출물의 부재** → Violation(R8). 증거란에 `[부재] STEP 0 인벤토리 인용` 을 첨부하고 "결과 영향 경로" 대신 "재현 차단 지점"을 쓴다 ② 판정 근거 자료만 없으면 → Insufficient-Evidence |

**P3 예외 3종 (도달 불가로 취급하지 않는다)**

```
E1. if not exists(...) / try-except / 환경변수 분기로 감싸인 블록이라도,
    그 산출물이 제출물에 동봉되어 있으면 그 블록은 실행된 것으로 간주한다.
E2. importlib / __import__ / getattr / exec / 설정 주도 디스패치 / 문자열 레지스트리가
    개입한 지점의 하류는 "도달 불가"가 아니라 reachable="unknown" 이다.
E3. 자동 루프의 부재는 R7-c 준수의 증명이 아니다. 사람이 킷 결과를 보고 코드를
    고치는 것은 동일한 위반이다(조문은 행위 주체를 한정하지 않는다). → C11
```

---

## 5. 등급과 처분

**등급 서열(높음→낮음): Violation > Warning > Dormant-Risk > Gray > Pass**
`Insufficient-Evidence` 는 서열 밖 **보류 상태**. "한 단계 강등"의 대상도 결과도 아니다.

### 5.1 결정 순서 (배타적 — 복수 등급 병기 금지)

> **적용 단위는 finding 이다(팀 단위 아님).** 한 finding = 하나의 `(사실, 조문)` 쌍.
> **다른 finding 이 파일:라인으로 성립한 Violation 은, 이 팀의 무관한 자료 부재로 강등되지 않는다.**

```
Q0. 이 finding 이 R8-b 필수 제출물의 부재를 지적하는가?  → Violation (R8) [종료, P8-①]
Q1. 이 finding 의 판정 근거자료(해당 코드·로그·자산)가 없는가? → Insufficient-Evidence [종료]
Q2. 유일한 근거가 T4인가? (§2.1 선포섭검사 전부 NO)      → Gray [종료]
Q3. 진입점에서 도달 불가를 정적으로 증명했는가?
    (P3 예외 E1·E2 해당 시 증명 실패)                    → Dormant-Risk [종료]
Q4. 도달 확인 + 조문 요건 충족을 코드로 보였는가?         → Violation [종료]
Q5. 위반 가설이 남아 있는가?                             → Warning [종료]
Q6. 그 외                                                → Pass
```

### 5.2 처분 코드

`D1` 수상 제외 · `D2` 평가 대상 제외 · `D3` 리더보드 기록 삭제 · `D4` 수상 취소(후순위 승계)
(용어 통일: "시상 취소"=D4, "평가 제외"=D2)

| 위반 | 처분 | 근거 강도 |
|---|---|---|
| R3-a 평가 데이터 정보 활용 | **D1** | 유의사항 **명문** |
| R3-b 외부 데이터 | 대회 진행 중 D3 / **코드 검증 단계 D4** | 시점 요건 주의 |
| R7 킷 목적 외 사용 | 최종 결과 반영 확인 → **D1** / 미확인 → **D2** | 417076 **재량 문언**("처리될 수 있습니다") |
| R4 킷 변경·출력값 변경 | D2 | 명문 처분 없음 → 평가 탭 포괄 조항 |
| R6 CSV 조작·보정·수정 | D2 | 명문 처분 없음 → 포괄 조항 |
| R1 / R2 | D2 | 명문 처분 없음 → 포괄 조항 |
| R5 / R8 재현 불가·예산 초과 | **D4** | 평가 탭 **명문** |
| R9 자료 미제출 2일 경과 | D3 | 대회 진행 중 한정 |

> 명문 처분이 없는 행은 보고서에 **"명문 처분 없음 — 포괄 조항 적용"** 을 병기한다.
> 소명 기한은 R9 에 따라 **요청일로부터 2일**.

---

## 6. 부정행위 검사 C1~C15

> rg 패턴은 **1차 스크리닝**이다. 히트 = 위반 아님. 모든 명령은 ripgrep 14.1.0 에서 실행 검증됨.
> **종료 코드 판정(ripgrep 규약: 0=매치, 1=무매치, ≥2=오류)**: `exit=0`·`exit=1` 은 **둘 다 정상 수행**으로 기록한다. **`exit≥2` 만 "미수행/오류"** 이며 "히트 없음"으로 간주하지 않는다. `find`·`sha256sum`·python 은 각 도구 규약대로 `exit≠0` 을 오류로 본다. 결과는 `{수행됨|미수행}` 과 `{히트 n건}` 을 **분리 기록**한다.
> **파이프 뒤에 `rg -v` 를 걸지 말 것** — rg 출력 줄에는 파일 경로가 포함되어 배제하려던 것과 반대의 줄이 삭제된다. 경로 제외는 `--glob '!패턴'` 으로.

---

### C1. 원격 API·외부 서버 의존 〔R2〕

```bash
rg -n -i "openai|anthropic|google\.generativeai|vertexai|runwayml|stability" --type py --type jupyter --glob '!**/.ipynb_checkpoints/**'
rg -n -i "\b(import|from)\s+(together|groq|cohere|replicate|dashscope|zhipuai|mistralai|google\.genai)\b" --type py --type jupyter
# ↑ together·groq 등 일상어와 겹치는 벤더명은 import 문맥으로만 매칭(오탐 억제)
rg -n -i "api[_-]?key|Authorization|sk-[A-Za-z0-9]{20,}|InferenceClient"
rg -n "requests\.(post|get)|httpx|aiohttp|urllib\.request"
```

| 상황 | 등급 |
|---|---|
| API 결과가 모델 입력·출력·선택에 반영 | **Violation (R2)** |
| 실행 경로에 API 호출은 없으나, 코드가 읽는 캐시·중간 산출물(.json/.npy/.pt)이 **API 유래임이 코드·주석·파일명·커밋으로 확인** | **Violation (R2)** |
| 위와 같으나 출처 특정 불가 | Warning + 소명("이 파일 생성 코드를 제출하라. 미제출 시 R8 필수 제출물 누락") |
| 결과가 mp4 에 기여하지 않는 외부 의존(원격 로깅 등) | Warning (R2 제2문 후단) |
| `from_pretrained` 등 **가중치 다운로드만** | Pass (R1) |

---

### C2. 평가 데이터 누수 〔R3-a, R3-c, R7-b-2〕

```bash
rg -n -e "data/eval" -e "eval/images" -e "eval/actions" --type py --type jupyter
rg -n -i "pseudo|self.?label|distill.*eval|ConcatDataset|semi.?supervised"
rg -n "register_buffer|running_mean|running_var|\.mul_\(|\.add_\(|momentum|ema|accumul"
rg -n "os\.listdir|glob\.glob|rglob|iglob"     # 재귀 glob 이 eval 을 삼키는 사고가 잦다
```

| 단계 | 형태 | 등급 |
|---|---|---|
| L4 | eval 이 학습 루프 손실 계산에 진입 | **Violation (R3-a)** |
| L3 | eval pseudo-label 후 재학습 | **Violation (R3-a)** — 유의사항 명문 |
| L2 | eval 통계량으로 정규화·보정 | **Violation (R3-a·R3-c)** |
| **L2′** | **eval 유래 통계가 상수·캐시 파일로 고형화되어 코드엔 로드만 남음** | **Violation (R3-a)** — 코드에 eval 경로가 없다는 사실은 무죄 근거가 아니다 |
| L1 | eval 특징값으로 후보 **선택·보정** | **Violation (R3-a + R7-0 "영상 선택·보정·수정")** |
| L0 | eval 샘플의 **입력 지정 파일만** 읽어 그 샘플의 출력을 생성 | Pass |

**결정선**: eval 은 **추론 함수의 인자로 들어가 그 샘플의 출력을 만드는 것**까지만 허용. **집합 전체를 훑어 학습·추정·선택하는 순간(transductive)** 위반.
→ 통계 모집단이 단일 샘플 / 샘플 단위 TTA → Pass. **2개 이상 eval 샘플에 걸침**, `model.train()` 추론 중 running stat 갱신, test-time training·adaptation → **Violation (R3-c)**.
→ 코드로 안 잡히는 형태는 **V7** 로 판정한다.
→ **L2′ 필수 절차**: 로드 전용 통계 자산(`.npy/.pt/.json`)은 **생성 스크립트를 신고된 입력(train)으로 재실행**해 값 일치를 확인한다(→ **V8-②**). 불일치 = 신고 출처와 다름(=eval 유래) → **Violation (R3-a)**. 캐시가 고형화되면 어떤 순서로 실행해도 결과가 같아 **V7 만으로는 잡히지 않는다.**

---

### C3. 킷 구성요소의 학습·추론 사용 〔R7-0, R7-b-1·4, R4〕

```bash
rg -n "submission_kit|action_extractor|feature_csv_utils|make_submission_csv"
rg -n "sys\.path\.(append|insert).*submission"
rg -n "importlib|__import__|\bexec\(|\beval\([^)]"          # model.eval() 제외됨
rg -n -i "teacher|distill|student|soft.?label|aux_target|pseudo.?target|kd_loss"
rg -n "frombuffer|b85decode|b64decode|zlib\.decompress|lzma|pickle\.loads|np\.load\("
rg -n --pcre2 '["'"'"'][A-Za-z0-9+/=~!#$%&()*_-]{4096,}["'"'"']'   # 4KB 초과 문자열 리터럴
find . -size +1M ! -name "*.mp4" ! -name "*.png"                   # 대용량 자산 전량 열거
```

| 상황 | 등급 |
|---|---|
| 킷 checkpoint 출력이 학습 손실(perceptual/critic)에 진입 | **Violation (R7-b-1·4)** |
| 킷 모델로 후보 영상을 점수화해 best-of-K 선택 | **Violation (R7-b-3·4)** |
| 킷 코드 import → 로컬 모의채점 → 체크포인트 선택 | **Violation (R7-b-3·5)** |
| 킷 checkpoint 로 자체 IDM 초기화 | **Violation (R7-b-1·4)** |
| **킷 모델의 출력을 라벨·타깃·보조손실로 사용(라벨 파일 포함)** | **Violation (R7-0 "이를 통해 추출한 정보")** |
| 킷 파일 복사만 하고 실행 경로 미사용 | Dormant-Risk + 소명 |
| **킷 아키텍처 구조만 참고**해 백지 설계 (§2.1 3검사 전부 NO) | Gray + 주최측 확인 요청 |

**C3-f. 함수 동치성 검사** (해시 대조를 대체하는 필수 절차 — 부록 A-3 규격)

```python
import torch, sys; sys.path.insert(0, "submission_kit")
from action_extractor import load_so100_action_extractor_checkpoint
ref = load_so100_action_extractor_checkpoint(
        "submission_kit/checkpoints/action_extractor.ckpt", map_location="cpu").eval()
torch.manual_seed(0)
x = torch.rand(2, 3, 16, 320, 512) * 2 - 1        # 부록 A-3 입력 규격, [-1,1]
with torch.no_grad(): y_ref = ref(x)              # (2, 16, 6)
# 팀의 모든 .pt/.ckpt/.pth/.safetensors 에 대해 동일 x 를 통과시켜 y_team 산출 후 비교
```

→ **차원별 상관계수 최대치 ≥0.99** 또는 **출력 순열 매칭 후 MAE <1e-3** → **Violation (R4·R7-b-1)**.
→ 키 리네이밍·채널 순열·Identity 래퍼·`strict=False` 로 바이트·레이어 수를 바꿔도 통과하지 못한다.
→ **1차 스크리닝**(torch 없이도 가능): ckpt zip 내 `data.pkl` 을 `pickletools` 로 판독해 **state_dict 텐서 키집합**을 뽑고 부록 A-3 의 지문 `8da3368604755b422721432ed24289b0def67f39306b0a94b57730379c5e676e` 와 대조. 일치 = 원본 이식 확정, 불일치 = 무죄 아님(리네이밍 가능) → 본 검사로 진행.

**C3-g. 출처 선언표** (입증 책임 전환)
모든 weight·라벨·코드북에 **「파일명 / 생성 스크립트 경로:라인 / 출처」** 3열 선언 요구 → 생성 스크립트 미지목 시 **Warning (R4)**, 라벨이면 **Violation (R8·P8-①)**.

라벨↔원본 ckpt 출력(train 영상 적용) 대조 — **단일 상관 임계는 노이즈 주입으로 뚫린다.** 셋 중 하나면 **Violation (R7-0)**:
- **Pearson >0.75** 또는 **Spearman 순위상관 >0.85** 또는 **상호정보량 상위 10%**
- **잔차 구조성**: 팀 라벨에서 원본 출력을 회귀로 제거한 잔차가 백색잡음이 아니면(자기상관·주성분 편중) → 원본 출력에 노이즈를 얹은 것으로 판정
- **동봉 스크립트 실행 대조**: 신고된 '자체 IDM' 생성 스크립트를 실제 실행해 그 출력과 제출 라벨의 상관이 **<0.6** → 생성 경위 허위 → **Violation (R8·P8-①)**

> 순위 정보만 보존한 채 상관을 임계 아래로 낮추는 위장이 실재한다. **상관 하나로 판정하지 않는다.**

---

### C4. 킷 실행 순서 역전 〔R7-c〕

grep 으로 안 잡힌다. **위상 정렬**으로 판정한다.

```
정상: [train]→[infer]→mp4 확정→[make_submission_csv.py]→CSV        (단방향)
위반: [infer]→후보 mp4→[kit]→score
             ↑___________________|                                  ← 이 화살표
```

1. 진입점(`main.sh`, `run_all.py`, README 순서)을 위상 정렬
2. `make_submission_csv.py` / `submission_features.csv` 를 **읽는 노드** 탐색
3. 그 노드가 `input_videos/` 에 **쓰는 노드**보다 먼저 오는 경로가 있으면 역전

| 상황 | 등급 |
|---|---|
| 킷 실행 → 결과 읽기 → mp4 재생성/재선택 → 킷 재실행 | **Violation (R7-b-3·c)** |
| `for` 루프 안의 킷 호출 / 반복 실행 로그 | **Violation** |
| 킷 산출물을 read 하는 파일:라인 + mp4 도달 경로를 **모두 인용 가능** | **Violation** |
| read 는 인용되나 도달 경로를 못 그림 | Warning |
| 킷을 1회만, 마지막에, 수정 없이 | Pass |

- `.ipynb` 의 `execution_count` 역전은 **단독으로 Warning 을 넘지 못한다**(P5).
- 자동 루프가 없어도 면책이 아니다 → **C11**.

**C4-b. CSV 자기채점 누수** 〔R7-b-2·3, R3-a〕 ★ 킷 실측으로 새로 드러난 경로

킷이 쓰는 `Action Component` 값은 특징 벡터가 아니라 **그 샘플의 Action MAE 스칼라 그 자체**다(부록 A-1).
→ **킷 1회 실행 = 배점 0.4 의 샘플 단위 정답 채점표 획득.** 재실행 루프도, 킷 코드 import 도 필요 없다.
→ C4 의 위상 검사(킷 재실행 화살표)로는 **원리적으로 잡히지 않는다.**

```bash
rg -n "Action Component|feature_component|submission_features"
rg -n "read_csv|json\.loads|pd\.read_|csv\.reader" -A5 | rg -n "Action|argmin|argsort|sort_values|idxmin|best"
```

| 상황 | 등급 |
|---|---|
| 코드가 생성 CSV 의 `Action Component` 행을 읽어 후보 선택·보정·재생성에 사용 | **Violation (R7-b-2·3)** — 최중대 |
| CSV 를 읽는 코드는 없으나, 샘플별 상수·후보 선택 흔적이 있음 | **C11-a·C12 로 이관** (사람이 CSV 를 눈으로 보고 반영한 경우) |
| 킷 실행이 1회이고 그 산출 CSV 를 읽는 노드가 없음 | Pass |

> **입증 곤란 인정**: 사람이 CSV 를 열어 보고 손으로 반영하면 코드에 흔적이 없다(C11 과 동형).
> 따라서 검증자는 **C11-b 필수 제출물(실험 로그·git log·로컬 킷 실행 흔적)** 과 **C11-d 포렌식**을
> 이 항목에서도 반드시 수행하고, 수행 사실을 보고서에 기록한다.

---

### C5. 킷 파일·가중치 변조 〔R4〕

```bash
find submission_kit -type f -print0 | sort -z | xargs -0 sha256sum   # 부록 B 와 대조
rg -n "action_extractor.*\.(train|fit|backward|requires_grad_?\(True\))"
rg -n "monkey|patch|setattr\(|\.forward\s*=|register_forward_(pre_)?hook"
```

| 상황 | 등급 |
|---|---|
| `action_extractor.ckpt` 재학습·파인튜닝·덮어쓰기 | **Violation (R4)** — 최중대 |
| **실행 경로상의** 킷 `.py` 변경(해시 불일치) | **Violation (R4)** |
| monkey-patch / forward hook 으로 출력 변경 | **Violation (R4 제2문)** |
| 킷 출력에 영향 주는 라이브러리(torch, torchvision, timm, av/decord/imageio-ffmpeg, pillow, numpy)를 킷 `requirements.txt` 와 다른 버전으로 실행 | **Violation (R4) 후보** → **V6** 에서 재실행 대조 후 확정 |
| 무관한 라이브러리 버전 차이 | Warning → V6 |
| 킷 `input_videos/` 에 mp4 배치 | Pass |
| 킷 파일 해시는 전부 일치하나 **실행 인자가 기본값과 다름** | **C5-b 로 이관** |

**C5-b. 킷 실행 인자 변조** 〔R4 제2문, R7-0 "수정 없이 실행"〕 ★ 킷 실측으로 새로 드러난 경로

`make_submission_csv.py` 는 CLI 인자를 받는다. **파일을 1바이트도 고치지 않고 출력만 바꿀 수 있다** → STEP 1 해시 대조를 100% 통과한다.

| 인자 | 기본값 | 변경 시 효과 |
|---|---|---|
| `--action-extractor-ckpt` | `checkpoints/action_extractor.ckpt` | **다른 ckpt 로 교체 — R4 직접 회피** |
| `--pad` / `--no-pad` | `True` (letterbox) | 전처리 변경 → 3성분 전부 달라짐 |
| `--target-height` / `--target-width` | 320 / 512 | 동상 |
| `--temporal-length` | 16 | 프레임 수 검증 우회 |
| `--challenge-root` | `../data/eval` | 정답 action 소스 변경 |
| `--action-stats-path` | `../data/train/so100_action_statistics.json` | 정규화 스케일 변경 → Action MAE 조작 |
| `--feature-precision` | 6 | CSV 반올림 자리수 변경 |

```bash
rg -n "make_submission_csv" -A3 -g '*.sh' -g '*.md' -g '*.py' -g '*.ipynb' -g '*.yaml' -g '*.yml'
rg -n -- "--pad|--no-pad|--target-height|--target-width|--temporal-length|--feature-precision|--action-extractor-ckpt|--challenge-root|--action-stats-path"
```

| 상황 | 등급 |
|---|---|
| 기본값이 아닌 인자로 킷 실행(스크립트·README·노트북에 인용 가능) | **Violation (R4·R7-0)** |
| `--action-extractor-ckpt` 를 다른 파일로 지정 | **Violation (R4)** — 최중대 |
| 실행 명령 기록이 전혀 없음 | Insufficient-Evidence + 소명(실행 명령 제출 요구) |
| `python make_submission_csv.py` 인자 없이 실행 | Pass |

**결정적 검사(동적 모드)**: 팀의 최종 mp4 216개를 **원본 킷 + 기본 인자**로 재실행해 CSV 를 생성하고,
팀이 제출한 CSV 와 대조한다 → **불일치 = 인자 변조 또는 CSV 후처리 확정** (→ V6·C6).

> **부록 B 기준 해시는 확정되었다(v3.1).** 해시 대조를 수행하고 불일치는 Violation 으로 기록한다.
> **단 해시 일치는 필요조건일 뿐이다** — 파일을 그대로 두고 실행 인자로 출력을 바꾸는 경로는 **C5-b** 가 담당한다.

---

### C6. CSV 조작·보정·수정 〔R6〕

```bash
rg -n "submission_features\.csv|to_csv|read_csv|feature_json|feature_component"
```

| 상황 | 등급 |
|---|---|
| read_csv → 값 변경 → to_csv | **Violation (R6)** |
| 복수 CSV 앙상블·평균·가중합 | **Violation (R6 + R7-b-4)** |
| 재작성하나 **값은 비트 단위 동일**(정렬·컬럼순·인코딩만 변경) | **Violation (R6)** — "수정"에 해당 |
| 행 수·파일명만 assert 하고 **write 경로 없음** | Pass |
| `sample_submission.csv` 참고 read 만 | Pass |
| read 는 있으나 write 경로 추적 불완전(동적 경로) | Warning |

**C6-b. 산식 표적 변환** 〔R7-b-5〕 — 대상 시점은 "최종 mp4 이후"가 **아니라 모델 순전파 출력(프레임 텐서) 이후의 모든 결정론적·비학습 변환**이다. 감마·샤프닝·블렌딩·리샘플링·색보정 + **인코딩 파라미터**(crf, pix_fmt, colorspace, tune)를 전량 열거하고 도출 근거를 요구한다. 근거가 **킷 실행 결과 또는 eval 점수**면 → **Violation (R7-b-5·R3-a)**. → **V9**.
> 시점을 "최종 mp4 이후"로 두면 **프레임 배열 생성 직후·인코딩 직전**에 변환을 끼우고 "증강"이라 주석하는 회피가 성립한다. 모델이 아니라 **파이프라인이 만든 변환은 전부 대상**이다.

---

### C7. 외부 데이터 〔R3-b〕

```bash
rg -n -i "datasets\.load_dataset|kaggle|gdown|urlretrieve|snapshot_download|git clone"
rg -n -i "ego4d|epic.?kitchens|something.?something|bridge|rt-1|open.?x|droid|calvin|rlbench|kinetics|webvid|laion|ucf101"
rg -n --glob '!**/data/**' -i "\.(zip|tar|tar\.gz|tgz|parquet|arrow|7z|rar)([\"'\`)[:space:]]|$)"
find . -type f \( -name '*.zip' -o -name '*.tar*' -o -name '*.parquet' -o -name '*.arrow' \)
rg -n -i "ego4d|epic.?kitchens|something.?something|bridge|rt-1|open.?x|droid|calvin|rlbench|kinetics|webvid|laion|ucf101" --glob '**/config.json' --glob '**/trainer_state.json' --glob '**/*.json'
# ↑ 경로 리터럴을 `--` 뒤에 두지 말 것(bash globstar off 에서 exit=2). rg 자체 --glob 은 ** 를 처리한다.
# ↑ 확장자 뒤 따옴표·괄호·공백·줄끝을 모두 허용($ 단독 앵커는 실제 코드를 전부 놓친다)
```

| 상황 | 등급 |
|---|---|
| 외부 데이터셋 로드 후 학습 | **Violation (R3-b)** |
| 외부 데이터로 사전학습한 자체 체크포인트 사용 | **Violation (R3-b)** |
| 공개 사전학습 가중치 로드 (라이선스 적합) | Pass (R1) → **C14** |
| 제공 데이터만으로 만든 증강·합성 | Pass |
| 실행 경로 밖 외부 데이터 경로 | Dormant-Risk (단 P3 예외 E1 확인) |

**C7-b. 데이터-담지 파라미터 검사**
첫 차원이 **10⁴ 이상인 임베딩·코드북·버퍼**를 전량 열거하고:
① 생성 학습 코드가 제출물에 없음 → **Violation (R8)** + R3-b Warning
② train 전량을 제출 인코더로 임베딩한 집합과의 **최근접 매칭률**이 낮음(cos>0.95 매칭 20% 미만) → **Violation (R3-b) 추정**, 반증 책임 팀
③ 행 수가 train 클립 수와 자릿수 이상 차이 → 소명 요청
④ **백본 암기 검사** — 외부 지식은 임베딩 테이블이 아니라 **일반 conv/linear 가중치**에도 녹일 수 있다. 첫 차원 필터를 통과한 정상 형태의 백본에도 기준("특정 외부 샘플 복원 가능성")을 적용한다: 제출 백본에 train·외부 후보 영상을 통과시켜 **선형 프로빙 재구성 오차**를 측정, 외부 클립 쪽이 유의하게 낮으면 → **Violation (R3-b) 추정**, 반증 책임 팀

> **조문 해석**: R1 이 허용하는 것은 **모델 파라미터**다. 데이터의 저차원 표현(임베딩·코드북·룩업테이블·암기 디코더)을 파라미터 형식으로 반입하는 것은 R3-b 의 외부 데이터 사용으로 본다. 기준은 파일 형식이 아니라 **그 텐서가 특정 외부 샘플을 복원 가능한가**.

---

### C8. 단일 GPU 실행 가능성 〔R5·R8〕 — 주관할: 실행 가능성만

```bash
rg -n "DataParallel|DistributedDataParallel|torch\.distributed|init_process_group|torchrun|deepspeed|FSDP"
rg -n "cuda:[1-9]|device_ids=|CUDA_VISIBLE_DEVICES|nproc_per_node|world_size"
```

| 상황 | 등급 |
|---|---|
| `cuda:1` 이상 하드코딩 / `world_size>1` 고정 → 단일 GPU 실행 불가 | **Violation (R5·R8)** |
| `world_size` 를 환경에서 읽고 **1일 때의 분기가 인용 가능** | Pass |
| 폴백 분기를 코드로 확인 불가 | Warning (동적 모드에서 재확인) |
| 다중 GPU 전제이며 **환산 시간이 쟁점** | 등급 부여하지 않음 — **C9 에서 단독 판정** |

---

### C9. 시간·자원 예산 〔R5〕 — 주관할: 예산 전체

**추론**
```
T_max  = 3600초 (eval 216샘플 전체, I/O·모델 로딩 포함)
t_load = 1회성 모델 로딩·초기화 시간 (팀 로그의 최초 추론 시작 타임스탬프로 산정,
         근거 없으면 30초를 보수적으로 가정하고 보고서에 명기)
t_max  = (3600 − t_load) ÷ 216      ← T_max 에는 로딩이 포함되므로 t_est 와 직접 비교하면 안 된다
t_est = (샘플당 diffusion step) × (프레임 수) × (step당 latency) × K(후보 수)

t_est > t_max            → Violation
0.8·t_max < t_est ≤ t_max → Warning (마진 20% 미만, 실측 로그 요구)
t_est ≤ 0.8·t_max        → Pass
(t_load=30초 가정 시 t_max ≈ 16.53초, Warning 구간 ≈ 13.2~16.53초)
```

**학습** — 모든 단계(사전학습→파인튜닝→증류)의 **합**. FLOPs→시간 환산은 **bf16 실효 처리량 + MFU 0.35** 가정, **가정치를 보고서에 명기**(근거 없으면 Insufficient-Evidence).
**다중 GPU 환산** = `Σ(GPU 가동시간) × (실효 처리량 비)`. 통신 오버헤드는 0으로 두어 **팀에게 유리하게** 계산.

| 상황 | 등급 |
|---|---|
| 학습 총합 4일 초과 / 추론 t_est > 16.67초 | **Violation (R5)** |
| 학습 로그 첨부 + **교차검증 3항 전부 통과** | Pass |
| 로그는 있으나 교차검증 실패·불가 | Warning |
| 산정 근거 부재 | Insufficient-Evidence + 소명 |

**교차검증 3항** (로그는 팀 제출 텍스트다 — 단독 Pass 근거 불가)
① step 수 × 배치가 코드의 epochs·데이터셋 크기와 정합 ② 타임스탬프÷step 이 모델 규모와 물리적으로 정합 ③ ckpt mtime 이 로그 구간에 포함

**C9-e. ckpt 메타데이터 포렌식** (미제출 학습 단계의 유일한 물증)
```python
sd = torch.load(w, map_location="cpu")
print({k: sd[k] for k in ("global_step","epoch","step") if k in sd},
      sd.get("optimizer",{}).get("state",{}).get(0,{}).get("step"))
```
→ ckpt 의 step 수가 **제출 학습 스크립트 총 스텝 수를 초과**하면 **Violation (R5) 확정**.
→ **증거 인멸을 보류로 보상하지 않는다.** 재현에 필요한 학습 메타데이터(`global_step`/`epoch`/optimizer state) 중 **최소 1종도 남아 있지 않고** R8-c-③(로그 부재 시 Epoch 등 명시)도 미충족이면 → **Violation (R8-c-②·③ 의무 위반, D2)**. 예산 초과 정황(대용량 ckpt·다단계 파이프라인)이 병존하면 R5 근거 **D4**. 원본 ckpt 재요청은 병행하되 **미제출 2일 경과 시 위 판정을 확정**한다.
→ **부분 실행 latency 측정** (로그·메타데이터에 의존하지 않는 물증): 제출 학습 스크립트를 **1/20 step 만 실행**해 step당 벽시계 시간을 측정 → `전체 step × 측정 latency` 가 4일 초과면 **Violation (R5)**.

> 메타데이터를 남긴 정직한 팀만 걸리고 지운 팀은 보류로 시간을 버는 구조를 만들지 않는다.

**C9 추가 점검**: `□ 학습 스크립트가 로드하는 모든 초기 가중치가 (a) C14 공개모델 선언표에 있는가, 또는 (b) 제출 학습 코드의 torch.save 로 생성됨을 추적 가능한가` — 아니면 **Violation (R5)**.

---

### C10. 결과 위조 〔R8〕 — 위조 전용. 실행 편의성 결함은 V3/V4 로 보낸다

```
□ 216개 mp4 전량 생성 루프의 존재
□ 제출 weight 를 추론 코드가 실제 로드하는가 (키·shape 대조)
□ 학습 코드 산출 체크포인트 형식 == 제출 weight 형식
□ 하드코딩 출력·캐시 산출물(.npy/.pt/.json)이 추론을 대체하고 있지 않은가
□ 제출 weight 내 **모든 non-parameter buffer 값**이 학습 코드 실행만으로 재현되는가
□ 추론이 읽는 사전계산 자산의 **eval 216개 조회 적중률**을 측정했는가
```

- 재현 불가 버퍼 값 → **Violation (R3-c 추정)**, 반증 책임 팀
- 사전계산 자산 적중률이 train 조회보다 유의하게 높음 → **Violation (R3-a)** / ≈100% → **Violation (R3-a·R5) 확정**
- 자산의 첫 차원·원소 수가 **216(또는 Private 분할 크기)과 일치** → **C12**
- 판정: **산출물이 코드로부터 생성되지 않았음**을 시사하면 → Violation (R8)

---

### C11. 인간 매개 누수 〔R7-0, R7-b-3·5, R7-c, R3-a〕 ★ 최대 사각지대

**정의**: 팀이 로컬에서 킷을 반복 실행해 점수를 보고, **사람 손으로** 코드의 상수·설정을 조정하는 구조. 자동 루프가 없어 C4 위상 검사를 통과한다.

> **P3 예외 E3**: 자동 루프의 부재는 R7-c 준수의 증명이 아니다. 조문은 "실행 결과를 활용하여 최종 mp4 를 생성·선택·수정"을 **행위 주체 무관하게** 금지한다.

| 절차 | 내용 |
|---|---|
| C11-a 매직 상수 소명 | 실행 경로의 **학습으로 도출되지 않은 수치 리터럴** 전량 열거 → 「도출 절차/사용 데이터/코드 경로」 요구. 미기재 **Warning** / eval·킷 결과 사용 확인 **Violation (R3-a)** |
| C11-b 제출물 확대 | 실험 로그·노트북·**git log**·로컬 킷 실행 흔적. 미제출 → Insufficient-Evidence + 2일 내 요구 |
| C11-c 대역폭 대조 | 제출 횟수 vs 손튜닝 파라미터 수. **유효숫자 3자리 이상 스칼라 3개 초과** → 로컬 채점 루프 가정, 소명. **군집 배정 함수가 입력에서 결정론적으로 계산됨을 코드로 못 보이면**(사람 눈 분류의 룰 근사) 그 자체가 소명 대상. C13 이력과 교차: `군집 수 × 조정 스칼라 ≈ 제출·실행 횟수` → Warning |
| C11-d 포렌식 | `input_videos/` mp4 mtime, `__pycache__`, `.ipynb` execution_count, 잔여 로그 → **킷 실행 ≥ mp4 확정** 물증 |

---

### C12. 샘플별 상수 〔R7-b-3, R3-a〕

**정의**: eval 샘플 ID·인덱스·해시를 키로 하는 상수 집합(시드·step 수·guidance scale·후보 인덱스·노이즈 텐서)이 추론에 영향을 주는 구조. **시드 고정은 C10 의 Pass 요건이라 최상의 위장 명분이 된다.**

```bash
rg -n "seeds?\.json|per_sample|sample_[0-9]{6}\"|SEEDS\[|CFG\[|best_idx|chosen|selected"
# 모든 json/npy 에 대해: 원소 수 == eval 샘플 수(216) 인가
```

| 상황 | 등급 |
|---|---|
| **원소 수가 216 또는 216의 약수**(2·3·4·6·8·9·12·18·24·27·36·54·72·108)이거나, eval 을 그 크기로 균등 분할한 **군집 인덱스와 정합**하는 상수 집합이 추론에 영향 | **Violation (R7-b-3·R3-a) 추정** — 도출 절차를 증명 못 하면 확정 |
| 전 샘플 공통 단일 시드 | Pass |
| 샘플별 상수가 **입력에서 결정론적으로 계산**(`seed = crc32(sample_id)`)됨이 코드로 확인 | Pass |

**결정적 검사**: 샘플별 상수를 **전부 동일 값 또는 `crc32(sample_id)` 로 치환**하고 재실행. 점수가 유의하게 악화되면 → 그 상수가 **eval 채점 결과에서 유래한 선택**임이 확정 → **Violation**. (P5 단서: **개입 전후 차분**은 메커니즘 증거다.)

---

### C13. 리더보드 프로빙 〔R3-a, R7-d〕

**정의**: 1일 10회 제출로 Public 30% 응답에서 평가 정보를 역추출해 후보·하이퍼파라미터 결정에 반영. **코드에 남지 않으므로 운영진의 제출 이력이 필요하다.**

신호: ① 총 제출 횟수가 상위 팀 중앙값의 3배 이상 ② 연속 제출 간 변경점이 단일 스칼라뿐인 이력 20회 이상 ③ Public 점수 개선 폭이 단조 감소하는 격자 탐색 패턴

| 상황 | 등급 |
|---|---|
| 신호 2개 이상 + **코드에 제출 이력 참조 선택 로직** | **Violation (R3-a)** |
| 신호만 있고 코드 연결 없음 | Warning |
| 제출 이력 미수령 | Insufficient-Evidence (운영진에 요청) |

> 제출 '이력의 형태'는 점수 자체가 아니므로 P5 대상이 아니다. 단 **단독으로 Violation 을 만들지 못한다.**

---

### C14. 사전학습 가중치 출처·라이선스 〔R1, R3-b〕

**필수 산출물 7열표**: `모델명 / 출처 URL / 라이선스 / 가중치 공개 여부 / **최초 공개일자** / **배포 주체** / **배포 주체와 팀의 관계**(무관·팀원·소속기관·미상)`

| 상황 | 등급 |
|---|---|
| 배포 주체가 **팀원·팀 소속 계정·미상** + 최초 공개일이 **2026-07-16 이후** | **Violation (R3-b) 추정** — 반증 책임 팀 |
| 대회 기간 중 공개이나 배포 주체가 제3자 기관 | Warning + 소명 |
| 대회 시작 이전 공개 + 제3자 배포 + 모델카드·다운로드 이력 존재 | Pass |
| 라이선스 불명·재배포 제한·가중치 미공개 | Warning |
| 제출 weight 의 키·shape 가 제출 학습 코드 산출물과 불일치 | **Violation (R8)** → C10 |

포렌식: 로컬 캐시 `config.json` 의 `_name_or_path` / `trainer_state.json` 에서 **원 학습 데이터 경로 문자열** 검색.
**공개일·배포주체는 자기 신고로 인정하지 않는다** — HF API 의 **최초 커밋 타임스탬프**와 **모델카드 최초 리비전**을 독립 확인한다. 다운로드 수는 인위적으로 부풀릴 수 있으므로 단독 신호로 쓰지 않는다.
확인 불능이면 Warning 이 아니라 **"제출 학습 코드로 4일 예산 내 재현 의무"** 로 전환한다(V8 학습판). 재현 실패 → **Violation (R3-b·R5)**.

---

### C15. 실행 환경 주입에 의한 킷 우회 〔R4, R7-0〕

킷 해시가 온전한 채로 인터프리터 시작 시점에 동작을 바꾸는 경로.

```bash
rg -n "sitecustomize|usercustomize|PYTHONSTARTUP|PYTHONPATH|conda/etc/conda/activate\.d"
find . -name "sitecustomize.py" -o -name "*.pth" -o -name "conftest.py"
rg -n "torch\.(nn\.functional\.[a-z_]+|load)\s*=" ; rg -n "sys\.modules\["
# 킷 실행 스크립트가 설정하는 환경변수 전량 열람
```

| 상황 | 등급 |
|---|---|
| 킷 실행 시점에 로드되는 경로에 동작 변경 주입 | **Violation (R4)** — 최중대 |
| 주입 파일은 있으나 킷 실행 경로에서 미로드 | Dormant-Risk |

---

## 7. 실행·재현 검사 V1~V9 (부정행위와 독립 축)

> **검증 모드를 STEP 0 에 명시한다.** 동적 = 실행 환경 있음 / 정적 = 없음.
> **정적 모드에서 V1·V3 는 자동으로 `Insufficient-Evidence (미실행)`** 이며 **D4 를 제안하지 않는다.**

| ID | 검사 | 근거 | 판정 |
|---|---|---|---|
| **V1** | **엔드투엔드 콜드 재현** — 제출물만으로 격리 환경에서 raw `data/eval` → 최종 CSV 전 과정 1회 실행. 대상은 **팀이 최종 선택한 제출 파일 1개**의 Private Score | R8-c-① | 상대오차 ≤1e-3 Pass / ≤1e-2 Warning / 초과·완주 실패 **Violation** |
| **V2** | **단일** RTX PRO 6000, 학습 ≤4일 / 추론 ≤1시간 | R5·R8-c-④ | 초과 시 D4 |
| **V3** | 상대경로·UTF-8·확장자·무오류 실행 | R8-a | 실패 시 검증 탈락 |
| **V4** | 개발 환경(OS)·라이브러리 버전 기재 | R8-a | 실패 시 검증 탈락 |
| **V5** | 필수 제출물 3종 완비 | R8-b | 실패 시 검증 탈락 |
| **V6** | **킷 requirements 버전 일치 + 불일치 시 CSV 값 변동** | R4·R8 | 전량 일치 Pass / 불일치+상대오차 ≤1e-6 Warning / **초과 Violation (R4 제2문)** |
| **V7** | **순서·독립성 불변 검사** | R3-c | 216개 **셔플 실행** + **1샘플씩 별도 프로세스 실행**. 정상 실행 대비 **점수 상대차 >1e-2**, 또는 **프레임 평균절대차(정규화 [0,1]) >0.02 인 샘플이 5% 초과** → **Violation (R3-c)**. 그 이하는 **GPU 추론 비결정성**(cuDNN 비결정 커널·atomic 누적·TF32)으로 보고 Pass — 무임계 적용은 정직한 팀을 전량 오탐한다 |
| **V8** | **자산 재생성 검사** | R8·R5·R3-a | ① 학습 코드 생성 추적 불가한 바이너리 자산 삭제 후 실행 → 216개 중 1개라도 미생성 또는 **점수 상대 악화 >1e-2** → **Violation** (1e-3~1e-2 Warning). ② **"추적 가능"은 자기 신고로 인정하지 않는다** — 동봉된 생성 스크립트를 **신고된 입력으로 실제 실행**해 값이 일치할 때만 인정. 불일치 시 출처 허위 → **Violation (R3-a 또는 R8)** |
| **V9** | **변환 불변 검사** | R7-b-5 | 동일 프레임 배열을 **킷 README·베이스라인 기본 인코딩**으로 재인코딩 → 점수 상대차 **>1e-3 C6-b 소명 승격 / >1e-2 Violation 후보**. **프레임 배열 단계의 비학습 변환에도 동일 적용** |

**V6~V9 는 정적 분석으로 잡히지 않는 위장을 기계적으로 검출한다.** 우선순위: **V1(엔드투엔드) → V7 → V8 → C3-f → C9-e → C11 → C12**

> **개별 V 는 V1 안에서의 부분 교란 실험으로 수행한다** (특정 자산만 재생성 / 특정 단계만 셔플 / 특정 변환만 제거).
> V6~V9 를 서로 분리된 검사창으로 두면 **창 사이의 틈**으로 빠져나가는 경로가 생긴다.

> **전제조건**: C3-f·V1·V6~V9 는 **배포 `submission_kit/` 원본 실물** 확보를 전제한다 — **v3.1 시점 확보 완료**(부록 A·B).
> 검사 환경에 원본이 없으면 `Insufficient-Evidence (킷 원본 미확보)` 로 고정하며 **Pass 도 Violation 도 기록하지 않는다.** 비교 대상 없이 Pass 로 기록해서는 안 된다.

---

## 8. 오탐 방지

| 겉보기 | 실제 | 판정 |
|---|---|---|
| `from_pretrained()` 네트워크 접속 | 가중치 다운로드 | Pass (R1) — 단 C14 선언표 필수 |
| timm·torchvision 의 DINOv2·R3D 사용 | 킷 미포함 공개 가중치 | 기본 Pass — 단 **부록 A 판정 규칙** 적용 |
| 코드에 `eval` 다수 | `model.eval()` | Pass |
| train 홀드아웃 자체 검증 | eval 아님 | Pass — 단 **킷 코드로 채점했다면 C3** |
| `DataParallel` 존재 | 단일 GPU 폴백 분기 인용 가능하면 무해 | Pass |
| CSV 행 수만 assert, write 없음 | 값 불변 | Pass |
| 점수가 크게 좋음 | 성능은 증거 아님 (P5) | 단독 지적 불가 |
| 데이터 증강(회전·색상·프레임 드롭) | **학습 데이터**의 내부 변형 | Pass — 단 **추론 경로(생성→저장) 상의 변환은 증강이 아니라 C6-b 대상**이다 |
| 코드가 매우 복잡함 | 복잡도는 위반 아님 | Pass |
| 순수 ASCII `.py` 에 `charset=us-ascii` | ASCII 는 UTF-8 의 부분집합 | Pass — `file -i` 문자열 대조 금지 |

---

## 9. 출력 형식

### 9.0 기계 판독 블록 (필수, 보고서 최상단)

~~~json
{
  "spec_version": "rule_v2.md v2.0",
  "team_id": "", "track": "undergrad|grad", "private_rank": 0,
  "review_mode": "static|dynamic",
  "reviewed_at": "", "reviewer": "",
  "artifacts": [],
  "findings": [{
    "finding_id": "T03-C02-01", "check_id": "C2", "subtype": "L1", "rule_id": "R3-a",
    "grade": "Violation", "reachable": "true|false|unknown", "confidence": "high|medium|low",
    "evidence": [{"path":"src/select.py","line_start":120,"line_end":134,
                  "symbol":"select_best","sha256":""}],
    "impact_path": ["read data/eval/*","compute feat","argmin over candidates","final mp4"],
    "counter_hypothesis": "", "counter_hypothesis_refuted": true,
    "clarification_questions": [], "proposed_sanction": "D1"
  }],
  "checks": {"C1":"","C2":"","C3":"","C4":"","C5":"","C6":"","C7":"","C8":"","C9":"",
             "C10":"","C11":"","C12":"","C13":"","C14":"","C15":""},
  "repro": {"V1":"","V2":"","V3":"","V4":"","V5":"","V6":"","V7":"","V8":"","V9":""},
  "tally": {"Violation":0,"Warning":0,"Gray":0,"Dormant-Risk":0,
            "Insufficient-Evidence":0,"Pass":0},
  "unresolved_edges": 0,
  "recommendation": "keep|rehear_after_clarification|D1|D2|D3|D4",
  "recommendation_confidence": "high|medium|low"
}
~~~

**규칙**
- `grade`·`checks`·`repro` 값은 반드시 6종 enum: `Pass | Gray | Dormant-Risk | Warning | Violation | Insufficient-Evidence`. "통과/탈락" 같은 처분어 금지.
- 한 check_id 에 findings 가 여럿이면 §5 서열상 **가장 높은 등급**을 `checks` 에 기입. `Insufficient-Evidence` 는 다른 findings 가 없을 때만 항목 등급이 된다.
- **하나의 사실이 복수 항목에 해당하면 주관할 항목 1개에만 findings 를 만든다.** 나머지 항목의 `checks` 는 `"→C? 참조"`. `tally.Violation` 은 findings 배열 기준(칸 수 아님).
- `recommendation_confidence`: 모든 Violation 이 `reachable=="true"` + `counter_hypothesis_refuted==true` → high / 하나라도 unknown → medium / 정적 모드이며 V1·V3 가 Insufficient-Evidence → **최대 medium**.

### 9.1 사람용 보고서

**요약표** — C1~C15 / V1~V9 전 항목: 판정 + 근거 조문 + 핵심 증거(파일:라인). 누락 금지.
**상세 소견** — Pass 이외 **모든 등급**(Insufficient-Evidence 포함):

~~~
[C?] 항목명 — 판정: <6종 enum>
▸ 근거 조문 : R?-? "원문 인용"
▸ 증거      : path:line-line (함수명) + 코드 3~10줄
▸ 결과 영향 경로 : 코드 → ? → 최종 mp4 / CSV  (못 그리면 등급을 한 단계 낮춘다)
▸ 판단 이유 : 조문의 어느 요건을 어떻게 충족하는지
▸ 반대 가설 : 무해 해석 1개 이상 + 코드 인용으로 배제했는지
▸ 소명 요청 : 구체적 질문 1~3개 (기한 2일)
▸ 제안 처분 : D1~D4 (명문 처분 없으면 "포괄 조항 적용" 병기)
~~~

**종합 결론** — `Violation / Warning / Gray / Dormant-Risk / Insufficient-Evidence` **5개 카운터 전부** + 재현성 판정 + 최종 권고(`keep / rehear / D1~D4`) + 신뢰도 + **미해결 간선 n건**.

---

## 10. 수행 절차

```
STEP 0   : 제출물 인벤토리 + 검증 모드(정적/동적) 명시 + 누락 항목(V5)
STEP 0.5 : 팀 집합 전체 1회 — 함수 단위 정규화 후 토큰 3-gram 유사도 행렬 (담합 검사)
           비자명 블록 유사도 ≥0.85 이며 공개 베이스라인으로 설명 불가 → Violation (R9)
           0.6~0.85 → Warning + 양 팀 동시 소명 / 공개 베이스라인 유래 → Pass
STEP 1   : 킷 해시 대조 — 부록 B 기준 해시가 있을 때만. 결과는 "일치/불일치 목록"으로만
           기록하고 **등급은 STEP 8 에서 부여**한다
STEP 2   : 정적 스크리닝 — §6 명령 전량 실행(bash, globstar off, rg 14.1.0).
           **rg 는 exit=0·1 모두 정상 수행, exit≥2 만 "미수행"**. 아직 판정 금지
STEP 3   : 진입점 파악 — README·실행 스크립트에서 실제 실행 순서 복원
STEP 4   : 호출 그래프 — 표준 `ast` 로 import·호출 간선 추출.
           ★ importlib/__import__/getattr/exec/설정 주도/문자열 레지스트리 개입 지점은
             **"미해결 간선 목록"에 기록**하고 하류를 도달 불가로 취급하지 않는다
             (reachable="unknown"). 보고서에 미해결 간선 n건을 반드시 싣는다
STEP 5   : 데이터 흐름 — data/eval → ? 와 submission_kit → ? 두 갈래를 끝까지
STEP 6   : 위상 검사 — C4 역방향 화살표
STEP 7   : 예산 추정 — C9 (가정치 명기)
STEP 7.5 : 실행 검증 — 동적 모드면 V1·V3·V6~V9 수행. 정적 모드면 미수행을 명시
STEP 8   : 등급 부여 — §5.1 결정 순서 Q1~Q6
STEP 9   : 반대 가설 — 모든 Violation 에 무해 해석 1개 이상.
           ★ **코드 인용으로 배제할 수 있을 때만 "반박됨"**. "가능성이 낮다"는 반박이 아니다.
             배제 불가 시 서열에서 한 단계 강등. counter_hypothesis_refuted=false 인
             Violation 은 보고서에 남길 수 없다(Warning 으로 내린다)
STEP 10  : §9 형식으로 출력
```

---

## 11. 부록 A — 제출킷 사실관계 (배포본 직접 판독, 2026-08-25)

```
submission_kit/
  action_extractor.py        7,607 B    SO100ActionExtractor 정의 + ckpt 로더
  feature_csv_utils.py      16,306 B    전처리·3성분 특징 추출·CSV 기록 전부
  make_submission_csv.py     3,156 B    CLI 진입점
  checkpoints/action_extractor.ckpt   55,172,743 B   Lightning ckpt, 텐서 키 88개
  sample_submission.csv     27,794,395 B
  requirements.txt · README.md · input_videos/README.md
```

### A-1. 3성분이 실제로 무엇인가 (`write_feature_csv`)

| 성분 | 모델 | 전처리 | **CSV 에 기록되는 값** |
|---|---|---|---|
| Video Feature | `torchvision.r3d_18(R3D_18_Weights.DEFAULT)`, `fc=Identity` | trilinear → 112×112, KINETICS mean/std | **특징 벡터** (거리 아님) |
| DINO | `timm.create_model("vit_small_patch14_dinov2.lvd142m", pretrained=True, num_classes=0)` | resize+pad → 모델 기대 크기, IMAGENET mean/std | **프레임별 특징 벡터** (거리 아님) |
| Action | **킷 포함 `action_extractor.ckpt`** | `[-1,1]` 정규화, 320×512 | **`mean(|pred − target|)` 스칼라 = 그 샘플의 Action MAE 그 자체** |

> ⚠️ **결정적 사실**: `extract_action_features()` 는 eval 의 정답 action 을 읽어 **MAE 를 직접 계산해 CSV 에 쓴다.**
> 즉 **참가자는 킷을 1회 실행하는 것만으로 자기 Action 점수(배점 0.4)를 샘플 단위로 정확히 알 수 있다.**
> DINO·VideoFeature 는 정답 feature 가 없어 자기채점이 불가능하다 — **이 비대칭이 부정행위 유인을 지배한다.** → **C4-b**

### A-2. 킷이 읽는 것

```
--challenge-root      기본 ../data/eval          → images/*.png, actions/*.npy (정답 action)
--action-stats-path   기본 ../data/train/so100_action_statistics.json  → action mean/std 정규화
--action-extractor-ckpt 기본 checkpoints/action_extractor.ckpt
```
→ 킷이 eval 을 읽는 것은 **정상 동작**이다. 참가자 코드가 읽는 것과 혼동하지 말 것.

### A-3. `SO100ActionExtractor` 구조 (C3-f 판정용)

```
입력  (B, C=3, T=16, H=320, W=512), 값 범위 [-1, 1]
stem  Conv3d(3→32, k=(3,7,7), s=(1,2,2)) + GroupNorm + SiLU
encoder Residual3DBlock ×8  (채널 32→32→64→128→128, 공간만 다운샘플)
        → 공간 평균 → (B, T, C)
temporal nn.GRU(hidden=256, layers=1, bidirectional=True)
head  LayerNorm(512) → Linear(512→256) → SiLU → Dropout → Linear(256→6)
출력  (B, T=16, A=6)   ※ action 은 train mean/std 로 정규화된 공간
하이퍼파라미터 키: action_dims·base_channels·blocks_per_stage·dropout·in_channels·
                   learning_rate·mlp_hidden·temporal_hidden·temporal_layers
state_dict 텐서 키 88개 · 키집합 지문 sha256:
  8da3368604755b422721432ed24289b0def67f39306b0a94b57730379c5e676e
```

### A-4. 킷 미포함 가중치 — DINOv2 / R3D-18

킷 코드가 `timm` · `torchvision` 에서 **런타임에 내려받는다.** 킷 파일이 아니다.

| 상황 | 등급 |
|---|---|
| 일반적 표현 학습·백본으로 사용 | **Pass (R1)** |
| (i) 출력이 손실·후보 선택기·후처리에 직접 진입 **AND** (ii) §13 산식 컴포넌트와 **동일 거리척도·동일 특징 추출 지점**을 재현 | **Violation (R7-b-5)** |
| (i) 충족 + **(ii′) 레이어·척도는 다르나 팀 프록시 feature 와 산식 컴포넌트의 상관 ρ>0.9**(train 홀드아웃 측정) | **Violation (R7-b-5)** — 산식 표적 프록시 |
| (i)만 충족, (ii)·(ii′) 모두 못 보임 | Warning + 소명 |

**(ii) 판정에 쓸 킷 실측 규격** — 아래와 **동일 조합**이면 "동일 추출 지점" 성립:
- DINO: `vit_small_patch14_dinov2.lvd142m`, `num_classes=0`, **CLS 토큰**(`_normalize_image_model_output` 이 `output[:,0]`), resize+**zero pad**, IMAGENET mean/std
- Video: `r3d_18` DEFAULT + `fc=Identity`, **trilinear 112×112**, KINETICS mean/std

> "채점 산식을 겨냥했는가"를 인상으로 판단하지 않는다. **(ii)/(ii′) 대응표·상관값을 보고서에 첨부**한다.
> 중간층 feature + 학습된 프로젝션으로 "동일 지점 아님"을 만드는 회피가 가능하므로 (ii) 단독 기준은 불충분하다.
> 이 항목은 T1(R7-b-5) 사안이므로 **Gray 로 처리하지 않는다.** Gray 는 T4-only 전용(§5.1 Q2).

---

## 12. 부록 B — 킷 원본 기준 해시 ✅ **확정**

배포본 `submission_kit/` 실물에서 직접 산출 (sha256, 2026-08-25).

| 경로 | 크기(B) | sha256 |
|---|---|---|
| `README.md` | 1,013 | `1099d5c00c67c4286697aa336e65e918f76cc9d7ef0a23b409229e0d7202de52` |
| `action_extractor.py` | 7,607 | `d4f075380f830aaa4f44457605395a98a69f5ef55b924fa32325b11332eca756` |
| `checkpoints/action_extractor.ckpt` | 55,172,743 | `de7758b4f26ee9448c983d63f137aa5a024c86751edb779b050f54a36cfaa959` |
| `feature_csv_utils.py` | 16,306 | `dc89068beb9a529e54333fbce80609fed8f4137aad1b63e3dd3b22d53815a827` |
| `input_videos/README.md` | 191 | `a2bea42a53c3d0901dae93c1f6e431c9f29bf9ed4c68fb92e9a7dff7c77fbc76` |
| `make_submission_csv.py` | 3,156 | `e32ac6bd4076c21749a5c2360a56647deb075c1180c3ca977774c4dfc7dd10f7` |
| `requirements.txt` | 700 | `e32465179da7d60dcc5a54631226a3d8d830b65706ebd32a8bc8a27ba3104ed1` |
| `sample_submission.csv` | 27,794,395 | `746d7317970a768082a65d20468c7f9c2bb5e8e76d681fe04c4b7c1e3fc1ddef` |

```bash
# 대조 명령 (STEP 1)
find submission_kit -type f -print0 | sort -z | xargs -0 sha256sum
```

> **해시 불일치 = C5 Violation (R4)**. 크기만 같고 해시가 다른 경우도 동일하다.
> **주의**: 해시 일치는 "변조 없음"의 **필요조건일 뿐 충분조건이 아니다.** 파일을 그대로 두고
> **실행 인자**로 출력을 바꾸는 경로가 존재한다 → **C5-b**.

---

## 13. 부록 C — 산식과 조사 우선순위

```
Score = 0.3×DINO + 0.3×VideoFeature + 0.4×Action   (0에 가까울수록 우수)
DINO·VideoFeature = Cosine Distance / Action = MAE / Public 30% · Private 70% · 순위 = Private 100%
```

**유인 비대칭 (부록 A-1 실측)** — 이것이 조사 우선순위를 결정한다.

```
Action 0.4      → 킷이 CSV 에 MAE 스칼라를 직접 기록한다.  참가자 자기채점 = 완전 가능
DINO 0.3        → CSV 에 특징 벡터만. 정답 feature 없음.   자기채점 = 불가능
VideoFeature 0.3 → 동상.                                    자기채점 = 불가능
```
⇒ **로컬에서 정확히 최적화 가능한 것은 Action 0.4 뿐이다.** 부정행위는 여기로 몰린다.

| 배점 | 반칙 기대이득이 큰 지점 | 우선 항목 |
|---|---|---|
| **Action 0.4** | **킷 1회 실행으로 얻은 CSV 의 Action MAE 로 후보 선택·상수 튜닝** | **C4-b, C11, C12** ← 최우선 |
| Action 0.4 | 킷 extractor 를 손실·교사로 | **C3, C3-f, C3-g** |
| DINO 0.3 | eval 프레임 통계로 색·밝기 보정 | **C2 L2·L2′, V8-②, V9** |
| VideoFeature 0.3 | 킷 실행 결과 기반 best-of-K | **C4, C12, V7** |
| 전 항목 | 미제출 계산·자산 반입 | **C9-e, V8, C7-b** |
| 전 항목 | **킷 인자 변조**(해시 무결) | **C5-b** |

---

## 14. 개정 이력

### v3.0 → v3.1 (킷 원본 실물 판독 반영)

| 항목 | 내용 |
|---|---|
| **부록 B 확정** | 배포본 8개 파일 sha256 산출 완료 → C5·C3 해시 판정의 `Insufficient-Evidence` 고정 해제 |
| **부록 A 실측 교체** | 3성분의 모델·전처리·**CSV 기록값**을 코드에서 직접 판독. DINO/R3D 판정의 (ii) 기준을 실측 규격(CLS 토큰·zero pad·IMAGENET / trilinear 112·KINETICS)으로 확정. `SO100ActionExtractor` 구조·입력 shape·텐서 키 88개·키집합 지문 기록 |
| **C4-b 신설** | ⚠️ `extract_action_features()` 가 **eval 정답 action 을 읽어 MAE 스칼라를 CSV 에 직접 기록**한다. **킷 1회 실행 = 배점 0.4 의 샘플 단위 채점표 획득**이며, 재실행 루프가 없어 **C4 위상 검사로 원리적으로 잡히지 않는다** |
| **C5-b 신설** | `make_submission_csv.py` 는 CLI 인자를 받는다. **파일 해시를 100% 유지한 채 출력을 바꿀 수 있다** — 특히 `--action-extractor-ckpt` 교체는 R4 직접 회피. 해시 대조는 필요조건일 뿐임을 명문화 |
| **C3-f 구체화** | 실행 가능한 스니펫 + 입력 규격 `(B,3,16,320,512) ∈ [-1,1]` + torch 없이 가능한 1차 지문 대조 |
| **부록 C** | **유인 비대칭** 명문화 — 자기채점이 가능한 것은 Action 0.4 뿐이다 |

### v2.0 → v3.0 (외부 감사 `audit_report_rule_v2.md` 반영)

| 등급 | 반영 |
|---|---|
| **치명** | **F-01** §6 종료코드 규칙 역결선 수정 — ripgrep 은 **무매치가 exit=1** 이므로 v2 규칙대로면 **깨끗한 팀의 스크리닝이 전부 "미수행"** 이 된다. `exit≥2` 만 오류로 정정 / **F-02** C7 의 `-- '**/config.json'` 이 bash globstar off 에서 **exit=2 실패** → `--glob` 으로 교체 (외부데이터 포렌식 복구) / **F-03** 부록 B 공백 시 C3-f 가 "수행 가능"이라는 단정 철회 — **킷 원본 실물 미확보면 수행 불능**, 비교 대상 없이 Pass 기록 금지 |
| **중대** | **F-04** R8-c 를 **5종으로 복원** — 417168 원문 3번 *"학습 로그가 없는 경우 실제 학습한 Epoch 수 등 주요 학습 정보를 반드시 명시"* 가 v2 에서 누락됐다(4종 축약) / **F-05** R7-a~b 5개 사례를 **417076 원문 그대로** 교체 / **F-06** §5.1 적용 단위를 **finding** 으로 명시 + **Q0** 신설(P8-① 흡수 방지) / **F-07·F-08** V7·V8·V9 **수치 임계 부여** — 무임계 "프레임 불일치"는 GPU 비결정성으로 정직한 팀을 전량 오탐 / **F-09** P6 예외에 C7-b·부록 A(ii) 등재 / **F-10** `\.zip$` 앵커 → 따옴표·괄호·공백 허용(실제 코드 미탐 해소) / **F-11** **증거 인멸 역유인 제거** — 메타데이터 전량 결손 + R8-c-③ 미충족 시 보류가 아니라 **D2**, 예산 초과 정황 병존 시 D4 |
| **경미** | **F-12** C9 에 `t_load` 도입 / **F-13** C1 벤더명을 import 문맥으로 한정 / **F-14** R5 [학습]/[추론] 블록 복원("추론도 1대 기준") / **F-15** C12 를 216의 **약수·군집**까지 확장 / **F-16** §14 의 "rg 전량 실측" 주장 한정 + 재현 환경(bash·globstar off·rg 14.1.0) 명시 |
| **레드팀 D-1~D-8** | C3-g **3중 임계**(Pearson·Spearman·상호정보량) + 잔차 구조성 + 동봉 스크립트 실행 대조 / C2 L2′ **자산 재생성 절차**(→V8-②) / C11-c 임계 3개 + 군집 배정 함수 소명 / C9-e **부분 실행 latency 측정** / C7-b **백본 암기 검사** / C14 **HF API 독립 확인** / 부록 A **(ii′) 상관 ρ>0.9 프록시 기준** / C6-b **시점을 프레임 텐서 이후로 재정의** / **V1 을 엔드투엔드 콜드 재현으로 격상**, V6~V9 를 그 안의 부분 교란 실험으로 재정의(창 사이 틈 제거) |

### v1.0 → v2.0

| 구분 | 내용 |
|---|---|
| **원문 정정** | **R7-0 신설** — T1 규칙 7)의 *"및 이를 통해 추출한 정보"*·*"영상 선택·보정·수정"* 이 v1.0 에 없었다 / R4 제2문을 파생해석→**조문**으로 복원 / R2 제2문 후단·R3 강화어·R6 금지문언·R5 "단일"·R9 3항 복원 / 처분표의 명문 부재 표기 + R3-a·R3-b 분할 |
| **구조** | §5.1 등급 결정순서(MECE) · 처분코드 D1~D4 / §2.1 T4 선포섭검사 / P3 예외 E1~E3 · P5 차분 단서 · P8 이원화 / §9.0 기계판독 JSON · 미해결 간선 / STEP 0.5·7.5 · 정적/동적 모드 / **C9 산술 정정 216초→16.67초** |
| **신설** | C11~C15 / C3-f·C3-g·C6-b·C7-b·C9-e / V6~V9 |
