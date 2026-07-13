# Network and Emotional Structure of Text as a Cognitive Signal

## Introduction

Dementia leaves a measurable trace in language. As the disease progresses, patients tend to produce shorter, more repetitive, and less structurally complex sentences, reflecting the gradual deterioration of the semantic memory system that underlies word retrieval and concept association (Huff, Corkin, & Growdon, 1986). Standard NLP methods like TF-IDF pick up some of this signal through word frequency, but treat words as independent tokens rather than as a structured web of associations.

This project takes a different angle: instead of modeling text as a bag of words, each document is modeled as a network, where words are nodes and syntactic relationships are edges. This kind of representation, a Textual Forma Mentis Network (TFMN), has been used before to study public perception (Stella, 2020), creativity (Haim et al., 2026), and authorship attribution (Segarra, Eisen, & Ribeiro, 2013). Here it's applied to a two-group comparison: text simulating Alzheimer's disease (AD) patients versus age-matched healthy controls (HC).

Two hypotheses guided the analysis:

1. AD-simulated text shows a degraded syntactic network — smaller, less connected, more reliant on repeated concepts than HC-simulated text.
2. AD-simulated text shows reduced emotional expressivity, consistent with the affective flattening reported in the clinical literature.

A third, more exploratory question was whether the resulting network and emotion features could actually classify the two groups, and, if the accuracy came out unexpectedly high, whether that reflected a real signal or an artifact of how the data was generated.

## Data

The dataset is entirely synthetic. Text was generated with a large language model (Mistral Large 3), prompted to describe a typical day at a local market while writing in a style characteristic of either an AD patient or a healthy control, incorporating the semantic, syntactic, and emotional patterns associated with each profile.

Using LLM-generated text instead of real patient transcripts has one clear advantage (no privacy concerns, easy to generate at scale) and one clear risk: synthetic text can end up too internally consistent within each group and too cleanly different between groups, in a way that doesn't reflect how real patients and controls actually write. That risk turns out to matter a lot for how the classification results should be read (see Results and Discussion).

## Method

**Network construction.** Each text was parsed with spaCy's dependency parser and converted into a TFMN via EmoAtlas, linking words based on syntactic proximity in the dependency tree rather than simple co-occurrence. From each network, 11 topological features were extracted: number of nodes and edges, density, mean degree, average clustering coefficient, number of connected components, number of isolated nodes, and — restricted to the largest connected component — its size, size ratio, average shortest path length, and diameter.

**Emotional profiling.** EmoAtlas also computed emotion z-scores for each text by cross-referencing tokens against the NRC Emotion Lexicon, mapped onto Plutchik's eight core emotions (joy, trust, fear, surprise, sadness, disgust, anger, anticipation). Unlike a transformer-based sentiment model, this approach combines syntactic parsing with a psychologically grounded lexicon, keeping the emotion scores interpretable and computationally light.

**Statistical comparison.** Each topological feature was compared between groups with a two-sided Mann-Whitney U test (chosen over a t-test given the non-normal, bounded nature of network metrics), with Benjamini-Hochberg FDR correction applied across all 11 comparisons.

**Classification.** Two classifiers — logistic regression and a decision tree — were trained on the 8 emotion z-scores under 10-fold stratified cross-validation, evaluated on accuracy and ROC AUC. Emotion features were chosen over topological ones for this step specifically to reduce dependence on document length, which is a known confound when working with LLM-generated text of varying length.

**Checking for a spurious separation.** Given the risk of overly homogeneous synthetic data, a Linear Discriminant Analysis and a Principal Component Analysis were run on the same 8 emotion features, to see whether the two groups separated cleanly even along an unsupervised axis.

## Results

**Network structure.** All 11 topological features differed significantly between groups after FDR correction (all adjusted p < .05). AD-simulated networks were consistently smaller: fewer nodes (41.6 vs. 87.5), fewer edges (108.3 vs. 262.5), and a smaller largest connected component (38.1 vs. 84.6). They also had shorter average path lengths (2.49 vs. 3.93) and smaller diameters (5.24 vs. 9.01) — smaller and structurally simpler networks, not just smaller ones. AD networks were also more fragmented, with more connected components (2.58 vs. 1.25) and more isolated nodes (0.69 vs. 0.03), suggesting disconnected "conceptual islands" not integrated into the main discourse. One counter-intuitive result: AD networks were denser (0.13 vs. 0.07) — but density scales inversely with network size, so this is very likely a side effect of having fewer nodes rather than evidence of richer connectivity.

**Emotional profile.** HC-simulated text showed a clearly more differentiated emotional signature: joy, anticipation, trust, and surprise all stood out well above baseline, while fear, anger, sadness, and disgust fell well below it. AD-simulated text followed the same overall direction but in an attenuated form, trust and disgust, in particular, were statistically indistinguishable from a random baseline. This pattern is consistent with the flattened affect reported in the clinical literature on Alzheimer's disease.

**Classification.** Both classifiers performed strikingly well on emotion features alone: logistic regression reached AUC = 0.97 and accuracy = 0.91; the decision tree reached AUC = 0.93 and accuracy = 0.87. That level of performance from just 8 features on a two-class problem is unusually high, so it was checked rather than taken at face value. Both LDA and PCA showed a near-total, one-dimensional separation between the two groups — meaning the groups were separable even without using the labels at all. That's the signature of low within-group variability rather than a subtle, hard-won signal: the synthetic groups are probably too internally consistent and too different from each other compared to how real patients and controls actually write.

## Discussion

The topological results line up well with what's known about semantic-syntactic decline in dementia (Liu et al., 2022): smaller networks, shorter paths, more fragmentation. The emotional results also track the clinical picture of anhedonia and affective flattening reported in Alzheimer's disease (Shaw et al., 2021).

The classification result is where this analysis earns its "exploratory" label. High accuracy and AUC would normally be good news, but the LDA/PCA check makes clear that the separation is close to total and one-dimensional, a pattern more consistent with the LLM producing two internally homogeneous, easily distinguishable writing styles than with a genuine, subtle clinical marker. Emotion features were deliberately chosen over topological ones to reduce sensitivity to surface-level text length, on the reasoning that emotional tone is less tied to how much a model writes, but the results here show that reasoning wasn't sufficient on its own: emotion features weren't fully immune to the same homogeneity confound.

This isn't a reason to discard the approach. It's a reason to treat the classification number as a property of this particular synthetic dataset, not as evidence that TFMN + emotion features can detect Alzheimer's disease from text in general. The natural next step is running the same pipeline on real transcript data — the DementiaBank Pitt Corpus or the ADReSS challenge dataset are the standard benchmarks, and comparing classification performance on synthetic versus real data directly. If the same features that separate LLM-generated groups almost perfectly turn out to separate real patients and controls only modestly, that gap is itself an informative and reportable finding.

## Conclusion

Modeling text as a network surfaces structural differences between simulated Alzheimer's and healthy writing that align with the clinical literature on semantic-syntactic decline, and the emotional analysis picks up a plausible signature of affective flattening. The classification result, on the other hand, is a good example of why a good result deserves the same scrutiny as a bad one: near-perfect separability on synthetic text is more likely to reflect how the data was generated than a validated clinical marker, and the LDA/PCA check is what catches that before the number gets over-interpreted.

## References

Haim, E., Fischer, N., Citraro, S., Rossetti, G., & Stella, M. (2026). Forma mentis networks predict creativity ratings of short texts via interpretable artificial intelligence in human and AI-simulated raters. *Journal of Computational Social Science*, 9(1), 22.

Huff, F. J., Corkin, S., & Growdon, J. H. (1986). Semantic impairment and anomia in Alzheimer's disease. *Brain and Language*, 28(2), 235-249.

Liu, Z., Paek, E. J., Yoon, S. O., Casenhiser, D., Zhou, W., & Zhao, X. (2022). Detecting Alzheimer's disease using natural language processing of referential communication task transcripts. *Journal of Alzheimer's Disease*, 86(3), 1385-1398.

Pataranutaporn, P., Powdthavee, N., Archiwaranguprok, C., & Maes, P. (2025). Simulating human well-being with large language models: Systematic validation and misestimation across 64,000 individuals from 64 countries. *Proceedings of the National Academy of Sciences*, 122(48), e2519394122.

Segarra, S., Eisen, M., & Ribeiro, A. (2013). Authorship attribution using function words adjacency networks. In *2013 IEEE International Conference on Acoustics, Speech and Signal Processing* (pp. 5563-5567).

Semeraro, A., Vilella, S., Improta, R., De Duro, E. S., Mohammad, S. M., Ruffo, G., & Stella, M. (2025). EmoAtlas: An emotional network analyzer of texts that merges psychological lexicons, artificial intelligence, and network science. *Behavior Research Methods*, 57(2), 77.

Shaw, S. R., El-Omar, H., Ramanan, S., Piguet, O., Ahmed, R. M., Whitton, A. E., & Irish, M. (2021). Anhedonia in semantic dementia — exploring right hemispheric contributions to the loss of pleasure. *Brain Sciences*, 11(8), 998.

Stella, M. (2020). Text-mining forma mentis networks reconstruct public perception of the STEM gender gap in social media. *PeerJ Computer Science*, 6, e295.
