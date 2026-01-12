[Installation - nerfstudio](https://docs.nerf.studio/quickstart/installation.html)

```
conda create --name nerfstudio -y python=3.8
conda activate nerfstudio
python -m pip install --upgrade pip
```

```
pip install torch==2.1.2+cu118 torchvision==0.16.2+cu118 --extra-index-url https://download.pytorch.org/whl/cu118
```

```
conda install -c "nvidia/label/cuda-11.8.0" cuda-toolkit
```

```
conda install -c conda-forge gcc_linux-64=11 gxx_linux-64=11 -y
```

```
conda install -y -c conda-forge libxcrypt ninja
```

***

```
sudo find /usr -name "libcuda.so*" 2>/dev/null
```

查询结果为：

```
(nerfstudio) diobrando@R9000P:~$ sudo find /usr -name "libcuda.so*" 2>/dev/null
[sudo] password for diobrando: 
/usr/lib/wsl/drivers/nvlti.inf_amd64_26c7d450745b4e73/libcuda.so.1.1
/usr/lib/wsl/lib/libcuda.so
/usr/lib/wsl/lib/libcuda.so.1
/usr/lib/wsl/lib/libcuda.so.1.1
```

***

TCNN_CUDA_ARCHITECTURES后的数字与显卡架构有关，40系用89（官网查询）

```
export TCNN_CUDA_ARCHITECTURES=89
export LIBRARY_PATH="/usr/lib/wsl/lib:$LIBRARY_PATH"
export LDFLAGS="-L/usr/lib/wsl/lib -L/usr/local/cuda-11.8/lib64/stubs $LDFLAGS"
```

```
pip install git+https://github.com/NVlabs/tiny-cuda-nn/#subdirectory=bindings/torch
```

***

```
pip install nerfstudio
```

