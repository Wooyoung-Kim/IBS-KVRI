# IBS-KVRI
# scLENS: 단일세포 전사체 데이터의 구조 기반 임베딩 분석

이 문서는 [scLENS (Structural Cell Lineage Embedding via Noise Suppression)](https://github.com/Mathbiomed/scLENS)의 설치 및 분석 방법을 정리한 가이드입니다.\
Julia 언어 기반으로 개발된 `scLENS`는 노이즈를 억제하고 구조적 특징을 보존하는 단일세포 표현 학습을 제공합니다.

---

## 📦 설치 방법

### 1. Julia 설치

공식 웹사이트에서 Julia ≥ 1.6 버전을 다운로드합니다.

👉 [https://julialang.org/downloads/](https://julialang.org/downloads/)

#### 환경변수 설정 (권장)

```bash
# Julia 실행파일 경로
export PATH="$HOME/local/julia-1.11/bin:$PATH"
source ~/.bashrc
# Julia 패키지 디렉토리 별도 설정 (권한 문제 해결)
export JULIA_DEPOT_PATH="$HOME/julia_depot"
```

해당 내용은 `.bashrc` 또는 `.bash_profile`에 추가하고, `source ~/.bashrc`로 반영하세요.

---

### 2. scLENS 레포지트리 클론

```bash
git clone https://github.com/Mathbiomed/scLENS.git
cd scLENS # 작업을 원하는 디렉토리로 설정 가능
```

---

### 3. julia 실행

```julia
import Pkg
Pkg.activate(".")       # scLENS 폴더를 경로로 설정 (앞서 설정한 디렉토리가 scLENS일 경우 ".")
Pkg.instantiate()       # 의존성 패키지 설치, GPU가 없을 경우 CUDA로 인한 defendency 오류 발생 하지만 무시 가능
```

‼️ Optional: GPU를 사용하지 않는 경우 `CUDA` 제거 가능:

```julia
Pkg.rm("CUDA") 
Pkg.resolve()
```

---

## 📁 데이터 준비

scLENS는 다음 포맷을 지원합니다:

- `csv.gz` 형식의 count matrix (gene × cell)
- 10X 형식: `matrix.mtx.gz`, `features.tsv.gz`, `barcodes.tsv.gz`
- `.jld2` 형식의 사전 처리된 파일 (아래 함수로 생성 가능)

### 예: 10X 형식을 jld2로 변환

```julia
using scLENS
scLENS.tenx2jld2("경로/10x", "경로/저장파일명.jld2")
```

---

## 🧦 분석 실행

```julia
using scLENS

# GPU가 있다면 자동 감지
cur_dev = has_cuda() ? "gpu" : "cpu"

#jld 파일 생성
```julia
# Import necessary packages
using scLENS

# Example usage
scLENS.tenx2jld2("/path/to/10x/data", "output_data.jld2")

# jld2 파일 로드
pre_df = JLD2.load("output/sample.jld2", "data")

# 임베딩 생성
pre_df = scLENS.preprocess(ndf)

sclens_embedding = scLENS.sclens(pre_df, device_=cur_dev)
```

---

## 📊 시각화 및 결과 저장

```julia
# MP 분포 및 군집 안정성 확인
scLENS.plot_mpdist(sclens_embedding)
scLENS.plot_stability(sclens_embedding)

# UMAP 적용
scLENS.apply_umap!(sclens_embedding)
panel = scLENS.plot_embedding(sclens_embedding, pre_df.cell)

# Save the PCA and UMAP results to CSV files
using DataFrames
using CSV
CSV.write("out/pca.csv", sclens_embedding[:pca_n1])
CSV.write("out/umap.csv", DataFrame(sclens_embedding[:umap], :auto))

# Save the scLENS outcome as an AnnData file
scLENS.save_anndata("out/test_data.h5ad", sclens_embedding)

# Save the UMAP plot as an image
using CairoMakie
CairoMakie.activate!(type = "png")
save("out/umap_dist.png", panel_1)
```

---

## 📌 기타 팁

- `matrix.mtx`는 gene × cell 방향이 기본입니다.
- 여러 샘플을 병합하려면 `JoinLayers()` 사용 후 10X 파일로 저장하세요.
- 인터넷 없이 사용할 경우:

```bash
export JULIA_DEPOT_PATH=~/julia_depot
```

---

## 📚 참고 문서

- 공식 GitHub: [https://github.com/Mathbiomed/scLENS](https://github.com/Mathbiomed/scLENS)
- 논문: [출판 전일 경우 bioRxiv 리크]
- 예제 노트북: `example/` 폴더 참조

---

## 👤 작성자

작성자: @kwy7605\
환경: Julia 1.11, Miniconda, Python 3.9, JupyterLab\
GPU: 사용 가능 / 또는 CPU 전용 설정
