---
publish: true
---

> 参照文件：[[New_IEEEtran_how-to.pdf]]

IEEEtran模板包包含五个主要文件
1. bare_jrnl.tex
2. bare_conf.tex
3. bare_jrnl_compsoc.tex
4. bare_conf_compsoc.tex
5. bare_jrnl_comsoc.tex
conf - conference; jrnl - journal
compsoc - computer society
comsoc - communication society

## A. 论文标题
```latex
\title{paper title}
```
避免使用数学或化学公式

## B. 作者名称与背景
```latex
\author{name}
\IEEEmembership %% 使用斜体字表示IEEE会员身份 %%
\thanks{} %% 将命令变为脚注 %%
```

## C.运行头
