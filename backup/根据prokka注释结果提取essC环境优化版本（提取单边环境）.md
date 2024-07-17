> 运行目录在：/mnt/sdb1/HAU/wa1/work/shi_shengrui/All_lianqiu/

```bash
#！ bin/bash

cd ./prokka/anno

# 为xls前面添加菌名

mkdir xiugai
for file in *.xls; do     awk -v filename="$file" '{print filename "\t" $0}' "$file" > "./xiugai/$file"; done

# 筛选essC条目信息
cd ./xiugai
 cat *.xls | grep "essC" | grep "CDS" | cat > all.txt
 sed -i 's/.xls//g' all.txt
 cp ./all.txt ../../..
 cd ../../..
 
# 开始提取序列
 mkdir -p ./essC_tiqu

while read line; do
    num=$(echo $line | awk '{print $1}')
    contig=$(echo $line | awk '{print $2}')
    start=$(echo $line | awk '{print $3}')
    end=$(echo $line | awk '{print $4}')
    strand=$(echo $line | awk '{print $5}')

    # 先判断$strand是 + 还是 -
    if [ "$strand" = "+" ]; then
       seqkit subseq -r "$start:$((end+100000))" --chr $contig --gtf ./prokka/gbk/${num}.gbk ./prokka/$num/*.fna -o ./essC_tiqu/${num}_${start}_${strand}.fa
    else
        # 如果是 - ，再判断$start是否 < 10K
        if [ "$start" -lt "100000" ]; then
                seqkit subseq -r "1:$end" --chr $contig --gtf ./prokka/gbk/${num}.gbk ./prokka/$num/*.fna -o ./essC_tiqu/${num}_${start}_${strand}.fa
        else
            seqkit subseq -r "$((start-100000)):$end" --chr $contig --gtf ./prokka/gbk/${num}.gbk ./prokka/$num/*.fna -o ./essC_tiqu/${num}_${start}_${strand}.fa
        fi
    fi

done < all.txt

# 最后删除seqkit生成的fai文件
rm ./*.fai

到此essC及其身后的环境及提取完成
 



```

