## Code Review for "Winter break? The effect of overwintering on immune gene expression in wood frogs" by Vaziri et al.(2024).

This paper focuses on exploring the immune gene expression responses to transitions into and out of the cold season in a winter-adapted
amphibian, the wood frog (*Lithobates sylvaticus*). While the paper's main objective is to understand how these ectothermic animals respond to parasitism in a reduced physiological state during winter hibernation, I chose to review the code because of the bioinformatic pipeline they have used. They have implemented an RNAseq pipeline to look at differential gene expression at different timepoints. This is relevant to my work as I am also looking at the transcriptomic responses of skipping or experiencing first-year cold dormancy in gopher tortoises (*Gopherus polyphemus*). 

Link to the paper: [here](https://www.sciencedirect.com/science/article/pii/S1744117X24001096?via%3Dihub)

Link to the code: [here](https://github.com/gracevaziri/Naturally-Hibernating-Frog-Transcriptomics)

### Overview of the paper:

Vaziri and colleagues investigated how immune gene expression changes across the hibernation cycle in wood frogs (*Lithobates sylvaticus*), a winter-adapted North American amphibian capable of surviving full-body freezing. The study is motivated by the **thermal mismatch hypothesis** — the idea that when environmental temperatures shift, host immune function and parasite activity may respond at different rates, which may make hosts more vulnerable. This is particularly relevant in the context of climate change, as northern winters are warming rapidly and unpredictably, with unknown consequences for host-parasite dynamics in ectotherms.
To address this, the authors housed wild-caught juvenile wood frogs in outdoor enclosures through a natural Connecticut winter and sampled two immunologically distinct tissues, ventral skin and spleen, at four timepoints spanning fall, winter, emergence from hibernation, and four weeks post-emergence. They investigated gene expression using RNAseq and analyzed it through a combination of differential expression analysis (DESeq2), co-expression network analysis (WGCNA), and GO term enrichment analysis, with nematode parasite infection intensity included as a covariate throughout. The study finds that immune gene expression is far from static during hibernation, showing tissue-specific, seasonal, and infection-driven dynamics that have implications for understanding amphibian vulnerability during a period of rapid environmental change. As requested by the editor, my review below focuses on the code associated with this manuscript.

### Overall Code Summary:
1) The code is well organised with each scripts found in both the author's [GitHub repository](https://github.com/gracevaziri/Naturally-Hibernating-Frog-Transcriptomics) and all metadata can be accessed through [FigShare](https://figshare.com/articles/dataset/Overwintering_wood_frog_2019-2020/25676361). In the GitHub repository, all scripts are named intuitively which gives the reader an overall idea of their function. In addition, scripts are numbered sequentially (e.g. 01, 02, 03, etc.) so the reader knows what preceeds and what follows.

2) I am convinced the authors have provided enough information on how to reproduce their results. Things that are essential for reproducibility that I easily found include:
- climate data
- individual IDs, sample period, tissue type, mass and SVL, sex, parasite species and load per tissue
- gene IDs, gene counts and gene annotations 
- bioinformatic tools such as StringTie and R packages such as  DESeq2, *adegenet*, WCGNA, *limma* and *goseq* with the versions they used   

3) I really like that for the most part there is internal documentation, i.e.  there are small annotations before blocks of code where they explain what the next couple of lines are supposed to do but more importantly, I really like how they have small summary statements and sanity checks. For example, in `02_Prepare Annotation.R` they wrote **"so from above, I can see that there are obviously multiple transcripts assoiated with a single gene. Each of these transcripts was retained from transdecoder output because it was the isoform with the highest score. Now what I want to do is to find and retain one single transcript for each gene that has the longest transcript length"**. Another nice example is in the same script: **"At this point, I should just have one line in my entap file that corresponds to a geneid in my count file"**, followed by a validation if indeed they got one line in the entap file that corresponds to a geneid in their count/file

4) The authors have properly cited any external sources in cases where they used other people's code or servers

### High-level Suggestions:

1) Consider adding a **README.md** file to the repository which would greatly improve reproducibility. Things that may be included:
- an overview of the project and biological questions being asked
- dependancies and how to install packages
- what input data is needed, where to download it and the expected file formats
- not absolutely necessary but a repository structure is useful to orientate the reader
- How to Run/FollowMe- a step by step instruction on the order in which to run scripts
- Authors and Credits- here you can also include links to the GenBank Accession number for the reference genome as well as  reference databases you used (RefSeq complete protein database, UniProt SwissProt database and NCBI NR database) 

2) Consider creating an R Project (.Rproj) file. This is a simple but highly effective practice for managing multi-script pipelines. It automatically sets the working directory to the project root when the project is opened in RStudio, making `setwd()` unnecessary and allowing all file paths to be written relative to the project root. This would also work seamlessly with the here package, further improving path management across all 14 scripts.

3) The authors show partially expressive coding. The tissue-based naming `vs_`, `sp_`, `all_` is applied consistently across most scripts and aids readability. Some variable names are descriptive, such as `rhabdias_log`, `oswaldocruzia_log`, `vs_go_sig_res_over`, and contrast names like `vs_WvF` and `sp_PH1vPH2`. However, there are areas of weakness, for instance, important objects like the main gene count matrix `m`, the raw count input table from StringTie `tab`, and filter vectors `subg` use very short and uninformative names that give no indication of their contents or purpose. Documentation through inline comments exists but is inconsistent: some sections have clear comments explaining biological intent while others have none.

4) The code is largely not modular. Across all scripts reviewed, only one function is defined, `DESeq.2.GO()` in the `10_GO enrichment analysis on DEGs.R` script, and even that function is poorly implemented, relying on global variables rather than its own parameters. The vast majority of the analysis is written as long, flat scripts with repeated copy-pasted blocks. The script `06_Make PCA.R` is the clearest example, where identical code is repeated three times for `vs`, `sp`, and `all` instead of being wrapped in a reusable function. This copy-paste pattern also appears in `04_Set up DESeq2 analysis.R`, `05_DGE results.R`, and `09_GO enrichment on WGCNA modules.R`. This is a significant code quality issue as it makes the code harder to maintain, harder to read, and more prone to subtle bugs.

5) **Git version control** was never implemented; there is only one commit in the entire history, meaning Git was never used to track changes, iterations, or development of the code over time. The authors uploaded all .R files directly through the **GitHub** website which indicates that they did not commit and push through **RStudio** or the **command line** frequently nor used descriptive commit messages which interferes with reproducibility and transparency.

### Low-level Suggestions:

1) For WGCNA analyses, there are two reusable functions `make_module_heatmap_vs()` and `make_module_heatmap_sp()` with docstrings and default arguments, which is good. However, the code is essentially duplicated, resulting in one long file. A key improvement would be a single parameterized function for the whole WGCNA pipeline that can run for any tissue and return a list of results specifically for ventral skin or spleen. 

2) Inconsistent use of `=` vs `<-` for assignment. For example,` m = m %>%` and `vs_sampleTable = filter(...)` use `=` while most other assignments use `<-`. In R, `<-` is the recommended assignment operator.

3) Column indices like `c(2:52)` are repeated multiple times across scripts with no explanation. If the number of samples ever changes, every instance would need to be manually updated. A named variable like `count_cols <- 2:52` defined once at the top would be safer and clearer.

4) There are inconsistent file paths, for example, the script `11_UpSetR plots_Figure_4-5.R` uses both placeholder paths and what appears to be a real absolute path. This is inconsistent and suggests the code was not cleaned up before being shared. The real path also reveals the author's personal folder structure, which is not portable. Additionally, there is a typo in  `youtpath` instead of `yourpath` that would cause the script to fail immediately.

5) There are multiple blocks of commented-out code left throughout the scripts which suggests incomplete cleanup. This reduces readability and might confuse the reader why it is there.
