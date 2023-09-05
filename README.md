# OnPrem.LLM

<!-- WARNING: THIS FILE WAS AUTOGENERATED! DO NOT EDIT! -->

> A tool for running large language models on-premises using non-public
> data

**OnPrem.LLM** is a simple Python package that makes it easier to run
large language models (LLMs) on non-public or sensitive data and on
machines with no internet connectivity (e.g., behind corporate
firewalls). Inspired by the
[privateGPT](https://github.com/imartinez/privateGPT) GitHub repo and
Simon Willison’s [LLM](https://pypi.org/project/llm/) command-line
utility, **OnPrem.LLM** is designed to help integrate local LLMs into
practical applications.

## Install

Once [installing PyTorch](https://pytorch.org/get-started/locally/), you
can install **OnPrem.LLM** with:

``` sh
pip install onprem
```

For fast GPU-accelerated inference, see additional instructions below.

## How to use

### Setup

``` python
import os.path
from onprem import LLM

llm = LLM()
```

### Send Prompts to the LLM to Solve Problems

This is an example of few-shot prompting, where we provide an example of
what we want the LLM to do.

``` python
prompt = """Extract the names of people in the supplied sentences. Here is an example:
Sentence: James Gandolfini and Paul Newman were great actors.
People:
James Gandolfini, Paul Newman
Sentence:
I like Cillian Murphy's acting. Florence Pugh is great, too.
People:"""

saved_output = llm.prompt(prompt)
```


    Cillian Murphy, Florence Pugh

### Talk to Your Documents

Answers are generated from the content of your documents.

#### Step 1: Ingest the Documents into a Vector Database

``` python
llm.ingest('./sample_data')
```

    2023-09-03 16:30:54.459509: I tensorflow/core/platform/cpu_feature_guard.cc:193] This TensorFlow binary is optimized with oneAPI Deep Neural Network Library (oneDNN) to use the following CPU instructions in performance-critical operations:  SSE4.1 SSE4.2 AVX AVX2 FMA
    To enable them in other operations, rebuild TensorFlow with the appropriate compiler flags.
    Loading new documents: 100%|██████████████████████| 2/2 [00:00<00:00, 17.16it/s]

    Creating new vectorstore
    Loading documents from ./sample_data
    Loaded 11 new documents from ./sample_data
    Split into 62 chunks of text (max. 500 tokens each)
    Creating embeddings. May take some minutes...
    Ingestion complete! You can now query your documents using the LLM.ask method

#### Step 2: Answer Questions About the Documents

``` python
question = """What is  ktrain?""" 
answer, docs = llm.ask(question)
print('\n\nReferences:\n\n')
for i, document in enumerate(docs):
    print(f"\n{i+1}.> " + document.metadata["source"] + ":")
    print(document.page_content)
```

     Ktrain is a low-code machine learning library designed to augment human
    engineers in the machine learning workow by automating or semi-automating various
    aspects of model training, tuning, and application. Through its use, domain experts can
    leverage their expertise while still benefiting from the power of machine learning techniques.

    References:



    1.> ./sample_data/ktrain_paper.pdf:
    lection (He et al., 2019). By contrast, ktrain places less emphasis on this aspect of au-
    tomation and instead focuses on either partially or fully automating other aspects of the
    machine learning (ML) workﬂow. For these reasons, ktrain is less of a traditional Au-
    2

    2.> ./sample_data/ktrain_paper.pdf:
    possible, ktrain automates (either algorithmically or through setting well-performing de-
    faults), but also allows users to make choices that best ﬁt their unique application require-
    ments. In this way, ktrain uses automation to augment and complement human engineers
    rather than attempting to entirely replace them. In doing so, the strengths of both are
    better exploited. Following inspiration from a blog post1 by Rachel Thomas of fast.ai

    3.> ./sample_data/ktrain_paper.pdf:
    with custom models and data formats, as well.
    Inspired by other low-code (and no-
    code) open-source ML libraries such as fastai (Howard and Gugger, 2020) and ludwig
    (Molino et al., 2019), ktrain is intended to help further democratize machine learning by
    enabling beginners and domain experts with minimal programming or data science experi-
    4. http://archive.ics.uci.edu/ml/datasets/Twenty+Newsgroups
    6

    4.> ./sample_data/ktrain_paper.pdf:
    ktrain: A Low-Code Library for Augmented Machine Learning
    toML platform and more of what might be called a “low-code” ML platform. Through
    automation or semi-automation, ktrain facilitates the full machine learning workﬂow from
    curating and preprocessing inputs (i.e., ground-truth-labeled training data) to training,
    tuning, troubleshooting, and applying models. In this way, ktrain is well-suited for domain
    experts who may have less experience with machine learning and software coding. Where

### Speeding Up Inference Using a GPU

The above example employed the use of a CPU.  
If you have a GPU (even an older one with less VRAM), you can speed up
responses.

#### Step 1: Install `llama-cpp-python` with CUBLAS support

``` shell
CMAKE_ARGS="-DLLAMA_CUBLAS=on" FORCE_CMAKE=1 pip install --upgrade --force-reinstall llama-cpp-python==0.1.69 --no-cache-dir
```

It is important to use the specific version shown above due to library
incompatibilities.

#### Step 2: Use the `n_gpu_layers` argument with [`LLM`](https://amaiya.github.io/onprem/core.html#llm)

``` python
llm = LLM(model_name=os.path.basename(url), n_gpu_layers=128)
```

With the steps above, calls to methods like `llm.prompt` will offload
computation to your GPU and speed up responses from the LLM.
