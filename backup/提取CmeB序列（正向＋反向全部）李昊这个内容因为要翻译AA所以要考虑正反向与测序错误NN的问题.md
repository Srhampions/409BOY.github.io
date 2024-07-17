提取CmeB序列（正向＋反向全部）

```bash
mkdir ncbi_exc_NN

cat ./ab_CmeB/CmeB/*.fa.txt > ncbi_exc_NN/res_sum.txt

cat ./ncbi_exc_NN/res_sum.txt | grep -v "#" | awk '{print $1,$6,$2,$3-1,$4,$5}' | sed 's#/#_#g' | cat > ncbi_exc_NN/id.txt

split -l 1 ncbi_exc_NN/id.txt -d ncbi_exc_NN/roll

for i in ` ls ncbi_exc_NN/roll* |sed 's#ncbi_exc_NN/##g' `
do
        str=$(cat ncbi_exc_NN/$i |awk '{print $1}' |sed 's/.fa//g')
        res=$(cat ncbi_exc_NN/$i  |awk '{print $2}')
        start=$(cat ncbi_exc_NN/$i | awk '{print $4}')
        strand=$(cat ncbi_exc_NN/$i | awk '{print $6}')
        mkdir -p ncbi_exc_NN/$str
        cat ncbi_exc_NN/$i |awk -v OFS="\t" '{print $3,$4,$5}' |cat >ncbi_exc_NN/${str}_${res}_${start}_id.txt
        cp ncbi_exc_NN/${str}_* ncbi_exc_NN/${str}/
        seqkit subseq --bed ncbi_exc_NN/${str}/${str}_${res}_${start}_id.txt ./206_fa/$str.fa -o ncbi_exc_NN/$str/${res}_${str}_${start}_${strand}.fa
done

rm ./ncbi_exc_NN/roll*
rm ./ncbi_exc_NN/*_id.txt

mkdir ncbi_exc_NN_all
find ./ncbi_exc_NN/ -name "*.fa" -exec cp {} ./ncbi_exc_NN_all/ \;
```

接下来对反向序列进行反向互补

```bash
for file in ./ncbi_exc_NN_all/*-*; do
    filename=$(basename "$file")
    seqkit seq -r -p "$file" | sed "s/>.*/>$filename/" > "$file.modified.fasta"
done

# 这里需要更改，菌株名中会存在"-"符号，会干扰反转这个步骤，出现重复项


rm ./ncbi_exc_NN_all/*-.fa

mkdir ./ncbi_exec_AA/CmeB_AA
mkdir ./ncbi_exec_AA/RECmeB_AA

for i in ./ncbi_exc_NN_all/CmeB*; do
    output_file="./ncbi_exec_AA/CmeB_AA/$(basename "$i" .fa).translated.fa"
    seqkit translate "$i" > "$output_file"
done

# 因为有些提取的序列中存在n缺失碱基，所以需要用for循环处理而不是seqkit translate 全部fa

for i in ./ncbi_exc_NN_all/RECmeB*; do
    output_file="./ncbi_exec_AA/RECmeB_AA/$(basename "$i" .fa).translated.fa"
    seqkit translate "$i" > "$output_file"
done

```

```bash
for i in *.fa ; do filename=$(basename "$i") ; sed -i "s/^>\(.*\)/>${filename}/g" "$i" ; done

# 为文件内容中>后的内容替换为文件名

diff wenjian.txt CmeB_NN.txt | grep '^<' > unique_NN.txt
diff wenjian.txt CmeB_AA.txt | grep '^<' > unique_AA.txt

```

您可以使用 `diff` 命令来比较两个文件，并筛选出它们之间不重复的内容。以下是使用 `diff` 命令的示例：

```
1
2
diff file1.txt file2.txt | grep '^<' > unique_in_file1.txt
diff file1.txt file2.txt | grep '^>' > unique_in_file2.txt
```

这里的 `diff file1.txt file2.txt` 会比较两个文件的内容，然后 `grep '^<'` 会筛选出只在 `file1.txt` 中出现而在 `file2.txt` 中没有出现的行，并将这些行输出到 `unique_in_file1.txt` 文件中。同理，`grep '^>'` 会筛选出只在 `file2.txt` 中出现而在 `file1.txt` 中没有出现的行，并将这些行输出到 `unique_in_file2.txt` 文件中。

这样您就可以得到两个文件中不重复的内容分别存储在 `unique_in_file1.txt` 和 `unique_in_file2.txt` 中了。

