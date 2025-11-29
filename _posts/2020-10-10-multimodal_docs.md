---
title: 'Multimodal Document Classification'
date: 2020-10-10
permalink: /posts/multimodal-document-classification
tags:
  - natural language processing
  - image processing
  - data science
---
The automatic classification of documents remains an important and only partially solved information management problem within the upstream oil and gas industry. Companies in this sector have typically amassed vast repositories of unstructured data measuring in the range of tens to thousands of terabytes. [An estimated 80% of all data within the upstream oil and gas industry is stored within large unstructured document repositories](https://www.cgg.com/sites/default/files/2020-11/cggv_0000029162.pdf).

Retrieving relevant documents from these large unstructured data repositories is a major challenge for the industry, with some reports suggesting that [geoscientists and engineers spend over 50% of their time searching for data](https://www.sciencedirect.com/science/article/pii/S2405656118301421). Locating specific data such as porosity and permeability logs, well trajectories and composite wireline logs can be challenging.

Many of the document types found in such repositories are vital to informing exploration and risk management decisions, as well as to ensuring regulatory compliance. If crucial data is omitted from exploration models due to difficulties in its retrieval, this can lead to costly dry exploration wells or jeopardise the safety of a company's operations.

## Document Classification
Automated document classification algorithms aid information retrieval from these repositories by automatically identifying document types. These classifiers act in a similar manner to static search queries, which when combined with key word search, allow results to be refined to only include the data types a user is seeking to retrieve. The predicted classifications when used more broadly can also assist in providing data management oversight of the data within these repositories.

Recently, supervised machine learning has been widely used to create these algorithmic classifiers. With the machine learning algorithm learning a document classification prediction function from a pre-labelled corpus of representative documents. Historically these classifiers can learn document classifications at a granular level based on a documents text content or at a higher level based on image representations of their pages. 

## Multi-Modal Approach
The majority of documents in the upstream oil and gas domain are multimodal, meaning they rely on multiple channels to communicate information to the reader, [relying on both textual and visual modalities for communication](https://dl.acm.org/doi/10.1109/TPAMI.2018.2798607). For example most documents contain free text, often alongside text stored in tables, captions and figures. This text gives documents their semantic meaning and in many cases conveys the majority of the information.

However visual features such as a documents layout, colour distribution, text formatting, charts and image content provide supplementary meaning alongside text based features. In some cases visual features will convey the majority of the information, examples of exploration documents that primarily communicate information visually include sample images, maps, technical diagrams, seismic sections and wireline logs.

<p align="center">
  <img src="https://raw.githubusercontent.com/justinbt1/Multimodal-Document-Classification/refs/heads/main/report/media/page_samples.png" />
  <br>Figure 1: Sample NDR Page Images
</p>

Contemporary uni-modal document classification models used in the industry often rely on features from a single modality, either textual or visual. However, it is difficult to build a robust high performance uni-modal classifier over such a complex multi-modal dataset. For example, document classifiers making use of visual features extracted from page images may underperform due to the high inter-class similarity or intra-class variance of some document page images.

Document classifiers that rely on features from only the text modality tend to outperform image based classifiers at classifying oil and gas documents at a granular level. However these classifiers only perform well when enough text data is present in a document, some documents in the corpus such as diagrams and sample images do not contain any text making them impossible to classify using a text based approach alone. Other documents contain text that may be repeated across multiple document types. For example an oil well name and location being the only text data within visually different images such as between a map and a technical schematic, making them semantically similar but visually distinct.

This blog post explores using a multi-modal approach to oil and gas document classification, combining both text and visual modalities to create a more robust classifier for oil and gas exploration and production documents. With the hypothesis that a multi-modal classification approach combining text and visual feature input streams, will outperform a classifier trained on features from a single modality such as text or visual features.


