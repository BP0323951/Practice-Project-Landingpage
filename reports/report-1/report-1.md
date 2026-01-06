---
layout: default
---

# Deprivation and GP Quality Outcome Payments

### Executive Summary
This project will look at the NHS’s voluntary Quality & Outcome Framework (QOF)

_“QOF rewards GP practices for the provision of 'quality care' and helps to fund further improvements in the delivery of clinical care. QOF points are achieved based on the proportions of patients on defined disease registers who receive defined interventions”_ (NHS Digital, 2023)

In 2023 the Health Equity Evidence Centre published a compelling online article called “Structural inequalities in primary care – the facts and figures” (Appel and Ford, 2023). This examined QOF and included the graph in figure 1.1 which appears to indicate more deprived GP practices achieve less QOF points. Thus QOF does not reduce inequality.

<figure>
  <img src="{{ '/reports/report-1/images/fig_1_1.png' | relative_url }}"
       alt="Figure 1.1"
       style="max-width:75%; height:auto;">
  <figcaption>
    Figure 1.1 The average QOF points when split by index of multiple deprivation quintile,
    showing fewer points awarded to the most deprived areas.
  </figcaption>
</figure>

Note that in the figure 1.1 IMD quintile 1 relates to the least deprived, however this is not the industry standard and so within this study IMD Quintile 1 will relate to the most deprived.

QOF is designed to assist with the additional workload associated with higher prevalence of disease it would perhaps be logical to speculate that perhaps higher prevalence would result in lower QOF achievement points. This would reduce the burden of inequality across the country: 

_“We think that incentives are a valuable tool for effectively allocating resources towards priority clinical areas. Some studies demonstrate that the introduction of QOF resulted in enhanced quality of care, reduced variation and better patient outcomes. They also consistently demonstrate that incentives lead to increased levels of recorded activity.”_ (UK Government, 2024)

Having defined a subset of the population (using data only from the South West of England), a linear regression model will be used, written in Python code and deployed in VSCode. The model will include both prevalence and IMD features. Metrics, such as RMSE, will be applied to the model created and assessed for goodness of fit. 

This project demonstrates that neither prevalence of disease nor deprivation reliably predict the number of QOF points achieved. It will also fail to recreate the evidence found in figure 1.1 and whilst a general trend may suggest a relationship it is not sufficiently consistent enough to support a regression model.

### Data Engineering
A variety of open source data will be used from multiple sources. Using the Government Data Quality Framework (The Government Data Quality Framework, 2020) the data quality will be assessed and the data joined, illustrated in figure 1.2 

<figure>
  <img src="{{ '/reports/report-1/images/qof_data_eng_diagram.png' | relative_url }}"
       alt="Figure 1.2"
       style="max-width:60%; height:auto;">
  <figcaption>
    Figure 1.2 the data linkage model showing how 5 datasets are combined to create one wide dataset with Practice ID as the primary key. 
  </figcaption>
</figure>

All data is for the financial year 2023/24, the latest available and ensures data timeliness.

The data sources are reputable and will have undergone strict data quality checks before national release. However, columns used within the model were checked for NA values (data completeness), data outside of possible limits (data validation) and the data type across the columns (data consistency) as per the Government Data Quality Framework (The Government Data Quality Framework, 2020). As expected, all tests were passed.

All joins were performed on a 1-1 basis, or many-1, in both instances the left hand dataframe in the join would contain the same number of rows as the target dataframe. This was checked before and after the join. An example being ‘QOF data with practice code’: ‘Postcode with Deprivation’ which is a many-1 join that will not include all rows from the ‘Postcode with Deprivation’ table.

Before joining to practice postcode data, a derived field name ‘ACHIEVED’ is created from the QOF Data and checked for data consistency, see figure 1.3 for code and output.

<figure>
  <img src="{{ '/reports/report-1/images/fig_1_3.png'| relative_url }}"
       alt="Figure 1.3"
       style="max-width:75%; height:auto;">
  <figcaption>
    Figure 1.3 code used to create aggregated (sum) field ‘ACHIEVED’ being the sum of all QOF points awarded across all domains.
  </figcaption>
</figure>

The data set is split into df_test and df_train,a random state is specified within the code to ensure that any analysis can be reproduced.  

### Data Visualisation
Initially figure 1.1 was recreated, as closely as possible, using the subset dataset. Instead of ‘Percentage of Max Points’ the ‘Achieved QOF Points’ was used (absolute value and not percentage of total). Figure 1.4 shows that while IMD Quintile 5 (least deprived practices) do indeed receive more points that IMD Quintile 1 (most deprived) the difference becomes minimal when the y-axis is drawn to zero. Furthermore, this effect is negated when the points are converted into monetary values.

<figure>
  <img src="{{ 'images/fig_1_4.png'| relative_url }}"
       alt="Figure 1.4"
       style="max-width:80%; height:auto;">
</figure>

This suggests that standard deviation will be low, figure 1.5 shows a regression plot between IMD Rank (a component of IMD Quintile) and ACHIEVED QOF points, the relationship does appear weak. 

<figure>
  <img src="{{ '/reports/report-1/images/fig_1_5.svg'| relative_url }}"
       alt="Figure 1.5"
       style="max-width:80%; height:auto;">
</figure>

<figure>
  <img src="{{ '/reports/report-1/images/fig_1_6.png'| relative_url }}"
       alt="Figure 1.6"
       style="max-width:80%; height:auto;">
</figure>

Figure 1.6 illustrates the correlation matrix between all possible features. As a number of features are correlated >0.8 (using Pearsons Correlation) some features are dropped. Reducing the number of features protects against over-fitting the data. There is reduced correlation between the prevalence variables and the deprivation variables which all end with ‘Rank’.

### Data Analytics
Within Python the ‘statsmodels’ package was used. This package shares similarities with R syntax making it interpretable for a wider audience. The model was created using the ‘df_train’ dataframe and whilst the model summary is invaluable it does not test the df_test dataframe. A mixture of sklearn and numpy packages along with a locally defined function will be used to test the model.  

<figure>
  <img src="{{ '/reports/report-1/images/fig_1_7_table.png'| relative_url }}"
       alt="Figure 1.7"
       style="max-width:80%; height:auto;">
</figure>

<figure>
  <img src="{{ '/reports/report-1/images/fig_1_7_2.png'| relative_url }}"
       alt="Figure 1.7.2"
       style="max-width:80%; height:auto;">
</figure>

Before starting this project it was assumed that both prevalence and deprivation would create a linear regression model that accurately predicted Achieved QOF points, with either prevalence or deprivation showing overall to be a more successful predictor. This was not achieved and instead most of the variation in achieved points must be due to factors outside of the data model. The author has spent some time considering whether a pre-bias in perceived outcome prevented a full investigation into whether this was a feasible project idea.

However, with reference to the original graph which started the project (figure 1.1) it was compelling to hypothesis that deprivation could be aligned with QOF achievement. This was not quite the pattern observed within the subset of data used in the model, which whilst more quadratic in nature suggested only the least deprived 20% (IMD Quintil 5) has a different distribution. Additionally, to highlight the difference in interpretation where the y-axis is limited (left plot in figure 1.8) compared to removing the limits on the y-axis (right plot in figure 1.8)

<figure class="double-figure">
  <div class="double-figure-images">
    <img src="{{ '/reports/report-1/images/fig_1_8_1.png.png' | relative_url }}"
         alt="Figure 1.8.1">
    <img src="{{ '/reports/report-1/images/fig_1_8_2.png.png' | relative_url }}"
         alt="Figure 1.8.2">
  </div>

  <figcaption>
    Figure 1.8 Both the potential quadratic relationship between Achieved QOF points and IMD Quintile,
    along with the visual effect of limiting the y-axis vs using an absolute scale.
  </figcaption>
</figure>

Also, rather than using IMD Quintile, which is a categorical value the models themselves were run on rank, which is more continuous (the rank of each LSOA across the country from the most to least deprived). The difference between a descriptive statistic (figure 1.1 mean by IMD Quintile and figure 1.8 boxplot by IMD Quintile) and using the IMD Rank within a regression model is that the descriptive grouping by quintile does not show whether the variation in correlation/patterns required within a regression model is truly present. 

As a conclusion, a naïve approach was used to feature selection, perhaps using a feature reduction algorithm like LASSO linear regression could also improve performance by applying a penalty shrinking some coefficients, sometimes to zero and hence performing feature selection. It is also likely that it is features not included in the dataset which account for the differences observed. However, as previously discussed it is also likely that the small standard deviation makes it difficult for the model to find meaningful relationships between features and the output variable (QOF Achieved points).
