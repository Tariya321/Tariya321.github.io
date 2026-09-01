---
title: "CIMFormer"
tags:
  - transformer
  - paper
parent: "CIMFormer: A Systolic CIM-Array-Based Transformer Accelerator With Token-Pruning-Aware Attention Reformulating and Principal Possibility Gathering"
publish: yes
---
研究背景
---
transformer模型的基本组成

![\<img alt="" data-attachment-key="K7E7SJNY" data-annotation="%7B%22attachmentURI%22%3A%22http%3A%2F%2Fzotero.org%2Fusers%2F11606558%2Fitems%2FFDMNJUVG%22%2C%22annotationKey%22%3A%22QGLHTDKW%22%2C%22color%22%3A%22%23ffd400%22%2C%22pageLabel%22%3A%223318%22%2C%22position%22%3A%7B%22pageIndex%22%3A1%2C%22rects%22%3A%5B%5B47.33052400287829%2C566.147%2C299.709%2C738.9473684210526%5D%5D%7D%2C%22citationItem%22%3A%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F11606558%2Fitems%2F6S9Q38VF%22%5D%2C%22locator%22%3A%223318%22%7D%7D" width="421" height="288" src="attachment/K7E7SJNY.png" ztype="zimage">|698](/attachment/K7E7SJNY.png)

“In contrast to RNNs and CNNs, which process input data sequentially or with fixed-size filters, the MHSA captures contextual information from the entire token sequence, as depicted in Fig. 1. This allows for a more comprehensive understanding of the input. While the Transformer model outperforms CNNs in terms of accuracy, it does so at the cost of a higher number of model parameters and computations due to its processing of all input tokens.” <span class="citation" data-citation="%7B%22citationItems%22%3A%5B%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F11606558%2Fitems%2F6S9Q38VF%22%5D%2C%22locator%22%3A%223317%22%7D%5D%2C%22properties%22%3A%7B%7D%7D" ztype="zcitation">(<span class="citation-item"><a href="zotero://select/library/items/6S9Q38VF">Guo 等, 2024, p. 3317</a></span>)</span> 🔤与顺序处理输入数据或使用固定大小的过滤器处理输入数据的 RNN 和 CNN 相比，MHSA 从整个标记序列中捕获上下文信息，如图 1 所示。这可以更全面地理解输入。虽然 Transformer 模型在准确性方面优于 CNN，但由于要处理所有输入标记，因此需要更多的模型参数和计算量。🔤

local-attention，fixed attention span

*   对于某task而言，span可能较小或者较大，从而冗余或者精度损失
*   对于CV任务而言，token更多更复杂

token pruning，剪枝掉注意力机制不集中的token，从而在几乎不影响精度的情况下提升efficiency

transformer CIM的三个主要挑战

![\<img alt="" data-attachment-key="IQ5KY78C" data-annotation="%7B%22attachmentURI%22%3A%22http%3A%2F%2Fzotero.org%2Fusers%2F11606558%2Fitems%2FFDMNJUVG%22%2C%22annotationKey%22%3A%22LS6S32W4%22%2C%22color%22%3A%22%23ffd400%22%2C%22pageLabel%22%3A%223319%22%2C%22position%22%3A%7B%22pageIndex%22%3A2%2C%22rects%22%3A%5B%5B48.0884187397204%2C521.204%2C563.078%2C737.962%5D%5D%7D%2C%22citationItem%22%3A%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F11606558%2Fitems%2F6S9Q38VF%22%5D%2C%22locator%22%3A%223319%22%7D%7D" width="858" height="361" src="attachment/IQ5KY78C.png" ztype="zimage">|845](/attachment/IQ5KY78C.png)

结果对比
---

![\<img alt="" data-attachment-key="YTFTS5T8" data-annotation="%7B%22attachmentURI%22%3A%22http%3A%2F%2Fzotero.org%2Fusers%2F11606558%2Fitems%2FFDMNJUVG%22%2C%22annotationKey%22%3A%22B524H5SR%22%2C%22color%22%3A%22%23ffd400%22%2C%22pageLabel%22%3A%223327%22%2C%22position%22%3A%7B%22pageIndex%22%3A10%2C%22rects%22%3A%5B%5B46.193681897615136%2C256.244%2C563.457%2C513.9284256784539%5D%5D%7D%2C%22citationItem%22%3A%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F11606558%2Fitems%2F6S9Q38VF%22%5D%2C%22locator%22%3A%223327%22%7D%7D" width="862" height="429" src="attachment/YTFTS5T8.png" ztype="zimage">|845](/attachment/vault/99_archive/Zotero/attachment/YTFTS5T8.png)

