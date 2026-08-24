# Capstone Report — Structured Content Archetype Clustering

- **Author:** Büşra Yıldırım (busrayildirim.dev@gmail.com)
- **Lane:** Structured Content Archetype Clustering
- **Repo:** [Insert Your GitHub Repo URL]
- **Date:** August 25, 2026

## 0. Abstract

Can we identify distinct performance archetypes within organic search content to automate strategic SEO decisions? Using an anonymized FlyRank search intelligence dataset of 16,451 active pages, we applied K-Means clustering to segment the inventory based on ranking, traffic, and engagement signals. The algorithm successfully identified five distinct operational archetypes, achieving a Silhouette Score of 0.302. This model translates raw search performance into a ranked action playbook, allowing SEO teams to automatically assign interventions—such as metadata optimization or content pruning—at scale.

## 1. Problem framing

This analysis supports the strategic decision of resource allocation for content and SEO teams. The unit of analysis is a single content page. The output is a cluster assignment mapped to a specific operational action (e.g., `OPTIMIZE_METADATA`, `PRUNE_OR_REDIRECT`). The human action taken involves executing the recommended editorial update. The cost of a wrong call is high: wasting editorial budget rewriting "Cash Cow" pages that are already performing well, or ignoring "Striking Distance" pages that only need a title tweak to capture significant traffic. Machine learning helps here because unsupervised clustering can uncover multidimensional patterns (combining position, CTR, volume, and engagement) that simple single-metric thresholds often miss.

## 2. Data safety

The model utilizes the `content_refresh_anonymized.csv` dataset. We explicitly excluded zero-impression pages to prevent cold-start noise from skewing the established search behavior clusters. Furthermore, we dropped label-derived fields (`trend_direction` and `trend_pct`) to ensure the algorithm clustered based strictly on static performance snapshots rather than leaking future trajectory data. Pseudonymous identifiers (`content_id`, `client_id`) were used only for grouping and final mapping; they were strictly excluded from the feature matrix. No client-identifying information appears in this analysis.

## 3. Baseline

In unsupervised clustering, the baseline is a non-segmented, "one-size-fits-all" heuristic rule (such as the threshold logic built in Week 4). While a rigid rule like "If position > 20, rewrite" captures some intent, it fails to account for interaction effects (e.g., position > 20 but high engagement). The clustering model's Silhouette Score of 0.302 indicates a meaningful, mathematically sound multidimensional separation of the data compared to basic heuristic grouping.

## 4. Model / analysis

We utilized K-Means clustering (K=5), which perfectly fits the lane's goal of discovering hidden archetypes in unlabeled search data. The exact feature list includes `avg_position`, `ctr`, `word_count`, `engagement_rate`, and `scroll_rate`, alongside a log-transformed `search_volume` to handle its right-skewed heavy tail. We deliberately left out categorical SERP tiers and raw IDs. While there is no traditional supervised target, the proxy objective was to maximize cluster cohesion and separation (Silhouette Score) to define clear, distinct content archetypes.

## 5. Evaluation

The full filtered dataset (16,451 rows) was used for archetype discovery, which is standard practice for exploratory clustering. The model achieved a Silhouette Score of 0.302. Analyzing the errors (or cluster overlaps) via the 2D PCA projection reveals that while the `PRUNE_OR_REDIRECT` and `PROTECT_AND_MONITOR` clusters are highly distinct, there is some dimensional overlap between `OPTIMIZE_METADATA` and `EXPAND_CONTENT`. This is expected, as pages in both clusters often share similar mid-tier SERP positions but differ subtly in engagement metrics. 

## 6. Interpretation

The model successfully found five distinct content profiles:
*   **Cluster 0 (`OPTIMIZE_METADATA`):** Pages with good average positions (~14.9) and high search volume, but severely underperforming CTR (~0.25%).
*   **Cluster 1 (`PRUNE_OR_REDIRECT`):** Zombie pages with deep ranks (~65.2), terrible engagement (~0.08%), and thin content.
*   **Cluster 2 (`EXPAND_CONTENT`):** Pages stuck in lower tiers (~45.9) with thin content (~645 words) needing depth.
*   **Cluster 3 (`PROTECT_AND_MONITOR`):** The "Cash Cows" with strong rank (~18.8), high CTR (~0.38%), and deep word counts.
*   **Cluster 4 (`RESTRUCTURE_INTENT`):** High volume but low rank and low engagement pages.

Visually, the PCA projection demonstrates clear separation for extreme performers. The action distribution chart reveals that `OPTIMIZE_METADATA` is the most frequently recommended action across the analyzed inventory.

## 7. Recommendation

The output is a ranked action playbook. A FlyRank editor would use this tomorrow by filtering the dataset for their client, sorting by the `OPTIMIZE_METADATA` cluster (for quick-win CTR fixes), and assigning the `EXPAND_CONTENT` pages to writers. 
**Limits:** These recommendations provide directional decision-support, not causal guarantees. We confidently identify mathematical archetypes, but executing a metadata refresh does not guarantee a Google ranking algorithm bump. Furthermore, this is a static snapshot; it does not evaluate semantic topic decay.

## 8. Reproducibility

To reproduce these exact results from a fresh clone:
1. Load `data/raw/content_refresh_anonymized.csv`.
2. Filter for `search_volume > 0`.
3. Fill `word_count` NaNs with the median, and engagement/scroll NaNs with 0.
4. Apply `np.log1p()` to `search_volume`.
5. Scale features using `sklearn.preprocessing.StandardScaler`.
6. Fit `KMeans(n_clusters=5, random_state=42, n_init=10)`.

## 9. Acknowledgments & data credit

Built on the FlyRank ML Internship dataset. For more information, visit [https://flyrank.ai](https://flyrank.ai).
