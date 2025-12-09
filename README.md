
INTRO TO R & DATA VISUALISATION WITH CYTOKINE DATA
=================================================

This is a from-scratch tutorial designed for MSc Immunology / Immunotherapeutics students
learning R and data visualisation.
```bash
You will learn how to:
- Load data from Excel
- Understand basic R objects (vectors, data frames, factors)
- Use ggplot2 for publication-quality figures
- Recreate common immune data figures:
  * Histograms
  * Boxplots
  * Scatterplots
  * Contingency heatmaps
  * PCA
  * Heatmaps
  * Correlation matrices
  * Linear models + forest plots

The example dataset is:
sample_cytokine_data.xlsx
```

-------------------------------------------------
0. INSTALLING AND LOADING PACKAGES
-------------------------------------------------
```bash
Install once:

install.packages(c(
  "tidyverse",
  "readxl",
  "pheatmap",
  "broom",
  "viridis"
))
```
Load every session:

```bash
library(tidyverse)
library(readxl)
library(pheatmap)
library(broom)
library(viridis)
```

-------------------------------------------------
1. LOADING THE DATA
-------------------------------------------------
```bash
data <- read_excel("sample_cytokine_data.xlsx")

head(data)
str(data)
```
Expected columns:
- IL-2, IL-6, TNF, IL-10   (numeric cytokines)
- Cohort                  (Healthy Controls / Disease Active / Disease Remission)
- Sex                     (Male / Female)
- Age                     (years)

Make categorical variables factors:
```bash
data <- data %>%
  mutate(
    Cohort = factor(Cohort),
    Sex    = factor(Sex)
  )
```

-------------------------------------------------
2. CORE R OBJECTS
-------------------------------------------------

2.1 VECTORS
```bash
ages <- c(53.8, 59.8, 79.2, 46.7)

mean(ages)
length(ages)
```

2.2 DATA FRAMES
```bash
mini_df <- tibble(
  Age    = ages,
  Cohort = c("Remission", "Active", "Remission", "Remission")
)

mini_df
```

-------------------------------------------------
3. HISTOGRAM OF TNF
-------------------------------------------------
```bash
ggplot(data, aes(x = TNF)) +
  geom_histogram(
    bins = 30,
    colour = "white",
    fill   = "steelblue"
  ) +
  labs(
    x = "TNF",
    y = "Count"
  ) +
  theme_minimal()
```

-------------------------------------------------
4. TNF BY COHORT (BOXPLOT + POINTS)
-------------------------------------------------
```bash
ggplot(data, aes(x = Cohort, y = TNF, fill = Cohort)) +
  geom_boxplot(outlier.shape = NA, alpha = 0.5) +
  geom_point(
    size = 2.5,
    shape = 21,
    colour = "black",
    position = position_jitter(width = 0.15)
  ) +
  scale_fill_viridis_d(end = 0.85) +
  theme_minimal() +
  theme(legend.position = "none")
```

-------------------------------------------------
5. IL-10 vs TNF SCATTERPLOT
-------------------------------------------------
```bash
ggplot(data, aes(x = `IL-10`, y = TNF, fill = Cohort)) +
  geom_point(size = 3, shape = 21, colour = "black") +
  scale_fill_viridis_d(end = 0.85) +
  theme_minimal()
```

-------------------------------------------------
6. CONTINGENCY TABLE HEATMAP (COHORT x SEX)
-------------------------------------------------
```bash
ct <- data %>%
  count(Cohort, Sex)

ggplot(ct, aes(x = Sex, y = Cohort, fill = n)) +
  geom_tile(colour = "white") +
  geom_text(aes(label = n), size = 4) +
  scale_fill_viridis_c(end = 0.9) +
  theme_minimal()
```

-------------------------------------------------
7. PCA OF CYTOKINES
-------------------------------------------------
```bash
cytokines <- c("IL-2", "IL-6", "TNF", "IL-10")

pca <- prcomp(
  data %>% select(all_of(cytokines)),
  center = TRUE,
  scale. = TRUE
)

var_expl <- round((pca$sdev^2 / sum(pca$sdev^2)) * 100, 2)

pca_df <- as.data.frame(pca$x) %>%
  mutate(Cohort = data$Cohort)

ggplot(pca_df, aes(PC1, PC2, fill = Cohort)) +
  geom_point(size = 3, shape = 21, colour = "black") +
  scale_fill_viridis_d(end = 0.85) +
  labs(
    x = paste0("PC1 (", var_expl[1], "%)"),
    y = paste0("PC2 (", var_expl[2], "%)")
  ) +
  theme_minimal()
```

-------------------------------------------------
8. CYTOKINE HEATMAP
-------------------------------------------------
```bash
cyto_mat <- data %>%
  select(all_of(cytokines)) %>%
  as.matrix() %>%
  scale()

pheatmap(
  cyto_mat,
  show_colnames = FALSE,
  clustering_method = "ward.D2"
)
```

-------------------------------------------------
9. CYTOKINE CORRELATION MATRIX
-------------------------------------------------
```bash
cor_mat <- data %>%
  select(all_of(cytokines)) %>%
  cor(use = "pairwise.complete.obs")

pheatmap(
  cor_mat,
  display_numbers = TRUE
)
```

-------------------------------------------------
10. LINEAR MODEL + FOREST PLOT
-------------------------------------------------
```bash
lm_model <- lm(TNF ~ Cohort + scale(Age) + Sex, data = data)

lm_tidy <- tidy(lm_model, conf.int = TRUE) %>%
  filter(term != "(Intercept)")

ggplot(lm_tidy, aes(x = estimate, y = term)) +
  geom_vline(xintercept = 0, linetype = "dashed") +
  geom_point(size = 3) +
  geom_errorbarh(aes(xmin = conf.low, xmax = conf.high), height = 0.2) +
  theme_minimal() +
  labs(x = "Effect size (estimate ± 95% CI)", y = NULL)
```

-------------------------------------------------
11. AGE vs TNF WITH REGRESSION LINES
-------------------------------------------------
```bash
ggplot(data, aes(x = Age, y = TNF, fill = Cohort)) +
  geom_point(size = 3, shape = 21, colour = "black") +
  geom_smooth(method = "lm", se = FALSE, linetype = "dashed") +
  scale_fill_viridis_d(end = 0.85) +
  theme_minimal()
```

-------------------------------------------------
END
-------------------------------------------------
