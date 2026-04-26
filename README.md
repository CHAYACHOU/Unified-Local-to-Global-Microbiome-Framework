# Unified Local-to-Global Microbiome Framework
The unified local-to-global-microbiome framework is a three-stage framework, integrating microbial feature screening, contribution profiling, and local network inference. It improved microbiome-based health assessment and enhanced biological interpretability.
# Step1：Construction and Description of Main Model 
## GMWI model
### Environment Dependencies
```text
Python 3.8.3 
pandas 2.0.3
numpy 1.23.5
scikit-learn 1.3.2
```
### Model Parameters 
```text
use_shannon = Ture
theta_f = 1.4
theta_d = 0.1
thresh = 0.00001
```
### Construction of GMWI model
```text
In this study, GMWI model was primarily adapted from the original work, including both parameters and implementation code[1].
-Data：The GMWI model was applied to a species-level relative abundance matrix, in which rows represented samples and columns represented microbial species.The matrix values were relative abundances scaled to the range of 0 to 1.  Sample labels were encoded as healthy or non-healthy. 
-Feature Select: A species was considered present in a sample when its relative abundance exceeded 1×10⁻⁵. Based on this presence cutoff, species prevalence was calculated separately in healthy and non-healthy groups. Two criteria were then used for feature selection: prevalence fold change and prevalence difference. Health-abundant taxa were defined as species with a prevalence fold change ≥ 1.4 and a prevalence difference ≥ 0.1 in healthy versus non-healthy samples. Health-scarce taxa were defined as species with a prevalence fold change ≥ 1.4 and a prevalence difference ≤ -0.1. These thresholds were adopted from the original GMWI study, in which this parameter combination achieved the highest balanced accuracy for health classification. 
-Output: The final GMWI score was calculated as the log-ratio between the health-abundant and health-scarce feature sets, and samples with GMWI > 0 were classified as healthy.
```
## GWMI2 model
### Environment Dependencies
```text
- Python 3.8.3 
- pandas 2.0.3
- numpy 1.23.5
- scikit-learn 1.3.2
```
### Model Parameters 
```text
penalty = "l1"
solver = "liblinear"
random_state = 42
class_weight = "balanced"
PRESENCE_CUTOFF = 0.00001
C = 0.03
```
### Construction of GMWI2 model
```text
In this study, GMWI2 model was primarily adapted from the original work, including both parameters and implementation code[2].
-Data：The original input of GMWI2 is a sample × microbialfeature relative abundance matrix, with all values scaled to the range of 0–1. Before modeling, the relative abundance matrix was first binarized according to the presence threshold PRESENCE_CUTOFF = 1×10⁻⁵. A feature was marked as 1 if its relative abundance exceeded 1×10⁻⁵; otherwise, it was marked as 0. After binarization, the original relative abundance data were converted into a binary-encoded presence/absence matrix, which served as the actual input for the L1-penalized logistic regression model.  Sample labels were encoded as healthy or non-healthy. 
-Feature Select: GMWI2 was implemented as an L1-penalized logistic regression model. Unlike rule-based preselection procedures, feature selection in GMWI2 was embedded within the model training process. Under L1 regularization, coefficients of non-informative species were shrunk to zero, whereas species with non-zero coefficients were retained as informative features contributing to health-state discrimination. Thus, feature selection in GMWI2 was performed as an internal sparse-selection procedure. The model was configured with penalty = "l1", solver = "liblinear", random_state = 42, and class_weight = "balanced". The regularization parameter C was not arbitrarily assigned, but selected from the candidate values 0.001, 0.003, 0.01, 0.03, 0.1, 0.3, 1, and 3 by Leave-One-Group-Out cross-validation. Balanced accuracy (BA) was used as the optimization criterion, and the value yielding the highest BA was retained as the final REGULARIZATION parameter. 
-Output: After model fitting, a continuous GMWI2 score was generated using the decision_function. For classification, samples with GMWI2 > 0 were assigned as healthy, whereas samples with GMWI2 ≤ 0 were assigned as non-healthy. This threshold corresponds to the logistic regression decision boundary used to convert continuous scores into binary labels.
```
## hiPCA model
### Environment Dependencies
```text
- Python 3.8.3 
- pandas 2.0.3
- numpy 1.23.5
- scikit-learn 1.3.2
- scipy 1.9.3
```
### Model Parameters 
```text
Candidate significance thresholds:1e-60, 1e-50, 1e-40, 1e-30, 1e-20, 1e-10, 0.001, 0.01, 0.1
health-abundant taxa: p < 1×10⁻30
health-scarce taxa: p < 1×10⁻30
ε = 1×10⁻⁵
Principal components：sum_pcs = 0.9
The confidence level for calculating control limits：confr = 0.9
```
### Construction of hiPCA model
```text
In this study, hiPCA model was primarily adapted from the original work, including both parameters and implementation code[3].
-Data：The input to hiPCA was a sample-by-species relative abundance matrix. Rows represented samples, and columns represented microbial species. As the input abundance matrix was scaled to the range of 0–1, all values were transformed using log2(2X + ε) before PCA. A pseudo-count of 1×10⁻⁵ was used to avoid zero values during log transformation. The transformed matrix was subsequently processed by Z‑score standardization. 
-Feature Select: Feature screening was performed before PCA using the Kolmogorov–Smirnov test to identify health-abundant and health-scarce taxa according to distributional differences between healthy and non-healthy samples. Candidate significance thresholds included 1e-60, 1e-50, 1e-40, 1e-30, 1e-20, 1e-10, 0.001, 0.01, and 0.1, and the final setting adopted in the main workflow was p < 1e-30 for both health-abundant and health-scarce taxa. 
-Output: Principal components were retained until the cumulative explained variance reached 0.9, and a confidence level of 0.9 was used to calculate the control limits (T2lim, SPElim, and Flim). The hiPCA model generated sample-level health scores  for classification and feature-level Bacteria-to-Health-Index Contribution (BHC) profiles for downstream interpretation.
```
## BN model
### Environment Dependencies
Matlab R2021b
### Model Parameters 
```text
ε = 1×10⁻⁷
Scoring function: Bayesian score 
Conditional probability distribution type = tabular 
Number of MCMC samples = 100 × number of nodes
Burn-in period = 5 × number of nodes
The number of iterations of EM (max)：maxiter = 10
stopping criterion：epsilon=0.001
EM convergence threshold：thresh = 1×10⁻³
```
### Construction of BN model
```text
In this study, BN model was primarily adapted from the previously work[4]. Both parameters and implementation code were appropriately modified.
-Data：The original input to BN was a sample × microbial-feature abundance matrix, in which rows represented samples and columns represented microbial nodes/features. Before BN construction, the abundance profile was subjected to customized logarithmic transformation（log2(2X + ε)） and Z-score normalization to reduce scale differences across features and improve comparability among samples. After normalization, the processed abundance matrix was further discretized according to the predefined state definitions of each node, and then binary encoding was performed for downstream modelling.
-Model Construction: The BN structure was learned from the preprocessed healthy-sample matrix using an MCMC-based DAG learning algorithm. Candidate graphs were generated by adding, deleting, or reversing a single edge while preserving acyclicity through an ancestor-matrix constraint. For each node pair, three possible edge states were compared (A→B, B→A, or no edge), and the direction with the highest posterior probability was retained. The main tunable parameters included the Bayesian score as the default scoring function, the number of MCMC samples set to 100 × number of nodes, and the burn-in period set to 5 × number of nodes. After the DAG structure was obtained, a shared discrete latent variable was introduced for each continuous node, and the data distribution was modeled using three Gaussian mixture components. Conditional probability distributions (CPDs) and mixture parameters, including means, covariance matrices, and mixing coefficients, were then estimated by the EM algorithm until convergence or a preset iteration limit was reached. 
-Output: The BN model produced a Bayesian inference engine containing the learned causal DAG and node-specific CPDs among health-associated taxa. After entering sample-specific evidence, the engine generated posterior marginals for hidden nodes and a sample-level log-likelihood value, which was used for downstream health-state evaluation and regulatory analysis.
```
# Step2： Proposal of The Unified Local-to-Global Microbiome Framework
```text
Ensemble_method.py: Based on the predicted labels of the four models under their optimal thresholds, a majority voting strategy was adopted to obtain final predictions under different ensemble modes. The balanced accuracy was used as the evaluation metric.
```
# Step3： Regulatory Role Classification of Health-associated Species
```text
bn_inta.m: The Mahalanobis distance of each feature was calculated to quantify the deviation degree of local network interactions.
```
# References
[1] Gupta, V. K., Kim, M., Bakshi, U., Cunningham, K. Y., Davis, J. M., Lazaridis, K. N., Nelson, H., Chia, N., & Sung, J. (2020). A predictive index for health status using species-level gut microbiome profiling. Nature Communications, 11(1), 4635. https://doi.org/10.1038/s41467-020-18476-8.
[2] Chang, D., Gupta, V. K., Hur, B., Cobo-López, S., Cunningham, K. Y., Han, N. S., Lee, I., Kronzer, V. L., Teigen, L. M., Karnatovskaia, L. V., Longbrake, E. E., Davis, J. M., Nelson, H., & Sung, J. (2024). Gut Microbiome Wellness Index 2 enhances health status prediction from gut microbiome taxonomic profiles. Nature Communications, 15(1), 7447. https://doi.org/10.1038/s41467-024-51651-9.
[3] Zhu, J., Xie, H., Yang, Z., Chen, J., Yin, J., Tian, P., Wang, H., Zhao, J., Zhang, H., Lu, W., & Chen, W. (2023). Statistical modeling of gut microbiota for personalized health status monitoring. Microbiome, 11(1), 184. https://doi.org/10.1186/s40168-023-01614-x.
[4] Zhu, J., Ge, Z., Song, Z., Zhou, L., & Chen, G. (2018). Large-scale plant-wide process modeling and hierarchical monitoring: A distributed Bayesian network approach. Journal of Process Control, 65, 91-106. https://doi.org/10.1016/j.jprocont.2017.08.011.
