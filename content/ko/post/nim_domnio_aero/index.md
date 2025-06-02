---
title: "NIM For Aero"
date: 2025-06-01T10:00:00+09:00
draft: false
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





<details>
<summary>convert_stl.py</summary>

```python
#!/usr/bin/env python3
import os
import trimesh

INPUT_ROOT = 'drivAer_data_full'
OUTPUT_DIR = 'drivAer_single_solid_stls'
os.makedirs(OUTPUT_DIR, exist_ok=True)

for i in range(1, 101):
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

필요한 입력 파일들을 모두 받았으면 GPU 장비가 필요하다. 또 컨테이너 사용을 위햇 컨테이터 툴킷같은 것들이 필요한데 여기서는 NVIDIA에서 개발한 [enroot](https://github.com/NVIDIA/enroot), [pyxis](https://github.com/NVIDIA/pyxis) 조합을 사용하였다. 이 두 개의 패키지 설치에 대한 자세한 설명은 다른 곳을 참고하기 바란다. 
NIM 이미지를 사용하기위해서는 [NGC](https://catalog.ngc.nvidia.com)의 키가 필요하다. 키를 발급받고 enroot에서 import할 수 있다.

```
enroot import -o nim_domino.sqsh 'docker://$oauthtoken:[API_KEY]@nvcr.io#nim/nvidia/domino-automotive-aero:1.0.0' 
```

[API_KEY]란에 NGC에서 받은 토큰을 사용해야 한다.  

이제 아래 Slurm 스크립트로 NIM을 로컬 GPU 장비에서 띄울 수 있다. 

```
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

domino이미지는 도커의 ENTRYPOINT기능으로 NIM API 서버를 띄우게 되있어서 별도의 명령이 필요하지 않다. 그런데 srun은 항상 명령어가 필요하다. 그래서 `true`란 명령을 줘서 이미지의 entrypoint에서 실행하도록 하였다. 

이제 간단한 명령으로 접속이 되는지 확인할 수 있다.

```
curl -X 'GET' 'http://worker-1:8000/v1/health/ready' -H 'accept: application/json' 
```

이 출력결과가 다음과 같이 나오면 성공이다. 

```
ready
```


NIM for DoMINO-Automotive-Aero는 크게 두가지 예측을 해준다. 하나는 양력과 항력같은 공력이고 다른 하나는 압력, 속도등의 유동장이다. 본 글에서는 먼저 공력에 대해 비교를 하기로 한다. DoMINO 세트에는 실제 학습에 쓰인 CFD 데이터를 제공해주므로 이 값을 리얼 값이라고 생각하였다.

또 비교를 위해 U=38.89m/s를 적용하였고 rho는 1kg/m3를 사용하였다. 공력계수는 두 가지를 제공하는데 aRefRef값과 force_mom_constref의 파일을 적용하였다.

500개의 데이터를 한번에 추론하기위해 다음과 같은 post 파일을 만들었다.


<details>
<summary>post_process.py</summary>

```
import numpy as np
import pandas as pd

# --- ▒~D▒▒| ~U ---
URL         = "http://worker-1:8000/v1/infer"                # ▒~T▒|  API ▒~W~T▒~S~\▒~O▒▒~]▒▒~J▒
STREAM_VEL  = 38.89                                            # ▒~\| ▒~F~M (m/s)
RHO         = 1.0                                             # ▒~\| 체 ▒~@▒~O~D (kg/m^3), ▒~U~D▒~Z~T▒~K~\ ▒~H~X▒| ~U
STL_DIR     = "./drivAer_single_solid_stls"               # STL ▒~L~L▒~]▒ ▒~O▒▒~M~T
DATA_DIR    = "./drivAer_data_full"                         # CSV ▒~L~L▒~]▒▒~S▒▒~]▒ ▒~^~H▒~J~T ▒~\▒~C~A▒~\~D ▒~O▒▒~M~T
RUN_START   = 1                                               # ▒~K~\▒~^~Q run
RUN_END     = 100                                             # ▒~E▒~L run

results = []

for i in range(RUN_START, RUN_END+1):
    run_name = f"run_{i}"
    stl_path = os.path.join(STL_DIR, run_name, f"drivaer_{i}_single_solid.stl")

    if not os.path.exists(stl_path):
        print(f"[SKIP] STL not found for run_{i}: {stl_path}")
        continue

    # --- API ▒~X▒▒~\ ---
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

    # --- 결과 ▒~]▒기 ---
    with np.load(io.BytesIO(r.content)) as npz:
        output = {key: npz[key] for key in npz.files}

    # ▒~B▒ ▒~]▒▒~D▒~W~P ▒~T▒▒~]▒ ▒| ~A▒| ~H▒~^~H ▒~H~X▒| ~U▒~U~X▒~D▒▒~Z~T
    drag = output.get("drag_force")
    lift = output.get("lift_force")

    # --- 참조▒~R ▒~\▒~S~\ ---
    geo_csv   = os.path.join(DATA_DIR, run_name, f"geo_ref_{i}.csv")
    force_csv = os.path.join(DATA_DIR, run_name, f"force_mom_constref_{i}.csv")

    try:
        geo_df   = pd.read_csv(geo_csv)
        force_df = pd.read_csv(force_csv)
    except Exception as e:
        print(f"[SKIP] CSV not found for run_{i}: {e}")
        continue

    # aRef 칼▒~_▒▒~]~D 기▒~@ 면▒| ~A▒~\▒▒~\ ▒~B▒▒~Z▒
    aRef     = float(geo_df.loc[0, "aRefRef"])
    Cd_ref   = float(force_df.loc[0, "Cd"])
    Cl_ref   = float(force_df.loc[0, "Cl"])

    # --- ▒~D▒~H~X ▒~D▒~B▒ ---
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

# --- 결과 ▒~\▒| ▒ ---
if results:
    df = pd.DataFrame(results)
    out_csv = "summary_coefficients.csv"
    df.to_csv(out_csv, index=False)
    print(f"\nComplete! Results saved to {out_csv}")
else:
    print("No results to save.")
```

</details>


출력은 다음과 같이 출력 된다.
```

```


여기까지 길게 왔지만 내가 진짜 하고 싶은 것은 500개의 데이터를 한번에 분석하는 것이었다. 아직까지 ML for Surrogate 모델은 업계에서 많은 의문을 갖고 있는 것이 사실이다. 여기서 예시로 든 공력 같은 경우 내가 주로 했었던 에어포일을 기준으로 하면 양력, 항력계수는 비교적 잘 맞히지만 비선형성이 큰 모멘트 계수같은 경우는 정확도를 크게 벗어났었다. 
500개의 데이터를 scattrer plot으로 나타내었다. 

![alt text](Cd_scatter.png)
![alt text](Cl_scatter.png)
![alt text](Cd_error_hist.png)
![alt text](Cl_error_hist.png)






