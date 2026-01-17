# Anomaly Detection using Isolation Forest on EDR Telemetry (BETH Dataset)

## Project Summary
This project evaluates whether statistical baselining can serve as a meaningful signal for endpoint detection and response. Using kernel-level process telemetry from the BETH dataset, I built an unsupervised anomaly detection pipeline that models normal process behavior and flags deviations that may indicate malicious activity.

Rather than detecting known attacks, the system learns what normal execution looks like and treats sufficiently abnormal behavior as suspicious. The objective is not to build a blocking control, but to examine how far unsupervised detection can go before false positives outweigh operational value.

## Why This Problem Matters
Endpoint telemetry is inherently noisy. Many detection systems fail not because they miss attacks, but because they overwhelm analysts with alerts that lack context. This is especially true for living-off-the-land techniques, where legitimate binaries such as `powershell.exe` or `cmd.exe` are abused in subtle ways.

This project is framed around a practical question: can process-level behavioral context alone distinguish benign administrative activity from adversarial execution, and what tradeoffs does that introduce for a SOC?

## Data and Signals
The analysis uses the publicly available BETH dataset, which contains kernel-level process execution logs collected across multiple hosts.

For additional information on BETH, refer to:

*   Highnam, K., Arulkumaran, K., Hanif, Z., & Jennings, N. R. (2021). BETH dataset: Real cybersecurity data for anomaly detection research. In *ICML 2021 Workshop on Uncertainty and Robustness in Deep Learning*. http://www.gatsby.ucl.ac.uk/~balaji/udl2021/accepted-papers/UDL2021-paper-033.pdf

I focused on signals that encode **behavioral structure rather than identity**, including:
- Process genealogy, such as parent-child relationships
- Command-line argument patterns
- Execution context features related to how a process runs

High-cardinality identifiers and raw timestamps were intentionally excluded to reduce overfitting to host-specific schedules.

## Detection and Modeling Approach
An unsupervised Isolation Forest was used to model normal execution patterns, chosen specifically because malicious behavior is rare and continuously evolving. The model assigns an anomaly score to each event rather than producing a binary decision.

Feature engineering focused on representing process structure and execution context numerically. Dimensionality reduction was used only for exploratory validation, not as a detection mechanism.

The output anomaly score allows downstream systems to tune sensitivity based on operational constraints.

## Results and Tradeoffs
The model achieved very high recall, identifying nearly all known malicious events in the evaluation set. However, this came with a substantial false positive rate, with a significant fraction of benign events flagged as anomalous.

This outcome highlights a core reality of detection engineering: high sensitivity without sufficient context creates analyst fatigue. In a production SOC, this system would require additional filtering and enrichment before generating actionable alerts.

## Adversary Perspective
Because this approach relies on statistical deviation, a knowledgeable attacker could attempt to blend in by mimicking common process trees, reusing familiar argument patterns, or spreading actions out over time.

This reinforces that anomaly detection should augment other controls rather than operate in isolation.

## Operational Use
In practice, this system would function as a triage or threat-hunting signal. Anomalous events would be enriched and correlated with secondary indicators such as rare event IDs, identity context, or network activity before analyst review.

Analyst feedback could be used over time to reduce noise and improve precision without sacrificing coverage.

## Future Improvements
The most impactful next step would be incorporating sequence-aware modeling. Many attacks only become apparent when considering execution order rather than individual events. Lightweight temporal features or sequence-based models could significantly reduce false positives.

Improving explainability, such as surfacing which behavioral features contributed most to an alert, would also be critical for analyst trust.

## Reproducibility
The dataset is not included in this repository due to size constraints. To reproduce the analysis, download the BETH dataset from the source above and follow the instructions in the notebook.
