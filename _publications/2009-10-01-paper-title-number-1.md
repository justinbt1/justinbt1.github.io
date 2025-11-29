---
title: "GraphTranslate: Predicting Clinical Trial Translation using Graph Neural Networks on Biomedical Literature"
collection: publications
category: manuscripts
permalink: /publication/2009-10-01-paper-title-number-1
excerpt: 'A novel graph neural network that leverages both semantic and structural information to predict which research publications will lead to clinical trials. Our model analyses a comprehensive dataset of 19 million publication nodes, using transformer-based title and abstract sentence embeddings within their citation network context. Our graph-based architecture, which employs attention mechanisms over local citation neighbourhoods, outperforms traditional convolutional approaches by effectively capturing knowledge flow patterns. Our metadata is carefully selected to eliminate potential biases from researcher-specific information, while maintaining predictive power through network structural features.'
date: 2025-07-01
venue: 'Proceedings of the Fifth Workshop on Scholarly Document Processing, Association for Computational Linguistics.'
slidesurl: 'https://academicpages.github.io/files/slides1.pdf'
paperurl: 'https://academicpages.github.io/files/paper1.pdf'
bibtexurl: 'https://academicpages.github.io/files/bibtex1.bib'
citation: 'Muller E, Boylan-Toomey J, Ekinsmyth J, Robben A, Cardona MDLP, Langfelder A. 2025. GraphTranslate: Predicting Clinical Trial Translation using Graph Neural Networks on Biomedical Literature. In Proceedings of the Fifth Workshop on Scholarly Document Processing (SDP 2025), pages 31–41, Vienna, Austria. Association for Computational Linguistics (ACL).'
---
The translation of basic science into clinical interventions represents a critical yet prolonged
pathway in biomedical research, with significant implications for human health. While previous translation prediction approaches have
focused on citation-based and metadata metrics or semantic analysis, the complex network structure of scientific knowledge remains
under-explored. In this work, we present
a novel graph neural network approach that
leverages both semantic and structural information to predict which research publications
will lead to clinical trials. Our model analyses a comprehensive dataset of 19 million
publication nodes, using transformer-based title and abstract sentence embeddings within
their citation network context. We demonstrate
that our graph-based architecture, which employs attention mechanisms over local citation
neighbourhoods, outperforms traditional convolutional approaches by effectively capturing
knowledge flow patterns (F1 improvement of
4.5 and 3.5 percentage points for direct and
indirect translation). Our metadata is carefully selected to eliminate potential biases from
researcher-specific information, while maintaining predictive power through network structural features. Notably, our model achieves
state-of-the-art performance using only contentbased features, showing that language inherently captures many of the predictive features
of translation. Through rigorous validation on a
held-out time window (2021), we demonstrate
generalisation across different biomedical domains and provide insights into early indicators of translational research potential. Our
system offers immediate practical value for
research funders, enabling evidence-based assessment of translational potential during grant
review processes.
