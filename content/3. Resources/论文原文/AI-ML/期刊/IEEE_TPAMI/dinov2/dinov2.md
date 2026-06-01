---
abstract: |
  The recent breakthroughs in natural language processing for model
  pretraining on large quantities of data have opened the way for
  similar foundation models in computer vision. These models could
  greatly simplify the use of images in any system by producing
  general-purpose visual features, i.e., features that work across image
  distributions and tasks without finetuning. This work shows that
  existing pretraining methods, especially self-supervised methods, can
  produce such features if trained on enough curated data from diverse
  sources. We revisit existing approaches and combine different
  techniques to scale our pretraining in terms of data and model size.
  Most of the technical contributions aim at accelerating and
  stabilizing the training at scale. In terms of data, we propose an
  automatic pipeline to build a dedicated, diverse, and curated image
  dataset instead of uncurated data, as typically done in the
  self-supervised literature. In terms of models, we train a ViT
  model [@dosovitskiy2020image] with 1B parameters and distill it into a
  series of smaller models that surpass the best available
  general-purpose features, OpenCLIP [@ilharco_gabriel_2021_5143773] on
  most of the benchmarks at image and pixel levels.
author:
- |
  Maxime Oquab$^{**}$, Timothée Darcet$^{**}$, Théo Moutakanni$^{**}$,\
  Huy V. Vo$^*$, Marc Szafraniec$^*$, Vasil Khalidov$^*$, Pierre
  Fernandez, Daniel Haziza,\
  Francisco Massa, Alaaeldin El-Nouby, Mahmoud Assran, Nicolas Ballas,
  Wojciech Galuba,\
  Russell Howes, Po-Yao Huang, Shang-Wen Li, Ishan Misra, Michael
  Rabbat,\
  Vasu Sharma, Gabriel Synnaeve, Hu Xu, Hervé Jegou, Julien Mairal$^1$,\
  Patrick Labatut$^*$, Armand Joulin$^*$, Piotr Bojanowski$^*$\
  $\;\;\;\;\;\;\;\;$\
  $^*$core team $\;\;\;\;\;\;\;$$^{**}$equal contribution
bibliography:
- egbib.bib
title: |
  DINOv2: Learning Robust Visual Features\
  without Supervision
publish: true
---

# Introduction {#sec:intro}

Learning task-agnostic pretrained representations have become the
standard in Natural Language
Processing (NLP) [@radford2019language; @raffel2020exploring; @chowdhery2022palm; @hoffmann2022training; @touvron2023llama].
One can use these features "as they are", i.e., without fine-tuning, and
achieve performances on downstream tasks that are significantly better
than those produced by task-specific models [@brown2020language]. This
success has been fueled by pretraining on large quantities of raw text
using pretext objectives, such as language
modeling [@radford2017learning] or word vectors [@devlin2018bert], that
require no supervision.

Following this paradigm shift in NLP, we expect similar "foundation"
models to appear in computer vision [@bommasani2021opportunities]. These
models should generate visual features that work out of the box on any
task, both at the image level, e.g., image classification, and pixel
level, e.g., segmentation. Most promising efforts towards these
foundation models focus on text-guided pretraining, i.e., using a form
of textual supervision to guide the training of the
features [@joulin2016learning; @mahajan2018exploring; @radford2021learning].
This form of text-guided pretraining limits the information that can be
retained about the image since captions only approximate the rich
information in images, and complex pixel-level information may not
surface with this supervision. Furthermore, these image encoders require
aligned text-image corpora and hence, do not offer the flexibility of
their text counterparts, that is, to learn from raw data alone.

::: figure*
![image](new-figure-1.jpg){width="\\linewidth"}
:::

An alternative to text-guided pretraining is self-supervised learning
 [@caron2018deep; @chen2020simple; @he2021masked] where features are
learned from images alone. These approaches are conceptually closer to
pretext tasks such as language modeling and can capture information at
the image and pixel level [@caron2021emerging]. Additionally, the
features output by self-supervised models have been shown to exhibit
various useful properties, and have enabled enabled a wide variety of
applications [@amir2021deep; @tumanyan2022splicing; @ofri2023neural; @hamilton2022unsupervised].
However, despite their potential to learn general-purpose features, most
of the advances in self-supervised learning were made in the context of
pretraining on a small curated dataset,
ImageNet-1k [@russakovsky2015imagenet]. Some efforts on scaling these
approaches beyond ImageNet-1k have been
attempted [@caron2019unsupervised; @goyal2021self; @goyal2022vision],
but they focused on uncurated datasets, which typically lead to a
significant drop in the quality of the features. This is explained by
the lack of control over the data quality and diversity, which are
essential to produce good features.

In this work, we explore if self-supervised learning has the potential
to learn general-purpose visual features if pretrained on a large
quantity of curated data. We revisit existing discriminative
self-supervised approaches that learn features at both the image and
patch level, such as iBOT [@zhou2021ibot], and we reconsider some of
their design choices under the lens of a larger dataset. Most of our
technical contributions are tailored toward stabilizing and accelerating
discriminative self-supervised learning when scaling in model and data
sizes. These improvements make our approach around 2$\times$ faster and
require 3$\times$ less memory than similar discriminative
self-supervised methods, allowing us to leverage longer training with
larger batch sizes.

Regarding pretraining data, we have built an automatic pipeline to
filter and rebalance datasets from an extensive collection of uncurated
images. This pipeline is inspired by pipelines used in
NLP [@wenzek2019ccnet], where data similarities are used instead of
external metadata and do not require manual annotation. A major
difficulty when dealing with images in the wild is to rebalance concepts
and avoid overfitting on a few dominant modes. In this work, a naive
clustering approach works reasonably well to resolve this issue. We
gathered a small but diverse corpus of 142M images to validate our
approach.

Finally, we provide a variety of pretrained visual models, called
DINOv2, trained with different Vision
Transformers (ViT) [@dosovitskiy2016discriminative] architectures on our
data. We release all the models and the code to retrain DINOv2 on any
data. We validate the quality of DINOv2 on various computer vision
benchmarks at both image and pixel levels as we scale them, as
summarized in Fig. [1](#tab:pullfig){reference-type="ref"
reference="tab:pullfig"}. We conclude that self-supervised pretraining
alone is a good candidate for learning transferable frozen features that
are competitive with the best openly available weakly-supervised models.

# Related Work

#### Intra-image self-supervised training.

A first family of self-supervised methods focuses on pretext tasks built
from the image, i.e., extracting a signal from the image to be predicted
from the rest of the image. This idea has become prevalent with the work
of [@doersch2015unsupervised], where they train by predicting the
context of a given patch. Many other pretext tasks were introduced based
on, for example, re-colorizing images [@zhang2016colorful], predicting
transformations [@gidaris2018unsupervised],
inpainting [@pathakCVPR16context] or patch
re-ordering [@noroozi2016jigsaw; @misra2020self]. Recently, the
emergence of patch-based architectures, like ViTs, has led to a revisit
of inpainting for
pre-training [@he2021masked; @bao2021beit; @el2021large], potentially in
feature space [@assran2023self; @baevski2022data2vec]. Of particular
interest, [@he2021masked] show that a masked auto-encoder (MAE) learns
features that provide substantial improvements when finetuned on
downstream tasks. This property of MAEs has been further validated on
video [@tong2022videomae], audio [@xu2022masked], and across other
modalities [@girdhar2022omnimae]. However, their features require
supervised finetuning, while our features perform well out of the box.

![ **Evolution of performance when scaling in parameters.** We show
performance on eight types of vision tasks, as presented in
Sec. [7](#sec:results){reference-type="ref" reference="sec:results"},
and average metrics with each type. Features are extracted from our
self-supervised encoders, DINOv2 (dark blue), and we compare them with
self-supervised methods (pale orange), as well as weakly-supervised
methods (dark pink). We report the best-performing weakly-supervised
model's performance as a dashed horizontal line. Our family of models
drastically improves over the previous state of the art in
self-supervised learning and reaches performance comparable with
weakly-supervised features. See
Sec. [7](#sec:results){reference-type="ref" reference="sec:results"} for
a detailed analysis. []{#tab:pullfig label="tab:pullfig"}
](pullfigure_5.pdf){#tab:pullfig}

#### Discriminative self-supervised learning.

The second line of work, closer to ours, is using discriminative signals
between images or groups of images to learn features. This family of
methods has roots in early deep learning
work [@hadsell2006dimensionality] but became popular with the emergence
of instance classification
methods [@dosovitskiy2016discriminative; @bojanowski2017unsupervised; @wu2018unsupervised].
Several improvements were made based either on instance-level
objectives [@henaff2019data; @he2020momentum; @chen2020exploring; @chen2020simple; @grill2020bootstrap; @caron2021emerging]
or clustering [@caron2018deep; @asano2019self; @caron2020unsupervised].
These methods provide performant frozen features on standard benchmarks
like ImageNet [@russakovsky2015imagenet], but they are hard to scale to
larger model sizes [@chen2021empirical]. In this work, we revisit the
training of these approaches in the context of large pretraining
datasets and models. In particular, we build on top of [@zhou2021ibot]
that we find particularly suited for scaling.

#### Scaling self-supervised pretraining.

A growing body of work has focused on the scaling abilities of
self-supervised learning in terms of data and model
size [@caron2019unsupervised; @goyal2019scaling; @tian2021divide; @goyal2022vision].
Most of these works use large quantities of uncurated data to train
models without supervision. They show evidence that discriminative
methods scale with data, but because of the poor quality of the
pretraining data, most of the results are obtained by finetuning the
features. Of particular interest, [@goyal2021self] have also shown that
these methods benefit from scaling in model size given enough pretrained
data. This line of work questions the ability of self-supervised methods
to work on any data while we focus on producing the best pretrained
encoders.

#### Automatic data curation.

Our dataset construction borrows from the image retrieval
community [@weinzaepfel2021learning; @radenovic2018fine; @berman2019multigrain; @douze2009evaluation; @tolias2015particular; @revaud2019learning].
In particular, the use of retrieval to augment the training set has been
studied in the context of semi-supervised learning [@yalniz2019billion].
Similarly, others have used hashtags or other
metadata [@mahajan2018exploring; @radford2021learning] or pretrained
vision encoders [@schuhmann2021laion; @schuhmann2022laion] to filter
uncurated datasets. Unlike these works, we use no pretrained encoders,
metadata nor supervision to filter images and leverage visual similarity
between images. Our approach is inspired by text curation
pipelines [@wenzek2019ccnet], where a language model is trained on
Wikipedia to score texts extracted from an uncurated source.

# Data Processing {#sec:data_processing}

We assemble our curated LVD-142M dataset by retrieving, from a large
pool of uncurated data, images that are close to those in several
curated datasets. We describe below the main components in our data
pipeline including the curated/uncurated data sources, the image
deduplication step and the retrieval system. Our pipeline does not
require any metadata or text and directly works with images, as shown in
Fig. [2](#fig:retrieval-system){reference-type="ref"
reference="fig:retrieval-system"}. We refer the reader to appendix
[11](#sec:supp-data){reference-type="ref" reference="sec:supp-data"} for
more details on our approach.

![ **Overview of our data processing pipeline.** Images from curated and
uncurated data sources are first mapped to embeddings. Uncurated images
are then deduplicated before being matched to curated images. The
resulting combination augments the initial dataset through a
self-supervised retrieval system.
](LaViDa_datapipeline_figure.pdf){#fig:retrieval-system}

**Data sources.** Our selection of curated datasets is detailed in the
appendix (Table
[\[tab:lavida-details\]](#tab:lavida-details){reference-type="ref"
reference="tab:lavida-details"}) and contains ImageNet-22k, the train
split of ImageNet-1k, Google Landmarks and several fine-grained
datasets. For the uncurated data source, we collect a raw unfiltered
dataset of images from a publicly available repository of crawled web
data. From each web page in the repository, we extract URL links of
images from `<img>` tags. We discard URLs that are unsafe or restricted
by domains, and post-process the downloaded images (PCA hash
deduplication, NSFW filtering, and blurring identifiable faces). This
results in 1.2B unique images.

**Deduplication.** We apply the copy detection pipeline
of [@pizzi2022self] to the uncurated data and remove near-duplicate
images. This reduces redundancy and increases diversity among images. We
also remove near-duplicates of images contained in the test or
validation set of any benchmark used in this work.

**Self-supervised image retrieval.** We build our curated pretraining
dataset by retrieving images from our uncurated data source that are
close to images in our curated sources. In order to do this, we first
compute an image embedding using a self-supervised ViT-H/16 network
pretrained on ImageNet-22k, and use cosine-similarity as a distance
measure between images. Then, we perform k-means clustering of the
uncurated data. Given a query dataset for retrieval, if it is large
enough we retrieve $N$ (typically 4) nearest neighbors for each query
image. If it is small, we sample $M$ images from the cluster
corresponding to each query image. Although visual inspection seemed to
indicate good retrieval quality for $N$ much larger than 4, this leads
to more collisions (images that are nearest-neighbor retrievals of
multiple queries). We choose $N=4$ as it provides a good tradeoff in
that sense.

**Implementation Details.** The deduplication and retrieval stages of
our pipeline rely on the Faiss library [@johnson2019billion] to
efficiently index and compute batch searches of nearest embeddings. In
particular, we heavily leverage its support for GPU-accelerated indices,
using inverted file indices with product quantization
codes [@jegou2010product]. The whole processing is distributed on a
compute cluster of 20 nodes equipped with 8 V100-32GB GPUs and takes
less than two days to produce the LVD-142M dataset.

# Discriminative Self-supervised Pre-training {#sec:approach}

We learn our features with a discriminative self-supervised method that
can be seen as a combination of DINO and iBOT losses with the centering
of SwAV [@caron2020unsupervised]. We also add a regularizer to spread
features and a short high-resolution training phase. We rapidly
introduce each of these approaches, but more details can be found in the
related papers, or in our open-sourced code.

-   **Image-level objective [@caron2021emerging].** We consider the
    cross-entropy loss between the features extracted from a student and
    a teacher network. Both features are coming from the class token of
    a ViT, obtained from different crops of the same image.

    We pass the student class token through the student DINO head. This
    head is an MLP model outputting a vector of scores, that we call
    \"prototype scores\". We then apply a softmax to obtain $p_s$.
    Similarly, we apply the teacher DINO head to the teacher class token
    to obtain teacher prototype scores. We then apply a softmax followed
    by a centering with moving average (or a Sinkhorn-Knopp centering as
    detailed thereafter) to obtain $p_t$. The DINO loss term corresponds
    to: $${\mathcal L}_{DINO} = - \sum p_t \log p_s$$

    We learn the parameters of the student and build the teacher head
    with an exponential moving average of past
    iterates [@he2020momentum].

-   **Patch-level objective [@zhou2021ibot].** We randomly mask some of
    the input patches given to the student, but not to the teacher.

    We then apply the student iBOT head to the student mask tokens.
    Similarly, we apply the teacher iBOT head to the (visible) teacher
    patch tokens corresponding to the ones masked in the student. We
    then apply the softmax and centering steps as above, and obtain the
    iBOT loss term:
    $${\mathcal L}_{iBOT} = - \sum_i p_{ti} \log p_{si}$$, where $i$ are
    patch indices for masked tokens. Similarly to above, we learn the
    parameters of the student, and build the teacher head through
    exponential moving average.

-   **Untying head weights between both objectives.**

    Both the DINO and the iBOT loss use a learnable MLP projection head.
    It is applied to the output tokens and the loss is compute atop. In
    [@zhou2021ibot], an ablation study shows that sharing parameters
    between the DINO and iBOT heads leads to better performance. At
    scale, we observed that the opposite is true, and we therefore use
    two separate heads in all our experiments.

-   **Sinkhorn-Knopp centering [@caron2020unsupervised].**
    [@ruan2022weighted] recommend to replace the teacher
    softmax-centering step of DINO and iBot by the Sinkhorn-Knopp (SK)
    batch normalization of SwAV [@caron2020unsupervised]. We run the
    Sinkhorn-Knopp algorithm steps for 3 iterations. For the student, we
    apply the softmax normalization.

-   **KoLeo regularizer  [@sablayrolles2018spreading].** The KoLeo
    regularizer derives from the Kozachenko-Leonenko differential
    entropy estimator (see  @beirlant1997nonparametric
    [@delattre2017kozachenko]) and encourages a uniform span of the
    features within a batch. Given a set of $n$ vectors
    $(x_1,\dots,x_n)$, it is defined as
    $${\mathcal L}_{\mathrm{koleo}} = - \frac{1}{n} \sum_{i=1}^n \log( d_{n, i}),$$
    where $d_{n, i} = \min_{j \neq i} \| x_i - x_j \|$ is the minimum
    distance between $x_i$ and any other point within the batch. We also
    $\ell_2$-normalize the features before computing this regularizer.

-   **Adapting the resolution [@touvron2019fixing].** Increasing image
    resolution is key to pixel-level downstream tasks such as
    segmentation or detection, where small objects disappear at low
    resolutions. However, training at high resolution is time and memory
    demanding, and instead, we increase the resolution of images to
    $518\times 518$ during a short period at the end of pretraining.
    This is also similar to UniViT training from [@likhomanenko2021cape]
    and FlexiViT training from [@beyer2023flexivit].

# Efficient implementation {#sec:optim}

We consider several improvements to train models at a larger scale. We
train models on A100 GPUs using PyTorch 2.0. The code and pretrained
models are made available under Apache 2.0 license [^1]. The details of
our models are in the appendix,
Table [6](#tab:vit-hparams){reference-type="ref"
reference="tab:vit-hparams"}. With the same hardware, compared to the
iBOT implementation, the DINOv2 code runs around $2\times$ faster using
only $1/3$ of the memory.

#### Fast and memory-efficient attention.

We implemented our own version of
FlashAttention [@dao2022flashattention] to improve memory usage and
speed on the self-attention layers. Our version is on par with or better
than the original on all cases considered, while covering more use-cases
and hardware. Due to the GPU hardware specifics, the efficiency is best
when the embedding dimension per head is a multiple of 64, and the
matrix operations are even better when the full embedding dimension is a
multiple of 256. As a consequence, our ViT-g architecture slightly
differs from the architecture proposed by [@zhai2022scaling] in order to
maximize compute efficiency, and we use an embedding dimension of 1536
with 24 heads (64 dim/head), rather than 1408 with 16 heads (88
dim/head). Our experiments did not show significant differences in final
accuracy, and our ViT-g backbone counts 1.1B parameters.

#### Sequence packing.

The DINO algorithm requires forwarding both large crops (at resolution
224) and small crops (resolution 98). When split into patches, these two
groups are represented by token sequences of different lengths and
cannot be forwarded together. In order to accelerate training, we use a
trick called \"sequence packing,\" which originates from NLP
[@krell2022efficient]. The idea is simple: we concatenate the sequences
we must forward through the transformers into a single long sequence. We
pass this sequence through the transformer blocks as usual. However, a
block-diagonal mask is applied to the self-attention matrix in attention
layers, preventing attention between different sequences. This way, the
forward is strictly equivalent to forwarding each sequence separately.
This trick gives us significant compute efficiency gains compared to
using separate forward and backward passes, as in prior implementations.
The lower-level components of our setup are available in the xFormers
library[^2] ([@xFormers2022]).

#### Efficient stochastic depth.

We implement an improved version of stochastic depth [@huang2016deep]
that skips the computation of the dropped residuals rather than masking
the result. This saves memory and compute in proportion approximately
equal to the drop rate, thanks to specific fused kernels. With high drop
rates ($d = 40\%$ in this work), this allows a drastic improvement in
compute efficiency and memory usage. The implementation consists of
randomly shuffling the $B$ samples over the batch dimension, and slicing
the first $(1-d) \times B$ samples for the computations in the block.

#### Fully-Sharded Data Parallel (FSDP).

Minimizing our objective with the AdamW optimizer requires 4 model
replicas in float32 precision -- student, teacher, optimizer first
moments, optimizer second moments. This sums to $16~\mathrm{GB}$ of
memory for a billion-parameter model such as our ViT-g. In order to
reduce this memory footprint per GPU, we split the model replicas across
GPUs, i.e., sharding $16~\mathrm{GB}$ across GPUs using the PyTorch
implementation of FSDP. Consequently, the model size is not bounded by
the memory of a single GPU but by the total sum of GPU memory across
compute nodes. The Pytorch implementation of FSDP brings a second
advantage, which is to save on the cross-GPU communication costs: the
weight shards are stored in float32 precision as required by the
optimizer, but broadcasting weights and reducing gradients is done in
float16 precision for the backbone (MLP heads gradients are reduced in
float32 to avoid training instabilities). This leads to approximately
50% reduction in communication costs compared to the float32 gradient
all-reduce operation used in DistributedDataParallel (DDP), which is
used in other self-supervised pretraining
methods [@caron2021emerging; @zhou2021ibot]. As a consequence, the
training procedure scales more efficiently than DDP with float16
autocast when scaling the number of GPU nodes. Overall, Pytorch-FSDP
mixed-precision is superior to DDP with autocast in virtually all cases
we encountered.

#### Model distillation. {#sec:distill}

Most of our technical improvements to the training loop aim at improving
the training of large models over large quantities of data. For smaller
models, we distill them from our largest model, the ViT-g, instead of
training them from scratch. Knowledge
distillation [@hinton2015distilling] aims at reproducing the output of a
large model with a smaller model by minimizing some distance between
both outputs for a set of given inputs. Since our objective function is
a form of distillation from the teacher network to the student network,
we leverage the same training loop with a few exceptions: we use a
larger model as a frozen teacher, keep a spare EMA of the student that
we use as our final model, remove the masking and stochastic depth, and,
apply the iBOT loss on the two global crops. In our ablations, we
observe that this approach achieves better performance than training
from scratch, even for a ViT-L. Our distillation method ends up close to
the one described by [@duval2023simple], except we do not modify the
loss terms for distillation and evaluate the EMA of the student.

# Ablation Studies

We present a set of ablations to empirically validate different
components of our pipeline: the technical modifications described in
Sec. [4](#sec:approach){reference-type="ref" reference="sec:approach"},
the pretraining data and the impact of model distillation. We consider
various downstream tasks that are described in
Sec. [7](#sec:results){reference-type="ref" reference="sec:results"}.

## Improved Training Recipe {#sec:ablation-ibot}

Our approach improves over the iBOT method by combining it with several
existing components described in
Sec. [4](#sec:approach){reference-type="ref" reference="sec:approach"}.
To evaluate their importance, we train multiple models where we
successively add components to a baseline iBOT model. We report the
Top-1 accuracy on the validation set of ImageNet-1k with a k-NN and a
linear probe in Table [1](#tab:ibot-dino){reference-type="ref"
reference="tab:ibot-dino"}. Generally, we observe that each component
improves the performance on either k-NN or linear probing and even both
in most cases. Only LayerScale and Stochastic Depth incur a performance
drop in linear probing but significantly improve the training stability
in our experience.

::: {#tab:ibot-dino}
                                   INet-1k k-NN                                      INet-1k linear
  -------------------------------- ------------------------------------------------- -------------------------------------------------
  iBOT                             72.9                                              82.3
   +(our reproduction)             74.5 [$\uparrow 1.6$]{style="color: IncrGreen"}   83.2 [$\uparrow 0.9$]{style="color: IncrGreen"}
   +LayerScale, Stochastic Depth   75.4 [$\uparrow 0.9$]{style="color: IncrGreen"}   82.0 [$\downarrow 1.2$]{style="color: DecrRed"}
   +128k prototypes                76.6 [$\uparrow 1.2$]{style="color: IncrGreen"}   81.9 [$\downarrow 0.1$]{style="color: DecrRed"}
   +KoLeo                          78.9 [$\uparrow 2.3$]{style="color: IncrGreen"}   82.5 [$\uparrow 0.6$]{style="color: IncrGreen"}
   +SwiGLU FFN                     78.7 [$\downarrow 0.2$]{style="color: DecrRed"}   83.1 [$\uparrow 0.6$]{style="color: IncrGreen"}
   +Patch size 14                  78.9 [$\uparrow 0.2$]{style="color: IncrGreen"}   83.5 [$\uparrow 0.4$]{style="color: IncrGreen"}
   +Teacher momentum 0.994         79.4 [$\uparrow 0.5$]{style="color: IncrGreen"}   83.6 [$\uparrow 0.1$]{style="color: IncrGreen"}
   +Tweak warmup schedules         80.5 [$\uparrow 1.1$]{style="color: IncrGreen"}   83.8 [$\uparrow 0.2$]{style="color: IncrGreen"}
   +Batch size 3k                  81.7 [$\uparrow 1.2$]{style="color: IncrGreen"}   84.7 [$\uparrow 0.9$]{style="color: IncrGreen"}
   +Sinkhorn-Knopp                 81.7 $=$                                          84.7 $=$
   +Untying heads = DINOv2         82.0 [$\uparrow 0.3$]{style="color: IncrGreen"}   84.5 [$\downarrow 0.2$]{style="color: DecrRed"}

  :  **Ablation study of the training differences between iBOT and
  DINOv2.** We optimize for k-NN performance, as in our experience, the
  linear probe performance is lower-bounded by the k-NN performance.
  Some modifications, like LayerScale and a high Stochastic Depth
  (rate=$0.4$), incur a decrease in linear probe performance, but have
  the benefits of increasing the stability of training by avoiding NaN
  loss values during training [@touvron2022deit]. Overall, these
  modifications allowed for the next set of improvements to be added.
  Experiments are run using the ViT-Large architecture on ImageNet-22k.
:::

## Pretraining Data Source

The quality of features is directly related to the quality of the
pretraining data. In this experiment, we probe the impact of LVD-142M
compared to ImageNet-22k, a commonly used pretraining dataset, or using
directly raw and uncurated data. For the uncurated dataset, we randomly
sample $142$ million images from the same data source as LVD-142M. We
train a ViT-g/14 on each dataset for the same number of iterations. We
also include a variant of ImageNet-22k obtained by removing the synsets
of ImageNet-1k (INet-22k $\setminus$ INet-1k) for completeness. We
report the comparisons in
Table [2](#tab:ablation-data){reference-type="ref"
reference="tab:ablation-data"}.

::: {#tab:ablation-data}
  Training Data                      INet-1k      Im-A     ADE-20k    Oxford-M   iNat2018   iNat2021   Places205
  ------------------------------ -- ---------- ---------- ---------- ---------- ---------- ---------- -----------
  INet-22k                           **85.9**     73.5       46.6       62.5       81.1       85.6       67.0
  INet-22k $\setminus$ INet-1k         85.3       70.3       46.2       58.7       80.1       85.1       66.5
  Uncurated data                       83.3       59.4       48.5       54.3       68.0       76.4       67.2
  LVD-142M                             85.8     **73.9**   **47.7**   **64.6**   **82.3**   **86.4**   **67.6**

  :  **Ablation of the source of pretraining data.** We compare the
  INet-22k dataset that was used in iBOT to our dataset, LVD-142M. Each
  model is trained for the same number of iterations, that is smaller
  than in our final run, without high-resolution adaptation. Pretraining
  on LVD-142M maintains the performance over INet-1k while leading to
  models that perform better in other domains.
:::

The most salient observation is that training on a curated set of images
works better on most benchmarks than training on uncurated data. This
confirms the benefit of curating data, even in the case of
self-supervised pretraining. When compared with models trained on
ImageNet-22k, training on LVD-142M is also superior on all the
benchmarks but ImageNet-1k. This confirms that training on a more
diverse set of images improves the quality of the features in domains
that are not covered by ImageNet-22k. We also see that training on our
curated data increases the performances on domains that are not used for
the curation process (INaturalist 2018, 2021 and Places205), proving
that scale and diversity can benefit unseen domains. Overall, the
conclusion of this ablation is that our dataset provides a good balance
of different types of images that leads to the best performance overall.

## Model Size and Data

We quantify the importance of scaling data with the model size in
Fig. [\[fig:scaleperf\]](#fig:scaleperf){reference-type="ref"
reference="fig:scaleperf"}. As the size of models grow, training on
LVD-142M becomes more beneficial than training on ImageNet-22k. For
instance, a ViT-g trained on LVD-142M matches the performance on
ImageNet-1k of a model trained on ImageNet-22k while significantly
outperforming it on the other benchmarks.

::: figure*
![image](stamp_ImageNet-1k.pdf) ![image](stamp_ImageNet-V2.pdf)
![image](stamp_ImageNet-Sketch.pdf) ![image](stamp_Food101.pdf)
![image](stamp_Cars.pdf) ![image](stamp_AmsterTime.pdf)
![image](stamp_Oxford-H.pdf)
:::

## Loss Components

We validated the proposed technical improvements in
Sec. [6.1](#sec:ablation-ibot){reference-type="ref"
reference="sec:ablation-ibot"} by adding them incrementally. This
section analyzes the performance hit observed if we ablate specific loss
terms, starting from our best-performing model. We ablate the importance
of the KoLeo loss and the impact of the masked image modeling term. For
both, we report performance on ImageNet-1k using a linear classifier,
ADE-20k segmentation using a linear classifier, and nearest-neighbor
image retrieval on Oxford-M.
Table [\[tab:koleo\]](#tab:koleo){reference-type="ref"
reference="tab:koleo"} shows the impact of using the KoLeo loss. We see
that the instance retrieval performance improves by more than $8\%$,
confirming that this term helps spread features in the output space. At
the same time, the other metrics do not suffer from this regularization.
In Table [\[tab:ibot\]](#tab:ibot){reference-type="ref"
reference="tab:ibot"}, we show the impact of using the masked image
modeling term from iBOT. This term is critical for dense prediction
tasks, leading to almost $3\%$ performance improvement.

::: subtable
0.5
:::

::: subtable
0.5
:::

## Impact of Knowledge Distillation

For small architectures, we distill larger models instead of training
them from scratch. We use the distillation procedure described in
Sec. [5.0.0.5](#sec:distill){reference-type="ref"
reference="sec:distill"}. We evaluate the effectiveness of this approach
by comparing a ViT-L/14 trained from scratch with one distilled from a
ViT-g/14 over 12 benchmarks in
Fig. [3](#fig:distillation){reference-type="ref"
reference="fig:distillation"}. We also report the performance of the
ViT-g/14 used for distillation as a topline. The distilled model
outperforms the one trained from scratch on all 12 benchmarks,
validating our pretraining approach for small models.

<figure id="fig:distillation">
<figure>
<embed src="ablation_distil_spider.pdf" style="width:98.0%" />
<figcaption>Comparison on individual metrics</figcaption>
</figure>
<figure>

<figcaption>Averaged metrics on 8 vision tasks</figcaption>
</figure>
<figcaption> <strong>Effectiveness of knowledge distillation.</strong>
Comparison between a ViT-L trained from scratch or distilled from DINOv2
using ViT-g/14. For reference, we also report the performance of the
ViT-g/14 teacher. We show that a ViT-L model distilled from a frozen
ViT-g outperforms a the same model trained from scratch on all
benchmarks, sometimes even outperforming the distillation target.
</figcaption>
</figure>

## Impact of Resolution

We measure the impact of changing the resolution during the pretraining
on the performance of image and patch-level features. We consider models
trained from scratch using a fixed resolution of either $224\times 224$
or $416\times 416$, and a model trained from scratch at $224\times 224$,
then resumed for 10k more iterations at $416\times 416$. High-resolution
training is compute-intensive, so we conduct this ablation on a small
setup: a ViT-L/16 trained on ImageNet1k. In
Fig. [4](#fig:res){reference-type="ref" reference="fig:res"}, we report
the performance of a linear probe on ImageNet-1k and ADE-20k, evaluated
at various resolutions. The model trained on high-resolution images
performs best across resolutions, but this comes at a high cost:
training at $416$ is approximately $3\times{}$ more compute-intensive
than training at $224$. On the other hand, training at high resolution
for only 10k iterations at the end of the training is almost as good and
only requiring a fraction of the compute. As a consequence, we include
this step at the end of the training rather than training at a high
resolution from scratch.

![ **Role of resolution.** Performance of ViT-L/16 trained on
ImageNet-1k at fixed resolution ("224" and "416") or trained at 224 then
416 for a short duration ("224$\rightarrow$`<!-- -->`{=html}416"). We
train linear classifiers on top of frozen features at different
resolutions and report Top-1 accuracy on ImageNet and mIoU on ADE-20k.
We observe that performing SSL training at high resolution for a short
duration achieve behavior and results close to training at the same high
resolution for the full training, at a fraction of the cost.
](figure_res.pdf){#fig:res}

# Results {#sec:results}

In this section, we present the empirical evaluation of our models on
many image understanding tasks. We evaluate both global and local image
representations, on category and instance-level recognition, semantic
segmentation, monocular depth prediction, and action recognition. We
detail the list of benchmarks in Appendix
[13](#app:benchmarks_list){reference-type="ref"
reference="app:benchmarks_list"}. The goal of this evaluation is
twofold. First, we show that our self-supervised features outperform the
current state of the art by a very large margin. Second, we show that
they match, or surpass the performance of weakly-supervised ones on a
substantial number of tasks.

**Baselines.** In our comparisons, we use two kinds of models as
baselines. We compare to the best performing self-supervised models that
are openly available. First, we run our evaluations for
MAE [@he2021masked], DINO [@caron2021emerging],
SEERv2 [@goyal2022vision], MSN [@assran2022masked],
EsViT [@li2021efficient], Mugs [@zhou2022mugs] and iBOT [@zhou2021ibot].
When several architectural variants were proposed for a given method, we
report results for the one that leads to best top-1 accuracy on
ImageNet-1k. Second, we report performance of open-source
weakly-supervised models such as CLIP [@radford2021learning],
OpenCLIP [@ilharco_gabriel_2021_5143773; @cherti2023reproducible], and
SWAG [@singh2022revisiting]. When evaluating models on ImageNet-1k, we
report the performance for each of the aforementioned methods. For all
other evaluations, we report the four best-performing models amongst SSL
ones. Also, for reference, we report the best performing OpenCLIP-G for
weakly-supervised ones.

## ImageNet Classification {#sec:imagenet}

As a first evaluation, we probe the quality of the holistic image
representation produced by the model on the ImageNet-1k classification
dataset. We evaluate the quality of features by training a simple
classifier over a frozen backbone, and do not perform finetuning of the
backbone weights. Following previous work, we use a linear model for
simplicity, ensuring a reproducible evaluation, despite the fact that
classes may not be linearly separable. Because most SSL methods were
developped using ImageNet-1k validation performance as a debugging
signal, we also report the top-1 accuracy on ImageNet-ReaL and
ImageNet-V2. In order to report this additional validation performance,
for all models, we run the evaluation with our code. We compare our
frozen features to the best publicly available SSL features in
Table [\[tab:lin-inet1k\]](#tab:lin-inet1k){reference-type="ref"
reference="tab:lin-inet1k"}, regardless of architecture or pretraining
data. We see the components proposed in this work lead to a very
significant improvement ($+4.2\%$) over the previous state of the art
(iBOT ViT-L/16 trained on ImageNet-22k) on linear evaluation. At the
same time, we also see that the performance increase on the alternative
test sets is larger for our method, indicating stronger generalization.
We describe details of our linear evaluation in Appendix
[12.3](#app:linearprobing){reference-type="ref"
reference="app:linearprobing"}.

::: tabu
@ lll c cc cccc@ &&&&& kNN &&\
Method & Arch. & Data & Text sup. && val && val & ReaL & V2\
\
CLIP & ViT-L/14 & WIT-400M & && 79.8 && 84.3 & 88.1 & 75.3\
CLIP & ViT-L/14$_{\text{336}}$ & WIT-400M & && 80.5 && 85.3 & 88.8 &
75.8\
SWAG & ViT-H/14 & IG3.6B & && 82.6 && 85.7 & 88.7 & 77.6\
OpenCLIP & ViT-H/14 & LAION-2B & && 81.7 && 84.4 & 88.4 & 75.5\
OpenCLIP & ViT-G/14 & LAION-2B & && 83.2 && 86.2 & 89.4 & 77.2\
EVA-CLIP & ViT-g/14 & custom$^*$ & && **83.5 && 86.4 & 89.3 & 77.4\
\
MAE & ViT-H/14 & INet-1k & && 49.4 && 76.6 & 83.3 & 64.8\
DINO & ViT-S/8 & INet-1k & && 78.6 && 79.2 & 85.5 & 68.2\
SEERv2 & RG10B & IG2B & && -- && 79.8 & -- & --\
MSN & ViT-L/7 & INet-1k & && 79.2 && 80.7 & 86.0 & 69.7\
EsViT & Swin-B/W=14 & INet-1k & && 79.4 && 81.3 & 87.0 & 70.4\
Mugs & ViT-L/16 & INet-1k & && 80.2 && 82.1 & 86.9 & 70.8\
iBOT & ViT-L/16 & INet-22k & && 72.9 && 82.3 & 87.5 & 72.4\
& ViT-S/14 & LVD-142M& && 79.0 && 81.1 & 86.6 & 70.9\
& ViT-B/14 & LVD-142M& && 82.1 && 84.5 & 88.3 & 75.1\
& ViT-L/14 & LVD-142M& && **83.5 && 86.3 & 89.5 & 78.0\
& ViT-g/14 & LVD-142M& && **83.5 && **86.5 & **89.6 & **78.4\
************
:::

#### How far are we from weakly-supervised models?

We also want to validate that our features are competitive with
state-of-the-art open-source weakly supervised models. To this end, we
compare on ImageNet-1k, using the linear evaluation, to three
off-the-shelf methods with several architectural variants. For all
models, we run the linear evaluation using our code, after making sure
that our numbers match those reported in technical reports and papers.
We show the result of this evaluation in
Table [\[tab:lin-inet1k\]](#tab:lin-inet1k){reference-type="ref"
reference="tab:lin-inet1k"}. We see that our backbone, surpases the
performance of OpenCLIP with a ViT-G/14 architecture ($+0.3\%$) and
EVA-CLIP with a ViT-g/14 ($+0.1\%$). At the same time, we also observe
that our performance on the ImageNet-V2 test set is significantly better
($+1.1\%$ versus EVA-CLIP), indicating better generalization. For the
remainder of this section, we report OpenCLIP-G as a reference for
weakly-supervised models.

#### Can we finetune the encoders?

We question if the ability of our models to produce high quality frozen
features impact their performance when finetuned with supervision on a
specific dataset. While this is not core to this paper, this experiment
is indicative of whether we have involuntarily specialized our models to
the setting of linear evaluations of frozen features. To run this sanity
check, we apply the finetuning pipeline from @touvron2022deit, without
tweaking hyper-parameters. In
Table [3](#tab:ft-inet1k-alone){reference-type="ref"
reference="tab:ft-inet1k-alone"}, we show that the Top-1 accuracy on the
validation set of ImageNet-1k improves by more than $+2\%$ when the
backbone is finetuned. This is true both when using models at resolution
$224$ and $448$. Further gains can be obtained by tuning the
hyper-parameters of the finetuning, but this is beyond the goal of this
sanity check. Nonetheless, our best finetuned performance ($88.9\%$) is
only a couple of percent below ($-2.2\%$) the absolute state of the arts
($91.1\%$), obtained by @chen2023symbolic. As DINOv2 leads to features
that are strong in both the linear and finetuning settings, a strong
property of our approach is that *finetuning is optional*.

::: {#tab:ft-inet1k-alone}
  Arch.       Res.   Linear   Finetuned   $\Delta$
  ---------- ------ -------- ----------- ----------
  ViT-g/14    224     86.5      88.5        +2.0
              448     86.7      88.9        +2.2

  :  **Supervised finetuning on ImageNet-1k.** We use the pipeline
  of [@touvron2022deit] to finetune our encoders on ImageNet-1k at
  resolutions $224\times224$ or $448\times448$. We compare with the
  accuracy obtained with linear probing and observe only modest
  improvements with fine-tuning: this suggests that DINOv2 features
  already perform well out-of-the-box.
:::

#### Robustness analysis.

To complement our study, and probe the generalization of our features,
we evaluate our ImageNet-1k models trained with linear classification
heads on domain generalization benchmarks. We use the best performing
linear classifier as described above and simply run inference on those
benchmarks. Please note that most results in the literature are obtained
with models that are finetuned end-to-end on ImageNet-1k. We show the
result of this experiment in
Table [\[tab:robustness\]](#tab:robustness){reference-type="ref"
reference="tab:robustness"}. When comparing with state-of-the-art SSL
methods, our models shows drastically better robustness ($+29.6\%$ on
A [@hendrycks2021natural], $+22.1\%$ on R [@hendrycks2021many] and
$+23.0\%$ on Sketch [@wang2019learning] compared to iBOT). Our model
also improves upon the best weakly-supervised model on ImageNet-A while
lagging behind on R and Sketch.

::: tabu
@ lll c cccc @ Method & Arch & Data && Im-A & Im-R & Im-C$\downarrow$ &
Sketch\
OpenCLIP & ViT-G/14 & LAION-2B && 63.8 &**87.8 & 45.3 & **66.4\
MAE & ViT-H/14 & INet-1k && 10.2 & 34.4 & 61.4 & 21.9\
DINO & ViT-B/8 & INet-1k && 23.9 & 37.0 & 56.6 & 25.5\
iBOT & ViT-L/16 & INet-22k && 41.5 & 51.0 & 43.9 & 38.5\
& ViT-S/14 & LVD-142M&& 33.5 & 53.7 & 54.4 & 41.2\
& ViT-B/14 & LVD-142M&& 55.1 & 63.3 & 42.7 & 50.6\
& ViT-L/14 & LVD-142M&& 71.3 & 74.4 & 31.5 & 59.3\
& ViT-g/14 & LVD-142M&& **75.9 & 78.8 & **28.2 & 62.5\
********
:::

## Additional Image and Video classification Benchmarks

In this section we study the generalization of our features on
downstream classification benchmarks. We consider two sets of
evaluations in that context. On one hand, we use large and finegrained
datasets such as iNaturalist and Places205. On the other, we use the 12
image classification tasks originally proposed in
SimCLR [@chen2020simple]. For iNaturalist 2018, iNaturalist 2021, and
Places205, we train a linear classifier with data augmentations as in
Sec. [7.1](#sec:imagenet){reference-type="ref" reference="sec:imagenet"}
We report top-1 accuracy for those three datasets in
Table [\[tab:finegrained_video\]](#tab:finegrained_video){reference-type="ref"
reference="tab:finegrained_video"}. Interestingly, our model
significantly outperforms OpenCLIP ViT-G/14 on both variants of
iNaturalist ($+8.6\%$ and $+9.7\%$ for 2018 and 2021 respectively), and
lags slightly behind on Places 205 ($-2.3\%$).

::: tabu
ll c ccc c ccc & && &&\
Feature & Arch && iNat2018 & iNat2021 & Places205 && K400 & UCF-101 &
SSv2\
OpenCLIP & ViT-G/14 && 73.0 & 76.0 & **69.8 && 78.3 & 90.7 & 35.8\
MAE & ViT-H/14 && 31.0 & 32.3 & 52.4 && 54.2 & 70.6 & 29.2\
DINO & ViT-B/8 && 59.6 & 68.3 & 60.4 && 64.5 & 85.0 & 32.6\
iBOT & ViT-L/16 && 66.3 & 74.6 & 64.4 && 72.6 & 88.6 & **38.7\
& ViT-S/14 && 69.0 & 74.2 & 62.9 && 67.8 & 87.0 & 33.1\
& ViT-B/14 && 76.4 & 81.1 & 66.2 && 73.2 & 89.1 & 34.4\
& ViT-L/14 && 80.4 & 85.1 & 67.3 && 76.3 & 90.5 & 35.6\
& ViT-g/14 && **81.6 & **85.7 & 67.5 && **78.4 & **91.2 & 38.3\
************
:::

In a second set of evaluations, we measure the performance of our model
on video action recognition even though our features were not trained on
videos.. We evaluated features on three datasets, namely
UCF-101 [@soomro2012ucf101], Kinetics-400 [@kay2017kinetics] and
Something-Something v2 [@goyal2017something]. For this evaluation, we
pick $8$ evenly spaced frames in the video and train a linear classifier
on the average of the features for UCF and K-400. For SSv2, we opt for
concatenation to retain more temporal information than with feature
averaging. For each dataset, we measure average accuracy and report the
results in
Table [\[tab:finegrained_video\]](#tab:finegrained_video){reference-type="ref"
reference="tab:finegrained_video"}. We see that amongst self-supervised
approaches, our model clearly sets a new state of the art. Moreover, our
model matches the accuracy of the OpenCLIP features on UCF and Kinetics
($+0.1\%$ and $+0.5\%$ respectively) and clearly outperforms them on
SSv2 ($+2.5\%$). This is particularly interesting, as SSv2 requires a
much richer understanding of the video frames.

Finally, in
Table [\[tab:finegrained\]](#tab:finegrained){reference-type="ref"
reference="tab:finegrained"}, we compare selected frozen features on 12
transfer classification benchmarks initially proposed
by @chen2020simple. This benchmark covers scenes, objects (food, cars,
planes), and textures. We replace the Birdsnap dataset with CUB because
the former was not publicly available in its entirety. We follow the
experimental protocol as outlined by @chen2020simple, namely training a
logistic regression on precomputed features. Our model significantly
outperforms state-of-the-art SSL models, with most notable differences
on Stanford Cars ($+14.8\%$ versus DINO ViT-B/8) and FGVC Aircraft
($+14.8\%$ versus iBOT ViT-L/16). Even though these benchmarks favor
text-guided pretraining, our features are still competitive with
OpenCLIP on most classification benchmarks, with the exception of a few
datasets, especially SUN ($-5.3\%$) and Cars ($-4.7\%$).

::: tabu
\@l@l cccccccccccc c @ Feature & Arch & Food & C10 & C100 & SUN & Cars &
Aircr & VOC & DTD & Pets & Cal101 & Flowers & CUB & Avg\
OpenCLIP  & ViT-G/14 & 94.5 & 98.7 & 91.0 & **84.0 & **96.1 & 80.2 &
**89.3 & **86.0 & 95.7 & **98.1 & 99.5 & 89.9 & 91.9\
MAE & ViT-H/14 & 78.4 & 96.1 & 83.9 & 63.9 & 56.1 & 63.4 & 84.3 & 75.4 &
89.4 & 95.9 & 92.3 & 57.2 & 78.0\
DINO & ViT-B/8 & 85.1 & 97.2 & 86.9 & 70.3 & 76.6 & 70.6 & 86.7 & 79.6 &
93.2 & 95.4 & 97.6 & 81.7 & 85.1\
iBOT & ViT-L/16 & 91.0 & 99.0 & 92.8 & 75.6 & 71.8 & 72.4 & 89.0 & 80.7
& 87.7 & 97.5 & 99.6 & 82.1 & 86.6\
& ViT-S/14 & 89.1 & 97.7 & 87.5 & 74.4 & 81.6 & 74.0 & 87.8 & 80.6 &
95.1 & 97.0 & 99.6 & 88.1 & 87.7\
& ViT-B/14 & 92.8 & 98.7 & 91.3 & 77.3 & 88.2 & 79.4 & 88.2 & 83.3 &
96.2 & 96.1 & 99.6 & 89.6 & 90.1\
& ViT-L/14 & 94.3 & 99.3 & 93.4 & 78.7 & 90.1 & 81.5 & 88.3 & 84.0 &
96.6 & 97.5 & 99.7 & 90.5 & 91.2\
& ViT-g/14 & **94.7 & **99.5 & **94.4 & 78.7 & 91.4 & **87.2 & 89.0 &
84.5 & **96.7 & 97.6 & **99.7 & **91.6 & **92.1\
**************************
:::

## Instance Recognition

In this experiment, we probe our model on the task of instance-level
recognition using a non-parametric approach. Images from a database are
ranked according to their cosine similarity with a query image. We
evaluated our model and compare to baselines on Paris and Oxford, that
are landmark recognition benchmarks. We also evaluated on Met, a dataset
of artworks from the Metropolitan museum, and AmsterTime, containing
street view images matched to archival images of Amsterdam. We measure
performance by computing the mean average precision and report our
results in
Table [\[tab:retrieval\]](#tab:retrieval){reference-type="ref"
reference="tab:retrieval"}. We see that our features significantly
outperform both SSL ($+41\%$ mAP on Oxford-Hard), and weakly-supervised
($+34\%$ mAP on Oxford-Hard) ones. It is interesting to see that our
features perform well across task granularities, both at the
category-level and instance-level. This is a desirable property for
strong off-the-shelf computer vision features.

::: tabu
ll c cc c cc c ccc c c & && && && && AmsterTime\
Feature & Arch && M & H && M & H && GAP & GAP- & ACC && mAP\
OpenCLIP & ViT-G/14 && 50.7 & 19.7 && 79.2 & 60.2 && 6.5 & 23.9 & 34.4
&& 24.6\
MAE & ViT-H/14 && 11.7 & 2.2 && 19.9 & 4.7 && 7.5 & 23.5 & 30.5 && 4.2\
DINO & ViT-B/8 && 40.1 & 13.7 && 65.3 & 35.3 && 17.1 & 37.7 & 43.9 &&
24.6\
iBOT & ViT-L/16 && 39.0 & 12.7 && 70.7 & 47.0 && 25.1 & 54.8 & 58.2 &&
26.7\
& ViT-S/14 && 68.8 & 43.2 && 84.6 & 68.5 && 29.4 & 54.3 & 57.7 && 43.5\
& ViT-B/14 && 72.9 & 49.5 && 90.3 & 78.6 && 36.7 & 63.5 & 66.1 && 45.6\
& ViT-L/14 && **75.1 & **54.0 && **92.7 & **83.5 && **40.0 & 68.9 & 71.6
&& **50.0\
& ViT-g/14 && 73.6 & 52.3 && 92.1 & 82.6 && 36.8 & **73.6 & **76.5 &&
46.7\
****************
:::

## Dense Recognition Tasks

We probe the quality of patch-level features extracted from our network
on several dense downstream tasks. We consider semantic image
segmentation and monocular depth estimation in several settings and we
conduct evaluations on multiple datasets for each.

#### Semantic segmentation.

For our semantic segmentation evaluation, we consider two different
setups. **Linear**: a linear layer is trained to predict class logits
from a patch tokens. It is used to produce a low-resolution logit map
(eg 32x32 for a model with patch size 16), which is then upsampled to
full resolution (512x512) to obtain a segmentation map. This procedure
is extremely simple but cannot easily produce high-resolution
segmentations. **+ms**: a boosted version of the linear setup. We
concatenate the patch tokens of the 4 last layers, use a larger image
resolution of 640, and use multiscale test-time augmentations to improve
the predictions. We report the performance of our model variants as well
as the baselines on three datasets under the two setups in
Table [\[tab:semseg\]](#tab:semseg){reference-type="ref"
reference="tab:semseg"}.

Our models show very good performance on all datasets and for all
setups. Interestingly, our evaluation using **+ms** is on par with fully
finetuning MAE with an Upernet decoder ($53.0$ versus $53.6$ mIoU). This
is surprising because we use a significantly simpler predictor. Also,
our best model, when evaluated using the boosted recipe, almost matches
the state of the art on Pascal VOC ($86.2$ versus $89.0$ mIoU).

#### Frozen backbone in a SOTA pipeline.

In a final experiment, we freeze our backbone, and plug it into a
ViT-Adapter [@chen2022vision] with a Mask2former
head [@cheng2022masked]. We tune the weights of the adapter and head,
but keep the backbone frozen, meaning 66% of the weights are frozen.
This allows for a lighter segmentation training than full end-to-end
fine-tuning. With this setup, we reach $60.2$ mIoU on ADE20k, close to
the competitive state of the art, standing at 62.9
mIoU [@wang2022internimage]. Although our setup for this experiment
doesn't makes use of the optimisations described in
Sec. [5](#sec:optim){reference-type="ref" reference="sec:optim"}, the
segmentation training in this experiment took 28 hours on 16 V100 GPUs.

::: tabu
\@ll c cc c cc c cc@ & && && &&\
& && && &&\
Method & Arch. && lin. & +ms && lin. & +ms && lin. & +ms\
OpenCLIP & ViT-G/14 && 39.3 & 46.0 && 60.3 & 70.3 && 71.4 & 79.2\
MAE & ViT-H/14 && 33.3 & 30.7 && 58.4 & 61.0 && 67.6 & 63.3\
DINO & ViT-B/8 && 31.8 & 35.2 && 56.9 & 66.2 && 66.4 & 75.6\
iBOT & ViT-L/16 && 44.6 & 47.5 && 64.8 & 74.5 && 82.3 & 84.3\
& ViT-S/14 && 44.3 & 47.2 && 66.6 & 77.1 && 81.1 & 82.6\
& ViT-B/14 && 47.3 & 51.3 && 69.4 & 80.0 && 82.5 & 84.9\
& ViT-L/14 && 47.7 & **53.1 && 70.3 & 80.9 && 82.1 & 86.0\
& ViT-g/14 && **49.0 & 53.0 && **71.3 & **81.0 && **83.0 & **86.2\
************
:::

#### Depth estimation.

In this experiment, we evaluate our patch-level features on three
monocular depth estimation benchmarks: NYUd, KITTI and zero-shot
transfer from NYUd to SUN3d. We follow the evaluation protocol
of @li2022binsformer. We consider three different setups for this
evaluation. **lin. 1**: we extract the last layer of the frozen
transformer and concatenate the `[CLS]` token to each patch token. Then
we bi-linearly upsample the tokens by a factor of 4 to increase the
resolution. Finally we train a simple linear layer using a
classification loss by dividing the depth prediction range in 256
uniformly distributed bins and use a linear normalization following
 [@bhat2019adabins]. **lin. 4**: we use the same protocol that we use
with one layer, but concatenate the tokens from layers $l=\{3,6,9,12\}$
for ViT-S/B, $l=\{5,12,18,24\}$ for ViT-L, and $l=\{10, 20, 30, 40\}$
for ViT-g. **DPT**: we use the DPT decoder [@ranftl2021vision] on top of
our frozen models and setup a regression task. We scale the size of the
head following the dimension of the features for each architecture. We
show results for all baselines, all datasets and all setups in Table
[\[tab:depth\]](#tab:depth){reference-type="ref" reference="tab:depth"}.

From this table, we see that our features clearly surpass the best SSL
and WSL features available. It is interesting to see that iBOT features
extracted from a ViT-L outperform the ones of OpenCLIP with a ViT-G.
This observation supports the intuition that caption-based feature
learning fails to learn subtle patterns like this one. Also, our model,
with the DPT decoder and frozen backbone, matches or exceeds the
performance of the recent work of @li2022binsformer. Finally, the
out-of-domain generalization result on SUN-RGBd shows that our features
allow very good transfer between domains. A depth prediction module
trained on indoor scenes from NYUd generalizes pretty well to the
outdoor examples of SUN-RGBd.

::: tabu
\@ll@ c ccc c ccc c ccc@ & && && &&\
& && && &&\
Method & Arch. && lin. 1 & lin. 4 & DPT && lin. 1 & lin. 4 & DPT && lin.
1 & lin. 4 & DPT\
OpenCLIP & ViT-G/14 && 0.541 & 0.510 & 0.414 && 3.57 & 3.21 & 2.56 &&
0.537 & 0.476 & 0.408\
MAE & ViT-H/14 && 0.517 & 0.483 & 0.415 && 3.66 & 3.26 & 2.59 && 0.545 &
0.523 & 0.506\
DINO & ViT-B/8 && 0.555 & 0.539 & 0.492 && 3.81 & 3.56 & 2.74 && 0.553 &
0.541 & 0.520\
iBOT & ViT-L/16 && 0.417 & 0.387 & 0.358 && 3.31 & 3.07 & 2.55 && 0.447
& 0.435 & 0.426\
& ViT-S/14 && 0.449 & 0.417 & 0.356 && 3.10 & 2.86 & 2.34 && 0.477 &
0.431 & 0.409\
& ViT-B/14 && 0.399 & 0.362 & 0.317 && 2.90 & 2.59 & 2.23 && 0.448 &
0.400 & 0.377\
& ViT-L/14 && 0.384 & 0.333 & 0.293 && 2.78 & 2.50 & 2.14 && 0.429 &
0.396 & 0.360\
& ViT-g/14 && **0.344 & **0.298 & **0.279 && **2.62 & **2.35 & **2.11 &&
**0.402 & **0.362 & **0.338\
******************
:::

## Qualitative Results

In this final section of the empirical evaluation of our features, we
propose a few qualitative analyses.

#### Semantic Segmentation and Depth Estimation.

We show some qualitative results for our dense prediction evaluations:
segmentation on ADE20K in
Fig. [\[fig:segm_qual\]](#fig:segm_qual){reference-type="ref"
reference="fig:segm_qual"} and depth estimation on NYUd, KITTI and SUN
RGB-D in Fig. [5](#fig:depth_qual){reference-type="ref"
reference="fig:depth_qual"}. We compare DINOv2 with OpenCLIP with a
linear classifier on each dataset. While not perfect, the linear
segmentation model using our DINOv2 backbone produces good results and
behaves much better than the OpenCLIP one under this evaluation setup.
Indeed, the segmentation mask produced by OpenCLIP-G shows many
artifacts and disconnected components. The qualitative results on depth
estimation clearly illustrate the quantitative gap between OpenCLIP and
DINOv2. These results highlight that our features, as well as the
features extracted from OpenCLIP, are able to linearly separate complex
information such as depth, even though neither was trained with this
type of information. However, our features lead to a much smoother depth
estimation, with less artifacts. Some objects such as the chair on the
SUN RGB-D image are completely ignored by OpenCLIP and correctly
positioned using our features.

![ **Segmentation and depth estimation with linear classifiers.**
Examples from ADE20K, NYUd, SUN RGB-D and KITTI with a linear probe on
frozen OpenCLIP-G and DINOv2-g features.
](new-figure-7.jpg){#fig:depth_qual width="\\linewidth"}

#### Out-of-distribution generalization.

We show a few examples of applying the depth prediction and segmentation
linear classifiers to out-of-distribution examples in
Fig. [6](#fig:depth_ood){reference-type="ref"
reference="fig:depth_ood"}. The qualitative results support our claim
that our features transfer between domains. The quality of the depth and
segmentation predicted for pictures of animals, or paintings is very
good, even though the domains are very different.

![ **Examples of out-of-distribution examples** with frozen DINOv2-g
features and a linear probe. ](new-figure-8.jpg){#fig:depth_ood
width="\\linewidth"}

![ **More visualization of the first PCA components.** We compute the
PCA between the patches from all of the images and show their first 3
components. Each component corresponds to a specific color channel. Same
parts are matched between related images depsite changes of pose, style
or even objects. Background is removed by removing patches with a
negative score of the first PCA component. ](new-figure-9.jpg){#fig:pca
width="\\linewidth"}

#### PCA of patch features.

We show the results of the principal component analysis (PCA) performed
on the patch features extracted by our model. We keep only patches with
a positive value after we threshold the first component. This procedure
turns out to separate the image's main object from the background. We
compute a second PCA on the remaining patches across three images
depicting the same category. We color the three first components with
three different colors and present the results in
Fig. [\[fig:qualitative\]](#fig:qualitative){reference-type="ref"
reference="fig:qualitative"} and [7](#fig:pca){reference-type="ref"
reference="fig:pca"}. There are two interesting observations: first, our
unsupervised foreground / background detector, based on detecting the
highest variance direction, performs very well and is capable of
delineating the boundary of the main object in the picture. Second, the
other components correspond to \"parts\" of objects and match well for
images of the same category. This is an emerging property -- our model
was not trained to parse parts of objects.

#### Patch matching.

Finally, we explore what type of information our patch-level features
contain by matching them across images. We start by detecting the
foreground object using the procedure described above. Then, we compute
the euclidean distance between patch features extracted from two images
and map them by solving an assignment problem. In order to reduce the
number of matches, we then apply a non-maximum suppression to keep only
the salient ones. In
Fig. [\[fig:matching\]](#fig:matching){reference-type="ref"
reference="fig:matching"}, we show some examples of such matchings.

We observe that the features seem to capture information about semantic
regions that serve similar purpose in different objects or animals. For
instance, the wing of a plane matches the wing of a bird. We also
observe that the model is robust to style (image versus drawing), and to
large variation of poses (see the elephant).

::: figure*
![image](new-figure-10.jpg){width="\\linewidth"}
:::

# Fairness and Bias Analysis

We conduct two fairness evaluations of our models. We probe for
geographical fairness and potential harmful label associations. For both
evaluations, we experiment with our largest ViT-g model.

## Geographical Fairness

We evaluate geographical fairness on the Dollar Street dataset
introduced in [@de2019does] using the evaluation protocol
of @goyal2022fairness. This benchmark compares performance across
countries and income levels. It contains 16,073 images from 289
households across 54 countries. The task is to recognize 94 concepts
that vary visually between households based on income or location. In
Table [\[tab:dollar\]](#tab:dollar){reference-type="ref"
reference="tab:dollar"}, we compare our model with
SEERv2 [@goyal2022vision], a model trained on a geographically diverse
set of images. Our model is slightly fairer across regions and incomes
than the SEERv2 model and significantly better than the supervised
baseline reported by @goyal2022vision. However, we still observe a
significant difference between regions, particularly in Africa, where
our model performance drops by 25.7% compared to Europe. This shows that
our model is still biased toward Western countries. Similarly, our model
performs significantly better on high-income households than low-income
ones, with a difference of 31.7%. Despite improvements, we observe
significant biases in our models toward wealthy households from Western
countries.

::: tabu
\@lll c ccc c cccc@ && && &&\
Method & Arch. & Data && low & medium & high && Africa & Asia & Americas
& Europe\
SEERv2 & RG-10B & IG-1B && 59.7 & 78.5 & 86.6 && 65.9 & 76.3 & 81.1 &
85.6\
DINOv2& ViT-g/14 & LVD-142M&& 67.4 & 83.3 & 90.5 && 74.0 & 81.6 & 86.2 &
89.7\
:::

## Gender, Skintones and Age

In a second set of evaluations, we question how our model classifies
images of people of different gender, skin tone, and age (all
self-reported). We follow the protocol of [@goyal2022fairness], where we
train a multiclass classifier on a subset of 619 classes of
ImageNet-22k. We group the 619 classes into four broader categories:
Human, Possibly Human, Non-Human, or Crime. Non-Human and Crime are
considered harmful. Using this classifier, we run inference on 2955
images from the Casual Conversations dataset [@hazirbas2021towards] and
keep all labels in the top-5 that are assigned a probability of 0.1 or
more. Because of that, we can assign multiple classes to each image. We
make one modification to the original evaluation protocol: we do not
backpropagate gradients to the backbone and keep it frozen. We compare
our model to SEERv2 in
Table [\[tab:gsa\]](#tab:gsa){reference-type="ref" reference="tab:gsa"}.

Our model often classifies images of all groups as Human without large
deviations across skin tones. Neither SEERv2 nor DINOv2 predict harmful
labels from the Non-Human or Crime meta-categories (except for two
instances where the background contains bars visually similar to prison
bars). We see that our model triggers the Possibly-Human classes often.
This class is constructed from objects in ImageNet-22k that are often
related to Humans, such as Scarf, Glasses, or Beard. Our model often
predicts the Possibly-Human class for men because of the prevalence of
the Beard class. No clear pattern indicates a bias against a particular
group in this study. While this is encouraging, we also acknowledge that
a more thorough evaluation of biases may reveal flaws in our model.

::: tabu
\@ll rrrr c rrrr@ && &&\
**Model** & **Assoc.** & & & & && & & &\
SEER & `Non-Human` & 0.0 & 0.0 & 0.0 & 0.0 && 0.0 & 0.0 & 0.0 & 0.0\
RG-10B & `Crime` & 0.0 & 0.0 & 0.0 & 0.0 && 0.0 & 0.0 & 0.0 & 0.0\
& `Human` & 94.9 & 95.8 & 86.6 & 79.0 && 90.5 & 88.3 & 91.9 & 82.3\
& `Possibly-Human` & 13.6 & 6.7 & 65.0 & 60.2 && 32.8 & 37.2 & 29.4 &
6.5\
DINOv2& `Non-Human` & 0.0 & 0.0 & 0.0 & 0.0 && 0.0 & 0.0 & 0.0 & 0.0\
ViT-g/14 & `Crime` & 0.0 & 0.0 & 0.2 & 0.0 && 0.0 & 0.1 & 0.0 & 0.0\
& `Human` & 97.3 & 97.7 & 86.1 & 84.0 && 91.2 & 90.2 & 93.2 & 88.7\
& `Possibly-Human` & 15.8 & 17.2 & 52.2 & 48.1 && 35.3 & 37.3 & 23.0 &
9.7\
:::

# Estimating the Environmental Impact of Training our Models

::: {#tab:carbon}
  ----------- ----------- ------------- ----------- ----- ------------- ----------------
  Model to     GPU Type     GPU Power    GPU-hours   PUE   Total power   Carbon emitted
  Reproduce                consumption                     consumption    (tCO$_2$eq)
  DINOv2-g     A100-40GB      400W        22,016     1.1     9.7 MWh          3.7
  ----------- ----------- ------------- ----------- ----- ------------- ----------------

  :  **Carbon footprint of reproducing DINOv2.** We report the potential
  carbon emission of reproducing DINOv2-g when assuming a power
  consumption for the A100-40GB of 400W, a PUE of 1.1 and carbon
  intensity factor of 0.385 kg CO$_2$e per KWh.
:::

Training foundation models consumes a significant amount of energy,
resulting in carbon dioxide emissions. @patterson2021carbon propose a
methodology to report an estimation of the carbon emitted during the
training of a model based on the specifics of the data center and its
power grid. This computation informs the design of the data center used
for the training of models and the choice of location for data centers.
This methodology requires to know the specifics of the data center used
for training, which can be complex when multiple data centers are
involved over time. Additionally, these specifics are most often not in
the control of the AI practitioner, and hence, this methodology is less
helpful when practioners make technical decisions about future
trainings. Instead, in this section, we follow an alternative that
reports the potential carbon emission of retraining a similar model in
an average data center located in the US. This methodology was used in
previous work in natural language
processing [@strubell2019energy; @touvron2023llama] to establish an
apple-to-apple comparison between pretraining schemes. More precisely,
we fix the value of all exogenous variables, i.e., the Power Usage
Effectiveness (PUE) and carbon intensity factor of a power grid to the
same values as in @touvron2023llama, that is, a PUE of 1.1 and the
carbon intensity factor to the US average of 0.385 kg CO$_2$eq/KWh. We
use the same formula as in [@patterson2021carbon] to estimate the
potential energy consumption and the carbon emission. For the power
consumption of an A100-80GB, we take the thermal design power for NVLink
systems, which is 400W. We report the potential carbon emission of
retraining a DINOv2 ViT-g in Table [4](#tab:carbon){reference-type="ref"
reference="tab:carbon"}. For comparison, retraining an OpenCLIP ViT-L or
OpenCLIP ViT-G would require 22.4 MWh and 118.9 MWh, respectively, if
run in the same data center. This is 10$\times$ more carbon emission.
Note that this comparison is not fair to them, since they also train a
text encoder in parallel, and we thus do not report them in the table.
However, it gives a reasonable guideline for those who are interested in
training only visual features: in this context, training a
self-supervised model is preferable in terms of carbon emission.
Training a text-guided model still makes sense when planning to reuse
the text encoder.

#### Carbon footprint of the whole project.

Additionally, we estimate the footprint of the whole project to be
between $0.5$k and $1$k tCO$_2$eq using the same grid as presented above
[^3]. This carbon footprint represents in the order of $200$k GPU-days.
The primary sources of emissions are the self-supervised pre-trainings
of the models. For example, a single pre-training of a ViT-g model (22k
GPU-hours) emits 3.7 tons of CO$_2$eq, while a finetuning on ImageNet-1k
(1k GPU-hours) emits 0.2 tons. This estimate only considers the GPUs'
electricity consumption and ignores other emissions, such as their
manufacturing and disposal.

# Future work and Discussion

In this work, we present DINOv2, a new series of image encoders
pretrained on large curated data with no supervision. This is the first
SSL work on image data that leads to visual features that close the
performance gap with (weakly) supervised alternatives across a wide
range of benchmarks and without the need for finetuning.

We can attribute the strong performance of the DINOv2 family of models
to several factors: **i**) an improved training recipe with better
hyperparameters and regularization (Table
[1](#tab:ibot-dino){reference-type="ref" reference="tab:ibot-dino"}),
**ii**) a larger model scale with improved results regardless of the
data used for training
(Fig. [\[fig:scaleperf\]](#fig:scaleperf){reference-type="ref"
reference="fig:scaleperf"}), **iii**) a larger dataset (Fig.
[\[fig:scaleperf\]](#fig:scaleperf){reference-type="ref"
reference="fig:scaleperf"}) and **iv**) the distillation process that
makes smaller models benefit from the performance of the strongest ViT-g
model (Fig. [3](#fig:distillation){reference-type="ref"
reference="fig:distillation"}).

A few properties emerge from these models, such as an understanding of
object parts and scene geometry regardless of the image domains. We
expect that more of these properties will emerge at larger scales of
models and data, akin to instruction emergence in large language models,
and plan to continue scaling along these axes. This paper also
demonstrates that these visual features are compatible with classifiers
as simple as linear layers - meaning the underlying information is
*readily available*. In future work, we plan to leverage this ability to
train a a language-enabled AI system that can process visual features as
if they were word tokens, and extract the required information to ground
the system.

### Acknowledgments. {#acknowledgments. .unnumbered}

We thank Mathilde Caron for initial discussions that led to this work.
Julien Mairal was supported by the ERC grant number 101087696 (APHELAIA
project) and by ANR 3IA MIAI@Grenoble Alpes (ANR-19-P3IA-0003). We thank
Olivia Joulin for the horse drawing used in
Fig. [\[fig:matching\]](#fig:matching){reference-type="ref"
reference="fig:matching"}. We thank Madeleine and Léon for posing for
Fig. [6](#fig:depth_ood){reference-type="ref" reference="fig:depth_ood"}
We also thank the rest of FAIR and Meta AI for feedback on this work
through the entire project.

# Data Processing {#sec:supp-data}

## Data selection {#subsec:supp-data-sources}

Our selection of datasets for building LVD-142M is detailed in
Tab. [\[tab:lavida-details\]](#tab:lavida-details){reference-type="ref"
reference="tab:lavida-details"}. This collection is intended to provide
images covering well various downstream vision tasks both for
image-level and dense recognition.

## Image similarity {#subsec:supp-data-similarity}

We employ cosine similarity to compare image features (whether ours or
feature generated for deduplication) with the following similarity
function $m$:
$$m(s, r) = \text{cosine-similarity}\left(f\left(s\right),f\left(r\right)\right) = \frac{f(s)\cdot{}f(r)}{\lVert f(s)\rVert_2\lVert f(r)\rVert_2}$$
where $s$ and $r$ are a pair of images to compare and $f$ is the model
generating features.

## Deduplication {#subsec:supp-data-dedup}

#### Self-deduplication.

To deduplicate our uncurated data source of 1.3B images, we compute and
use the embeddings generated by [@pizzi2022self] and retrieve the
$k = 64$ nearest neighbors of each image (using cosine similarity).
Considering only neighbors with a similarity ${>}0.6$, we extract the
connected components of the associated $k$-NN graph thanks to a scalable
disjoint set data structure implementation. We then only keep one
representative for each component of duplicate images. This results in a
self-deduplicated data source of 1.1B images.

#### Relative deduplication

To reduce redundancy and also properly evaluate the performance of our
features, we discard remaining images of our self-deduplicated data
source that are too similar to train and test splits of our evaluation
datasets. To achieve this, we apply a similar procedure as for
self-deduplication, with a stricter similarity ${>}0.45$, this time
identifying the duplicate components (if any) to which each reference
image belong and discarding it entirely. This results in a self- and
relatively-deduplicated data source of 744M images.

## Retrieval {#subsec:supp-data-retrieval}

We employ two approaches to augment dataset via retrieval: sample-based
and cluster-based. The first one, sample-based, applies to datasets
larger than 1M images and consists in collecting a fixed number $k$ of
nearest images for each sample image of the dataset to retrieve,
effectively trying to multiply by $k$ the size of the dataset. We use
$k = 4$ for Google Landmarks v2 and ImageNet-22k but a larger $k = 32$
to make this specific retrieval a core part of our LVD-142M dataset. For
smaller datasets, the second approach, cluster-based, consists in first
clustering our uncurated data source into $100,000$ separate clusters
thanks to a distributed $k$-means implementation. Each cluster should
capture different types of image concept and contents. We then pick
$10,000$ images from each cluster associated with more than $3$ images
of the retrieved dataset. As this can result in a very large number of
retrieved images for some dataset, we restrict such retrievals to a
maximum of 1M images to maintain the balance between the different
datasets within LVD-142M.

# Implementation Details

## Unsupervised pre-training

For unsupervised pre-training we build on the DINO and iBOT codebases.
We use hyperparameters shown in
Table [5](#tab:appendix-hparams){reference-type="ref"
reference="tab:appendix-hparams"}, ViT architectures described in
Table [6](#tab:vit-hparams){reference-type="ref"
reference="tab:vit-hparams"}.

::: {#tab:appendix-hparams}
                              Arch.     Drop-rate     LR     Batch size        
  ------------------------- ---------- ----------- -------- ------------ -- -- --
  DINOv2-S (distilled)       ViT-S/14       0        1e-3       2048           
  DINOv2-B (distilled)       ViT-B/14       0        1e-3       2048           
  DINOv2-L (distilled)       ViT-L/14       0        1e-3       2048           
  DINOv2-L (from scratch)    ViT-L/14      0.4      3.5e-4      3072           
  DINOv2-g (from scratch)    ViT-g/14      0.4      3.5e-4      3072           

  :  **Training hyperparameters for DINOv2-S, DINOv2-B, DINOv2-L and
  DINOv2-g.** All models run for 625k iterations with optimizer AdamW,
  an initial LayerScale value of 1e-5, a weight decay cosine schedule
  from 0.04 to 0.2, a learning rate warmup of 100k iterations, a teacher
  momentum cosine schedule from 0.994 to 1, and we train in float16
  precision in all cases (except for the DINO heads where we reduce the
  gradients in float32).
:::

::: {#tab:vit-hparams}
  Arch.                      Embed dim   Heads   Blocks   FFN layer
  ------------------------- ----------- ------- -------- -----------
  ViT-S/14 (distilled)          384        6       12        MLP
  ViT-B/14 (distilled)          768       12       18        MLP
  ViT-L/14 (distilled)         1024       16       24        MLP
  ViT-L/14 (from scratch)      1024       16       24      SwiGLU
  ViT-g/14 (from scratch)      1536       24       40      SwiGLU

  :  **Architecture details of the ViT-S/B/L/g networks used in this
  work.** We use MLP feed-forward networks for distilled models, and
  SwiGLU  [@shazeer2020glu] when training from scratch.
:::

#### KoLeo regularization.

We apply the KoLeo regularizer with a weight of 0.1 between the class
tokens of the first global crop, for all samples within a GPU without
cross-communication for this step.

#### EMA update for the teacher.

The teacher is initialized with the same state as the student, and is an
exponential moving average of the student network, with a momentum value
in \[0.994, 1.0\] following a cosine schedule. It is updated at the end
of every training step.

## High-Resolution adaptation

We initialise the model with the pretrained weights then train it for
10k iterations with the same procedure as the original pretraining. All
the schedules are kept the same as in the original training, but
compressed to fit in 10k iterations. All the hyperparameters are kept
the same as in the first pretraining, except the base learning rate
which is reduced.

## Linear probing evaluation {#app:linearprobing}

For linear probing we define 3 evaluation parameters: the learning rate,
how many output layers we use, whether we concatenate the average-pooled
patch token features with the class token (or use only the class token).
We train our linear layer with SGD for 12500 iterations, using
random-resized-crop data augmentation, and perform the following grid
search:

-   learning rate in
    $\{0.0001, 0.0002, 0.0005, 0.001, 0.002, 0.005, 0.01, 0.02, 0.05, 0.1, 0.2, 0.3, 0.5 \}$

-   output layers in $\{1,4\}$

-   concatenate average-pooled tokens in $\{yes, no\}$

We then report the highest accuracy value obtained on the validation set
as is common practice. Note that this grid search is not expensive,
because at each iteration we perform inference on the backbone only
once, then feed the output to all linear classifiers (each performing a
single matrix multiplication).

# List of Datasets used {#app:benchmarks_list}

We show in Table
[\[tab:datasets_list\]](#tab:datasets_list){reference-type="ref"
reference="tab:datasets_list"} the list of benchmarks and datasets used
and their purposes.

::: tabular
p2.5cmcccp2cmp5cm & & & & Task & Citation\
ImageNet-1k & []{style="color: DecrRed"}& []{style="color: IncrGreen"}&
[]{style="color: IncrGreen"}& Classif. &  [@russakovsky2015imagenet]\
ImageNet-22k & []{style="color: IncrGreen"}&
[]{style="color: IncrGreen"}& []{style="color: DecrRed"}& &
 [@deng2009imagenet]\
ImageNet-V2 & []{style="color: DecrRed"}& []{style="color: DecrRed"}&
[]{style="color: IncrGreen"}& Classif. &  [@recht2019imagenet]\
ImageNet-ReaL & []{style="color: DecrRed"}& []{style="color: DecrRed"}&
[]{style="color: IncrGreen"}& Classif. &  [@beyer2020we]\
ImageNet-A & []{style="color: DecrRed"}& []{style="color: DecrRed"}&
[]{style="color: IncrGreen"}& Classif. &  [@hendrycks2021natural]\
ImageNet-C & []{style="color: DecrRed"}& []{style="color: DecrRed"}&
[]{style="color: IncrGreen"}& Classif. &  [@hendrycks2018benchmarking]\
ImageNet-R & []{style="color: DecrRed"}& []{style="color: DecrRed"}&
[]{style="color: IncrGreen"}& Classif. &  [@hendrycks2021many]\
ImageNet-Sk. & []{style="color: DecrRed"}& []{style="color: DecrRed"}&
[]{style="color: IncrGreen"}& Classif. &  [@wang2019learning]\
Food-101 & []{style="color: DecrRed"}& []{style="color: IncrGreen"}&
[]{style="color: IncrGreen"}& Classif. &  [@dataset-food101]\
CIFAR-10 & []{style="color: DecrRed"}& []{style="color: IncrGreen"}&
[]{style="color: IncrGreen"}& Classif. &  [@krizhevsky2009learning]\
CIFAR-100 & []{style="color: DecrRed"}& []{style="color: IncrGreen"}&
[]{style="color: IncrGreen"}& Classif. &  [@krizhevsky2009learning]\
SUN397 & []{style="color: DecrRed"}& []{style="color: IncrGreen"}&
[]{style="color: IncrGreen"}& Classif. &  [@dataset-sun397]\
StanfordCars & []{style="color: DecrRed"}& []{style="color: IncrGreen"}&
[]{style="color: IncrGreen"}& Classif. &  [@dataset-stanfordcars]\
FGVC-Aircraft & []{style="color: DecrRed"}&
[]{style="color: IncrGreen"}& []{style="color: IncrGreen"}& Classif. &
 [@dataset-aircraft]\
VOC 2007 & []{style="color: DecrRed"}& []{style="color: IncrGreen"}&
[]{style="color: IncrGreen"}& Classif. &  [@everingham2010pascal]\
DTD & []{style="color: DecrRed"}& []{style="color: IncrGreen"}&
[]{style="color: IncrGreen"}& Classif. &  [@dataset-dtd]\
Oxford Pets & []{style="color: DecrRed"}& []{style="color: IncrGreen"}&
[]{style="color: IncrGreen"}& Classif. &  [@dataset-pets]\
Caltech101 & []{style="color: DecrRed"}& []{style="color: IncrGreen"}&
[]{style="color: IncrGreen"}& Classif. &  [@dataset-caltech101]\
Flowers & []{style="color: DecrRed"}& []{style="color: IncrGreen"}&
[]{style="color: IncrGreen"}& Classif. &  [@nilsback2008automated]\
CUB200 & []{style="color: DecrRed"}& []{style="color: IncrGreen"}&
[]{style="color: IncrGreen"}& Classif. &  [@dataset-cub]\
iNaturalist 2018 & []{style="color: DecrRed"}&
[]{style="color: DecrRed"}& []{style="color: IncrGreen"}& Classif. &
 [@van2018inaturalist]\
iNaturalist 2021 & []{style="color: DecrRed"}&
[]{style="color: DecrRed"}& []{style="color: IncrGreen"}& Classif. &
 [@van2021benchmarking]\
Places-205 & []{style="color: DecrRed"}& []{style="color: DecrRed"}&
[]{style="color: IncrGreen"}& Classif. &  [@zhou2014learning]\
UCF101 & []{style="color: DecrRed"}& []{style="color: DecrRed"}&
[]{style="color: IncrGreen"}& Video &  [@soomro2012ucf101]\
Kinetics-400 & []{style="color: DecrRed"}& []{style="color: DecrRed"}&
[]{style="color: IncrGreen"}& Video &  [@kay2017kinetics]\
SSv2 & []{style="color: DecrRed"}& []{style="color: DecrRed"}&
[]{style="color: IncrGreen"}& Video &  [@goyal2017something]\
GLD v2 & []{style="color: IncrGreen"}& []{style="color: IncrGreen"}&
[]{style="color: DecrRed"}& &  [@weyand2020google]\
R-Paris & []{style="color: DecrRed"}& []{style="color: IncrGreen"}&
[]{style="color: IncrGreen"}& Retrieval &  [@radenovic2018revisiting]\
R-Oxford & []{style="color: DecrRed"}& []{style="color: IncrGreen"}&
[]{style="color: IncrGreen"}& Retrieval &  [@radenovic2018revisiting]\
Met & []{style="color: DecrRed"}& []{style="color: IncrGreen"}&
[]{style="color: IncrGreen"}& Retrieval &  [@ypsilantis2021met]\
Amstertime & []{style="color: DecrRed"}& []{style="color: IncrGreen"}&
[]{style="color: IncrGreen"}& Retrieval &  [@yildiz2022amstertime]\
ADE20k & []{style="color: DecrRed"}& []{style="color: IncrGreen"}&
[]{style="color: IncrGreen"}& Seg. &  [@zhou2017scene]\
Cityscapes & []{style="color: DecrRed"}& []{style="color: IncrGreen"}&
[]{style="color: IncrGreen"}& Seg. &  [@cordts2016cityscapes]\
VOC 2012 & []{style="color: DecrRed"}& []{style="color: IncrGreen"}&
[]{style="color: IncrGreen"}& Seg. &  [@everingham2010pascal]\
Mapillary SLS & []{style="color: IncrGreen"}&
[]{style="color: DecrRed"}& []{style="color: DecrRed"}& &
 [@warburg2020mapillary]\
NYU-Depth V2 & []{style="color: DecrRed"}& []{style="color: IncrGreen"}&
[]{style="color: IncrGreen"}& Depth &  [@silberman2012indoor]\
KITTI & []{style="color: DecrRed"}& []{style="color: IncrGreen"}&
[]{style="color: IncrGreen"}& Depth &  [@geiger2013vision]\
SUN-RGBD & []{style="color: DecrRed"}& []{style="color: IncrGreen"}&
[]{style="color: IncrGreen"}& Depth &  [@song2015sun]\
DollarStreet & []{style="color: DecrRed"}& []{style="color: DecrRed"}&
[]{style="color: IncrGreen"}& Fairness &  [@de2019does]\
Casual Conv. & []{style="color: DecrRed"}& []{style="color: DecrRed"}&
[]{style="color: IncrGreen"}& Fairness &  [@hazirbas2021towards]\
& & & & &
:::

[^1]: <https://github.com/facebookresearch/dinov2>

[^2]: <https://github.com/facebookresearch/xformers>

[^3]: For context, a full Boeing 777 return flight between London and
    New York corresponds to approximately 560 tCO$_2$eq.
