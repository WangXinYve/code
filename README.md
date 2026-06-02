  bedtools intersect \
  -a pansvscope.bed \
  -b survivor.bed \
  -f 0.01 \
  -v \
  > pansvscope_specific_0.01.bed

bedtools intersect -wa -wb -a DEL.bed -b software/MELTv2.2.2/duroc-te.bed -f 0.75 > te-class.txt

