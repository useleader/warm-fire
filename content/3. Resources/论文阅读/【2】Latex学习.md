---
publish: true
---

# 文本排版
## Latex格式
1. 文档类及其选项 `\documentclass[options]{class}`
![[Pasted image 20251110102017.png]]
![[Pasted image 20251110102027.png]]
2. 使用宏包 `\usepackage{}`
3. 正文内容
	- `\begin{document}`
	- `\end{document}`
4. 内容分节
	- `\section{}` 文档章节，自动编号，自动出现在目录中
	- `\subsection{}` `\subsubsection{}` ...
	- `\paragraph{}`文档段落，这个使用频率很短，不会自动编号，也不会自动出现在目录中
5. 文本编辑
	- 空格和制表符等空白字符视为相同的空白距离
	- 多个连续的空白字符视为一个空白字符
	- 每行开始的空白字符被忽略，单个回车视为空格
	- 分段使用两个以上的空行
6. 特殊字符
	- `# $ % ^ & _ { } - \`
	- `\`用于转义
	- `\\`是断行，用`$\backslash$`

## Latex分栏
- 单栏 <=> 双栏
- `\documentclass[onecolumn, draftcls]{IEEEtran}`
- `\documentclass[journal]{IEEEtran}`

# 图片与表格
## 图片

- 调用宏包 （导言部分添加`\usepackage{graphicx}`)
- figure环境举例
	- ![[Pasted image 20251110110826.png]]
- 插入图片文件格式
	- Latex图片插入非一般类型，eps格式，矢量图，普通的图效果不好
	- 一般不可强制指定图片插入位置，Latex根据排版要求自动规划
- 给图片加标题且编号
	- 在插图指令后面加入`\caption{xxx}`，图注前自动生成标号
## 表格
- 制表
	- `\begin{tabular}[pos]{table spec}` pos描述表格与周围文本的竖直位置关系，table spec描述表格样式
	- ![[Pasted image 20251110111750.png]]
	- `&`分割列，`\\`换行
	- ![[Pasted image 20251110112000.png]]

# 数学公式
可以通过在线转换实现复杂的，不必太深究
[[8.LaTeX介绍与使用.pdf]]

# 参考文献
![[Pasted image 20251110112146.png]]
