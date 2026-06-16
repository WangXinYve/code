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





HiFisvcall

##Align PacBio HiFi reads to a reference genome(ARS-UCD1.2), the recommended aligner is pbmm2.
Reference_Fa=$1
HiFi_Fa=$2
Sample_prefix=$3
pbmm2 align ${Reference_Fa} ${HiFi_Fa} ${Sample_prefix}.sort.bam --sort --preset HiFi --sample ${Sample_prefix} --rg "@RG\tID:${Sample_prefix}"

##Using pbsv to call structural variants
#Discover signatures of structural variation
pbsv discover ${Sample_prefix}.sort.bam ${Sample_prefix}.svsig.gz
#Call structural variants and assign genotypes
pbsv call --ccs -m 50 ${Reference_Fa} ${Sample_prefix}.svsig.gz ${Sample_prefix}.vcf

##Using cuteSV to call structural variants
Sort_BAM=$1
Reference_Fa=$2
Sample_Prefix=$3
Threads=$4
cuteSV ${Sort_BAM} ${Reference_Fa} ${Sample_Prefix}.vcf ./ \
	--max_cluster_bias_INS 1000 \
	--diff_ratio_merging_INS 0.9 \
	--max_cluster_bias_DEL	1000 \
	--diff_ratio_merging_DEL 0.5 \
	--genotype --sample ${Sample_Prefix} --threads ${Threads} --min_mapq 20 --min_size 30 --max_size 100000

##Using SVIM to call structural variants
BAM=$1
Reference=$2
SampleName=`basename ${BAM} .sort.bam`
svim alignment ./ ${BAM} ${Reference} --min_mapq 20 --min_sv_size 30 --max_sv_size 100000 --minimum_depth 3 --sample ${SampleName} --sequence_alleles --insertion_sequences

##Using Sniffles to call structural variants
Ref=$1
BAM=$2
prefix=$3
#Add MD tag to BAM:
samtools calmd -bS ${BAM} ${Ref} > ${prefix}.add_md.sort.bam
#SV calling
sniffles -t $(nproc) --input ${sam}.add_md.sort.bam --vcf $sam.sniffles.vcf.gz --snf $sam.sniffles.snf 

