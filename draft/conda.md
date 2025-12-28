# 命令

`conda info --envs` ：列出有哪些环境

`conda update conda` ：更新conda

创建虚拟环境：

 `conda create --name myenv python=3.8`

`conda create --name myenv python=3.8 numpy pandas` 还可以加上一些需要的包

激活虚拟环境：

```bash
conda activate myenv
```

删除虚拟环境：

```bash
 conda remove --name myenv --all
```

安装 Jupyter Notebook：

```bash
 conda install jupyter notebook
```


![](Pasted%20image%2020250111164019.png)

为了确保所有conda环境都可见，可以安装nb_conda_kernels：

```bash
conda install nb_conda_kernels
```