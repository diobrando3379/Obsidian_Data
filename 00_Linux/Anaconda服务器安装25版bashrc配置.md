.bashrc

```shell
export PATH=/home/LVM_date2/zjnu/anaconda3/bin:$PATH
# >>> conda initialize >>>
__conda_setup="$('/home/LVM_date2/zjnu/anaconda3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
    eval "$__conda_setup"
else
    if [ -f "/home/LVM_date2/zjnu/anaconda3/etc/profile.d/conda.sh" ]; then
        . "/home/LVM_date2/zjnu/anaconda3/etc/profile.d/conda.sh"
    else
        export PATH="/home/LVM_date2/zjnu/anaconda3/bin:$PATH"
    fi
fi
unset __conda_setup
# <<< conda initialize <<<

# 更新为新配置，禁用自动激活 base 环境
conda config --set auto_activate true
# conda activate qianlf

```

.condarc

```shell
# auto_activate_base: true
auto_activate: true
```

