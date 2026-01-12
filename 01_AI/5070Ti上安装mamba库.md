5070Ti上安装mamba库

```shell
conda create -n mamba python=3.11 -y
```

```shell
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu128
```

```shell
# 1) 先把打包工具和编译常用依赖补齐（你日志里甚至缺 numpy）
pip install -U pip setuptools wheel packaging ninja numpy

# 2) 在当前 conda 环境内安装 CUDA 12.8 工具链（提供 nvcc）
conda install -y -c nvidia/label/cuda-12.8.1 cuda-toolkit

# 3) 告诉构建脚本 CUDA 在哪儿（避免 bare_metal_version 未定义的路径）
conda env config vars set CUDA_HOME="$CONDA_PREFIX"
# 让上面的变量生效
conda deactivate && conda activate mamba

# 4) Blackwell/RTX 50 系列：明确让扩展为 sm_120 编译（很关键）
export TORCH_CUDA_ARCH_LIST="12.0"

# 5) 确认 nvcc、CUDA、PyTorch 的可见性
nvcc --version
python - <<'PY'
import torch, os
print("torch:", torch.__version__, "CUDA in torch:", torch.version.cuda, "CUDA avail:", torch.cuda.is_available())
print("CUDA_HOME:", os.environ.get("CUDA_HOME"))
PY

# 6) 先装 conv 依赖（可选但更稳）
pip install --no-build-isolation "causal-conv1d>=1.5.0"

# 7) 再装 mamba-ssm（避免隔离，确保能看到你当前的 torch/cu128）
pip install --no-build-isolation "mamba-ssm[causal-conv1d]==2.2.5"

# 8) 运行一个最小 GPU 自测
python - <<'PY'
import torch
from mamba_ssm import Mamba
x=torch.randn(2,64,16, device='cuda')
m=Mamba(d_model=16,d_state=16,d_conv=4,expand=2).cuda()
y=m(x)
print("OK", y.shape)
PY
```

