---
title: "NIM For Aero"
date: 2025-06-01T10:00:00+09:00
draft: true
featured: false
authors:
  - admin
tags: 
 - NIM
 - drivAer
 - DoMINO
categories:
 - HPC

---

최근 CFD(Computational Fluid Dynamics) 분야에서는 머신러닝을 접목한 연구가 활발히 진행되고 있다. 이런 연구 중 NVIDIA에서 오픈소스로 공개한 [PhysicNeMo](https://github.com/NVIDIA/physicsnemo) 프레임워크는 Physics ML 기법을 이용해 물리 기반 AI 모델을 빠르게 학습 배포할 수 있게 해준다. 그 중 [DoMINO](https://docs.nvidia.com/deeplearning/physicsnemo/physicsnemo-core/examples/cfd/external_aerodynamics/domino/readme.html) 예제는 DrivAer 형상을 대상으로 국소 스케일, 포인트 클라우드 기반 신경 연산자를 학습하는 외부 차량 공력 모델의 워크플로우를 제공한다. 추가로 NVIDIA는 [NIM for DoMINO-Automotive-Aero](https://docs.nvidia.com/nim/physicsnemo/domino-automotive-aero/latest/overview.html)도 공개하여, 수십 TB 데이터와 대규모 GPU 없이도 자체 추론 파이프라인을 구축할 수 있다.

이 글에서는 NIM 서비스를 사용하고 분석한 결과를 공유하려고 한다. [홈페이지](https://docs.nvidia.com/nim/physicsnemo/domino-automotive-aero/latest/overview.html)에서 공개한 기본 예제는 형상 1개를 추론하는 경우를 보여주는데 여기서는 공개된 데이터세트를 한 번에 추론하고, 예측한 공력이 실제 CFD 값과 얼마나 일치하는 분석한 기록이다.

## 1. 준비 사항
이 글의 실험을 시작하려면 GPU가 있는 리눅스 서버가 필요하다. 저자는 A100 80G GPU로 테스트 하였지만 20G이하의 메모리가 달린 GPU로도 가능할 것으로 보인다. 스케쥴러는 없어도 되지만 저자는 Slurm이 설치된 환경에서 테스트 하였다. 컨테이너를 띄우기 위해 [enroot](https://github.com/NVIDIA/enroot)와 [pyxis](https://github.com/NVIDIA/pyxis)를 설치해야 한다. 또 [NGC](https://catalog.ngc.nvidia.com)와 [허깅페이스](https://huggingface.co)에 접속할 수 있는 API 토큰이 있어야 한다.



## 2. 데이터 다운로드
[DrivAerML](https://caemldatasets.org/drivaerml/)은 500가지 모핑 변형을 포함한 고해상도 공력 데이터셋으로, [Hugging Face](https://huggingface.co/datasets/neashton/drivaerml)와 AWS에서 공개 중이다. 기본 예제는 AWS의 S3에서 다운로드는 하는 방법이 있는데 여기서는 허깅페이스에서 직접 내려받는 방법을 사용한다. 허깅페이스 계정과 토큰이 필요하지만, 가입 및 발급 과정은 생략한다. 

먼저 Python에서 Hugging Face API를 다루기 위해 `huggingface_hub[cli]` 패키지를 설치한다.

```
pip install huggingface_hub[cli]
```

데이터셋의 전체 용량은 수십 테라바이트에 이르기 때문에 추론에 꼭 필요한 파일만 선택적으로 받을 수 있게 했다. 실제로 추론에 필요한 건 `boundary.vtp` 파일 하나뿐이고, 예측 결과를 검증하려면 추가로 공력 계수 `csv` 파일들과 이미지 폴더가 필요하다.

이를 위해 `download_hf.py` 스크립트를 만들었다. 이 스크립트는 `--download-type` 옵션으로 받을 파일 종류를 지정할 수 있게 되어 있다. 예를 들어 다음과 같이 쓸 수 있다:


- 형상 파일만 받을 경우:
```
python download_hf.py --download-type boundary
```

- 형상 + 공력 계수 + 이미지까지 전부 받을 경우 (기본값):
```
python download_hf.py
```

옵션에는 csv, boundary, images, all이 있으며, 원하는 조합을 선택할 수 있다. 각 파일은 run_1부터 run_500까지 순차적으로 다운로드된다. 

<details>
<summary>download_hf.py</summary>

```python
import argparse
from huggingface_hub import HfApi, hf_hub_download
import os

# Configuration
REPO_ID    = "neashton/drivaerml"
REPO_TYPE  = "dataset"
HF_TOKEN   = ""
LOCAL_DIR  = "./drivAer_data_full"
RUN_RANGE  = range(1, 501)
CSV_PREFIXES = [
    "force_mom_",
    "force_mom_constref_",
    "geo_parameters_",
    "geo_ref_",
]

def download_csv(run_dir, index):
    for prefix in CSV_PREFIXES:
        filename = f"{run_dir}/{prefix}{index}.csv"
        try:
            print(f"[CSV] {filename}")
            hf_hub_download(
                repo_id   = REPO_ID,
                repo_type = REPO_TYPE,
                filename  = filename,
                local_dir = LOCAL_DIR,
                token     = HF_TOKEN,
            )
        except Exception:
            print(f"    skip CSV: {filename}")

def download_boundary(run_dir, index):
    filename = f"{run_dir}/boundary_{index}.vtp"
    try:
        print(f"[VTP] {filename}")
        hf_hub_download(
            repo_id   = REPO_ID,
            repo_type = REPO_TYPE,
            filename  = filename,
            local_dir = LOCAL_DIR,
            token     = HF_TOKEN,
        )
    except Exception:
        print(f"    skip VTP: {filename}")

def download_images(run_dir):
    api = HfApi(token=HF_TOKEN)
    try:
        tree = api.list_repo_tree(
            repo_id   = REPO_ID,
            repo_type = REPO_TYPE,
            path_in_repo = f"{run_dir}/images",
        )
    except Exception as e:
        print(f"    skip images listing: {e}")
        return

    for entry in tree:
        path = entry.path if hasattr(entry, "path") else entry["path"]
        try:
            print(f"[IMG] {path}")
            hf_hub_download(
                repo_id   = REPO_ID,
                repo_type = REPO_TYPE,
                filename  = path,
                local_dir = LOCAL_DIR,
                token     = HF_TOKEN,
            )
        except Exception:
            print(f"    skip IMG: {path}")

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--download-type",
        choices=["csv", "boundary", "images", "all"],
        default="all",
    )
    args = parser.parse_args()
    do_csv      = args.download_type in ("csv", "all")
    do_boundary = args.download_type in ("boundary", "all")
    do_images   = args.download_type in ("images", "all")

    os.makedirs(LOCAL_DIR, exist_ok=True)

    for i in RUN_RANGE:
        run_dir = f"run_{i}"
        print(f"\nProcessing {run_dir}")

        if do_csv:
            download_csv(run_dir, i)

        if do_boundary:
            download_boundary(run_dir, i)

        if do_images:
            download_images(run_dir)

    print("\nAll done.")

if __name__ == "__main__":
    main()
```

</details>

## 3. STL 변환
다운로드 받은 STL 파일은 바로 사용할 수 없고, 간단한 전처리가 필요하다. 기존에는 하나의 STL 파일만 변환할 수 있었지만, 지금은 여러 파일을 한꺼번에 처리할 수 있도록 스크립트를 수정했다.

```
python convert_stl.py
```

<details>
<summary>convert_stl.py</summary>

```python
#!/usr/bin/env python3
import os
import trimesh

INPUT_ROOT = 'drivAer_data_full'
OUTPUT_DIR = 'drivAer_single_solid_stls'
os.makedirs(OUTPUT_DIR, exist_ok=True)

for i in range(1, 501):
    run_dir = f"run_{i}"
    input_path = os.path.join(INPUT_ROOT, run_dir, f"drivaer_{i}.stl")
    output_path = os.path.join(OUTPUT_DIR, run_dir, f"drivaer_{i}_single_solid.stl")
    os.makedirs(os.path.dirname(output_path), exist_ok=True)
    if os.path.isfile(output_path):
        print(f"[SKIP] Already processed: {output_path}")
        continue

    if not os.path.isfile(input_path):
        print(f"[SKIP] Input STL not found: {input_path}")
        continue

    mesh = trimesh.load_mesh(input_path)
    if isinstance(mesh, trimesh.Scene):
        mesh = trimesh.util.concatenate(list(mesh.geometry.values()))

    mesh.export(output_path)
    print(f"[OK] Saved: {output_path}")
```

</details>

이 스크립트를 실행하면 drivAer_data_full 안에 있는 STL 파일들을 모두 읽어서, 각 파일을 하나의 solid로 병합한 후 drivAer_single_solid_stls 디렉토리에 저장한다.
이제 이 변환된 STL 파일을 그대로 추론에 사용할 수 있다.


## 4. NIM 실행 환경 구성

필요한 STL 파일과 CSV 파일을 모두 준비했다면, 이제 NIM을 실행할 환경을 구성해야 한다. 추론에는 GPU 장비가 필요하고, 컨테이너 기반으로 NIM을 실행해야 하므로 관련 도구도 설치되어 있어야 한다.

여기서는 NVIDIA에서 제공하는 [enroot](https://github.com/NVIDIA/enroot)와 [pyxis](https://github.com/NVIDIA/pyxis)를 조합해서 컨테이너를 실행했다. 설치 방법은 공식 문서를 참고하면 된다.

NIM 이미지를 받기 위해서는 [NGC (NVIDIA GPU Cloud)](https://catalog.ngc.nvidia.com)에서 API 키를 발급받아야 한다. 키를 발급받은 후 아래 명령어를 사용해 enroot 형식으로 이미지를 불러올 수 있다.

```bash
enroot import -o nim_domino.sqsh 'docker://$oauthtoken:[API_KEY]@nvcr.io#nim/nvidia/domino-automotive-aero:1.0.0'
```

위 명령에서 `[API_KEY]` 자리에 발급받은 API 키를 입력하면 된다.

이미지를 준비한 후에는 아래처럼 Slurm 배치 스크립트를 작성해 NIM 서버를 실행할 수 있다.

```bash
#!/bin/bash
#SBATCH --job-name=nim_infer
#SBATCH --output=nim_%j.log
#SBATCH --gres=gpu:1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#SBATCH --time=01:00:00
#SBATCH --partition=main

export NGC_API_KEY="#######"

IMAGE=/app/enroot/nim_domino.sqsh

declare -a SLURM_ARGS=(
    --container-image $IMAGE
    --container-entrypoint
    --container-env NGC_API_KEY
    --container-mounts=/root/2.domino_nim
)

srun -l "${SLURM_ARGS[@]}" true
```

여기서 사용하는 NIM 이미지는 도커의 ENTRYPOINT 기능으로 API 서버가 자동으로 실행되도록 되어 있기 때문에 별도의 실행 명령이 필요 없다. 다만 `srun`은 명령어를 필수로 요구하므로 `true`를 전달해 컨테이너 내부의 ENTRYPOINT가 실행되도록 했다.

서버가 정상적으로 뜨면 아래 명령으로 상태를 확인할 수 있다.

```bash
curl -X 'GET' 'http://[gpu-node01]:8000/v1/health/ready' -H 'accept: application/json'
```

여기서 `[gpu-node01]`대신 실제 Slurm 잡이 실행된 노드를 적어야 한다. 응답으로 `ready`라는 문자열이 출력되면 정상적으로 기동된 것이다.


## 5. 공력 계수 추론 및 결과 분석

NIM for DoMINO-Automotive-Aero는 크게 두 가지 예측을 수행한다. 하나는 양력과 항력 같은 공력 계수 예측이고, 다른 하나는 압력과 속도 분포 등 유동장 예측이다. 이 글에서는 먼저 공력 계수에 집중해서, 예측값이 실제 CFD 결과와 얼마나 일치하는지를 비교했다. DoMINO에서 제공하는 데이터셋에는 실제 학습에 사용된 CFD 결과도 포함되어 있어 이를 기준값으로 삼았다.

추론 조건으로는 유입 속도 𝑈=38.89m/s
밀도 𝜌=1.0kg/m3를 적용했고, 공력 계수 계산에는 geo_ref.csv에 있는 𝑎ref 값과 force_mom_constref.csv의 참조 계수(Cd, Cl)를 사용했다. 500개의 데이터를 한꺼번에 추론하기 위해 `post_process.py`라는 스크립트를 작성했고, 이 스크립트는 각 STL 파일을 API 서버에 전송해 예측된 힘 값을 받아온 뒤, 이를 공력 계수로 환산하고 CSV 기준값과 비교해 정리하는 작업을 수행한다.

<details>
<summary>post_process.py</summary>
import os
import io
import httpx
import numpy as np
import pandas as pd

# --- Configuration ---
URL         = "http://worker-1:8000/v1/infer"     # NIM API endpoint
STREAM_VEL  = 38.89                               # Free-stream velocity (m/s)
RHO         = 1.0                                 # Fluid density (kg/m^3)
STL_DIR     = "./drivAer_single_solid_stls"       # Directory containing converted STL files
DATA_DIR    = "./drivAer_data_full"               # Directory containing CSV reference data
RUN_START   = 1                                   # Starting run index
RUN_END     = 100                                 # Ending run index

results = []

for i in range(RUN_START, RUN_END+1):
    run_name = f"run_{i}"
    stl_path = os.path.join(STL_DIR, run_name, f"drivaer_{i}_single_solid.stl")

    if not os.path.exists(stl_path):
        print(f"[SKIP] STL not found for run_{i}: {stl_path}")
        continue

    # --- Send STL to API ---
    files = {"design_stl": (os.path.basename(stl_path), open(stl_path, "rb"))}
    data = {
        "stream_velocity": str(STREAM_VEL),
        "stencil_size":    "1",
        "point_cloud_size": "500000",
    }
    try:
        r = httpx.post(URL, files=files, data=data, timeout=120.0)
        r.raise_for_status()
    except Exception as e:
        print(f"[ERROR] API failed for run_{i}: {e}")
        continue

    # --- Read inference results ---
    with np.load(io.BytesIO(r.content)) as npz:
        output = {key: npz[key] for key in npz.files}

    drag = output.get("drag_force")
    lift = output.get("lift_force")

    # --- Load reference data ---
    geo_csv   = os.path.join(DATA_DIR, run_name, f"geo_ref_{i}.csv")
    force_csv = os.path.join(DATA_DIR, run_name, f"force_mom_constref_{i}.csv")

    try:
        geo_df   = pd.read_csv(geo_csv)
        force_df = pd.read_csv(force_csv)
    except Exception as e:
        print(f"[SKIP] CSV not found for run_{i}: {e}")
        continue

    # Extract reference area and coefficients
    aRef     = float(geo_df.loc[0, "aRefRef"])
    Cd_ref   = float(force_df.loc[0, "Cd"])
    Cl_ref   = float(force_df.loc[0, "Cl"])

    # --- Compute predicted coefficients ---
    q_inf   = 0.5 * RHO * STREAM_VEL**2
    Cd_pred = drag / (q_inf * aRef)
    Cl_pred = lift / (q_inf * aRef)

    results.append({
        "run":      i,
        "Cd_pred":  Cd_pred,
        "Cd_ref":   Cd_ref,
        "Cl_pred":  Cl_pred,
        "Cl_ref":   Cl_ref,
    })
    print(f"[OK] run_{i}: Cd_pred={Cd_pred:.4f}, Cd_ref={Cd_ref:.4f}, Cl_pred={Cl_pred:.4f}, Cl_ref={Cl_ref:.4f}")

# --- Save results to CSV ---
if results:
    df = pd.DataFrame(results)
    out_csv = "summary_coefficients.csv"
    df.to_csv(out_csv, index=False)
    print(f"\nComplete! Results saved to {out_csv}")
else:
    print("No results to save.")
</details>


예측값과 참값의 산점도:


예측 오차 분포 (히스토그램):





결과적으로, 양력(Cl)과 항력(Cd)은 대체로 잘 맞았지만, 이전에도 자주 경험했던 것처럼 모멘트 계수와 같은 비선형성이 큰 항목은 오차가 클 가능성이 높다. 아직까지 ML 기반 Surrogate 모델에 대해 업계의 신뢰는 완전하지 않지만, 이런 대규모 실험을 통해 점차 검증해 나갈 수 있을 것으로 기대된다.