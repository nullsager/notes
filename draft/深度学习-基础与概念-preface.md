#ai #读书笔记

Deep learning uses multilayered neural networks trained with large data sets to
solve complex information processing tasks and has emerged as the most successful paradigm in the field of machine learning.

深度学习使用多层神经网络，并借助大型数据集的训练来解决复杂的信息处理任务，并且已经成为机器学习领域最成功的范式


> [!NOTE] 
> 注意 with large data sets 以及 has emerged 的翻译


Over the last decade, deep learning has revolutionized many domains including computer vision, speech recognition, and natural language processing, and it is being used in a growing multitude of applications across healthcare, manufacturing, commerce, finance, scientific discovery, and many other sectors. 

在过去 10 年，深度学习已经革新了许多领域，包括计算机视觉，语音识别，以及自然语言处理，并且它正在被应用于越来越多的领域，包括医疗保健，制造业，商业，金融，科学探索，以及其他的行业。


> [!NOTE] 
> a growing multitude of 译为越来越多的


Recently, massive neural networks, known as large language
models and comprising of the order of a trillion learnable parameters, have been
found to exhibit the first indications of general artificial intelligence and are now
driving one of the biggest disruptions in the history of technology.

近期，被称为大语言模型的超大型神经网络，包含高达万亿数量级的可学习参数，已经显示出通用人工智能的初步迹象，它们正在引发技术史上的一次最大的颠覆


> [!NOTE]
> known as 这里翻译为被称为，comprise 译为组成，包含，the order of 译为... 的数量级，exhibit 这里译为显示出，first indications 译为初步迹象，disruptions 译为颠覆

## Goals of the book

This expanding impact has been accompanied by an explosion in the number
and breadth of research publications in machine learning, and the pace of innovation continues to accelerate. 

这种不断扩大的影响伴随着机器学习研究出版物数量和广度的爆炸式增长，创新的步伐也在不断加快。


> [!NOTE] 
> expanding impact 译为不断扩大的影响，accompany by 译为伴随着，breadth 译为广度，the pace of 译为... 的步伐

For newcomers to the field, the challenge of getting
to grips with the key ideas, let alone catching up to the research frontier, can seem
daunting. 

对于这一领域的新人来说，仅是掌握核心思想就已经足够令人畏惧，更不用说赶上研究前沿。


> [!NOTE]
> get to grep with 掌握，let alone 更不用说，catch up 赶上，frontier 前沿，daunting 使人畏惧/气馁的

Against this backdrop, Deep Learning: Foundations and Concepts aims
to provide newcomers to machine learning, as well as those already experienced in the field, with a thorough understanding of both the foundational ideas that underpin deep learning as well as the key concepts of modern deep learning architectures and techniques. 

在这种背景下，本书将帮助机器学习的新手，以及那些已经在这一领域有经验的人深入了解支撑深度学习的基本思想以及现代深度学习架构和技术的关键概念。

> [!NOTE] 
> against this backdrop 在这种背景下，underpin 支撑

This material will equip the reader with a strong basis for future specialization. 

这些材料将为读者将来的专业化打下坚实的基础。

Due to the breadth and pace of change in the field, we have deliberately
avoided trying to create a comprehensive survey of the latest research. Instead, much of the value of the book derives from a distillation of key ideas, and although the field itself can be expected to continue its rapid advance, these foundations and concepts are likely to stand the test of time. 

鉴于深度学习领域知识的广度和变化速度，我们有意避免写一本涵盖最新研究的全面综述。相反，这本书的大部分价值来自对关键思想的提炼，尽管这个领域本身有望继续快速发展，但这些基础和概念很可能经受住时间的考验。

> [!NOTE] 
> comprehensive survey 全面综述，derive from 来自 distillation 提炼 rapid advance 快速发展 

For example, large language models have been evolving very rapidly at the time of writing, yet the underlying transformer architecture and attention mechanism have remained largely unchanged for the last five years, while many core principles of machine learning have been known for decades.

例如，在撰写本书时，大预言模型正在迅速演进，然而其底层的 Transformer 架构和注意力机制在过去 5 年基本保持不变，并且机器学习的许多核心原则已被人们熟知数十年。

> [!NOTE] 
> mechanism 机制

## Responsible use of technology

Deep learning is a powerful technology with broad applicability that has the potential to create huge value for the world and address some of society’s most pressing challenges. 

深度学习是一项功能强大、适用范围广泛的技术，具有为世界创造巨大价值和应对社会最紧迫挑战的潜力。

However, these same attributes mean that deep learning also has potential both for deliberate misuse and to cause unintended harms. 

然而，这些相同的属性也意味着深度学习技术可能会被故意滥用并造成意外伤害


> [!NOTE] 
> attribute 特点，属性

We have chosen not to discuss ethical or societal aspects of the use of deep learning, as these topics are of such importance and complexity that they warrant a more thorough treatment than is possible in a technical textbook such as this. 

我们选择不去讨论深度学习使用中的伦理和社会层面的问题，因为这些话题非常的重要和复杂，它们需要比在这样的技术教科书中更彻底的处理。


> [!NOTE] 
> warrant 需要，理应得到


Such considerations should, however, be informed by a solid grounding in the underlying technology and how it works, and so we hope that this book will make a valuable contribution towards these important discussions. 

然而，这些考虑应该通过对底层技术及其工作原理的坚实基础来了解，因此我们希望本书将对这些重要的讨论做出有价值的贡献。


> [!NOTE] 
> be informed by 通过... 了解 grounding 基础 underlying 根本的，潜在的


The reader is, nevertheless, strongly encouraged to be mindful about the broader implications of their work and to learn about the responsible use
of deep learning and artificial intelligence alongside their studies of the technology itself.

尽管如此，强烈建议读者注意他们工作的更广泛含义，并在研究技术本身的同时，学习如何负责任地使用深度学习和人工智能。


> [!NOTE] 
> implication 含义 be mindful about 注意 alongside 同时

## Structure of the book

The book is structured into a relatively large number of smaller bite-sized chapters, each of which explores a specific topic.

这本书由相对大量的小章节组成，每个章节都探讨了一个特定的主题。

The book has a linear structure in the sense that each chapter depends only on material covered in earlier chapters. 

这本书的结构是线性的，因为每一章都只依赖于前几章的内容。


> [!NOTE] 
> in the sense that 即，也就是说

It is well suited to teaching a two-semester undergraduate or postgraduate course on machine learning but is equally relevant to those engaged in active research or in self-study.

它非常适合教授两个学期的机器学习本科或研究生课程，但同样适用于那些从事积极研究或自学的人。


> [!NOTE] 
> be relevant to 与... 有关 engage in 从事


A clear understanding of machine learning can be achieved only through the
use of some level of mathematics. 

对机器学习的清晰理解只能通过使用一定程度的数学来实现。

Specifically, three areas of mathematics lie at the heart of machine learning: probability theory, linear algebra, and multivariate calculus. 

具体来说，机器学习的核心由三个数学领域构成：概率论、线性代数和多元微积分（也称多变量微积分）​。

The book provides a self-contained introduction to the required concepts in
probability theory and includes an appendix that summarizes some useful results in linear algebra. 

本书提供了概率论所需概念的独立介绍，并包括一个附录，总结了线性代数中一些有用的结果。

It is assumed that the reader already has some familiarity with the
basic concepts of multivariate calculus although there are appendices that provide
introductions to the calculus of variations and to Lagrange multipliers. The focus
of the book, however, is on conveying a clear understanding of ideas, and the emphasis is on techniques that have real-world practical value rather than on abstract
theory. Where possible we try to present more complex concepts from multiple complementary perspectives including textual description, diagrams, and mathematical
formulae. In addition, many of the key algorithms discussed in the text are summarized in separate boxes. These do not address issues of computational efficiency, but
are provided as a complement to the mathematical explanations given in the text.
We therefore hope that the material in this book will be accessible to readers from a
variety of backgrounds.