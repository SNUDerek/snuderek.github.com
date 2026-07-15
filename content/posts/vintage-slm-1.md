---
title: "Vintage SLM 1: Introduction - Data and Pretraining"
date: 2026-07-15T17:22:00+09:00
draft: false
tags:
  - research
  - llms
categories:
  - vintage slm
---

## Why This Project

This is the first in a series about my attempt to train a small 'vintage' language model. I wanted to tackle this because in my day job, I was mostly working on agentic systems built on proprietary model backbones, so most of the work was harness construction, tool-use development, and agent evaluation. I did experiment with basic fine-tuning for style transfer using OpenAI's now-defunct fine-tuning API for GPT 3.5-turbo, and also conducted limited open-weight model LoRA fine-tuning using HuggingFace Deep Learning Containers on GCP, but I haven't had the opportunity to train a model from scratch.

I decided on training a ["vintage"](https://github.com/entanglr/awesome-vintage-llms) small language model for a number of reasons. First, I wanted to train on ethical data. While I originally wanted to use only public-domain data, I eventually allowed myself 19th-century datasets realsed under an explicit permissive license; I am not a lawyer and I believe that the ethical debate around LLM training data is complex, but I don't think the authors of any pre-1930's work are in a position to raise a complaint. Second, as an amateur history fan, the idea of a model representing the knowledge of a "snapshot in time" was appealing, and as an amateur [mechanistic interpretability](https://learnmechinterp.com) student, I thought that this knowledge cutoff may provide for interesting opportunities for MI study.

As for the "small" part, both training time efficiency and financial constraints were major influences, but I also wanted to target sizes that would reasonably be able to run on a consumer GPU (or even CPU) for local inference, so that others could use it as well. Since many others have also experimented with small language models, there is also a lot of wisdom available regarding best practices. Finally, I was curious how modern training regimens (pre-training, mid-training, SFT, preference alignment and verifiable reward training) would apply to a sub-1B scale model.

Check out the repo here: [mini-lm-training](https://github.com/SNUDerek/mini-lm-training)

## Pretraining Dataset Collection

Here I'll break down the sources I drew from to assemble the pretraining corpus, some key decisions I made along the way, and what the final training and validation data distribution looks like.

### Data Sources

**Project Gutenberg English Texts**

Gutenberg works are under the permissive [Project Gutenberg License](https://www.gutenberg.org/policy/license.html)

You can download a collection of all gutenberg.org files as txt from [this page](https://www.gutenberg.org/cache/epub/feeds/) (it's the ~10 GB 'txt-files.tar.zip' file), long with a csv of the entire catalog. Since the catalog does not list publication date, I instead filtered by author birth year; if any listed author had a birth year before 1875, the document was kept. This did result in dropping works where the author's birth year was not listed or was not formatted in ####-####. I then filtered by English language texts. String matching was used to strip the Gutenberg preamble and footer, and heuristics were used to remove table of contents, ascii-formatted tables and figures, and glossaries, indices and footnotes lists. 

**British Library Books (1800-1899)**

British Library Books dataset is released under a CC0-1.0 license. 

The other main source of the pretraining dataset, available on [Huggingface Datasets](https://huggingface.co/datasets/TheBritishLibrary/blbooks/blob/main/blbooks.py). The entire dataset spans 1510 to 1899, but because the collection is scanned from images of the books with OCR, I was worried that the quality of early texts' transcriptions would be low, so I set the lower cutoff at 1800; the bulk of the collection lies in this time period anyways. The Huggingface dataset has a py file for downloading the archives by decade, but I created a shell script to download the files I wanted directly. 

**PleIAs Post-OCR Correction Dataset**

The pleIAs post-OCR Correction dataset is released under a CC0-1.0 license. 

This is a multi-lingual dataset of newspaper and monograph texts with original OCR and corrected texts. The English portion comes from the Chronicling America collection. I had originally wanted to use the [American Stories](https://huggingface.co/datasets/dell-research-harvard/AmericanStories) dataset, but unfortunately after visual inspection, I found the text quality to be too low. This corrected dataset, while smaller, is of much higher quality. There were some sections of repeated text due to the LLM; I visually inspected all final files and manually removed long sections of repeated texts.

**Old Bailey Corpus v2**

The Old Bailey dataset is available under the [CC BY-NC-SA 4.0](http://creativecommons.org/licenses/by-nc-sa/4.0/) license.

A corpus of trial transcripts from London's criminal court, ranging from 1674-1913, available to download [here](https://fedora.clarin-d.uni-saarland.de/oldbailey/downloads.html). The data comes as XML files with annotation tags; I created a script to parse the text content and save it as plain-text. 

**Nineteen Century Serials Edition (NCSE v2.0)**

The NCSE v2.0 dataset is available under the [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) license.

The NCSE v2.0 dataset contains OCR-transcribed English newspapers from teh 19th century. It's available to download [here](https://rdr.ucl.ac.uk/articles/dataset/NCSE_v2_0_A_Dataset_of_OCR-Processed_19th_Century_English_Newspapers/28381610).

### Final Dataset Distribution

After corpus-specific pre-preprocessing and some limited manual cleaning of the smaller corpora (e.g. to remove the LLM repetitions in the PleIAs dataset), the data was split into a 99:1 training-validation split.

**Training Data Split**

| corpus     | size on disk | file count |
|------------|--------------|------------|
| Gutenberg  | 15 GB        | 37427      |
| BL Books   | 15 GB        | 26490      |
| PleIAs     | 1.3 GB       | 30920      |
| Old Bailey | 150 MB       | 632        |
| NCSE v2.0  | 5.5 MB       | 35         |
| **TOTAL**  | **31 GB**    | **95504**  |


## Tokenizer

I fit a custom byte-pair encoding Huggingface [`tokenizers`](https://github.com/huggingface/tokenizers) to the training data. While many "quickstart" tutorials recommend using a pretrained tokenizer, usually for both pretrained model compatibility and ease of use, I felt a custom tokenizer was a better fit. First of all, I wanted to save parameters by shrinking the vocabulary. Because the data is English only, and the domain is rather constrained, I felt that a Llama-2 size of around 32,000 was more reasonable than Llama-3's 128,256 or Qwen3's 151,646 token sizes, which to my understanding are meant to better capture multilingual data. At the small size we're targeting, even with weight tying, the token embeddings can occupy a substantial fraction of the model's total parameters. Secondly, *because* my data is from a different distribution than general modern text, I wanted to fit a custom vocabulary that fit the time period from which my training data is drawn.

I did follow Llama-3 in that I did not use a BOS (beginning-of-sentence) token, only an EOS token that separates documents during pretraining. However I broke with Llama-3 in adding an initial space, so text-initial tokens and intermediate tokens appear similar to the model. While this may impact the model's performance on initial tokens, I felt that this space processing would allow for a better representation of the token vocabulary given the smaller vocabulary size.

One "gotcha" I faced was that when training the initial `Tokenizer`, was that you need to pass the tokenizer's `pre_tokenizer.alphabet()` to the Trainer's `initial_alphabet` so that all possible bytes are used to initialize the vocabulary. I thought that this would be done automatically by the trainer, so I was initially confused when testing the tokenizer on non-English texts and was receiving `unk` outputs due to the initial vocabulary being set only to the bytes in the training data (I suppose this is a design choice, but I believe one of the *benefits* of BPE is allowing models better handling of OOV terms, so I didn't want to worry about `unk` tokens)

## Model Pretraining

### Training Code

I wanted to make a small note here. Originally, my intention was to use HuggingFace's [`nanotron`](https://github.com/huggingface/nanotron), which was used to train their SmolLM3, to train; if you check the github repo, you can see my still-wip code for this. However, I was having trouble getting it to work due to outdated dependencies, and since I was not doing distributed training anyways, I went with the simpler option of just using basic Huggingface `transformers` instead. Since I wanted to use the Llama 3 architecture as a base, with single-GPU training to start, I felt that this was satisfactory, though I want to revisit the `nanotron` effort later.  

### Model Dimensions

I was targeting a model around 350 million parameters, around the size of GPT-2 Medium. I wanted to use a Llama-3 style architecture (SwiGLU, RoPE, RMSNorm, grouped-query attention), and at this size, I decided weight tying was best - it seems that rule of thumb is that below 500M parameters or so, the parameter savings with weight tying outweigh the downsides (*though I'd like to look into this further, maybe in another post*).

To be honest, I sort of set the parameter budget, and then sort of winged it, drawing from various other small models for inspiration in setting each parameter. Popular sub-1B models seem to have around 18 to 24 layers (see Gemma-3 270m, GPT-2 Medium) so 20 seems okay; hidden size to MLP ratios seem to be around 2.5 ~ 4x, so 1024 and 4096 are nice "round" numbers. For grouped-query attention, I set the attention heads to 16 (following GPT-2 Medium, again) and then set key heads to 4 in a 4:1 ratio.

**340M Model Configuration**

| component    | value  |
|--------------|--------|
| vocabulary   | 32000  |
| hidden size  | 1024   |
| mlp size     | 4096   |
| layers       | 20     |
| query heads  | 16     |
| kv heads     | 4      |
| weight tying | yes    |
| **total params** | **336.9M** |

 ### Pretraining Data Packing

The pretraining data was pre-tokenized and pre-packed into training and validation Hugginface `Dataset`s. A context window of 4096 was used, so each row consists of 4097 tokens' no padding is added; instead documents are separated with EOS token. A streaming file loader with token buffer was used to efficiently process the entire dataset. The total processed training dataset clocks in at 28 GB, with 1,828,842 samples x 4096 = 7,490,936,832 tokens seen in training. This approximately fits the Chinchilla scaling law of roughly 20 tokens per parameter, as 337M * 20 = 6.7B ([*Training Compute-Optimal Large Language Models*](https://arxiv.org/pdf/2203.15556), Table 3). ...then I sort of ruined that by setting the training epochs to `2` on the 340M run. Why did I do that? Honestly, not sure myself. I guess I was worried from reading the [**awesome-vintage-llms**](https://github.com/entanglr/awesome-vintage-llms) page, especially the part about "just how data-hungry 'public-domain-only' LLMs are" and deciding that the (*relative*) cleanness of the dataset and its relatively small size, combined with the model's small size and thus fewer parameters to "memorize" the entire dataset, may make it worth it to risk a second pass over the data.

### Setting up on vast.ai

I initially tried to use Runpod, as I had some credits there, even spending the time to transfer my data to a network volume atached to a CPU instance, which took hours due to slow network transfer speeds. However, once I got the data transferred over, I couldn't launch an instance - despite showing "low" availability, apparently that meant "no" availability. So I checked vast.ai and noticed that not only were their 'secure' server prices better (RTX 5090 was cheaper than Runpod's RTX 4090), I could also get an instance in Korea. Even though there are no CPU instances of cheaply loading data to a network drive, in-country data transfer was incredibly fast; I sent the entire packed dataset in a couple minutes (capping at around 400 Mbps, hurray for gigabit internet). And sorry, reading it back this sounds like an ad for vast.ai, which it's not (I'm purposely not linking to their service here), I am just generally pleased with their offering so far.

Anyways, at context length 4096, I could fit a batch size of 12 into 1x RTX 5090 with bfloat16 precision and [`sdpa`](https://huggingface.co/docs/transformers/attention_interface#sdpa-context-manager) attention mode and gradient checkpointing. Since I was seeing recommendations of effective batch size between 64 to 128 with gradient accumulation, I set the gradient accumulation for 10 minibatches, or effective batch size of 120. With these settings, I'm using `29382MiB / 32607MiB` VRAM on the RTX 5090, with an iteration time of 13.26 seconds (for 10x minibatches + optimizer step), so aobut 2.7 days per epoch, or 5.4 days for both epochs.

Since I want to examine intermediate checkpoints for mechanistic interpretability work, I am currently checkpointing every 200 steps, with each checkpoint clocking in at ~3.8 GB. I setup a `cron` job on my home server to attempt to `rsync` all new checkpoints and tensorboard data from the vast.ai pod to my server. So far this has been working well, though probably only effective thanks to the fast transfer speeds.

## Sample Text Completions

Here are some sample text completions generated from checkpoint 5400, after about 20 hours of training, and training on about a third of the corpus.

**settings**: temperature of 0.80, top-p of 0.90, top-k of 100, max generation tokens: 200.

The model occupies about 1100 MiB of VRAM while inferencing.

Note: this model is trained on data from another time, with potentially outdated beliefs and attitudes about race and culture, gender, politics, sciences, and many other potentially sensitive topics. I don't think the samples below have anything abhorrent in them, but they do make reference to wealth and poverty, indigenous peoples, and gender roles.

**It is a truth universally acknowledged, that a single man in possession of a good fortune, must be in want of**

> It is a truth universally acknowledged, that a single man in possession of a good fortune, must be in want of the necessaries of life." The writer of this history adds, that " there are several families who do not want even these necessaries. The reason why it is so common for all the people who are rich to have an allowance of beef, butter, or poultry, is because their wants are few, and they do not know how to buy. The whole family is not of the same rank in life as the farmer, and it is not uncommon for the farmer to be rich when he can afford it, and spend as much money as he can. But they are very seldom troubled with poverty, because it is very seldom that the farmer can afford to have any thing else, and they rarely have any thing else besides that which they earn by their labour." A correspondent, who, in a recent paper, has published a graphic account of the manner in which the people of the United States have become rich, writes, "The most prevalent idea prevails among the people in every part of the world, that they

> It is a truth universally acknowledged, that a single man in possession of a good fortune, must be in want of a sufficient number of persons, and of a well-to-do population, to support his family, and maintain himself, during a year, if he can afford it. It is, then, a positive fact, that a man, in possession of a fortune, would be able to support a family of the best quality, in a condition of poverty. A man, who, if he had an enemy, would be able to subsist upon his wife and children, would be able to support a family of the most respectable families in the kingdom, and of the greatest respectability, and the most respectable families in the kingdom. It is no more a maxim of nature, that a man, having a family, can have no property, or the means of support, than what he might have had from his children, and from a stranger, if he had not possessed a fortune. And that is the case of a man, who, having an estate, can have no other means of support than those

> It is a truth universally acknowledged, that a single man in possession of a good fortune, must be in want of a large portion of it.
Having now briefly stated my case, I will briefly inform you of it.
I have no doubt of the fact, that when a man has a large fortune, he often spends it upon the purchase of property; and that it is rarely, if ever, that any one man, who has property, will do so. When a man is rich, and has property of his own, he sometimes spends his money in this way, for the sake of the money, or of the interest he has derived from it; and by these means he becomes rich, and the greater the sum the greater the amount. This is, in fact, the case in all cases, where the man has no capital, and is therefore very poor. But if a man has large capital, he is more frequently ruined than the man who has a large capital. For the same reason, I have known a man who, by means of a large capital, acquired a considerable fortune.

(from *Pride and Prejudice*)

**You will rejoice to hear that no disaster has accompanied the commencement of an enterprise which you have regarded with such evil forebodings.**

> You will rejoice to hear that no disaster has accompanied the commencement of an enterprise which you have regarded with such evil forebodings. The last year or two of your life has been in imminent peril. I have felt it my duty to tell you this, that the whole affair may be explained in this way. You have not been informed by me of the causes which have led to the disaster which has happened. I have not yet seen the facts. It has been impossible for me to state the facts. I must have them. If they are true, and the consequences are the same, I must have the responsibility of your life.
"And now, sir, what shall I say? Do you not know what I have often felt? that, if the consequences of the failure of your plans were known, you would not have the courage to follow them? And if, after having, as you say, made your plans, you had the courage to put them in execution, you would not have the courage to face a man who has been insulted by the enemy, who has been the aggressor of his country, who

> You will rejoice to hear that no disaster has accompanied the commencement of an enterprise which you have regarded with such evil forebodings. You have lost the confidence of your people, and you are determined to be their friend. In a word, you will not abandon the hope of conquering the rebellious tribes of the Sioux and the Shawnees. I hope that the great Father of the West will have you at his mercy, and that you will not be driven from your stronghold by any of his people.
“In the words of the prophet, ‘The mountains are in our hands, and the hills are our heritage,’ you will find yourself in a condition to begin a new struggle. You will not have to fight against the powers of the sky, but you will conquer. You will be able to conquer all the tribes and tribes of the nation. You will be able to conquer the tribes, and you will have to fight with all the tribes of the Sioux and the Shawnees, and all the tribes of the Shawnees and the Shawnees.
“It is for you to decide which will be the greatest

> You will rejoice to hear that no disaster has accompanied the commencement of an enterprise which you have regarded with such evil forebodings. What, then, will it be to me to go to my father at once, and tell him that the whole country is in arms to aid him against the enemy? A good thing indeed, that the people are not to know that he has lost his daughter! No, not a single man in the land is willing to remain at home, and we are still looking forward to the time when he may be able to go to the help of the king. Let us hope so, and not, as many suppose, from the fear of the consequences of our own folly, let us, on the other hand, expect to hear from him, and he will be sure to let us know when the time is ripe.
"But, meanwhile, if there be anything that is to be done at home, I am of opinion that the people of England will not fail to do it. I do not, at present, think that there is anything that the king, or any of the people of

(from *Frankenstein*)

**In my younger and more vulnerable years my father gave me some advice that**

> In my younger and more vulnerable years my father gave me some advice that would have made a man of me. It was not a subject on which I had any concern, and I did not like to tell him to have made a fool of himself; and when I had heard this, and had seen the state of the boy, I could not tell him why I had not done the thing.
One day, about a year after I was married, a gentleman called to see me, and when I saw him, I said, "Mr. Gooch, you are the same man that came to see me about a month ago. I am an old man, but I have never seen you, and I have not seen you since."
"I knew you to be a gentleman," said he; "and I know you to be a very worthy young man. I am proud of your son, but I wish to give my son some of my company, that he may be better able to judge what I have done for you, and I will do

> In my younger and more vulnerable years my father gave me some advice that I should not be tempted to do so.
"I have known some who would have been proud to take their sons to the universities, and would have been glad to have seen them at the universities. I cannot see any reason why, unless it be that my mother, who has lived so happily with the family, should have been so careful to bring them up to the mark, that they would be likely to have taken to the pursuit of virtue.
"As for myself, I know not what to do. I am resolved to keep my thoughts to myself. My mother's advice and the doctor's, my father's, and I, are both at stake.
"If my mother should see me in the race for the prize she would have to give me up to her. I should have to be ready to meet her, and I should have to see her to decide.
"My mother would be very likely to find her way into my father's house by the way of

> In my younger and more vulnerable years my father gave me some advice that he did not want. He said he was afraid that I might have to take a dislike to him before the doctor could come. He wanted to have me go to the sea, but I could not take him. I knew he would not object to my staying with him, and I wanted to get him to come to my house, as I had to be there to see the doctor before he came. He came and went, and I was sorry for him, and went to see him, and took an interest in him. He was my best friend. I found the doctor and his wife very kind and good. I did not know what to do about him, and I did not think it safe to say that I was afraid of him. I had to be careful not to betray him. I told him that he had been in my house, but I did not think of him. I knew he was in a very dangerous place. I was sure that he would come to harm. I

(from *The Great Gatsby*)