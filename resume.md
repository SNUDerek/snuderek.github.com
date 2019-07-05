# Derek Hommel

Machine Learning/Natural Language Processing Researcher

master of computational linguistics, SNU 2018

## contact

currently living in Seoul, South Korea. 

please contact me via email at [derekshommel at gmail.com](mailto:derekshommel@gmail.com) or through [LinkedIn](https://www.linkedin.com/in/derek-hommel-4a646869/) 

message me from Korea at 010 - 9482 - four two zero eight

## technical skills

- languages : `python`
- deep learning libraries : `tensorflow`, `keras`, `pytorch`
- engineering : `amazon aws`, `docker`, `kafka`, `tensorflow-serving`

## professional projects

listed in chronological order of participation, earliest to latest

**retrieval-based dialog assistant and conversation classifier**  

clients: korean e-commerce startup, international bank

tools: `pandas`, `sklearn`, `word2vec`, `sent2vec`, `keras`
- data analysis of 1M+ historical chats with unsupervised clustering methods
- ML models for classifying at sentence and dialog level (dialog level: 83 F1)
- response recommendation using cosine similarity and statistical speech-act transition modeling
- worked with technical advisors, engineering team to guide project development after departure of ML team lead
- mentored company interns and advised them on project sub-tasks
- helped port models & system from Korean to English for second client in new domain

**named entity recognition/joint SLU system**  

client: korean e-commerce company (with Seoul Innovation Challenge Grant)

tools: `keras`
- researched and implemented a flexible neural NER/joint-SLU tool for English and Korean
- led a team of two data analyst interns and two junior ML software engineers to complete project
- worked on data processing, data bootstrapping; researched input methods (tokenization, subword labeling), joint models
- korean grocery shopping domain entity (product, brand, size/type, count) detection (recall: 99.4)
- english travel domain using ATIS corpus (intent accuracy: 96.3, slot F1: 93.71) 

**limited-domain neural machine translation**  

client: joint venture with VC firm

tools: `keras`, `pytorch`
- supervised one junior ML software engineer (duties: preliminary research, webcrawling, deployment)
- worked on data bootstrapping (webcrawling) and processing; data input methods including `wordpiece`
- researched `seq2seq` translation methods, transfer learning for low-resource translations
- implemented custom `keras` network for product title and composition translations
- developed `open-NMT-py` fork for product description translations (BLEU: 28.64, cf. Google NMT: 2.05)

**NLU model serving for large-scale deployment**

client: korean e-commerce company

tools: `keras`, `tensorflow-serving`, `kafka`, `sklearn`, `cherrypy`, `ws4py`
- created CLI-based model trainer and exporter to train and save `tensorflow` models for `tensorflow-serving`
- created and deployed lightweight REST-based model server for `sklearn`-trained models
- configured and deployed Dockerized `tensorflow-serving` model server
- created and deployed multiprocessed data preprocessing service that communicated with above services over kafka, websocket and REST API
- worked with team of researchers and engineers to integrate above components into complete voice-powered navigation and search service 

## personal projects

**[MLPy](https://github.com/SNUDerek/MLPy)**  
[ongoing] education-focused implementation of machine learning algorithms in `python+numpy` with detailed comments

**[Magic The Gathering Card Generation with LSTM Language Modeling](https://github.com/SNUDerek/mtgcardgenerator)**  
[ongoing] a fun side-project for generating MtG cards using neural networks.  
see some cards at [my MtG Cardsmith page](https://mtgcardsmith.com/user/dsh9470/cards)

**[Named Entity Recognition with Bidirectional LSTM-CRF in `Keras`](https://github.com/SNUDerek/NER_bLSTM-CRF)**  
a `keras` implementation of word-level bi-LSTM-CRF network for entity extraction

**[Multipurpose LSTM_CRF Network with Attention for NER & Intent Detection](https://github.com/SNUDerek/multiLSTM)**  
an extension of the above project for a unified NER and intent detection network

## work experience

**Principal NLP Researcher** (March 2017 – Present)  
*[Atlas Labs](http://atlaslabs.ai), Seoul, Korea*  
assisting in researching and developing AI & NLP technologies for dialog assistant software.

**Guest English Teacher, English Program in Korea (EPIK)** (2012 – 2014)  
*Ohyun High School, Jeju Girl’s High School & Jeju Girl’s Commercial High School, Jeju City, South Korea*  
developed syllabus and curriculum, and taught weekly conversational English classes to high school students.  
volunteered to lead student-driven extracurricular activities including persuasive writing & magazine publishing club, debate club, and programming/game development club. 

**Fulbright ETA Orientation Coordinator** (2010)  
*Jungwon University, Goesan-gun, South Korea*  
conducted intensive seven-week orientation and training program covering Korean educational system, teaching methodology, and cultural adjustment. primary liason for maintaining site logistics with university.

**Fulbright English Teaching Assistant (ETA)** (2009 – 2012)  
*Aphae High School, Jeollanamdo, South Korea*  
*Daejeon Yongsan High School, Daejeon, South Korea*  
developed syllabus and curriculum, and taught weekly conversational English classes to high school students.

## education

**Seoul National University** (2015 – 2018)  
*Master's in Computational Linguistics*  
thesis: [Data-Driven Input Feature Augmentation for Named Entity Recognition](abstract.md)

**University of Rochester** (2006 – 2009)  
*Bachelor of Arts* magna cum laude *in Japanese Language in Culture*  
with minor in Linguistics

**Rochester Institute of Technology** (2004 – 2005)  
*Software Engineering*
on the RIT Computing Medal scholarship

## awards & honors

**Naver AI Hackathon 2018**  
*Round 3/Finalist with Team Atlas*

**NIIED Korean Government Scholarship Program Graduate Scholarship** (2014)  
*Seoul National University – Linguistics*

**Test of Proficiency in Korean (TOPIK)** (2014)  
*Level 4 Proficiency*

**Teaching English as a Foreign Language (TEFL)** (2012)  
*100-hour Certificate*

**Fulbright Scholar** (2009)  
*English Teaching Assistant grant to South Korea*
