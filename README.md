  bedtools intersect \
  -a pansvscope.bed \
  -b survivor.bed \
  -f 0.01 \
  -v \
  > pansvscope_specific_0.01.bed

bedtools intersect -wa -wb -a DEL.bed -b software/MELTv2.2.2/duroc-te.bed -f 0.75 > te-class.txt

# 1. 删除可能损坏的环境
conda deactivate
conda remove -n svcall --all -y

# 2. 清理缓存
conda clean -a
mamba clean -a

# 3. 重新设置频道优先级（使用 strict 模式）
conda config --set channel_priority strict
conda config --add channels conda-forge
conda config --add channels bioconda
conda config --add channels defaults

# 4. 用 mamba 重新创建环境
mamba create -n svcall python=3.9 -y

# 5. 激活环境
conda activate svcall

# 6. 仅从 conda-forge 和 bioconda 安装（避免混合源）
mamba install -c conda-forge -c bioconda pbmm2 pbsv cutesv svim sniffles samtools -y
