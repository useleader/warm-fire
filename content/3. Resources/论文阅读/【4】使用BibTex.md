---
publish: true
---

1. 创建.bib文件

```bibtex
@文献类型{你的引用标识,
  author    = {作者信息},
  title     = {文献标题},
  journal   = {期刊名},
  year      = {年份},
  volume    = {卷},
  number    = {期},
  pages     = {页码},
  ...       = {其他字段}
}
```

**文献类型**：根据文献的种类选择，常用的有：

- `@article`：期刊或杂志文章
    
- `@inproceedings`：会议论文
    
- `@book`：书籍
        
- `@phdthesis`：博士论文
    
- `@misc`：当其他类型都不符合时使用

```bibtex
@inproceedings{hara2018can,
  title     = {Can Spatiotemporal 3D CNNs Retrace the History of 2D CNNs and ImageNet?},
  author    = {Hara, Kensho and Kataoka, Hirokatsu and Satoh, Yutaka},
  booktitle = {Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition},
  pages     = {6546--6555},
  year      = {2018}
}
```
2. 在latex中设置引用
**设置参考文献样式 (`\bibliographystyle`)**
- `plain`：按作者字母顺序编号，是标准样式之一
    
- `unsrt`：按引用的先后顺序编号
    
- `alpha`：使用作者姓氏和年份的缩写作为标号
    
- `ieeetr`：国际电气电子工程师协会（IEEE）常用的样式

里面引用到的时候加一个\cite{引用标识}
然后在最后加一个**`\bibliography`**
```
\bibliography{myrefs} % 告诉LaTeX使用 myrefs.bib 文件
\end{document}
```