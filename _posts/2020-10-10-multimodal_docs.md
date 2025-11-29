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

## Data Pre-processing
The document corpus for this project comes from the UK National Data Repository (NDR), an online repository maintained by the UK Oil and Gas Authority for the storage of petroleum related information and samples. The NDR contains hundreds of thousands of documents, representing 65 document types defined by labels known as CS8 Codes. 

For the purposes of our experiment a document corpus was created by taking a random sample of approximately 1000 documents from each of 6 key document classes present in the NDR, with each document's provided CS8 Code being used as a classification label. The document classes selected for inclusion in this dataset were; geological end of well reports (geol_geow), geological sedimentary reports (geol_sed), general geophysical reports (gphys_gen), well log summaries (log_sum), pre-site reports (pre_site) and vertical seismic profile files (vsp_file).

### Feature Extraction
As this is a new dataset, the text and page image features needed to be extracted from each document in the NDR corpus, for use as training, validation and test data for evaluating each of the classification models. The raw documents were downloaded manually via the free to access NDR website where they are available under an open data license and processed using a feature extraction pipeline.

Unfortunately features were not successfully extracted from all documents, some older Microsoft Office documents could not be processed, while others were corrupt or had non-standard file formats. Several documents also did not contain any text content and therefore lack a text feature set.

### Text Pre-Processing
The text feature dataset consists of a vector of integers T for each document representing the first 2,000 informative terms following text extraction and pre-processing. To create each vector, each document's text first had to be extracted. Text extraction from ascii format files was achieved using native Python. For Microsoft Office files and PDF files with an embedded searchable text layer text was extracted using Apache Tika Server. For scanned PDF files without an embedded text layer and image format files, text was extracted using the Tesseract OCR Engine.

```python
def start_tika_server(tika_path):
    command = f'java -cp "{tika_path}" org.apache.tika.server.TikaServerCli ' \
    '--port 80 --host 127.0.0.1'
    tika_server = subprocess.Popen(
        command, stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL
    )

    try:
        requests.get('http://127.0.0.1:80/tika')
    except requests.exceptions.ConnectionError as e:
        raise SystemExit(f'Unable to connect to Tika Server. {e}')

    return tika_server
  
def extract_text(file):
    tika_response = requests.put(
        url=f'http://127.0.0.1:80/tika',
        data=file,
        headers={
            'X-Tika-PDFOcrStrategy': 'no_ocr',
            'X-Tika-OCRLanguage': 'eng',
            'X-Tika-OCRTimeout': '1500',
            'Accept': 'text/plain'
        },
        timeout=1500
    )

    tika_response = {
        'content': tika_response.text,
        'status': tika_response.status_code
    }

    return tika_response
```

The raw text extracted from each document was case folded to lower case and tokenized using the pre-trained Word Punkt tokenizer available in the NLTK library, into a sequence of individual tokens. Any tokens with low semantic value were then dropped from the sequence, including non-word tokens such as numbers and punctuation, as well as frequently occurring stopwords. Each token was then lemmatized to it's base form using the Word Net lemmatizer in NLTK, this reduces dimensionality while preserving the semantic meaning of each token term.

```python
def text_processing(text):
    lemmatizer = nltk.stem.WordNetLemmatizer()
    text = text.casefold()
    text = text.translate(text.maketrans({'\'': None, '-': ' '}))
    text = nltk.word_tokenize(text)
    clean_tokens = []

    for token in text:
        if token in nltk.corpus.stopwords.words('english'):
            continue
        if not token.isalpha():
            continue
        if len(token) < 2:
            continue

        token = lemmatizer.lemmatize(token)
        clean_tokens.append(token)

    clean_string = ' '.join(clean_tokens)

    return clean_string
```

The processed tokens were then converted to integers, this was achieved by creating a vocabulary set V containing all unique tokens extracted from each document and mapping each to a unique integer value v. The first 2,000 processed tokens for each document are then mapped to their corresponding v integer value in V to create a vector of integers T for each document. Any vectors with less than 2,000 integers are padded with 0 so that \|T\| = 2,000 giving a consistent input size for our models.


