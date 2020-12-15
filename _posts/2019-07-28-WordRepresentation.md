---
layout: post
title: "Word Representation"
description: "frequently used word representation methods"
categories: [nlp]
tags: [word representation]
redirect_from:
  - /2019/07/28/
---

* Karmdown table of content
{:toc .toc}

# GloVe

## 理论

关于模型理论的讲解可以参考博客 [理解GloVe模型（+总结）](https://blog.csdn.net/u014665013/article/details/79642083)

不得不惊叹作者在损失函数中加入了exp()的操作，使复杂度从N<sup>3</sup>降到了N<sup>2</sup>实为惊人之举。但是从直觉上来讲，以下结论可以直接得到：如果单词i和单词j相关，P<sub>ij</sub>趋近于1。那么久绕开了原来的ratio<sub>i,j,k</sub>这一步。

另一个惊艳的点在于，处理log(P<sub>i,j</sub>)=v<sub>i</sub><sup>T</sup>v<sub>j</sub>的不对称性过程中，引入了残差项。再次惊为天人。

## train

因为我是希望把glove应用在医疗任务中，所以需要自己使用医疗预料进行预训练。这里讲一下自己预训练的过程。

### data

数据来自于以下几个公开数据集：

> 1. CCKS 2019医疗领域命名实体识别任务，1.3K语句
> 2. huangshi central hospital 病历中的主诉、现病史，各1.8w条。
> 补充： 还有一些其他可以找到的数据，但是我这里没用。可以参考博客https://blog.csdn.net/qq_27590277/article/details/108031071

分词过程推荐以下两款工具。
> 1. jieba，使用说明可见于[官方github](https://github.com/fxsjy/jieba/)
> 2. PKUSEG。使用说明见[官方github](https://github.com/lancopku/pkuseg-python)

pkuseg广为诟病的点在于他执行速度太慢了，不过如果可以预处理并进行存储的话，倒是不算啥问题。而且PKUSEG安装也很方便，不比jieba差，可以尝试一下。

之后将训练数据中的语句先使用分词工具分词后，用空格隔开存储。

![smiley](\assets\images\usedInBlogs\WordRepresentation\1.png)


### 源码与训练

在github上可以找到[斯坦福实现的源码](https://github.com/stanfordnlp/GloVe)，并且提供了一些英文预训练的词向量。clone下来。

之后修改demo.sh文件：

~~~
#!/bin/bash
set -e

# Makes programs, downloads sample data, trains a GloVe model, and then evaluates it.
# One optional argument can specify the language used for eval script: matlab, octave or [default] python

# make
# if [ ! -e text8 ]; then
#   if hash wget 2>/dev/null; then
#     wget http://mattmahoney.net/dc/text8.zip
#   else
#     curl -O http://mattmahoney.net/dc/text8.zip
#   fi
#   unzip text8.zip
#   rm text8.zip
# fi

#CORPUS=text8
CORPUS=data/medical_data
VOCAB_FILE=data/vocab.txt
COOCCURRENCE_FILE=cooccurrence.bin
COOCCURRENCE_SHUF_FILE=cooccurrence.shuf.bin
BUILDDIR=build
SAVE_FILE=vectors
VERBOSE=2
MEMORY=4.0
VOCAB_MIN_COUNT=5
VECTOR_SIZE=50
MAX_ITER=15
WINDOW_SIZE=15
BINARY=2
NUM_THREADS=8
X_MAX=10
if hash python 2>/dev/null; then
    PYTHON=python
else
    PYTHON=python3
fi

echo
echo "$ $BUILDDIR/vocab_count -min-count $VOCAB_MIN_COUNT -verbose $VERBOSE < $CORPUS > $VOCAB_FILE"
$BUILDDIR/vocab_count -min-count $VOCAB_MIN_COUNT -verbose $VERBOSE < $CORPUS > $VOCAB_FILE
echo "$ $BUILDDIR/cooccur -memory $MEMORY -vocab-file $VOCAB_FILE -verbose $VERBOSE -window-size $WINDOW_SIZE < $CORPUS > $COOCCURRENCE_FILE"
$BUILDDIR/cooccur -memory $MEMORY -vocab-file $VOCAB_FILE -verbose $VERBOSE -window-size $WINDOW_SIZE < $CORPUS > $COOCCURRENCE_FILE
echo "$ $BUILDDIR/shuffle -memory $MEMORY -verbose $VERBOSE < $COOCCURRENCE_FILE > $COOCCURRENCE_SHUF_FILE"
$BUILDDIR/shuffle -memory $MEMORY -verbose $VERBOSE < $COOCCURRENCE_FILE > $COOCCURRENCE_SHUF_FILE
echo "$ $BUILDDIR/glove -save-file $SAVE_FILE -threads $NUM_THREADS -input-file $COOCCURRENCE_SHUF_FILE -x-max $X_MAX -iter $MAX_ITER -vector-size $VECTOR_SIZE -binary $BINARY -vocab-file $VOCAB_FILE -verbose $VERBOSE"
$BUILDDIR/glove -save-file $SAVE_FILE -threads $NUM_THREADS -input-file $COOCCURRENCE_SHUF_FILE -x-max $X_MAX -iter $MAX_ITER -vector-size $VECTOR_SIZE -binary $BINARY -vocab-file $VOCAB_FILE -verbose $VERBOSE
if [ "$CORPUS" = 'text8' ]; then
   if [ "$1" = 'matlab' ]; then
       matlab -nodisplay -nodesktop -nojvm -nosplash < ./eval/matlab/read_and_evaluate.m 1>&2
   elif [ "$1" = 'octave' ]; then
       octave < ./eval/octave/read_and_evaluate_octave.m 1>&2
   else
       echo "$ $PYTHON eval/python/evaluate.py"
       $PYTHON eval/python/evaluate.py
   fi
fi
~~~~

之后执行

~~~
make
sh demo.sh
~~~

### 训练结果

结果存储在vectors.txt中，如图所示：

![smiley](\assets\images\usedInBlogs\WordRepresentation\2.png)

# LSA

# word2vec