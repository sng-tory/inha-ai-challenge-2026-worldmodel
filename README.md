# 2026 인하 인공지능 챌린지 — World Model 트랙

첫 프레임 1장과 로봇팔 관절 명령 16스텝으로 미래 16프레임 영상을 생성하는 대회입니다.
이 저장소는 대회의 **베이스라인 파이프라인, 제출 킷, 채점 킷, 데이터 입출력 규격**을 보존하고,
참가팀들이 문제를 어떤 방식으로 풀어갔는지를 요약합니다.

참가팀 제출물(모델 가중치, 학습 코드, 체크포인트)은 포함하지 않습니다.

---

## 1. 대회 개요

| 항목 | 내용 |
|---|---|
| 대회명 | 2026 인하 인공지능 챌린지 (World Model) |
| 주최 / 플랫폼 | 인하대학교 / DACON |
| 기간 | 2026-07-16 ~ 2026-08-21 (코드 제출 마감) |
| 트랙 | 대학원 트랙, 학부 트랙 |
| 과제 | 첫 프레임 이미지 + SO-100 로봇팔 6D 액션 시퀀스(16스텝) → 16프레임 미래 영상 생성 |
| 평가 샘플 | 216개 (Public 30% / Private 70%) |
| 자원 제한 | RTX PRO 6000 (96GB) 1대 기준, 학습 최대 4일, 추론 1시간 이내 |
| 데이터 제한 | 제공 데이터만 사용, 평가 데이터는 어떤 형태로도 학습에 사용 불가 |
| 모델 제한 | 가중치가 공개되고 라이선스가 허용된 사전학습 모델만 사용 가능, 원격 API 사용 불가 |
| 제출 킷 | CSV 변환 용도로만 사용 가능. 킷의 코드·모델·출력을 학습, 추론, 영상 선택에 활용 금지 |

### 태스크

```text
입력   sample_XXXXXX.png   첫 프레임 1장
       sample_XXXXXX.npy   액션 시퀀스, float32 (16, 6)  — SO-100 6관절 절대값
출력   sample_XXXXXX.mp4   16프레임 영상 (6 fps), 216편
제출   submission_kit/make_submission_csv.py 로 mp4 → feature CSV 변환 후 DACON 업로드
```

### 채점 (낮을수록 좋음)

```text
score = 0.3 × DINO + 0.3 × VideoFeature + 0.4 × Action
```

| 성분 | 추출 모델 | 비교 방식 |
|---|---|---|
| DINO | DINOv2 ViT-S/14 (`timm`), 프레임별 CLS 토큰 | 정답 영상 특징과의 cosine distance |
| Video Feature | R3D-18 (`torchvision`, Kinetics 사전학습) | 정답 영상 특징과의 cosine distance |
| Action | 킷 포함 `action_extractor.ckpt` (3D CNN + BiGRU 액션 회귀기) | 생성 영상에서 역추정한 액션과 정답 액션의 MAE 비율 |

Action 성분은 킷이 정답 액션과의 MAE를 직접 CSV에 기록하므로 참가자가 자기 점수를 알 수 있는 반면,
DINO와 Video Feature는 정답 특징이 비공개라 자기채점이 불가능합니다.

---

## 2. 저장소 구성

```text
.
├── baseline/            베이스라인 (DynamiCrafter 기반 action-conditioned latent video diffusion)
│   ├── baseline.ipynb          데이터 확인 → 설치 → backbone 다운로드 → 영상 생성 → CSV 생성
│   ├── challenge_kit/          학습·추론 코드, config, LeRobot 데이터로더
│   └── shared_libs/video_utils/
├── submission_kit/      mp4 → 제출 CSV 변환 킷 (배포 원본)
│   ├── make_submission_csv.py
│   ├── feature_csv_utils.py    전처리 + DINO/R3D/Action 특징 추출 + CSV 기록
│   ├── action_extractor.py     SO100ActionExtractor 정의
│   └── checkpoints/action_extractor.ckpt
├── dacon_score/         채점 스크립트와 정답 CSV
├── data/                데이터 형식 설명과 예시 (원본 미포함, data/README.md 참고)
└── docs/
    ├── rule_v3.md                       코드 검증 사양서 (규칙 원문 인용 포함)
    ├── grad_team_ideas.html / .pdf      대학원 트랙 팀별 접근법 정리
    └── undergrad_team_ideas.html / .pdf 학부 트랙 팀별 접근법 정리
```

용량 문제로 제외한 항목과 복구 방법:

| 제외 항목 | 크기 | 복구 방법 |
|---|---|---|
| `baseline/checkpoints/backbone.ckpt` | 10.4GB | `baseline.ipynb` 3번 셀이 HuggingFace `Doubiiu/DynamiCrafter_512`에서 자동 다운로드 |
| `data/train/` | 8.6GB | HuggingFace LeRobot 커뮤니티 SO-100 데이터셋 128개 (형식은 `data/README.md`) |
| `data/eval/` | 52MB | 대회 배포 `open.zip`의 `eval` 폴더 (sample_id 목록은 `data/eval/sample_ids.txt`) |

---

## 3. 베이스라인 파이프라인

### 모델

DynamiCrafter 512 (1.4B, image-to-video latent diffusion)의 UNet에 6차원 액션 조건을 추가한 구조입니다.
VAE, CLIP 인코더, image projection은 backbone에서 로드해 동결하고 UNet만 학습합니다. 텍스트 캡션은 사용하지 않습니다.

| 항목 | 값 |
|---|---|
| latent 크기 | 40 × 64 × 4ch (영상 320 × 512 기준) |
| 프레임 수 | 16 |
| parameterization | v-prediction, zero-SNR rescale, 1000 steps |
| 학습 | lr 1e-4, batch 1 × accum 2, fp16, 최대 100k step / 48h |
| 추론 | DDIM 50 step, guidance scale 1.0, `uniform_trailing`, eta 1.0 |

### 실행 순서

```bash
# 1. 설치
pip install -r baseline/requirements.txt
pip install --no-deps -e baseline/challenge_kit \
                      -e baseline/challenge_kit/libs/dynamicrafter \
                      -e baseline/shared_libs/video_utils

# 2. backbone 다운로드 → baseline/checkpoints/backbone.ckpt  (노트북 3번 셀)

# 3. (선택) 학습
cd baseline/challenge_kit
bash scripts/train.sh --config configs/train/inha_action_diffusion_11M.yaml \
                      --script scripts/train_diffusion.py

# 4. 평가 입력 216개에 대해 mp4 생성 → submission_kit/input_videos/
python scripts/inference/generate_baseline_videos.py \
  --checkpoint ../checkpoints/baseline_diffusion.ckpt \
  --challenge-root ../../data/eval \
  --prediction-root ../../submission_kit/input_videos

# 5. 제출 CSV 생성 → submission_kit/submission_features.csv
cd ../../submission_kit && python make_submission_csv.py
```

### 데이터 전처리

1. 영상: RGB uint8 → 종횡비 유지 리사이즈 + letterbox 패딩으로 320 × 512 → `[-1, 1]` → `(C, T, H, W)`
2. 액션: `(16, 6)` float32 → `so100_action_statistics.json`의 mean/std로 z-score 정규화
3. 추론 입력: 첫 프레임만 채우고 나머지 15프레임은 0인 영상 텐서 + 정규화 액션 + 빈 캡션 + fps 6

### 제출 킷 동작

1. `data/eval`의 이미지와 액션 교집합으로 216개 sample_id 확정. 누락 mp4가 있으면 에러
2. 각 mp4를 디코딩해 16프레임인지 확인하고 320 × 512로 통일
3. R3D-18 → 512D 특징 벡터 1개 / 샘플
4. DINOv2-S/14 → 384D 특징 × 16프레임 / 샘플
5. `action_extractor.ckpt`로 액션 예측 후 정답과의 MAE 스칼라 1개 / 샘플
6. `sample_id, feature_component, feature_json` 3컬럼 CSV로 기록 (648행)

### 채점

```bash
cd dacon_score/scoring_kit && pip install -r requirements.txt
python scripts/eval/score_feature_csv.py --submission-csv ../submissions/team_name/submission_features.csv
```

---

## 4. 참가팀들은 어떻게 접근했나

대학원 트랙 8팀, 학부 트랙 6팀의 제출 코드와 문서를 바탕으로 정리했습니다.
그림과 상세 제원은 `docs/grad_team_ideas.pdf`, `docs/undergrad_team_ideas.pdf`에 있습니다.

### 4.1 공통으로 마주친 문제

- **모델이 액션을 무시한다.** 액션을 다른 에피소드 것으로 바꿔도 오차가 거의 오르지 않고, 첫 프레임을 빼면 크게 오른다는 실측이 있었습니다. 모델이 "첫 프레임 복사"라는 쉬운 답에 안주한다는 뜻이고, 배점 0.4가 걸린 Action 성분이 여기서 결정됐습니다.
- **배경이 흔들리면 감점이다.** 점수의 60%가 프레임 특징의 cosine distance라 배경의 미세한 진동이 그대로 감점으로 이어졌습니다. 대학원 8팀 중 6팀이 움직이지 않아야 할 픽셀을 첫 프레임으로 되돌리는 후처리를 넣었고, 액션 이동량이 작은 샘플을 정지영상으로 대체한 팀도 있었습니다.
- **자원 제약이 구조를 정했다.** GPU 1장, 4일이라는 예산 때문에 대형 백본을 쓴 팀은 전부 백본을 동결하고 LoRA나 어댑터만 학습했습니다.

### 4.2 첫 갈림길: 백본 선택

| 백본 | 팀 수 | 전략 |
|---|---|---|
| DynamiCrafter 512 (대회 제공, 1.4B) | 5팀 | 예산을 학습 스텝에 투자. UNet을 직접 수정해 액션 경로 추가 |
| Wan 2.1 / 2.2 (TI2V-5B, VACE-14B) | 3팀 | 백본 동결 + LoRA + AdaLN 주입기. URDF 순기구학, 골격 렌더 등 기하 사전지식 주입 |
| Cosmos-Predict2.5-2B | 3팀 | 이미 action-conditioned인 모델에서 출발. 잠재 역동역학(IDM) 보조 손실, warped noise |
| Cosmos3-Nano | 2팀 | 대용량 LoRA + 액션 18채널 확장(현재값, 시작점 대비, 직전 대비) |
| SVD / Ctrl-World | 1팀 | 액션 대조 힌지 손실로 조건 무시를 직접 억제 |

### 4.3 두 번째 갈림길: 6D 액션을 어디로 넣는가

1. **정규화 변조 (AdaLN-Zero / AdaGN)** — `h = h·(1+γ) + β`. 마지막 층을 zero-init해 붙인 직후 출력이 원본과 동일합니다. 모델이 무시할 수단이 없어 대회 백본을 고친 팀들이 주로 선택했습니다.
2. **크로스 어텐션 / 어댑터** — 액션 토큰을 K·V로 주입하고 LoRA로 학습합니다. 대형 백본을 얼린 채 붙이기 좋아 외부 백본 팀이 선택했습니다.
3. **입력 채널 확장 / 구조 조건** — depth 채널이나 골격 렌더를 latent 옆에 붙입니다. 팔 위치를 픽셀 공간에서 직접 지정하지만 카메라 캘리브레이션이 필요합니다.
4. **텍스트 토큰 우회** — 액션 통계를 자연어 구절로 바꿔 CLIP 조건으로 투입합니다.

### 4.4 액션이 반영되도록 강제하는 처방

| 처방 | 개입 지점 | 내용 |
|---|---|---|
| 학습 태스크 교대 | 데이터·태스크 | 짝수 스텝은 액션을 주고 영상을 예측, 홀수 스텝은 액션까지 가리고 둘 다 예측 |
| 무시 불가능한 통로 | 모델 구조 | AdaLN으로 주입하고 움직이는 영역에 손실 가중치 부여 |
| 전용 통로 신설 | 모델 구조 | 액션 전용 cross-attention 부품 하나만 추가하고 나머지는 단계별로 동결 |
| 자체 심판 | 손실 | 영상에서 액션을 역추정하는 IDM을 별도 학습해 보조 손실로 쓰거나, 샘플링을 실제로 굴려 심판 점수를 직접 미분 |
| 추론 시 조건 증폭 | 추론 | 액션 CFG로 조건 방향을 강화. 반대로 CFG를 전부 기각한 팀도 있어 백본에 따라 효과가 갈림 |

### 4.5 그 외 눈에 띄는 아이디어

- 채점과 같은 특징 공간(DINOv2 dense 패치, R3D-18)을 지각 손실로 사용
- URDF 순기구학으로 관절각을 3D 팔 자세로 변환해 조건으로 투입. 학습 대상과 예산이 가장 가벼웠음
- 1-pass 잠재 예측기로 초안을 만들고 확산은 img2img로 정련만 하는 2단 구조
- 광류로 노이즈를 미리 뒤틀어(warped noise) 시간 일관성 확보
- 체크포인트 수프(마지막 여러 개 평균), 시드 여러 개로 생성 후 메도이드 선택, 학습 노이즈 분포를 추론 스텝 격자에 맞춤
- 사람 손 검출과 정지·흔들림 기준으로 학습 데이터를 걸러내는 품질 게이트

### 4.6 규정 해석에서 반복된 쟁점

`docs/rule_v3.md`는 상위 팀 코드 검증에 사용한 검사 사양서입니다. 검증 과정에서 반복적으로 문제가 된 지점은 다음과 같습니다.

- 킷이 출력하는 Action MAE를 보고 체크포인트나 후보 영상을 고르는 행위
- 평가 샘플을 통계 모집단에 포함하는 transductive 전처리
- 채점 산식과 동일한 특징 추출 지점을 보조 손실로 사용하는 것의 허용 범위
- 주최가 제공하지 않은 외부 자료(URDF 로봇 규격 등) 사용의 허용 범위
- 학습 로그 유실, 선택 도구 미제출 등 재현성 결함
- 추론 시간 측정에 모델 로딩과 저장 시간이 포함되었는지

---

## 5. 참고

- 베이스라인 backbone: [DynamiCrafter](https://github.com/Doubiiu/DynamiCrafter) (Apache-2.0)
- 데이터 형식: [LeRobot](https://github.com/huggingface/lerobot) v2.1 dataset format, SO-100 로봇팔
- 채점 특징 추출기: `torchvision` R3D-18, `timm` `vit_small_patch14_dinov2.lvd142m`
