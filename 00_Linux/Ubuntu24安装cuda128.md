```bash
source /etc/os-release
distro="ubuntu${VERSION_ID/./}"
arch="x86_64"

wget "https://developer.download.nvidia.com/compute/cuda/repos/${distro}/${arch}/cuda-keyring_1.1-1_all.deb"
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt update
```

```bash
sudo apt install cuda-toolkit-12-8
ls -l /usr/local/cuda-12.8/bin/nvcc
```

```bash
echo 'export PATH=/usr/local/cuda-12.8/bin${PATH:+:${PATH}}' >> ~/.bashrc
source ~/.bashrc
command -v nvcc
nvcc --version
```

