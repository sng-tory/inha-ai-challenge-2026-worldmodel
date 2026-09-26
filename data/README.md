# 데이터 형식

원본 데이터(`data/train` 8.6GB, `data/eval` 52MB)는 용량 문제로 이 저장소에 포함하지 않았습니다.
이 폴더에는 형식을 파악할 수 있는 최소한의 예시만 남겼습니다.

## 학습 데이터 `data/train/<owner>/<dataset>/`

LeRobot v2.1 형식의 SO-100 로봇팔 데이터셋 128개(촬영자 56명, 에피소드 11,132개, 프레임 약 102만 개)입니다.

```text
data/train/
  so100_action_statistics.json        # 6D action의 전체 mean/std (정규화용, 포함됨)
  <owner>/<dataset>/
    meta/info.json                    # fps, features, 경로 템플릿 (예시: example_meta_info.json)
    meta/episodes.jsonl               # episode_index, tasks(자연어), length (예시: example_meta_episodes.jsonl)
    data/chunk-000/episode_XXXXXX.parquet
                                      # 컬럼: action(6), observation.state(6), timestamp, frame_index, episode_index, index, task_index
    videos/chunk-000/<camera_key>/episode_XXXXXX.mp4
                                      # 640x480, 6 fps (원본 30fps를 stride 5로 다운샘플)
```

- `action` 6차원: `shoulder_pan, shoulder_lift, elbow_flex, wrist_flex, wrist_roll, gripper` (절대 관절값)
- 베이스라인 데이터로더(`baseline/challenge_kit/src/ldwma/datasets/lerobot_so100.py`)는
  `meta/info.json`에서 `action.shape == [6]`인 데이터셋만 자동 탐색하고, 이름에 `wrist/gripper/arm`이 들어간 카메라는 제외합니다.
- 학습 시 에피소드에서 연속 16프레임(`traj_len=16`)을 임의 시작점으로 잘라 `(video, act)` 쌍을 만듭니다.

## 평가 데이터 `data/eval/`

216개 샘플. 각 샘플은 첫 프레임 1장과 16스텝 액션 시퀀스로 구성됩니다.

```text
data/eval/
  images/sample_000000.png ... sample_000215.png     # 첫 프레임 (RGB PNG)
  actions/sample_000000.npy ... sample_000215.npy    # float32, shape (16, 6), 정규화 전 절대 관절값
```

- `sample_ids.txt`: 216개 sample_id 전체 목록
- `samples/`: 형식 확인용 예시 3개 (`sample_000000`, `sample_000100`, `sample_000215`)

## 모델 입출력 규격 (베이스라인/킷 공통)

| 항목 | 값 |
|---|---|
| 입력 영상 텐서 | `(B, 3, T=16, H=320, W=512)`, 값 범위 `[-1, 1]`, letterbox(pad) 리사이즈 |
| 입력 액션 텐서 | `(B, 16, 6)`, `so100_action_statistics.json`의 mean/std로 z-score 정규화 |
| 출력 영상 | `sample_XXXXXX.mp4`, 16프레임, 6 fps, libx264 |
| 제출 CSV | `sample_id, feature_component, feature_json` 3컬럼, 샘플당 3행(216 × 3 = 648행) |
