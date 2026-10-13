# From Forecasts to Decisions: Benchmarking Probabilistic Demand Forecasts for Retail Replenishment and Liquidation Across Settings and Periods

> Draft v3 (13/10/2026) for ICIIT 2027 — single column, at most 10 pages including figures, tables and references.
> Tables are generated in `v3_tables.md`; the numbers quoted in the text come from that file and from `06_experiment_results/phase2_*.md`.
> `[[PENDING-SF]]` marks the only passage that depends on a run still in progress (M5 with the 21 shared features).
> Author names, affiliations and the repository link are to be added.

## Abstract

Public retail forecasting benchmarks rank methods by forecast error, although forecasts are used to decide how much to stock. We benchmark seven probabilistic forecasting methods, three statistical and four based on gradient boosting, inside one transparent order-up-to policy with optional liquidation. The benchmark uses two public datasets from different retail settings, M5 (brick-and-mortar) and VN1 (multi-vendor e-commerce), keeps every series and repeats all experiments on three 26-week test windows. Because the datasets contain no costs, methods are compared at equal fill rate along inventory–fill-rate trade-off curves, with bootstrap confidence intervals. On M5, LightGBM quantile regression is the most accurate method and needs 3–11% less inventory than the other strong baselines in every window. On VN1 the most inventory-efficient method changes between windows. We trace this to demand dynamics: a negative-binomial TSB model needs 13–16% less inventory on series with stable demand in every window, whereas only the quantile model serves series whose demand grows sharply. Neither the ADI–CV² classes nor an unsupervised segmentation of the series separates these two kinds of series in advance, and method-selection rules built on either grouping do not reliably beat a single method. LightGBM quantile regression is the minimax-regret choice. A quantile-based liquidation rule costs almost no service, but no rule shows an economic benefit within 26 weeks.

**Keywords:** probabilistic forecasting; inventory management; intermittent demand; quantile regression; demand segmentation; liquidation; M5; VN1

## 1 Introduction

Retailers face two opposite risks. With too little stock they lose sales; with too much they tie up capital and eventually have to clear products at a loss. Setting stock levels requires the uncertainty of demand, not only its expected value, so quantile forecasts are the natural input to inventory decisions [3].

Public benchmarks have driven progress in probabilistic demand forecasting, but they score forecasts, not decisions. The M5 competition used the weighted RMSSE and the weighted scaled pinball loss [1], and its organisers state that it did not target a specific decision problem [3]. Studies on the more recent VN1 e-commerce data also compare accuracy and computational cost [4]. Yet higher accuracy does not necessarily give better inventory decisions [5, 6]. On M5, Theodorou et al. [7] report that the most accurate method may not be the best choice when lost-sales costs dominate, especially for intermittent products (we had access to the abstract of [7] only).

Three gaps remain. First, studies that link forecasts to inventory on M5 often keep only selected series, such as 8,000 high-volume series [8] or a subset of volatile food items [9], although most retail series are intermittent. Second, they rely on one dataset and one test period [5, 8, 9], so it is unknown whether their conclusions transfer to another retail setting or another period. Third, public datasets contain neither costs nor inventory positions, so cost-based evaluations depend on assumed parameters.

This paper addresses these gaps with a forecast-to-decision benchmark. It does not propose a new forecasting model. Our contributions are:

1. A reproducible benchmark of seven probabilistic forecasting methods inside one transparent replenishment-and-liquidation policy, on two public retail datasets from different settings, with all series retained. It runs on a laptop CPU.
2. A cost-free evaluation protocol: methods are compared at equal fill rate along their trade-off curves, with bootstrap confidence intervals, and liquidation is summarised by a break-even salvage ratio.
3. Evidence on robustness. Everything is repeated on three test windows, which separates findings that hold in every window from period-dependent ones.
4. An explanation of the period dependence on e-commerce data. We show that it follows from how much demand grows in the test period, that neither the standard demand classes nor an unsupervised segmentation anticipates this, and that group-wise method selection therefore does not help.

## 2 Related Work

**Retail forecasting benchmarks.** In the M5 accuracy track, LightGBM was used by all of the top 50 teams, and the winner combined LightGBM models trained with a Tweedie loss [2]. The winner of the uncertainty track trained a separate LightGBM model for each quantile [3]. These results motivate our main model and our point-forecast baseline. Zanotti [4] used M5 and VN1 to study the cost of ensembling global models; that study evaluates accuracy and computational cost, not inventory outcomes.

**Intermittent demand.** The TSB method updates a demand probability every period, which lets it react to obsolescence [10]. Demand is commonly classified by the average inter-demand interval (ADI) and the squared coefficient of variation of demand sizes (CV²) [11]; the thresholds 4/3 and 0.5 are used to describe M5 [1]. Damato et al. [12] compared local and global probabilistic models on intermittent data and concluded that no global architecture is established for such series.

**From forecasts to inventory decisions.** A structured review observes that most forecasting studies ignore the step from forecast to replenishment decision [6]. Kourentzes et al. [13] tune exponential smoothing with an inventory-based cost function and question whether forecast accuracy is a reasonable proxy for the decisions it supports. van der Haar et al. [14] learn order quantities end to end. On M5, Wang et al. [5] evaluate probabilistic forecast combinations with a newsvendor rule in a single-period setting, which they list as a limitation; Zabraoui et al. [9] simulate inventory with reinforcement-learning policies on selected items; Mohammed et al. [8] evaluate safety stock with one illustrative cost calculation on high-volume series. According to its abstract, [7] simulates an order-up-to policy with quantile forecasts on M5 for different demand patterns and cost structures.

Our study differs from these works in combining two retail settings, all demand classes, a liquidation decision, a comparison that needs no cost assumption, and three test windows. Multi-period simulation [7, 9, 14] and decision-level evaluation [5, 7] are not new in themselves.

## 3 Method

### 3.1 Data

M5 [1] contains daily unit sales of 3,049 products in 10 Walmart stores (30,490 product–store series). We aggregate them to 277 complete weeks. VN1 [21] is a weekly e-commerce dataset with 15,053 client–warehouse–product series over 196 weeks; despite its name it is not Vietnamese data. A series is evaluated if it starts at least 13 weeks before the test period and has at least one sale before it.

**Table 1.** Datasets. Class shares and zero weeks refer to the history before the main test window.

| | M5 | VN1 |
|---|---|---|
| Setting | one retailer, 10 physical stores | multi-vendor e-commerce, 46 vendors, 328 warehouses |
| Series in panel / evaluated (main, second, third window) | 30,490 / 30,381, 29,917, 28,824 | 15,053 / 13,844, 11,442, 9,383 |
| Weeks | 277 | 196 |
| Zero weeks since series start | 24.4% | 49.1% |
| Smooth / erratic / intermittent / lumpy | 53.0 / 5.8 / 33.9 / 7.3% | 18.8 / 13.7 / 47.2 / 20.3% |
| Exogenous information | price, events, SNAP days, product hierarchy | price in weeks with sales only |

Both datasets pass through the same code for selection, splitting and evaluation. The only definitional difference is the series start (first listed price in M5, first sale in VN1); it is immaterial, because the two coincide for 99.5% of the M5 series and recomputing the M5 classes with the VN1 definition changes their shares by at most 0.1 percentage points. The datasets nevertheless contain different kinds of series: a classifier separates them with an AUC of 0.91 from nine features of demand behaviour alone. Section 5.6 examines whether this explains the differences in our results.

### 3.2 Forecasting target and methods

At each weekly origin, every method forecasts the quantiles q ∈ {0.5, 0.8, 0.9, 0.95, 0.99} of the cumulative demand over h weeks directly, with h = 3 for replenishment and h = 13 for liquidation. Weekly quantiles are not summed, because the quantile of a sum is not the sum of quantiles.

- **EMP**: empirical quantiles of rolling h-week sums over the last 104 weeks.
- **ETS**: simple exponential smoothing with a normal distribution for the h-week total [18].
- **TSB-NB**: TSB [10] with a negative-binomial distribution matched to the mean and variance of the h-week total.
- **LGB-T**: a global LightGBM [15] point model with a Tweedie loss plus a normal safety stock from each series' validation residuals. This is the common "point forecast plus safety stock" practice.
- **LGB-C**: the same point model with split-conformal quantiles [17] pooled by demand class.
- **HGB-Q**: histogram gradient boosting [19] with a quantile loss, trained on a 300,000-row sample; a check that the results do not depend on one library.
- **LGB-Q**: LightGBM quantile regression [16], one global model per quantile (main model).

The four global models share features (lags, rolling means and standard deviations, zero share, weeks since the last sale, calendar, price, and a scale) and a target scaled by the series level. M5 adds events, SNAP days and the product hierarchy. Hyper-parameters are fixed; no model is tuned.

### 3.3 Decision layer

Each week, for every series: (1) the order placed L weeks earlier arrives; (2) the inventory position IP is on-hand plus on-order stock; (3) stock above a liquidation threshold T is liquidated; (4) an order max(S − IP, 0) is placed, with S the τ-quantile of demand over L + 1 weeks, unless stock was liquidated; (5) demand occurs and unmet demand is lost. The default lead time is L = 2 weeks.

Four liquidation policies use the same replenishment forecasts: none; **quantile** (T is the 0.95-quantile of 13-week demand from the same method); **fixed** (T is 26 times the mean weekly sales of the last 26 weeks); and **dead13** (a series without a sale in 13 weeks is liquidated completely and not replenished until it sells again).

### 3.4 Evaluation

*Accuracy.* The usual scaled quantile loss divides each series' pinball loss by its in-sample naive error. Its mean is unstable on VN1 (Section 5.1). We therefore report the volume-weighted quantile loss wQL (total pinball loss divided by total demand), the median per-series scaled loss, and mean per-series ranks with the Friedman test and the Nemenyi critical difference [20].

*Inventory.* Over the 22 test weeks after a 4-week warm-up we compute the fill rate (units sold / units demanded) and inventory in weeks of demand. At a fixed τ the methods reach different fill rates, so we run τ over the five quantile levels, draw each method's trade-off curve, and interpolate the inventory needed to reach a target fill rate. The frontier rank is a method's mean rank of required inventory over the targets 0.90–0.98. Confidence intervals come from resampling series (200 resamples).

*Liquidation.* For each policy we compute the break-even salvage ratio s*, the minimum salvage price per unit cost at which liquidating is at least as profitable as holding, over holding costs of 10–40% per year and gross margins of 30–100%.

## 4 Experimental Setup

The last 26 weeks of a panel form the test period, in two 13-week blocks; models are retrained at the start of each block and every test week is a forecast origin. To test robustness we drop the last 26 and the last 52 weeks and repeat everything, including model training, on the preceding 26 weeks. This gives three test windows per dataset (main, second, third). All experiments ran on a laptop CPU (Intel Core i5-12450H, 16 GB RAM). One horizon of LGB-Q took 1,251 s on M5 and 146 s on VN1; TSB-NB took 68 s and 30 s. Code and result files are available at [link to be added].

## 5 Results

### 5.1 Accuracy

*[Table 2 here — `v3_tables.md`]*

By volume-weighted loss, LGB-Q is the most accurate method in both datasets and all three windows, with HGB-Q within 0.002 (Table 2). It is also the best method in every demand class of both datasets and every window. On M5, per-series ranks agree: LGB-Q ranks first at h = 3 in every window, ahead of HGB-Q by more than the critical difference.

On VN1 the per-series view differs. At h = 3, LGB-Q, HGB-Q and TSB-NB are within 0.17 mean ranks, and LGB-Q and HGB-Q differ by less than the critical difference. At h = 13, TSB-NB ranks first in all three windows (2.92 / 2.99 / 3.37 against 4.02 / 3.48 / 3.85 for LGB-Q). The global models are thus not more accurate than TSB-NB for a typical VN1 series; their advantage in total loss comes from avoiding rare, very large errors. The mean scaled loss hides this: for LGB-Q on VN1 it is 0.336, 0.539 and 2.008 in the three windows, driven by a few series with a tiny naive error, whereas wQL is 0.103, 0.078 and 0.102.

### 5.2 Inventory at equal fill rate

*[Figure 2 here — trade-off curves, main window: (a) M5, (b) VN1]*

*[Table 3 here]*

**M5.** LGB-Q has the best frontier rank in every resample of every window. At a fill rate of 0.94 it needs 3.3–10.6% less inventory than LGB-T, LGB-C, TSB-NB and ETS, and every interval excludes zero (Table 3). HGB-Q is within 0.2–1.4%. Direct quantiles therefore beat the point forecast plus safety stock (LGB-T) by 3.3–6.8%.

**VN1.** The answer depends on the window. In the main window LGB-Q has the best frontier rank, but its lead over LGB-T, LGB-C and TSB-NB at a fill rate of 0.94 is not significant. In the second window TSB-NB leads, and TSB-NB, LGB-T and LGB-C need 4.1–8.8% less inventory than LGB-Q. In the third window LGB-Q leads again, and TSB-NB, ETS and LGB-T cannot reach a fill rate of 0.94 at any τ. The lowest loss (LGB-Q in every window) does not reliably give the lowest inventory.

By demand class, LGB-Q leads on M5 in the smooth, intermittent and lumpy classes in every window. On VN1 only the lumpy class has the same leader (LGB-Q) in all three windows; the smooth class is led by TSB-NB in two windows and by LGB-C in the third.

### 5.3 Why VN1 depends on the period

*[Table 5 here]*

We split the series of each window by how their demand changed between the 26 weeks before the test period and the test period itself: stable (0.67–1.5 times) or growing (more than 1.5 times). This split uses realised demand, so it is descriptive and cannot guide decisions.

The pattern is the same in all three windows (Table 5). On VN1 series with stable demand, TSB-NB needs 13.3–15.6% less inventory than LGB-Q. On series whose demand grows, TSB-NB fails to reach a fill rate of 0.92 in two windows, whereas LGB-Q reaches it. The aggregate result follows the share of demand that comes from growing series: 31.5% and 40.4% in the windows where LGB-Q leads, and 22.8% in the window where TSB-NB leads. In the third window the demand of the intermittent class is 3.2 times that of the previous half-year, and two thirds of it comes from series that grew more than tenfold. On M5, LGB-Q needs less inventory in both strata in every window.

### 5.4 Can a better grouping of series anticipate this?

*[Table 6 here]*

If growth were predictable from a series' history, a method could be chosen per group of series. We tested two groupings: the ADI–CV² classes, and an unsupervised segmentation built for this purpose. For the latter we clustered the series of both datasets on eight features computed before the test period (level, order size, ADI, CV², weeks since the last sale, 26-week trend, recent level relative to history, and peak size). The model was fitted on the earliest window and applied unchanged to the later ones. Among K-means and Gaussian mixtures with 3 to 8 components, a three-component Gaussian mixture was selected by criteria fixed in advance (bootstrap stability and minimum cluster size). It yields a regular, a fading and a sparse segment. The segmentation is stable (adjusted Rand index 0.94 between refits) and differs from the ADI–CV² classes (adjusted Rand index 0.19–0.41), but the clusters are not well separated: the silhouette is 0.21 and a density-based method labels 43% of the series as noise.

Neither grouping helps to choose a method (Table 6). A rule that assigns each group the method that was most inventory-efficient for it in the previous window stays within 0.3% of LGB-Q on M5. On VN1 the rule based on ADI–CV² classes saves 5.6% in one transition and tends to cost 5.1% in the other; the rule based on segments saves nothing and then costs 10.5%. Even an oracle that knows the best method of each segment in the test window does no better than the best single method of that window, because within a window the same method leads in every segment. The reason is that the share of series that will grow is almost the same in every group: 18–31% in the three segments and 14–39% in the four ADI–CV² classes on VN1. Growth is only partly predictable even by a supervised model (AUC 0.72–0.73 on VN1).

### 5.5 Liquidation

*[Table 7 here]*

With LGB-Q forecasts, the quantile rule costs 0.02 percentage points of fill rate or less in every window and dataset (Table 7). On M5 it almost never triggers; on VN1 it reduces inventory by 1.7–7.4%. The fixed rule and the dead-stock rule lose more service: they raise the stockout rate by 5–20% and 10–23%. Dead13 targets stock that would indeed have remained unsold on VN1, but on M5 products that have not sold for 13 weeks usually sell again.

No rule shows an economic benefit within 26 weeks. The break-even salvage ratio of the quantile rule on VN1 is 0.91–1.21 of unit cost, and that of the other rules is at or above 0.97. Liquidation would have to be justified by factors outside this simulation, such as obsolescence or storage space.

### 5.6 Are the differences between the datasets artefacts?

We checked three possible artefacts (details in the repository). First, composition: re-weighting the VN1 series so that their feature distribution matches M5 (mean standardised difference of the features reduced from 0.32–0.39 to 0.02–0.03) leaves the VN1 conclusions unchanged. LGB-Q is more accurate than TSB-NB on 49–51% of the re-weighted series, against 60–64% on M5, and TSB-NB still leads in the second window. Second, prices: VN1 prices are observed in less than half of the weeks, but refitting the four global models without price features changes the inventory needed by LGB-Q by less than 1% in every window. Third, information: [[PENDING-SF]] the global models on M5 use six features that VN1 lacks; refitting them on M5 with the shared features only *(result to be inserted: inventory needed by LGB-Q and its lead over TSB-NB and LGB-T, three windows)*.

## 6 Discussion

**Compare at equal service level and over several periods.** At τ = 0.9 the methods reach fill rates between 0.935 and 0.967 on M5, because each calibrates its quantiles differently. Comparing inventory at equal τ rewards methods that under-cover. Trade-off curves remove this effect and need no cost assumption. A single test period would have given a confident but wrong answer on VN1 in either direction.

**On e-commerce data the best method depends on what demand will do.** A well-specified statistical model is the more economical choice for series that continue as before, and a global quantile model is the only one that copes with series that take off. This is consistent with [7], who find on M5 that the most accurate method need not be the best for intermittent products; our results add that the divergence depends on the period and that it is not visible on M5 at the service levels we studied. Since growth cannot be read from the demand classes or from a richer segmentation, we see no reliable basis for switching methods by group.

**LGB-Q is the minimax-regret choice.** Averaged over the fill targets it reaches, LGB-Q needs at most 9.4% more inventory than the best method in any window of either dataset, and it reaches all five targets in five of the six runs. TSB-NB needs up to 20.7% more and reaches only two of the five targets in the third VN1 window. Among the global models on VN1, however, the order of LGB-Q, LGB-C and HGB-Q is not robust: they differ by a few per cent and change places when the price features are removed.

**Limitations.** Sales are used as demand, so historical stockouts are not visible. Unmet demand is lost and liquidation does not change demand. Hyper-parameters are not tuned. The demand-shift analysis is ex post. Three windows per dataset are few; the consistency of method rankings between the two datasets is weak (Spearman correlation of frontier ranks 0.86, 0.29 and 0.61). Twenty-six weeks are too short to observe the benefit of clearing obsolete stock. Two extensions are reported in the repository only: a zero-shot, univariate use of the Chronos-2 foundation model [22] on VN1, which was less accurate than LGB-Q on 69–73% of the series and never the most inventory-efficient method, and a case study on Vietnamese footwear retail data with actual unit costs, which gave similar break-even salvage ratios but has unresolved data-quality and licence issues.

## 7 Conclusion

We benchmarked seven probabilistic forecasting methods inside one replenishment-and-liquidation policy on two public retail datasets and three test windows. On brick-and-mortar data (M5) the conclusion is stable: LightGBM quantile regression is the most accurate method and needs 3–11% less inventory than the other strong baselines at a fill rate of 0.94. On e-commerce data (VN1) the most inventory-efficient method changes with the period, because a statistical model is more economical for stable series while only the quantile model serves series whose demand grows sharply. Neither the ADI–CV² classes nor an unsupervised segmentation anticipates which series will grow, so group-wise method selection does not help, and LightGBM quantile regression remains the minimax-regret choice. A quantile-based liquidation rule is safe for service, but within 26 weeks liquidation pays off only at salvage prices near unit cost. Future work should predict demand growth directly, extend the horizon for liquidation, and test covariate-informed foundation models.

## References

[1] S. Makridakis, E. Spiliotis, and V. Assimakopoulos. 2022. The M5 competition: Background, organization, and implementation. *International Journal of Forecasting* 38, 4, 1325–1336. https://doi.org/10.1016/j.ijforecast.2021.07.007

[2] S. Makridakis, E. Spiliotis, and V. Assimakopoulos. 2022. M5 accuracy competition: Results, findings, and conclusions. *International Journal of Forecasting* 38, 4, 1346–1364. https://doi.org/10.1016/j.ijforecast.2021.11.013

[3] S. Makridakis, E. Spiliotis, V. Assimakopoulos, Z. Chen, A. Gaba, I. Tsetlin, and R. L. Winkler. 2022. The M5 uncertainty competition: Results, findings and conclusions. *International Journal of Forecasting* 38, 4, 1365–1385. https://doi.org/10.1016/j.ijforecast.2021.10.009

[4] M. Zanotti. 2025. The cost of ensembling: Is it always worth combining? arXiv:2506.04677.

[5] S. Wang, Y. Kang, E. Spiliotis, and F. Petropoulos. 2026. Multi-objective probabilistic forecast combination for inventory demand. arXiv:2606.04900.

[6] T. E. Goltsos, A. A. Syntetos, C. H. Glock, and G. Ioannou. 2022. Inventory–forecasting: Mind the gap. *European Journal of Operational Research* 299, 2, 397–419. https://doi.org/10.1016/j.ejor.2021.07.040

[7] E. Theodorou, E. Spiliotis, and V. Assimakopoulos. 2025. Forecast accuracy and inventory performance: Insights on their relationship from the M5 competition data. *European Journal of Operational Research* 322, 2, 414–426. https://doi.org/10.1016/j.ejor.2024.12.033

[8] Z. Mohammed, C. Anas, and M. El Hammoumi. 2026. A hybrid learning framework for forecasting uncertainty and adaptive inventory planning in retail supply chains. *Supply Chain Analytics* 13, 100180. https://doi.org/10.1016/j.sca.2025.100180

[9] O. Zabraoui, Y. Hmamou, A. Chafi, and S. Kammouri Alami. 2025. A comparative study of multi-algorithm optimization for inventory analytics in supply chains. *Supply Chain Analytics* 12, 100154. https://doi.org/10.1016/j.sca.2025.100154

[10] R. H. Teunter, A. A. Syntetos, and M. Z. Babai. 2011. Intermittent demand: Linking forecasting to inventory obsolescence. *European Journal of Operational Research* 214, 3, 606–615. https://doi.org/10.1016/j.ejor.2011.05.018

[11] A. A. Syntetos, J. E. Boylan, and J. D. Croston. 2005. On the categorization of demand patterns. *Journal of the Operational Research Society* 56, 5, 495–503. https://doi.org/10.1057/palgrave.jors.2601841

[12] S. Damato, N. Rubattu, D. Azzimonti, and G. Corani. 2026. Intermittent time series forecasting: Local vs global models. arXiv:2601.14031.

[13] N. Kourentzes, J. R. Trapero, and D. K. Barrow. 2020. Optimising forecasting models for inventory planning. *International Journal of Production Economics* 225, 107597. https://doi.org/10.1016/j.ijpe.2019.107597

[14] J. F. van der Haar, A. P. Wellens, R. N. Boute, and R. J. I. Basten. 2024. Supervised learning for integrated forecasting and inventory control. *European Journal of Operational Research* 319, 2, 573–586. https://doi.org/10.1016/j.ejor.2024.07.004

[15] G. Ke, Q. Meng, T. Finley, T. Wang, W. Chen, W. Ma, Q. Ye, and T.-Y. Liu. 2017. LightGBM: A highly efficient gradient boosting decision tree. In *Advances in Neural Information Processing Systems 30*.

[16] R. Koenker and G. Bassett. 1978. Regression quantiles. *Econometrica* 46, 1, 33–50. https://doi.org/10.2307/1913643

[17] J. Lei, M. G'Sell, A. Rinaldo, R. J. Tibshirani, and L. Wasserman. 2018. Distribution-free predictive inference for regression. *Journal of the American Statistical Association* 113, 523, 1094–1111. https://doi.org/10.1080/01621459.2017.1307116

[18] R. Hyndman, A. Koehler, K. Ord, and R. Snyder. 2008. *Forecasting with Exponential Smoothing*. Springer. https://doi.org/10.1007/978-3-540-71918-2

[19] F. Pedregosa et al. 2011. Scikit-learn: Machine learning in Python. *Journal of Machine Learning Research* 12, 2825–2830.

[20] J. Demšar. 2006. Statistical comparisons of classifiers over multiple data sets. *Journal of Machine Learning Research* 7, 1–30.

[21] N. Vandeput. 2024. VN1 Forecasting – Accuracy Challenge. DataSource.ai.

[22] A. F. Ansari et al. 2025. Chronos-2: From univariate to universal forecasting. arXiv:2510.15821.
