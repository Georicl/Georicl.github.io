# genome annotation 
向旸

## 进行基因组结构注释的前期准备

1.  前期需要准备的软件\
    我们本次使用braker - maker 流程进行组装基因组的注释,所以需求的软件为:\
    braker2/3   本次使用的软件为braker-3.0.8,并且使用手动安装的方式安装\
    maker maker可以直接使用conda进行安装.
2.  为什么使用braker进行模型的训练?\
    [braker](https://github.com/Gaius-Augustus/BRAKER)是一个整合了GeneMark、Augustus、blast等软件进行基因组结构注释的一个工具,可以输
    入各种证据(如蛋白质序列、EST)进行基因组模型结构的预测, 也可以根据输入的数据选择不同的模式.\
    在本次注释流程中,我们使用Braker的物种训练功能,进行augustus物种模型的训练,但是braker同时也会生成一个gtf注释文件,与相应的aa蛋白序列文件和
    相应的cds序列.\
3.  braker-maker流程是什么样的?\
    我们使用braker整合的GeneMark-ET(真核生物)和augustus进行物种模型的训练,并且产生物种的模型,然后再使用maker输入拼接而成的EST序列(转录本序列
    )和同源蛋白序列等证据进行证据的整合,最后得到一个完整的该物种基因组的基因组结构注释文件.\
    [maker](https://github.com/Yandell-Lab/maker)在此处查看github的官网.
4.  安装软件

- 进行braker的安装\
  对于不具有管理员权限的普通用户而言,有两种使用braker的方法:分别为使用**容器**进行安装,或使用直接安装的方式
    - 使用容器安装的方式:\
      对于不具有管理员root权限的用户来说,可以通过装配singularity容器的方式,通过docker拉取braker的镜像进行安装.\
      singularity可以不需要管理员权限即可以安装,不需要像docker一样需要root权限,我们先安装singularity:\
```bash
wget https://github.com/sylabs/singularity/releases/download/v4.2.1/singularity-ce-4.2.1.tar.gz
#但是在此之前,singularity是由Go语言开发软件,我们要先安装Go语言:
wget https://golang.google.cn/dl/go1.23.3.linux-amd64.tar.gz #下载Go语言安装
sudo tar --directory=/usr/local -xzvf go1.13.linux-amd64.tar.gz 
#解压并放置在用户的local目录下,其实可以不需要sudo权限,如果放在你这个用户的local下的话可以不需要
#解压后我们还需要将包放入到环境变量中,需要编辑
vim ~/.bashrc
export PATH=$PATH:/usr/local/go/bin
#然后编译安装singularity
cd /path/to/singularity
./mconfig
cd builddir
make
make install
#运行以上步骤后,singularity就安装完毕了,可以使用测试数据进行测试下载:
singularity run library://godlovedc/funny/lolcow #拉取测试数据
```
安装好singularity后可以进行braker3镜像拉取:
```Bash
singularity build braker3.sif docker://teambraker/braker3:latest #拉取braker3的镜像到本地
singularity exec braker3.sif braker.pl #运行braker.pl
#我们无法确定braker是否可以稳定运行,可以对其进行测试(该测试方法均由github的braker项目中自带)
singularity exec -B $PWD:$PWD braker3.sif cp /opt/BRAKER/example/singularity-tests/test1.sh 
singularity exec -B $PWD:$PWD braker3.sif cp /opt/BRAKER/example/singularity-tests/test2.sh 
singularity exec -B $PWD:$PWD braker3.sif cp /opt/BRAKER/example/singularity-tests/test3.sh 
export BRAKER_SIF=/your/path/to/braker3.sif # may need to modify
bash test1.sh
bash test2.sh
bash test3.sh
### 使用容器进行安装可以避免安装braker遭遇环境问题,最难的一步实际为拉取braker3的镜像,需要自行寻找拉取的镜像网站,或者给服务器挂个东西..
### 我们无法更改容器的参数,也不对容器的环境以及软件目录具有写入的权限,但是augustus的物种模型会生成在/augustus/config/species/下,因此
### 我们需要将本地的augustus config目录挂载进singularity中,让其具有写入权限可以将物种模型生成在本地可写入的config目录中
```
由容器安装的方法可以很好地规避由于braker需要过多的软件依赖导致的环境问题,但由于容器的限制,对于环境以及软件本身我们不具有写入权限,因此我们也有一定限制.

- 直接安装法:\
  我们也可以选择直接安装braker3并且安装其所有相关的依赖,以在自己本地用户下构建一个完整的braker环境,具有更强的参数变更性.
```
wget https://github.com/Gaius-Augustus/BRAKER/archive/refs/tags/v3.0.8.tar.gz #下载braker3的源代码自行安装环境
#braker还需要许多附属的软件
在正式安装braker之前先使用conda构建一个稳定的环境
conda create -n braker python=3.11.9 #注意,此处的python版本一定要低于3.12,否则会在一行代码中出现字符串的warning
conda install augustus # 此处直接使用conda安装augustus,但是conda的版本较老,如果觉得有影响也可以直接手动安装,但是本次我们使用conda安装
```
其他大部分附属软件都可以使用conda进行安装,但是GeneMark-ETP的安装则较为不同\
我们先进入[GeneMark](https://genemark.bme.gatech.edu/GeneMark/license_download.cgi)的官网下载页面登记后,根据自己的系统下载GeneMark-ETP\
同时记得下载了软件本身的时候也要下载他的**gmkey**,这个非常重要,因为GeneMark属于授权文件,需要key文件进行进行授权.下载解压后使用`cp gm_key ~/.gm_key`
后才可以运行GeneMark,对genemark进行解压后,运行INSTALL后`cd tools`并且再`perl check_ETP_tools.pl`,根据其缺少的提示下载,直到出现\
```Bash
# checking for presence of GeneMark-ETP dependencies
## BEDTOOLS OK
## DIAMOND OK
## FASTQ-DUMP OK
## GFFREAD OK
## HISAT2-ALIGN-L OK
## HISAT2-ALIGN-S OK
## HISAT2-BUILD-L OK
## HISAT2-BUILD-S OK
## PREFETCH OK
## SAMTOOLS OK
## STRINGTIE OK
## VDB-CONFIG OK
# tools are OK
```
则显示为可用状态.此时GeneMark可以运行.而后我们解压braker,并进入scripts文件夹`perl braker.pl`产生了什么报错我们就下载相应的模块\
这个安装方法有一个优势,如果我们是在本用户下自己安装的conda,并且是conda安装的augustus,则物种模型会生成在conda/env/name/config/species/
下.

- 在运行braker之前,braker3会要求输入经过重复序列屏蔽的基因组,因此我们需要先进行基因组的**重复序列屏蔽**\
  使用repeatMasker进行重复序列的屏蔽`conda install RepeatModeler`使用conda下载可以直接下载,之后输入RepeatModeler测试运行,缺少了什么就
  下载什么.

## 进行重复序列的屏蔽
**在此之首先需要声明的是,根据braker3的官方提示,请使用软屏蔽的方式进行屏蔽,而非硬屏蔽.**
先构建repeatmodeler的文库`BuildDatabase -name species input.fasta #名字参数为你取的物种数据库的名字,build功能没有太多参数选择`\
构建文库后进行Modeler的比对构建(这个步骤运行需要较长时间)
```Bash
RepeatModeler -database species -pa 20 #物种的数据库,以及使用的线程数
#此处有一个特别参数
-engine <abblast|wublast|ncbi>
        The name of the search engine we are using. I.e abblast/wublast or
        ncbi (rmblast version).
#也就是可以指定的三个库里的一个进行repeatmodeler的构建,一般选择ncbi
```
经过模型的构建后,我们得到了一个species-families.fa的文件,这个就是我们构建的重复序列的数据模型,得到了该模型后,我们可以进行重复序列的屏蔽
```Bash
RepeatMasker -e rmblast -pa 20 -xsmall -lib species-families.fa input.fasta 1>repeatmasker.log 2>&1
#这里的参数解释
#-e 使用的屏蔽方式为rmblast -pa 使用20线程进行
#-xsmall参数则是较为特别的参数,意思为使用软屏蔽的方式进行重复序列的屏蔽,而非硬屏蔽,如果无该参数则为硬屏蔽
#一些参数说明
-small  Returns complete .masked sequence in lower case
 #返回完整的屏蔽使用更低case
-xsmall Returns repetitive regions in lowercase (rest capitals) rather than
        masked
 #使用大小写碱基的差异标出重复序列,并且以更低的case进行筛选
-x  Returns repetitive regions masked with Xs rather than Ns
 #使用大小写之间格式差异替换掉重复序列屏蔽使用N的问题,保留了原碱基
-qq Rush job; about 10% less sensitive, 4->10 times faster than default
        (quick searches are fine under most circumstances) repeat options
        #更快的运行速度,但是损失10%的严格度
```
在rmasker完成后,我们会得到一个input.fa.masked的文件,这个是我们最终需要braker输入的文件.同时我们可以观察一下rm的log文件看看比对情况.

## 运行braker
braker不止需要输入masked的序列,也需要输入转录组证据,可以直接先进行比对后,将比对得到的sam文件转为bam文件,并且合并后得到一个all.bam,用该bam
文件作为转录组证据的输入文件,节约大量服务器空间和输入fa.gz证据的比对时间.此处不演示如何比对转录组.\
`perl braker.pl --species=species_name --genome=input.fasta.masked --bam=all.bam --core=20 #使用20线程`\
braker的运行需要一定时间,运行结束后会产生一大堆文件如.aa、.gtf等文件,我们可以`less braker.log`观察结尾是否有
```Bash
#**********************************************************************************
#                               BRAKER RUN FINISHED                                
#**********************************************************************************
```
来判断是否有产生影响到braker运行的错误.如果是正确的运行结果会在env/config/species/下生成一个你输入的物种名的文件夹,以及在braker文件夹下
生成了一个braker生成的结构注释文件以及由这个结构注释文件生成的蛋白序列,cds序列等文件.到这一步我们就已经使用braker生成了物种的训练模型

## 使用maker整合多方证据
进行证据的整合的部分较为繁琐与复杂,需要分模块进行讲解,首先我们需要准备多个证据:**同源蛋白序列.aa文件**、**EST,转录本文件**、**AUGUSTUST生
成的物种模型**

- EST,转录本序列证据:\
  首先如果我们没有该物种的转录本,我们在送测基因组时也应该会有转录组数据的积累,我们可以使用转录组数据拼接生成一个转录本作为输入文件,在这个情况下
  使用的转录组的数据数量越大则证据越好,接下来将使用Trinity进行转录本的拼接
```Bash
nohup Trinity --genome_guided_bam input.bam 
              --max_memory 200G --genome_guided_max_intron 10000 --CPU 30 &
#这个模式为Trinity genome_guided mod 是在已经经过了参考基因组的比对后生成的bam文件的指导组装的模式,这样可以有效缩短trinity的比对时间
#提高trinity的运行效率,如果该物种此前无参,则建议走无参转录组流程拼装转录本(速度较慢)
#将转录组文件逐个比对后,得到多个fasta文件,将其合并后使用cd-hit进行聚类和降低冗余
cat *.fasta > all.fasta
cd-hit-est -i all.fasta -o cd_hit.fasta -T 40 -M 50000  
#cd-hit-est 对核酸序列进行聚类,输入文件为all.fasta 使用50G进行运算,使用足够的运存可以提高cd-hit的运转时间,cd-hit主要吃运存比较多
#输出一个cd_hit.fasta
```
至此我们就已经整理好了转录本的序列证据,同时我们可以在/env/config/species中找到由之前braker3生成的物种模型,之后我们需要获取蛋白序列的证据
可以通过[NCBI](https://www.ncbi.nlm.nih.gov/)中获取近缘物种的蛋白序列文件,如果已经是有该物种的基因组的文件,可以直接下载该物种的原基因组
的蛋白序列文件直接使用.\
至此我们已经准备好了三种证据:augustus训练的物种模型、蛋白序列、EST/转录本序列.\

- 配置maker文件\
  输入`maker -CTL`在当前目录下生成maker的配置文件,由于maker的配置项目繁多,因此maker可以通过生成配置文件的方式来启动,本次使用配置文件方式启动
  生成了四个文件,分别是:maker_bopts.ctl、maker_evm.ctl、maker_exe.ctl、maker_opts.ctl.接下来我们分别探讨这四个文件如何改动

- maker_bopts.ctl\
  blast模块的调整,一般保持默认即可.
```Bash
#-----BLAST and Exonerate Statistics Thresholds
blast_type=ncbi+ #set to 'ncbi+', 'ncbi' or 'wublast' # 使用的blast比对类型
use_rapsearch=0 #use rapsearch instead of blastx, 1 = yes, 0 = no # 使用rapsearch代替blastx

pcov_blastn=0.8 #Blastn 覆盖率阈值 EST 基因组比对
pid_blastn=0.85 #Blastn 识别到EST转录本的阈值
eval_blastn=1e-10 #Blastn 的E-value
bit_blastn=40 #Blastn bit cutoff
depth_blastn=0 #Blastn depth cutoff (0 to disable cutoff)

pcov_blastx=0.5 #Blastx Percent Coverage Threhold Protein-Genome Alignments
pid_blastx=0.4 #Blastx Percent Identity Threshold Protein-Genome Aligments
eval_blastx=1e-06 #Blastx eval cutoff
bit_blastx=30 #Blastx bit cutoff
depth_blastx=0 #Blastx depth cutoff (0 to disable cutoff)

pcov_tblastx=0.8 #tBlastx Percent Coverage Threhold alt-EST-Genome Alignments
pid_tblastx=0.85 #tBlastx Percent Identity Threshold alt-EST-Genome Aligments
eval_tblastx=1e-10 #tBlastx eval cutoff
bit_tblastx=40 #tBlastx bit cutoff
depth_tblastx=0 #tBlastx depth cutoff (0 to disable cutoff)

pcov_rm_blastx=0.5 #Blastx Percent Coverage Threhold For Transposable Element Masking
pid_rm_blastx=0.4 #Blastx Percent Identity Threshold For Transposbale Element Masking
eval_rm_blastx=1e-06 #Blastx eval cutoff for transposable element masking
bit_rm_blastx=30 #Blastx bit cutoff for transposable element masking

ep_score_limit=20 #Exonerate protein percent of maximal score threshold
en_score_limit=20 #Exonerate nucleotide percent of maximal score threshold
```

- maker_evm.ctl
  这个是证据相关的得分的evm,一般不做修改.
```Bash
#-----Transcript weights
evmtrans=10 #default weight for source unspecified est/alt_est alignments
evmtrans:blastn=0 #weight for blastn sourced alignments
evmtrans:est2genome=10 #weight for est2genome sourced alignments
evmtrans:tblastx=0 #weight for tblastx sourced alignments
evmtrans:cdna2genome=7 #weight for cdna2genome sourced alignments

#-----Protein weights
evmprot=10 #default weight for source unspecified protein alignments
evmprot:blastx=2 #weight for blastx sourced alignments
evmprot:protein2genome=10 #weight for protein2genome sourced alignments

#-----Abinitio Prediction weights
evmab=10 #default weight for source unspecified ab initio predictions
evmab:snap=10 #weight for snap sourced predictions
evmab:augustus=10 #weight for augustus sourced predictions
evmab:fgenesh=10 #weight for fgenesh sourced predictions
evmab:genemark=7 #weight for genemark sourced predictions
```

- maker_exe.ctl
  这里存放的是软件的识别路径,可以直接看到各个软件的识别路径,一般如果使用conda安装maker的话,他会自动识别的.\
```Bash
makeblastdb=/ME4012_Vol0002/Ahome_Dir/waiyong/miniconda3/envs/maker/bin/makeblastdb #location of NCBI+ makeblastdb executable
blastn=/ME4012_Vol0002/Ahome_Dir/waiyong/miniconda3/envs/maker/bin/blastn #location of NCBI+ blastn executable
blastx=/ME4012_Vol0002/Ahome_Dir/waiyong/miniconda3/envs/maker/bin/blastx #location of NCBI+ blastx executable
tblastx=/ME4012_Vol0002/Ahome_Dir/waiyong/miniconda3/envs/maker/bin/tblastx #location of NCBI+ tblastx executable
formatdb=/usr/local/bin/formatdb #location of NCBI formatdb executable
blastall= #location of NCBI blastall executable
xdformat= #location of WUBLAST xdformat executable
blasta= #location of WUBLAST blasta executable
prerapsearch=/ME4012_Vol0002/Ahome_Dir/waiyong/miniconda3/envs/maker/bin/prerapsearch #location of prerapsearch executable
rapsearch=/ME4012_Vol0002/Ahome_Dir/waiyong/miniconda3/envs/maker/bin/rapsearch #location of rapsearch executable
RepeatMasker=/ME4012_Vol0002/Ahome_Dir/waiyong/miniconda3/envs/maker/bin/RepeatMasker #location of RepeatMasker executable
exonerate=/ME4012_Vol0002/Ahome_Dir/waiyong/miniconda3/envs/maker/bin/exonerate #location of exonerate executable

#-----Ab-initio Gene Prediction Algorithms
snap=/ME4012_Vol0002/Ahome_Dir/waiyong/miniconda3/envs/maker/bin/snap #location of snap executable
gmhmme3=/ME4012_Vol0002/Ahome_Dir/waiyong/genome_build/Genemark/gmetp_linux_64/bin/gmes/gmhmme3 #location of eukaryotic genemark executable
gmhmmp= #location of prokaryotic genemark executable
augustus=/ME4012_Vol0002/Ahome_Dir/waiyong/miniconda3/envs/maker/bin/augustus #location of augustus executable
fgenesh= #location of fgenesh executable
evm= #location of EvidenceModeler executable
tRNAscan-SE=/ME4012_Vol0002/Ahome_Dir/waiyong/miniconda3/envs/maker/bin/tRNAscan-SE #location of trnascan executable
snoscan=/ME4012_Vol0002/Ahome_Dir/waiyong/miniconda3/envs/maker/bin/snoscan #location of snoscan executable

#-----Other Algorithms
probuild=/ME4012_Vol0002/Ahome_Dir/waiyong/genome_build/Genemark/gmetp_linux_64/bin/gmes/probuild #location of probuild executable (required for genemark)
```

- maker_opts.ctl
```Bash
#-----Genome (these are always required)
genome=input.fasta # 输入需要注释的基因组
organism_type=eukaryotic #eukaryotic or prokaryotic. Default is eukaryotic

#-----Re-annotation Using MAKER Derived GFF3 这个是重复注释的时候才使用该选项,如果不是重复注释则不要输入maker_gff
maker_gff= #MAKER derived GFF3 file
est_pass=0 #use ESTs in maker_gff: 1 = yes, 0 = no
altest_pass=0 #use alternate organism ESTs in maker_gff: 1 = yes, 0 = no
protein_pass=0 #use protein alignments in maker_gff: 1 = yes, 0 = no
rm_pass=0 #use repeats in maker_gff: 1 = yes, 0 = no
model_pass=0 #use gene models in maker_gff: 1 = yes, 0 = no
pred_pass=0 #use ab-initio predictions in maker_gff: 1 = yes, 0 = no
other_pass=0 #passthrough anyything else in maker_gff: 1 = yes, 0 = no

#-----EST Evidence (for best results provide a file for at least one) EST转录本证据的参数
est=cd_hit.fasta #输入EST、转录本证据的路径文件
altest= #EST/cDNA sequence file in fasta format from an alternate organism
est_gff= #EST、转录本的GFF文件
altest_gff= #aligned ESTs from a closly relate species in GFF3 format

#-----Protein Homology Evidence (for best results provide a file for at least one)
protein=input.aa  #输入蛋白序列证据
protein_gff=  #输入蛋白序列注释文件

#-----Repeat Masking 重复序列屏蔽选项,如果不需要屏蔽重复序列建议不填
model_org= #select a model organism for DFam masking in RepeatMasker
rmlib= #provide an organism specific repeat library in fasta format for RepeatMasker
repeat_protein= #provide a fasta file of transposable element proteins for RepeatRunner
rm_gff= #pre-identified repeat elements from an external GFF3 file
prok_rm=0 #forces MAKER to repeatmask prokaryotes (no reason to change this), 1 = yes, 0 = no
softmask=1 #use soft-masking rather than hard-masking in BLAST (i.e. seg and dust filtering)

#-----Gene Prediction 基因预测
snaphmm= #SNAP HMM file
gmhmm= #GeneMark HMM file
augustus_species=zj #augustus预测的物种模型的名字,会在config里找到的文件夹名字
fgenesh_par_file= #FGENESH parameter file
pred_gff= #ab-initio predictions from an external GFF3 file
model_gff= #annotated gene models from an external GFF3 file (annotation pass-through)
run_evm=0 #run EvidenceModeler, 1 = yes, 0 = no
est2genome=1 #infer gene predictions directly from ESTs, 1 = yes, 0 = no
protein2genome=1 #infer predictions from protein homology, 1 = yes, 0 = no
trna=0 #find tRNAs with tRNAscan, 1 = yes, 0 = no
snoscan_rrna= #rRNA file to have Snoscan find snoRNAs
snoscan_meth= #-O-methylation site fileto have Snoscan find snoRNAs
unmask=0 #also run ab-initio prediction programs on unmasked sequence, 1 = yes, 0 = no
allow_overlap= #allowed gene overlap fraction (value from 0 to 1, blank for default)

#-----Other Annotation Feature Types (features MAKER doesn't recognize)
other_gff= #extra features to pass-through to final MAKER generated GFF3 file

#-----External Application Behavior Options
alt_peptide=C #amino acid used to replace non-standard amino acids in BLAST databases
cpus=20 #线程选择,如果你的服务器有MPI的话,请输入1

#-----MAKER Behavior Options
max_dna_len=100000 #length for dividing up contigs into chunks (increases/decreases memory usage)
min_contig=10000 #skip genome contigs below this length (under 10kb are often useless)

pred_flank=200 #flank for extending evidence clusters sent to gene predictors
pred_stats=0 #report AED and QI statistics for all predictions as well as models
AED_threshold=1 #Maximum Annotation Edit Distance allowed (bound by 0 and 1)
min_protein=50 #require at least this many amino acids in predicted proteins
alt_splice=0 #Take extra steps to try and find alternative splicing, 1 = yes, 0 = no
always_complete=1 #extra steps to force start and stop codons, 1 = yes, 0 = no
map_forward=0 #map names and attributes forward from old GFF3 genes, 1 = yes, 0 = no
keep_preds=0 #Concordance threshold to add unsupported gene prediction (bound by 0 and 1)

split_hit=10000 #length for the splitting of hits (expected max intron size for evidence alignments)
min_intron=20 #minimum intron length (used for alignment polishing)
single_exon=0 #consider single exon EST evidence when generating annotations, 1 = yes, 0 = no
single_length=250 #min length required for single exon ESTs if 'single_exon is enabled'
correct_est_fusion=0 #limits use of ESTs in annotation to avoid fusion genes

tries=2 #number of times to try a contig if there is a failure for some reason
clean_try=0 #remove all data from previous run before retrying, 1 = yes, 0 = no
clean_up=0 #removes theVoid directory with individual analysis files, 1 = yes, 0 = no
TMP= #specify a directory other than the system default temporary directory for temporary files 
```
就此情况下我们的配置就简单完成了,之后可以直接输入:`maker`即可直接跑maker,但是该步骤会占用大量时间.