# What Affects Reaction Time? 
### *A Repeated-Measures Factorial Design on Cognitive Performance*


---

## Project Overview
Reaction time (RT) is a fundamental measure of human cognitive and motor performance, representing how quickly we perceive, process, and respond to stimuli. Whether returning a tennis serve, dodging a hazard while driving, or making split-second decisions in competitive gaming, milliseconds matter. 

This project investigates how **physiological** and **environmental** factors affect visual reaction time. By employing a rigorous **$2^3$ ($2 \times 2 \times 2$) repeated-measures factorial design** and **blocking on individual subjects**, we isolate the impact of short-term modifications from intrinsic baseline differences.

### Research Questions
1. Does breathing style (holding breath vs. normal breathing) alter oxygenation and focus enough to affect reaction speed?
2. Does a loud, high-stimulus background video significantly distract attention and slow reaction speed?
3. Does the time of day (morning vs. evening) correlate with circadian alertness and cognitive speed?
4. Do these factors interact (e.g., does distraction degrade performance more when fatigued in the morning or holding one's breath)?

---

## Experimental Design

To investigate these questions, we designed a balanced factorial experiment with the following structure:

### 1. Factors & Levels
| Factor | Type | Level 1 (-) | Level 2 (+) | Description |
| :--- | :--- | :--- | :--- | :--- |
| **Breathing** | Physiological | **Normal** | **Hold** | Breathe normally vs. taking a deep breath and holding it for the trial. |
| **Video** (Distraction) | Environmental | **No** (Control) | **Yes** (Distracted) | Testing in silence vs. playing a loud, visually stimulating video. |
| **Time of Day** | Circadian | **Morning** | **Evening** | Test within 3 hours of waking (before noon) vs. between 6:00 PM – 10:00 PM. |

### 2. Blocking on Subjects
Reaction times vary widely across individuals due to genetics, age, and gaming experience. To control this high inter-subject variability, we used a **repeated measures blocking design**.
* **Blocks:** 3 Subjects (Aditya, Benjamin, Isabel).
* **Treatment Combinations:** $2 \times 2 \times 2 = 8$ unique treatment combinations.
* **Replications:** Each subject completed every treatment combination multiple times (yielding a total sample size of **$N = 48$** observations, or 16 trials per block).

---

## Power Analysis & Sample Size Justification

Before collecting our final data, we ran a preliminary "Dry Run" ($N = 24$) to estimate the natural variance of reaction times.

### 1. Estimating Variance ($\sigma^2$)
Using a $2 \times 2 \times 2$ ANOVA with blocking on subjects from our dry run data, we calculated the Mean Squared Residuals:
$$\sigma^2 \approx 405.5 \text{ ms}^2 \quad (\sigma \approx 20.14 \text{ ms})$$

### 2. Simulation & Data Generating Process (DGP)
We simulated data using the following mathematical model:
$$y_{ijkl} = \mu + A_i + B_j + C_k + D_l + (AB)_{ij} + (AC)_{ik} + (BC)_{jk} + (ABC)_{ijk} + \epsilon_{ijkl}$$
Where:
* $\mu = 250 \text{ ms}$ (overall mean reaction time)
* $A_i, B_j, C_k$ are main effects ($\approx \pm 8.5 \text{ ms}$)
* $D_l$ is the Subject Block effect (Aditya: $-20 \text{ ms}$, Benjamin: $+10 \text{ ms}$, Isabel: $+10 \text{ ms}$)
* $\epsilon_{ijkl} \sim N(0, 405.5)$ represents random, irreducible error.

### 3. Power Curves
Simulating power over various sample sizes $n$ across 100 iterations at a significance level of $\alpha = 0.05$, we sought to achieve a target power of:
$$0.7 \le 1 - \beta \le 0.9$$

```r
# Power simulation loop in R (summarized)
power_data <- map_dfr(n_values, function(current_n) {
  map_dfr(factors_to_test, function(f) {
    rejections <- replicate(100, one_sim(mu, A, B, C, D, AB, AC, BC, ABC, 405.5, current_n, f))
    data.frame(n = current_n, power = mean(rejections), factor = f)
  })
})
```

> [!TIP]
> **Conclusion:** The simulation demonstrated that a sample size of **$N = 48$** was sufficient to detect relatively small effect sizes ($< 10 \text{ ms}$) with $80\%$ power, prompting us to scale up our planned experimental observations from 24 to 48.

---

## Experimental Protocol

To ensure reliability and minimize noise across trials, subjects strictly adhered to the following standard operating protocol:

### Step 1: Environmental & Hardware Standardization
* **Testing Environment:** Sit in a quiet room with consistent lighting and minimal physical noise.
* **Hardware Setup:** Connect laptops to a power source. **A physical mouse must be used** (rather than a trackpad) because trackpads introduce massive mechanical latency.
* **Software Environment:** Close all background applications, notifications, and unused browser tabs. Open only the [Human Benchmark Reaction Test](https://humanbenchmark.com/tests/reactiontime).

### Step 2: Randomization
A balanced randomization plan was generated in R to assign treatment combinations to block sequences (randomizing the order of trials for each subject):
```r
n <- 48
dat <- data.frame(
  breathing = factor(sample(rep(1:2, each = n/2))),
  time      = factor(sample(rep(1:2, each = n/2))),
  video     = factor(sample(rep(1:2, each = n/2))),
  person    = factor(sample(rep(1:3, each = n/3)))
)
```

### Step 3: Execution
Look at the assigned treatment for the specific trial and adjust behaviors:
1. **Breathing:** Normal vs. holding breath immediately before clicking "Start" and maintaining it through the end of the trial.
2. **Video:** Control (silence) vs. playing a loud, high-stimulus, visually conspicuous distraction video.
3. **Time of Day:** Morning trials must be completed within 3 hours of waking up (before 12 PM). Evening trials must be completed between 6:00 PM – 10:00 PM.

### Step 4: Measurement & Logging
* Tap "Start" and click as soon as the screen turns green.
* Record the result in milliseconds on the shared Google Sheet.
* Export the final spreadsheet for statistical analysis

---

## Results & Statistical Analysis

The gathered dataset was analyzed in R using a three-way Analysis of Variance (ANOVA).

### 1. The Statistical Model
$$y_{ijkl} = \mu + P_i + B_j + V_k + T_l + (BV)_{jk} + (BT)_{jl} + (VT)_{kl} + (BVT)_{jkl} + \epsilon_{ijkl}$$
Where:
* $y_{ijkl}$: Observed reaction time (ms)
* $P_i$: Effect of the Person block ($i \in \{\text{Aditya, Benjamin, Isabel}\}$)
* $B_j, V_k, T_l$: Main effects of Breathing, Video Distraction, and Time of Day
* Pairs and triples represent interaction terms
* $\epsilon_{ijkl}$: Residual error

### 2. ANOVA Results Table
Running `anova(lm(Reaction_time_.ms. ~ Person + Breathing * Time * Video, data = dat))` yielded the following:

| Source | Df | Sum Sq | Mean Sq | F value | Pr(>F) | Significance |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **Person (Block)** | 2 | 39669.1 | 19834.6 | 39.467 | < 0.001 | *** |
| **Breathing** | 1 | 823.1 | 823.1 | 1.638 | 0.209 | |
| **Time** | 1 | 1792.0 | 1792.0 | 3.565 | 0.067 | . |
| **Video** | 1 | 55.4 | 55.4 | 0.110 | 0.742 | |
| **Breathing:Time** | 1 | 42.4 | 42.4 | 0.084 | 0.773 | |
| **Breathing:Video** | 1 | 797.7 | 797.7 | 1.587 | 0.216 | |
| **Time:Video** | 1 | 49.3 | 49.3 | 0.098 | 0.756 | |
| **Breathing:Time:Video** | 1 | 50.8 | 50.8 | 0.101 | 0.752 | |
| **Residuals** | 38 | 19097.6 | 502.6 | | | |

*Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1*

### 3. Key Findings

#### The Power of Blocking (Person Effect)
* **Result:** Highly significant ($p < 0.001$).
* **Insight:** Individual subject variance accounted for **66.4%** of the total Sum of Squares ($39,669$ out of $59,671$). This confirms that blocking was highly effective; if we had not blocked by subject, this massive baseline variance would have flooded our residuals, making it impossible to detect any other effects.

```
Total SS = 59,671 ms² |█████████████████████░░░░░░░░░░|
Block SS = 39,669 ms² |██████████████       | (66.4% Explained by Blocking)
```

#### Circadian Rhythms (Time of Day Effect)
* **Result:** Marginally significant ($p = 0.067$ at a threshold of $\alpha = 0.10$).
* **Insight:** Participants reacted roughly **10 ms faster in the evening** compared to the morning. This fits general circadian expectations, where physiological alertness rises throughout the day.

#### Breathing & Video Distraction
* **Result:** Statistically non-significant ($p > 0.10$).
* **Insight:** Neither holding your breath nor playing a background video showed a statistically significant main effect on reaction times. The visual fluctuations observed in raw data were largely swallowed by residual noise.

#### Interactions
* **Result:** No significant two-way or three-way interactions were detected.
* **Insight:** Although the interaction plot between **Breathing & Video** showed visual divergence (indicating that the effect of breath-holding might differ when listening to a video), it did not pass the threshold for statistical significance ($p = 0.216$). The 3-way interaction was also highly non-significant ($p = 0.752$).

---

## Limitations & Lessons Learned

While the block design successfully isolated individual variability, several factors introduced experimental noise:
1. **Uncontrolled Testing Environments:** Subjects performed trials in different physical spaces, with varying ambient noise, table ergonomics, and room lighting.
2. **Device Discrepancies:** Different computer brands, screen refresh rates, and mouse types (varying in click latency) were used by each subject.
3. **Distraction Heterogeneity:** Rather than using a single standardized video distraction, subjects selected their own "high-stimulus" videos. Distraction intensity thus varied from subject to subject.
4. **Carry-over & Learning Effects:** Performing several reaction tests consecutively may have introduced rapid motor learning or cognitive fatigue, which we did not explicitly model or counter-balance.

---

## How to Reproduce

### Prerequisites
To compile the reports and run the analysis, you need [RStudio](https://posit.co/download/rstudio-desktop/) and the [Quarto CLI](https://quarto.org/docs/get-started/).

Install the required packages in R:
```r
install.packages(c("tidyverse", "ggplot2", "knitr", "gtable"))
```

### Analysis Steps
1. Clone the repository and navigate into the project directory:
   ```bash
   git clone https://github.com/bzacharia999/What-Affects-Reaction-Time-.git
   cd What-Affects-Reaction-Time-
   ```
2. Open `FP_Results.qmd` or `Project Protocol.qmd` in RStudio.
3. Render the document using Quarto:
   ```bash
   quarto render FP_Results.qmd --to pdf
   ```
   *This compiles the R analysis, executes the linear model and ANOVA, renders the plots, and outputs a formatted PDF report.*

---

## Repository Contents
* `Project Protocol.qmd` / `Project-Protocol (2).pdf` — Pre-registration experimental protocol and power simulation script.
* `FP_Results.qmd` / `Final Report - FP_Results.pdf` — The final project report containing methodology, plots, ANOVA analysis, and raw data appendix.


