# PhastCons_Doc



PhastCons 是一个用于在多序列比对中识别进化保守元件的程序，基于称为系统发育隐马尔可夫模型（phylo-HMM）的序列进化统计模型。如果您熟悉广受欢迎的 VISTA 程序（Mayor 等人，2000），该程序通过沿序列长度绘制两条比对序列的百分比相似性来可视化，那么可以将 PhastCons 看作是下一代的 VISTA。PhastCons 与 VISTA 的区别包括：PhastCons 处理的是 n 个物种而非两个；它考虑了这些物种之间的系统发育关系；并且它不再简单地用百分比相似性来衡量相似性或差异性，而是使用核苷酸替代的统计模型，允许一个位点发生多次替代，并且不同碱基对之间的替代速率可以不相等（例如，转换的频率高于颠换）。
PhastCons 是 UCSC 基因组浏览器中日益流行的保守性轨迹背后的分析引擎。
PhastCons 和 PHAST 软件包的其他部分均使用 C 语言编写。PHAST 软件包已经在各种 UNIX 和 Linux 平台以及 Mac OS X 上编译并安装。将该软件移植到其他平台应该相对容易。


### 2.1 使用概述 

PhastCons 主要执行以下三项功能：

1. 生成逐碱基的保守性评分（如 UCSC 浏览器中的保守性轨迹所显示）。

2. 预测离散的保守元件（如浏览器中的“最保守”轨迹所显示）。

3. 估计自由参数。（在本节中，我们假设逐碱基的评分为保守性评分，离散元件为保守元件，尽管它们可能有其他解释；详见第 5 节。）

#### 输入要求 

PhastCons 的输入包括以下内容：

- 多序列比对文件

- 保守区域的系统发育模型

- 非保守区域的系统发育模型
保守区域的模型是可选的。如果未提供该模型，则通过将非保守区域的模型按比例因子 rho 缩放来间接定义（详见 `--rho` 选项）。程序的基本原理是沿比对扫描，寻找更能被保守模型“解释”的区域而非非保守模型的区域；这些区域将作为保守元件输出，每个碱基属于这些区域的概率将作为该碱基的保守性评分输出。

#### 状态转移概率 

程序还需要指定 HMM 的状态转移概率。这些概率决定了模型在不同状态间切换的自由度，从而定义了重要特性，如：

- 保守元件的期望长度

- 保守元件的期望覆盖度

- 保守性图的“平滑性”
有关详情可参阅 PhastCons 的论文。状态转移概率可以通过 `--transitions` 选项直接设置，或使用 `--expected-length` 和 `--target-coverage` 选项设置（详见第 3 节）。此外，它们也可以留空，程序将自动通过最大似然估计来计算这些参数。设置这些参数以获得良好的结果可能较为棘手。我们暂时跳过这个复杂的问题，先专注于程序的基本用法。假设通过调节参数设置了状态转移概率，例如使用 `--target-coverage 0.25` 和 `--expected-length 12`。我们将在第 3 节中详细讨论这一问题。
#### 基本运行命令 

程序可以使用以下形式的命令运行：


```css
phastCons [OPTIONS] alignment [cons.mod,]noncons.mod > scores.wig
```
 
- **alignment**  是一个多序列比对文件，需使用适当的格式（详见 `--msa-format` 选项）。
 
- **cons.mod**  和 **noncons.mod**  分别是保守区域和非保守区域的系统发育模型文件，格式为 PHAST 的 `.mod` 格式。
 
- **scores.wig**  是保守性评分文件，格式为 UCSC 基因组浏览器支持的 fixed-step WIG 格式（参考 [UCSC WIG 格式指南](http://genome.ucsc.edu/encode/submission.html#WIG) ）。


例如：


```css
phastCons --target-coverage 0.25 --expected-length 12 \
    --msa-format MAF alignment.maf cons.mod,noncons.mod \
    > scores.wig
```

在上述示例中，输入比对文件格式为 UCSC 基因组浏览器使用的多序列比对格式（MAF）。另一种情况是，保守模型被定义为非保守模型的缩放版本，缩放因子为 rho = 0.4：


```css
phastCons --target-coverage 0.25 --expected-length 12 \
    --rho 0.4 --msa-format MAF alignment.maf noncons.mod \
    > scores.wig
```
通过添加 `--most-conserved` 选项，可以预测一组离散的保守元件。例如：

```css
phastCons --target-coverage 0.25 --expected-length 12 --rho 0.4 \
    --most-conserved most-cons.bed --msa-format MAF alignment.maf \
    noncons.mod > scores.wig
```
在此命令中，预测的保守元件的坐标将以 BED 格式写入文件 `most-cons.bed` 中（参见 [UCSC BED 格式指南](http://genome.ucsc.edu/goldenPath/help/customTrack.html#BED) ）。如果输出文件名使用 `.gff` 后缀，则会以 GFF 格式输出（参见 [GFF 规范](http://www.sanger.ac.uk/Software/formats/GFF/GFF_Spec.shtml) ）。如果不需要计算保守性评分，可以使用 `--no-post-probs` 选项，这样可以节省一些计算时间。如果同时使用 `--no-post-probs` 且未指定 `--most-conserved`，程序将不会预测离散元件或生成保守性评分。然而，如果指定，程序仍然可以估计自由参数（详见后续内容）。
#### 总结 

程序的默认行为是：

1. 估计任何未指定的自由参数；

2. 计算保守性评分并将其输出到标准输出（stdout）。

通过组合以下选项，可以选择程序执行的主要任务：
 
- 是否使用 `--most-conserved` 来预测保守元件；
 
- 是否使用 `--no-post-probs` 来避免计算保守性评分；

- 是否指定自由参数的值。


### 2.2 获取系统发育模型 
PhastCons 使用的系统发育模型（`.mod` 文件）可以通过以下三种主要方式获得： 

1. **直接通过 PhastCons 从数据中估计模型** 
使用一种“无监督”学习算法（PhastCons 论文中描述的 EM 算法）估计模型。
 
2. **从标注的训练数据中单独估计非保守模型** 
例如使用四倍简约位点（4d）或祖先重复序列（ARs）作为训练数据，并让 PhastCons 估计缩放参数 rho。
 
3. **从标注的训练数据中单独估计保守和非保守模型** 
在这种情况下，不使用 PhastCons 进行任何模型估计。

#### 选择建议 
通常推荐使用 **选项 1** ，但某些情况下可能不适用： 
- 当数据集中包含非常远缘的物种，这些物种的比对主要出现在保守区域（如编码外显子）时，非保守模型的分支长度可能会被低估。这是因为任何能够比对的“非保守”碱基实际上可能至少部分是保守的。在这种情况下，可能需要：
  - 从编码区域的 4d 位点（比对可靠）估计非保守模型；

  - 使用标注的训练数据估计保守模型（选项 3）；

  - 或通过让 PhastCons 估计 rho 参数（选项 2）来定义保守模型。

实验表明（基于脊椎动物数据，包括人类、小鼠、大鼠、鸡和河豚序列），从 4d 位点估计非保守模型与直接通过 PhastCons 估计相比，对预测的保守元件影响不大，即使分支长度的估计结果存在较大差异（详见 PhastCons 论文的补充材料）。EM 算法（选项 1）即使在非保守模型未能完全准确表示中性替代过程的情况下，仍能区分保守和非保守区域。
另一方面，当物种数量较多（例如超过 12 个）时，选项 1 的计算可能非常密集。出于实际考虑，可能更适合使用 **选项 2 或选项 3** （详见下面的 ENCODE 示例）。
#### 实用建议 
 
- 如果物种数量少于 12，推荐 **选项 1** 。
 
- 如果物种数量超过 12，或数据集中包含许多远缘物种且比对覆盖率较低，且您担心分支长度的偏差，推荐 **选项 2** 。


### 使用选项 2 或 3 估计系统发育模型 
对于选项 2 或 3，可以使用 PHAST 包中的 `phyloFit` 程序来估计模型。以下是具体步骤：
#### 示例 1: 基于祖先重复序列 (ARs) 
假设您有一个多序列比对文件 `data.fa`（FASTA 格式），其中包含人类、小鼠和大鼠的序列。另外，有一个 GFF 格式的注释文件 `AR.gff`，定义了祖先重复序列（ARs）在比对中第一个序列（人类）的坐标系中的位置。例如：

```bash
human   RepeatMasker    AR      14451829    14451925    .   .   .   type "L1MC4a.LINE.L1"
human   RepeatMasker    AR      14452228    14452443    .   .   .   type "L1MC4a.LINE.L1.2"
human   RepeatMasker    AR      14458153    14458278    .   .   .   type "L2.LINE.L2"
...
```
可以使用以下命令估计这些位点的系统发育模型，并将模型保存到 `nonconserved.AR.mod` 文件：

```bash
phyloFit --tree "(human,(mouse,rat))" --features AR.gff \
         --do-cats AR --out-root nonconserved data.fa
```
 
- **`--tree`**  指定系统发育树，例如 `(human,(mouse,rat))`。
 
- **`--features`**  指定包含注释信息的 GFF 文件。
 
- **`--do-cats`**  指定类别（例如 `AR`）。
 
- **`--out-root`**  设置输出文件的根名称（这里生成 `nonconserved.AR.mod`）。
更多参数详情可通过 `phyloFit --help` 查看。

---


#### 示例 2: 基于四倍简约位点 (4d sites) 
如果要从四倍简约位点估计非保守模型，可以使用 `msa_view` 工具提取这些位点。假设您有一个 GFF 格式文件 `genes.gff`，其中描述了比对中人类序列的编码区。例如：

```arduino
human    UCSC        CDS     4562    4692    .   -   0   transcript_id "NM_198943"; gene_id "NM_198943"
human    UCSC        CDS     7469    7605    .   -   0   transcript_id "NM_198943"; gene_id "NM_198943"
human    UCSC        CDS     7778    7924    .   -   0   transcript_id "NM_198943"; gene_id "NM_198943"
...
```

您可以通过以下步骤提取 4d 位点并估计模型：
 
1. **使用 `msa_view` 提取 4d 位点** 
假设多序列比对文件为 `data.fa`：

```bash
msa_view --4d --features genes.gff --out-format FASTA data.fa > 4d_sites.fa
```
 
2. **使用 `phyloFit` 估计模型** 
使用提取的 4d 位点：

```bash
phyloFit --tree "(human,(mouse,rat))" --out-root nonconserved_4d 4d_sites.fa
```
这样可以生成针对四倍简约位点的非保守模型 `nonconserved_4d.mod`。

---


#### 注意事项 

- 如果使用 GFF 注释，确保坐标与比对文件中第一个序列（参考物种）的一致性。
 
- 在使用选项 2 时，PhastCons 会进一步缩放非保守模型以生成保守模型（通过 `--rho` 参数）。

- 在选项 3 中，您需要手动为 PhastCons 提供保守和非保守模型。

通过这些方法，您可以生成精确的模型以适应特定的分析需求。


### 提取四倍简约位点 (4d sites) 并估计系统发育模型 

以下是通过 PHAST 工具提取 4d 位点并估计非保守模型的具体步骤：


---


#### 单个比对文件的 4d 位点提取和建模 
 
1. **提取包含 4d 位点的完整密码子** 
使用 `msa_view` 从输入比对（`data.fa`）和注释文件（`genes.gff`）中提取包含 4d 位点的完整密码子，并保存为 SS 格式：

```bash
msa_view data.fa --4d --features genes.gff > 4d-codons.ss
```
 
2. **提取仅包含 4d 位点的单碱基** 
使用 `msa_view` 将 SS 文件中的 4d 位点提取为单个碱基：

```bash
msa_view 4d-codons.ss --in-format SS --out-format SS --tuple-size 1 > 4d-sites.ss
```
 
3. **估计非保守模型** 
使用 `phyloFit` 对 4d 位点生成的 SS 文件估计非保守模型：

```bash
phyloFit --tree "(human,(mouse,rat))" --msa-format SS \
    --out-root nonconserved-4d 4d-sites.ss
```


---


#### 多个比对文件的 4d 位点提取和合并建模 
 
1. **从多个文件中提取 4d 位点** 
假设有多个比对文件和对应的 GFF 文件（如 `set1.maf` 和 `set1.gff`），可以使用以下循环逐一提取每组的 4d 位点：

```bash
for set in set1 set2 set3; do
    msa_view $set.maf --in-format MAF --4d --features $set.gff > $set.codons.ss
    msa_view $set.codons.ss --in-format SS --out-format SS --tuple-size 1 > $set.sites.ss
done
```
 
2. **合并多个 4d 位点文件** 
使用 `msa_view` 将所有 `.sites.ss` 文件合并为一个文件：

```bash
msa_view --aggregate human,mouse,rat *.sites.ss > all-4d.sites.ss
```
 
3. **估计合并的非保守模型** 
对合并的 4d 位点文件运行 `phyloFit`：

```bash
phyloFit --tree "(human,(mouse,rat))" --msa-format SS \
    --out-root nonconserved-all-4d all-4d.sites.ss
```


---


#### 提取保守区域模型（例如，第一个密码子位置） 
 
1. **提取第一个密码子位置** 
使用 `msa_view` 提取基因的第一个密码子位置（`CDS 1-3` 表示 CDS 包含 1 到 3 个密码子）：

```bash
msa_view data.maf --in-format MAF --features genes.gff \
    --catmap "NCATS = 3; CDS 1-3" --out-format SS --unordered-ss \
    --reverse-groups transcript_id > codons.ss
```
 
2. **估计保守区域模型** 
使用 `phyloFit` 对第一个密码子位置建模，指定分类 `--do-cats 1`：

```bash
phyloFit --tree "(human,(mouse,rat))" --msa-format SS --do-cats 1 \
    --out-root conserved-codon1 codons.ss
```


---


### 注意事项 
 
- **输入格式** ：确保比对文件和注释文件的格式正确（如 FASTA/MAF 和 GFF）。
 
- **`catmap` 参数** ：`--catmap` 用于定义密码子位置的分类信息。
 
- **系统发育树** ：调整 `--tree` 参数以匹配您的物种关系。
 
- **并行处理** ：对于多个比对文件，可以并行提取和建模以节省时间。

通过这些步骤，您可以提取目标区域的序列并生成适用于不同分析需求的系统发育模型。


使用 `phastCons` 的不同选项估计参数并预测保守区域

---

**使用 `phastCons` 的不同选项估计参数并预测保守区域

---

选项 3: 使用 `phyloFit` 估计模型后直接运行 `phastCons`**  
1. 使用 `phyloFit` 分别从注释的保守和非保守区域估计模型（如 `conserved.mod` 和 `nonconserved.mod`）。
 
2. 将这些模型直接输入 `phastCons`：

```bash
phastCons --target-coverage 0.25 --expected-length 12 \
    --msa-format MAF alignment.maf \
    conserved.mod,nonconserved.mod > scores.wig
```


---

**使用 `phastCons` 的不同选项估计参数并预测保守区域

---

**使用 `phastCons` 的不同选项估计参数并预测保守区域

---

选项 3: 使用 `phyloFit` 估计模型后直接运行 `phastCons`**  
1. 使用 `phyloFit` 分别从注释的保守和非保守区域估计模型（如 `conserved.mod` 和 `nonconserved.mod`）。
 
2. 将这些模型直接输入 `phastCons`：

```bash
phastCons --target-coverage 0.25 --expected-length 12 \
    --msa-format MAF alignment.maf \
    conserved.mod,nonconserved.mod > scores.wig
```


---

选项 2: 使用 `--estimate-rho` 优化 `rho` 参数**  
1. 提供一个初始非保守模型（例如从 4d 位点生成的 `nonconserved-4d.mod`）。
 
2. 启用 `--estimate-rho` 选项，以迭代优化参数 `rho`：

```bash
phastCons --target-coverage 0.25 --expected-length 12 \
    --rho 0.4 --estimate-rho mytrees --msa-format MAF \
    alignment.maf nonconserved-4d.mod > scores.wig
```
 
  - **关键参数** ： 
    - `--rho 0.4`：初始化 `rho` 参数为 0.4。
 
    - `--estimate-rho`：启用 `rho` 参数的 EM 算法估计。
 
    - 输出的新模型将保存为 `mytrees.cons.mod` 和 `mytrees.noncons.mod`。
 
3. 如果需要，可以将 `--estimate-rho` 的输出用于后续预测：

```bash
phastCons --target-coverage 0.25 --expected-length 12 \
    --msa-format MAF alignment.maf \
    mytrees.cons.mod,mytrees.noncons.mod > scores.wig
```


---

**使用 `phastCons` 的不同选项估计参数并预测保守区域

---

**使用 `phastCons` 的不同选项估计参数并预测保守区域

---

选项 3: 使用 `phyloFit` 估计模型后直接运行 `phastCons`**  
1. 使用 `phyloFit` 分别从注释的保守和非保守区域估计模型（如 `conserved.mod` 和 `nonconserved.mod`）。
 
2. 将这些模型直接输入 `phastCons`：

```bash
phastCons --target-coverage 0.25 --expected-length 12 \
    --msa-format MAF alignment.maf \
    conserved.mod,nonconserved.mod > scores.wig
```


---

**使用 `phastCons` 的不同选项估计参数并预测保守区域

---

**使用 `phastCons` 的不同选项估计参数并预测保守区域

---

选项 3: 使用 `phyloFit` 估计模型后直接运行 `phastCons`**  
1. 使用 `phyloFit` 分别从注释的保守和非保守区域估计模型（如 `conserved.mod` 和 `nonconserved.mod`）。
 
2. 将这些模型直接输入 `phastCons`：

```bash
phastCons --target-coverage 0.25 --expected-length 12 \
    --msa-format MAF alignment.maf \
    conserved.mod,nonconserved.mod > scores.wig
```


---

选项 2: 使用 `--estimate-rho` 优化 `rho` 参数**  
1. 提供一个初始非保守模型（例如从 4d 位点生成的 `nonconserved-4d.mod`）。
 
2. 启用 `--estimate-rho` 选项，以迭代优化参数 `rho`：

```bash
phastCons --target-coverage 0.25 --expected-length 12 \
    --rho 0.4 --estimate-rho mytrees --msa-format MAF \
    alignment.maf nonconserved-4d.mod > scores.wig
```
 
  - **关键参数** ： 
    - `--rho 0.4`：初始化 `rho` 参数为 0.4。
 
    - `--estimate-rho`：启用 `rho` 参数的 EM 算法估计。
 
    - 输出的新模型将保存为 `mytrees.cons.mod` 和 `mytrees.noncons.mod`。
 
3. 如果需要，可以将 `--estimate-rho` 的输出用于后续预测：

```bash
phastCons --target-coverage 0.25 --expected-length 12 \
    --msa-format MAF alignment.maf \
    mytrees.cons.mod,mytrees.noncons.mod > scores.wig
```


---

选项 1: 使用 `--estimate-trees` 优化所有参数**  
1. 准备一个初始模型（例如从全局比对中生成的 `init.mod`）：

```bash
phyloFit --tree "(human,(mouse,rat))" --msa-format MAF \
    --out-root init alignment.maf
```
 
2. 运行 `phastCons`，使用 `--estimate-trees` 来优化所有参数：

```bash
phastCons --target-coverage 0.25 --expected-length 12 \
    --estimate-trees mytrees --msa-format MAF \
    alignment.maf init.mod > scores.wig
```
 
  - **关键点** ： 
    - `--estimate-trees`：通过最大似然估计优化所有自由参数。
 
    - 输出的新模型将保存为 `mytrees.cons.mod` 和 `mytrees.noncons.mod`。
 
3. 如果需要，进行二次运行以预测保守区域：


```bash
phastCons --target-coverage 0.25 --expected-length 12 \
    --msa-format MAF alignment.maf \
    --most-conserved most-cons.bed \
    mytrees.cons.mod,mytrees.noncons.mod > scores.wig
```


---

**选项运行的策略建议**  
1. **选项 3** ：简单直接，适用于已有模型的情况。
 
2. **选项 2** ：适合需要优化 `rho` 参数的场景，尤其是当初始模型可信但 `rho` 尚未确定时。
 
3. **选项 1** ：适用于复杂数据集（如包含多个物种）或需要全模型优化的场景，但训练过程可能耗时较长。


---

**训练与预测分离的建议**  
- 对于大规模数据集，建议分离训练与预测步骤：
 
  1. **训练** ：
    - 在全数据集或其子集（如随机选择 100-200 个片段）上运行训练。

    - 如有计算资源，可以并行处理多个小片段。
 
  2. **预测** ：
    - 使用训练后的参数在全数据集上运行预测。
 
- 示例：


```bash
phastCons --target-coverage 0.25 --expected-length 12 \
    --estimate-trees mytrees --msa-format MAF alignment.maf \
    init.mod --no-post-probs
phastCons --target-coverage 0.25 --expected-length 12 \
    --msa-format MAF alignment.maf --most-conserved most-cons.bed \
    mytrees.cons.mod,mytrees.noncons.mod > scores.wig
```


---

**可选参数调整**  
- `--target-coverage` 和 `--expected-length` 是控制预测结果的重要参数，但可以根据具体数据和需求调整。
 
- 使用 `--no-post-probs` 时，将跳过输出后验概率的计算。

通过上述方法，用户可以根据数据特性灵活选择合适的选项并高效完成保守区域的预测分析。
