# bedtools intersect \
  -a pansvscope.bed \
  -b survivor.bed \
  -f 0.01 \
  -v \
  > pansvscope_specific_0.01.bed
