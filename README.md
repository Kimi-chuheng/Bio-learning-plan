# 详细学习计划：生物信息学、蛋白质序列分析与Linux技能培养

本学习计划专为Linux环境设计，强调通过命令行和脚本处理生物信息学数据，旨在同时提升生物信息学分析能力和Linux系统操作技能。

## 第一阶段：Linux基础与FASTA文件处理（预计 2 周）

### 第1周：Linux与命令行基础

#### 天1-2：Linux系统基础
- 学习资源：
  - [Linux Command Line Basics](https://ubuntu.com/tutorials/command-line-for-beginners)
  - [The Linux Command Line](http://linuxcommand.org/tlcl.php) (免费电子书)
- 实践任务：
  - 安装Linux系统（Ubuntu/CentOS）或使用WSL/虚拟机
  - 熟悉基本命令：`ls`, `cd`, `pwd`, `mkdir`, `cp`, `mv`, `rm`
  - 文件权限管理：`chmod`, `chown`

#### 天3-4：文本处理命令
- 学习重点：
  - 文本查看：`cat`, `less`, `head`, `tail`
  - 文本搜索：`grep`, `find`
  - 文本处理：`cut`, `sort`, `uniq`
- 练习：
  - 下载示例FASTA文件：`wget http://example.org/sample.fasta`
  - 使用`grep`提取序列标题：`grep ">" sample.fasta`
  - 使用`wc`计算序列数量：`grep -c ">" sample.fasta`

#### 天5-7：高级文本处理与管道
- 学习重点：
  - 流编辑器：`sed`, `awk`
  - 管道与重定向：`|`, `>`, `>>`
  - 环境变量与配置文件：`.bashrc`, `.bash_profile`
- 练习：提取FASTA序列长度
```bash
cat sample.fasta | grep -v ">" | awk '{ total += length } END { print total }'
```

### 第2周：Bash脚本与FASTA分析

#### 天8-9：Bash脚本基础
- 学习重点：
  - 脚本创建与执行权限
  - 变量声明与使用
  - 条件判断和循环
- 练习：创建第一个FASTA分析脚本
```bash
#!/bin/bash
# fasta_info.sh - 一个简单的FASTA文件分析脚本

if [ $# -eq 0 ]; then
    echo "用法: $0 <fasta文件>"
    exit 1
fi

echo "分析文件: $1"
echo "序列数量: $(grep -c ">" $1)"
echo "总碱基数: $(grep -v ">" $1 | tr -d '\n' | wc -c)"
```

#### 天10-11：FASTA序列提取与操作
- 开发Bash脚本实现：
  - 提取特定序列
  - 序列反向互补
  - 序列切片
- 示例脚本：
```bash
#!/bin/bash
# extract_sequence.sh - 根据ID提取FASTA序列

fasta_file=$1
seq_id=$2
in_target=0

while IFS= read -r line; do
    if [[ $line == ">"* ]]; then
        if [[ $line == *"$seq_id"* ]]; then
            in_target=1
            echo "$line"
        else
            in_target=0
        fi
    elif [[ $in_target -eq 1 ]]; then
        echo "$line"
    fi
done < "$fasta_file"
```

#### 天12-14：GC含量分析与批处理
- 编写完整的FASTA分析套件：
  - GC含量计算
  - 批处理多个文件
  - 结果汇总
- 使用Linux定时任务：
  - 学习`cron`命令设置定时任务
  - 设置定期运行分析脚本

## 第二阶段：Linux环境下的生物信息学工具链（预计 3 周）

### 第3周：工具安装与环境配置

#### 天15-16：软件包管理与环境变量
- 学习重点：
  - 软件包管理：`apt`, `yum`
  - 源码编译：`./configure`, `make`, `make install`
  - 环境变量配置：`PATH`, `LD_LIBRARY_PATH`
- 实践任务：
  - 安装编译工具：`sudo apt install build-essential`
  - 安装Python：`sudo apt install python3 python3-pip`
  - 安装Biopython：`pip3 install biopython`

#### 天17-19：BLAST安装与使用
- 安装BLAST：
```bash
wget ftp://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/LATEST/ncbi-blast-*+-x64-linux.tar.gz
tar -xzvf ncbi-blast-*+-x64-linux.tar.gz
export PATH=$PATH:$PWD/ncbi-blast-*+/bin
```
- BLAST命令行使用：
  - 创建BLAST数据库：`makeblastdb`
  - 执行序列比对：`blastp`, `blastn`
- 练习：编写脚本自动执行BLAST并解析结果
```bash
#!/bin/bash
# run_blast.sh - 运行BLAST并提取前10个结果

query=$1
db=$2
out="${query%.*}_results.txt"

blastp -query $query -db $db -outfmt "6 qseqid sseqid pident length evalue" -max_target_seqs 10 > $out
echo "BLAST完成，结果保存在 $out"
```

#### 天20-21：HMMER安装与配置
- 安装HMMER：
```bash
wget http://eddylab.org/software/hmmer/hmmer.tar.gz
tar -xzvf hmmer.tar.gz
cd hmmer-*
./configure
make
make check
sudo make install
```
- 实践任务：
  - 下载Pfam数据库：`wget http://ftp.ebi.ac.uk/pub/databases/Pfam/current_release/Pfam-A.hmm.gz`
  - 准备搜索数据库：`hmmpress Pfam-A.hmm`

### 第4周：Linux下的序列比对与分析

#### 天22-24：HH-suite配置与使用
- 安装HH-suite：
```bash
sudo apt install cmake
git clone https://github.com/soedinglab/hh-suite.git
mkdir -p hh-suite/build && cd hh-suite/build
cmake -DCMAKE_INSTALL_PREFIX=~/hh-suite -DCMAKE_CXX_COMPILER=g++ ..
make && make install
export PATH="$HOME/hh-suite/bin:$PATH"
```
- 下载UniRef30数据库并配置
- 使用HHblits生成多序列比对：
```bash
hhblits -i query.fasta -d /path/to/uniclust30 -oa3m query.a3m -n 3
```

#### 天25-26：Linux下的多序列比对工具
- 安装多种比对工具：
```bash
sudo apt install muscle clustalw mafft
```
- 编写脚本比较不同工具的结果：
```bash
#!/bin/bash
# compare_alignments.sh - 使用不同工具进行多序列比对

input=$1
base=$(basename "$input" .fasta)

# 运行不同的比对工具
muscle -in $input -out ${base}_muscle.aln
clustalw -infile=$input -outfile=${base}_clustalw.aln -output=clustal
mafft $input > ${base}_mafft.aln

# 计算比对结果的差异
echo "比对长度对比："
for aln in ${base}_*.aln; do
    echo -n "$aln: "
    grep -v "^>" $aln | tr -d ' \n' | wc -c
done
```

#### 天27-28：自动化流程与Shell处理
- 创建完整工作流：
  - 序列数据获取
  - 序列比对
  - 结果处理与分析
- 使用`screen`或`tmux`管理长时间运行的任务
- 使用`parallel`进行并行任务处理

### 第5周：机器学习与数据可视化

#### 天29-31：Linux下的Python与R环境
- 配置科学计算环境：
```bash
pip3 install numpy pandas matplotlib scikit-learn tensorflow
```
- 安装R和Bioconductor：
```bash
sudo apt install r-base
R -e "install.packages('BiocManager')"
R -e "BiocManager::install('Biostrings')"
```
- 在Linux服务器上配置Jupyter Notebook：
```bash
pip3 install jupyter
jupyter notebook --ip=0.0.0.0 --no-browser
```

#### 天32-35：数据分析与可视化脚本
- 编写Python脚本分析蛋白质序列特征
- 使用Shell脚本调用Python和R：
```bash
#!/bin/bash
# analyze_protein.sh - 分析蛋白质特征并生成可视化结果

fasta=$1
output_dir="${fasta%.*}_analysis"

mkdir -p $output_dir

# 运行Python分析
python3 analyze_protein_features.py $fasta $output_dir/features.csv

# 使用R绘图
Rscript plot_protein_features.R $output_dir/features.csv $output_dir/protein_features.pdf

echo "分析完成，结果保存在 $output_dir/"
```

## 第三阶段：CASP14单链蛋白质目标分析与Linux高级技能（预计 3 周）

### 第6周：Linux环境下的CASP数据处理

#### 天36-37：数据获取与组织
- 使用wget下载CASP14数据：
```bash
mkdir -p casp14/targets
cd casp14/targets
wget -r -np -nd https://predictioncenter.org/casp14/target.cgi?target=*\&view=sequence
```
- 编写脚本处理和整理CASP序列：
```bash
#!/bin/bash
# process_casp_targets.sh - 处理CASP目标序列

mkdir -p processed

for file in target.cgi*; do
  target=$(grep -o "T[0-9]\{4\}" $file | head -1)
  if [ ! -z "$target" ]; then
    echo "处理目标 $target"
    grep -A1 "SEQUENCE" $file | tail -1 > processed/${target}.fasta
    sed -i "1i >$target" processed/${target}.fasta
  fi
done
```

#### 天38-40：构建分析流程
- 使用Makefile自动化分析流程：
```makefile
# Makefile for CASP target analysis

TARGETS := $(wildcard processed/*.fasta)
MSA_FILES := $(TARGETS:.fasta=.a3m)
REPORTS := $(TARGETS:.fasta=_report.txt)

all: $(MSA_FILES) $(REPORTS)

%.a3m: %.fasta
	hhblits -i $< -d /path/to/uniclust30 -oa3m $@ -n 3

%_report.txt: %.a3m %.fasta
	./analyze_alignment.sh $^ > $@

clean:
	rm -f $(MSA_FILES) $(REPORTS)
```

#### 天41-42：并行计算与作业调度
- 学习使用GNU Parallel进行并行任务：
```bash
find processed -name "*.fasta" | parallel -j 4 './run_analysis.sh {}'
```
- 配置作业调度系统（如有集群环境）：
  - SLURM命令：`sbatch`, `squeue`, `scancel`
  - 编写作业提交脚本

### 第7周：高级序列分析与Linux系统管理

#### 天43-44：HHblits高级应用
- 编写HHblits批处理脚本：
```bash
#!/bin/bash
# batch_hhblits.sh - 批量运行HHblits并监控资源使用

log_dir="logs"
mkdir -p $log_dir

for fasta in processed/*.fasta; do
    target=$(basename $fasta .fasta)
    echo "运行 $target 的HHblits..."
    
    # 后台运行并记录时间和资源
    /usr/bin/time -v hhblits -i $fasta -d /path/to/uniclust30 -oa3m processed/${target}.a3m -n 3 \
    > $log_dir/${target}.log 2>&1 &
    
    # 限制并行数量
    while [ $(jobs -p | wc -l) -ge 4 ]; do
        sleep 10
    done
done

wait
echo "所有HHblits任务完成"
```

#### 天45-47：系统资源监控与优化
- 学习系统监控命令：
  - `top`, `htop`, `ps`, `free`, `df`
  - `iostat`, `vmstat`, `sar`
- 编写资源监控脚本：
```bash
#!/bin/bash
# monitor_resources.sh - 监控分析任务的资源使用

interval=60  # 秒
log_file="resource_usage.log"

echo "时间戳,CPU(%),内存使用(MB),磁盘IO(KB/s)" > $log_file

while true; do
    timestamp=$(date +"%Y-%m-%d %H:%M:%S")
    cpu=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}')
    mem=$(free -m | grep Mem | awk '{print $3}')
    io=$(iostat -d -k 1 1 | tail -n 2 | head -n 1 | awk '{print $3}')
    
    echo "$timestamp,$cpu,$mem,$io" >> $log_file
    sleep $interval
done
```

#### 天48-49：比对质量评估与过滤
- 开发MSA质量评估脚本：
```bash
#!/bin/bash
# assess_msa.sh - 评估多序列比对质量

a3m_file=$1
output="${a3m_file%.*}_stats.txt"

# 计算序列数量
seq_count=$(grep -c "^>" $a3m_file)

# 计算每个位置的氨基酸多样性
python3 - <<EOF > $output
import sys
from collections import Counter

with open("$a3m_file") as f:
    lines = f.readlines()

seqs = []
for line in lines:
    if not line.startswith(">"):
        seqs.append(line.strip())

positions = zip(*seqs)
diversities = []

for i, pos in enumerate(positions):
    counter = Counter(pos)
    diversity = len([aa for aa, count in counter.items() if aa != '-'])
    diversities.append(diversity)

print(f"序列数量: {len(seqs)}")
print(f"比对长度: {len(diversities)}")
print(f"平均多样性: {sum(diversities)/len(diversities):.2f}")
print(f"保守位点数: {len([d for d in diversities if d == 1])}")
EOF

echo "MSA质量评估完成，结果保存在 $output"
```

### 第8周：结果分析与Linux下的报告生成

#### 天50-51：数据可视化与报告
- 使用Python生成可视化：
```bash
#!/bin/bash
# visualize_results.sh - 生成MSA分析可视化

msa_dir="processed"
out_dir="reports"
mkdir -p $out_dir

# 运行Python脚本生成可视化
python3 <<EOF
import os
import matplotlib.pyplot as plt
import numpy as np
import seaborn as sns

msa_dir = "$msa_dir"
out_dir = "$out_dir"

# 收集所有MSA的统计数据
targets = []
seq_counts = []
conservations = []

for stats_file in [f for f in os.listdir(msa_dir) if f.endswith("_stats.txt")]:
    target = stats_file.split("_stats")[0]
    targets.append(target)
    
    with open(os.path.join(msa_dir, stats_file)) as f:
        lines = f.readlines()
        seq_counts.append(int(lines[0].split(": ")[1]))
        conservations.append(float(lines[3].split(": ")[1]))

# 绘制序列数量和保守性的关系
plt.figure(figsize=(10, 6))
sns.scatterplot(x=seq_counts, y=conservations)
for i, target in enumerate(targets):
    plt.annotate(target, (seq_counts[i], conservations[i]))
    
plt.xlabel("序列数量")
plt.ylabel("保守位点比例")
plt.title("CASP14目标蛋白的序列多样性分析")
plt.tight_layout()
plt.savefig(os.path.join(out_dir, "conservation_vs_seqcount.png"))
EOF

echo "可视化报告已生成到 $out_dir 目录"
```

#### 天52-54：LaTeX报告生成
- 安装LaTeX：`sudo apt install texlive-full`
- 编写脚本生成PDF报告：
```bash
#!/bin/bash
# generate_report.sh - 生成LaTeX格式的分析报告

report_dir="final_report"
mkdir -p $report_dir

# 创建LaTeX文件
cat > $report_dir/casp_analysis.tex <<EOF
\documentclass{article}
\usepackage{graphicx}
\usepackage{booktabs}
\usepackage{caption}
\usepackage{hyperref}
\title{CASP14 蛋白质目标序列分析报告}
\author{你的名字}
\date{\today}

\begin{document}
\maketitle

\section{介绍}
本报告分析了CASP14比赛中的单链蛋白目标序列，使用HHblits构建了多序列比对，并评估了序列保守性和结构预测可能性。

\section{方法}
使用了以下工具和数据库：
\begin{itemize}
    \item HHblits (版本 3.3.0)
    \item UniRef30 数据库（2021年版）
    \item 自定义脚本进行分析和可视化
\end{itemize}

\section{结果}
\begin{figure}[h]
    \centering
    \includegraphics[width=0.8\textwidth]{../reports/conservation_vs_seqcount.png}
    \caption{CASP14目标蛋白的序列多样性分析}
\end{figure}

\section{讨论}
通过分析发现，序列数量与保守性之间存在一定的相关关系...（此处根据实际结果撰写）

\section{结论}
本分析提供了对CASP14目标蛋白的深入理解，可以为蛋白质结构预测提供参考...

\end{document}
EOF

# 编译LaTeX文件
cd $report_dir
pdflatex casp_analysis.tex
pdflatex casp_analysis.tex  # 再次运行以生成完整引用

echo "报告已生成: $report_dir/casp_analysis.pdf"
```

#### 天55-56：演示准备
- 编写Shell脚本生成演示数据：
```bash
#!/bin/bash
# prepare_presentation.sh - 准备演示所需的数据

pres_dir="presentation"
mkdir -p $pres_dir

# 复制所需的图表和数据
cp reports/*.png $pres_dir/
cp processed/*_stats.txt $pres_dir/

# 创建摘要文件
cat > $pres_dir/summary.txt <<EOF
CASP14蛋白质目标分析摘要
=======================

分析的目标数量: $(ls processed/*.fasta | wc -l)
平均序列数量: $(grep "序列数量" processed/*_stats.txt | awk '{sum+=$NF} END {print sum/NR}')
平均保守位点比例: $(grep "保守位点数" processed/*_stats.txt | awk -F': ' '{conserved+=$2} {total+=$(NF-2)} END {print conserved/total}')

最具挑战性的目标: $(grep "平均多样性" processed/*_stats.txt | sort -k3 -nr | head -1 | awk -F'/' '{print $1}' | awk -F'processed/' '{print $2}' | awk -F'_stats' '{print $1}')
最保守的目标: $(grep "平均多样性" processed/*_stats.txt | sort -k3 -n | head -1 | awk -F'/' '{print $1}' | awk -F'processed/' '{print $2}' | awk -F'_stats' '{print $1}')

详细结果请参阅最终报告。
EOF

echo "演示材料已准备到 $pres_dir 目录"
```

## 额外资源

### Linux生物信息学书籍与教程
1. 《Bioinformatics Data Skills》by Vince Buffalo - 专注于Linux和命令行生物信息学
2. 《Linux Command Line and Shell Scripting Bible》by Richard Blum
3. [Bioinformatics Command Line Tools](https://link.springer.com/book/10.1007/978-1-4419-7744-3)

### 在线课程
1. [Command Line Tools for Genomics](https://www.coursera.org/learn/genomic-tools)
2. [Linux for Biologists](https://github.com/doxeylab/learn-genomics-in-linux)
3. [Software Carpentry: Unix Shell](https://swcarpentry.github.io/shell-novice/)

### 社区与论坛
1. [Biostars](https://www.biostars.org/) - 生物信息学问答网站，包含大量Linux相关问题
2. [SEQanswers](http://seqanswers.com/) - 专注于序列分析的论坛
3. [OMICtools](https://omictools.com/) - 生物信息学工具数据库

### 有用的GitHub仓库
1. [Bioinfo-Tools](https://github.com/jsh58/Bioinfo-Tools) - 生物信息学分析脚本集
2. [Bioinformatics-Workbook](https://github.com/ISUgenomics/bioinformatics-workbook) - 生物信息学工作流程教程
3. [Linux-command](https://github.com/jaywcjlove/linux-command) - Linux命令手册
