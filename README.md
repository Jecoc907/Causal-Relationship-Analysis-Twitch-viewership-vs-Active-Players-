# Causal-Relationship-Analysis-Twitch-viewership-vs-Active-Players-
![Screenshot 2024-08-30 at 10 37 14 PM](https://github.com/user-attachments/assets/681053cc-5dcb-418c-b7d1-6af56afba8af)

## Objective
To analyze the causal relationship between a game’s Twitch viewership and its active player engagement, using the case of Tom Clancy’s Rainbow Six Siege as a primary example, and extending the analysis to Counter-Strike 2 and Rust.

## Background
![Screenshot 2024-08-30 at 10 47 13 PM](https://github.com/user-attachments/assets/bb1c5729-c217-455b-ba7c-07e07fd7c2c8)
(Link_to_article: https://siege.gg/news/siege-reaches-its-highest-player-peak-on-steam-in-almost-three-years) \
In February 2024, Ubisoft’s Six Invitational 2024, co-streamed by Twitch’s most subscribed channel, Jynxzi, set a record for the highest peak viewership (521,349 peak viewers) in the event’s history. Subsequently, Rainbow Six Siege achieved a record high in average player count (over 60,000) since April 2021. This event highlights the potential influence of Twitch viewership on player engagement. Given the rapid growth of the live-streaming industry, understanding this relationship can offer valuable insights for gaming companies to strategize their marketing and engagement efforts.

## Data Description
Due to limited access to historical data from Twitch and Steam APIs, we used a combination of datasets from two third-party websites: SteamDB.com and Sullygnome.com. These websites are online statistics and analytics databases for Steam and Twitch, respectively. According to the platforms' websites, APIs are used to collect information and are polled every 5-10 minutes to ensure data completeness and accuracy.
Tom Clancy’s Rainbow Six Siege is the primary example for this project. Additionally, we included other games for reference based on the following criteria:
1. Must be online multiplayer shooting games.
2. Must have more than 5 years of data to ensure data sufficiency. 3. Must be available on the Steam platform.
Based on these criteria, Counter-Strike 2 and Rust were selected as reference games for our analysis. The original data from SteamDB and Sullygnome provided 26 columns with daily player engagement and Twitch streaming information for each game, such as Daily Peak Players, Twitch Daily Peak Viewers, and Number of Channels. We manually added 6 new columns to account for potential confounding variables that could affect Twitch viewership and active player counts: "Day of the Week'', "Original Price", "Discount" (the amount of discount in percentage), "Free Week/Weekend'' (a binary variable that indicates whether the game is free on that day), “Events” (a string variable that contains information of special events, such as marketing campaigns), and "Tournament (INTL)" (a binary variable that indicates whether there is international tournament). This resulted in three datasets, each with 32 columns, one for each target game. Below is a sample of the list of variables.
['DateTime', 'Day of the Week', 'Followers', 'Players', 'Average Players', 'Twitch Viewers', 'Positive reviews', 'Negative reviews', 'Rating', 'Final price', 'Historical low', 'Original price', 'discount', 'Free Weekend / Free Week', 'Tournament (INTL)', 'Events', 'Peak viewers', 'Average viewers', 'Peak channels', 'Average channels', 'Viewer ratio (Average view / Average Channel)', 'Top channel', '2nd channel', '3rd channel', '4th channel', '5th channel', 'Channels 6-10', 'Channels 11-25', 'Channels 26-50', 'Channels 51-100', 'Channels 101-250']

## Data Preprocessing

### Exploratory Data Analysis
The ideal KPI for study would be Daily Active Users (DAU), a commonly used metric in the gaming industry counting unique daily users, but due to privacy policies, it is not usually disclosed to the public. In our datasets, “Players”, daily peak players, and “Average Players” are two potential KPIs to be considered, and they display similar trends across the whole dataset. However, “Average Players” only have values for less than two years, and because it is unclear how API derives the average value, it is impossible for us to impute the average values. Therefore, we decided to use “Players”, the poxy that is similar to DAU and has no missing values, as our KPI although the values are slightly lower than real DAU.
Since there are missing values in some columns in our raw data, we omitted columns that are not helpful in this study or have better substitutes, such as “Followers”, “Average Players”, “Positive reviews”, and “Negative reviews”. Other columns, “Free Weekend / Free Week”, “Tournament (INTL), and “Events”, were converted into binary values, where NAs were denoted as 0 (not happening) and non-NAs were turned into 1 (happening). Besides, for the purpose of controlling a specific day’s influence on players, we encoded “Day of the Week” into seven separate dummy variables. Moreover, the lowest game price is an important factor leading to users’ behaviors, so to control for the price’s impact, we created a new binary variable “is_historical_low” by incorporating “Final price” and “Historical low”.

### Stationarity and Deseasonality
It is crucial to understand the trend of time series data and the autocorrelation, implying seasonality, within variables. Figure 1-3 plot the trend of “Peak viewers” and “Players” across the data. From a descriptive perspective, when “Peak viewers” reaches a spike, “Players” seems to have a peak immediately or within a short period of time, which laid the foundation of our study.

![Screenshot 2024-08-30 at 10 57 06 PM](https://github.com/user-attachments/assets/8f1cecce-d225-4b1b-b36d-cfc46066d38f)

![Screenshot 2024-08-30 at 10 57 16 PM](https://github.com/user-attachments/assets/0595c29f-f9c8-4d12-952d-9661c300ec62)

Time series data are expected to be stationary during any modeling, which refers to constant means and constant variance. We applied the Augmented Dicky-Fuller Test (ADF Test), a statistical significance test used to test time series’ stationarity, to all of the numerical features. The test result showed that the target variable “Players” is not stationary in all three games with a higher-than-threshold p-value. “Final price” in Counter-Strike 2 and “Rating” in Rust are also not stationary. Therefore, to keep scales on the same level, we took the first difference for all the numerical variables, and they became stationary when we reapplied the ADF Test. Figure 4-6 displays the plot of “Peak viewers” and “Players” after first differencing.

![Screenshot 2024-08-30 at 10 58 36 PM](https://github.com/user-attachments/assets/81cf2c93-dc0f-4767-ae2c-469b166f1c21)

![Screenshot 2024-08-30 at 10 58 58 PM](https://github.com/user-attachments/assets/35123db7-31f6-41c8-a56b-83849df0b525)

Moreover, seasonality is common in time series but it is not preferred when performing modeling analyses. Seasonality can be checked by autocorrelation scores ranging from -1 to 1, where the closer the magnitude to 1, the stronger the seasonality. We detected strong seasonality in “Players” in all three games. There are seasonal trends approximately every seven days with an autocorrelation score higher than 0.6 (Figure 5-7 in Appendix). Therefore, to remove the seasonality in “Players”, we derived a new variable “deseasonlized_players” by subtracting the decomposed seasonal value of players from “Players”. The new variable has no strong seasonality as most of the autocorrelation scores dropped under 0.25 (Figure 8-10 in Appendix).

### Lag

Not only can factors at time t have causal relationships with players, but historical data can also have potential causal relationships with players. To perform thorough analyses of causality, we shifted “Peak
viewers” from t-1 to t-14 and “Rating”, “Final Price”, and “is_historical_low” from t-1 to t-7 to mimic the lag effect. These lag variables are used selectively in the following modeling section.

## Modeling
### Linear Regression
Before implementing deeper analyses of the causal relationship between Players and Twitch viewers, we performed linear regressions on players to understand how Twitch viewership is potentially quantitatively associated with Players under the influence of confounding variables. To keep the results' robustness and reflect the true relationship between two variables, we used cleaned data to fit models for all three games even though outputs showed weaker associations.
#### Rainbow Six Siege
To control the confounding effect, we included “Rating”, “Final price”, Days of the Week, “Free Weekend / Free Week (bool)”, “Tournament (INTL)”, “is_historical_low_shift_5”, “Final price_shift_3”, and “Final price_shift_4” as our confounding variables. Trying a full model with “Peak viewers” and all its lags variables on “deaseaonalized_players”, we decided to check the association with “Peak viewers” and its 1, 6, and 13 lags respectively since they appear to have more significant results.
As the OLS outputs shown in Table 1, p-values of “Peak viewers” with shifts 1, 6, and 13 indicate that these three variables are statistically significant by using 0.05 as the threshold. Their positive coefficients and 95% confidence intervals imply there might be a potential positive effect of around 0.008 on the number of players. However, “Peak viewers” proves a weaker relationship. The p-value of 0.069 suggests the result is marginally significant with a likely positive influence but it is not convincing enough to be considered significant at the 5% significance level. The [-0.001, 0.017] confidence interval for the coefficient of “Peak viewers” also suggests that there might be some uncertainty about the true effects. Thus, we concluded that there is possibly a positive correlation between players and peak viewers but needs further analysis to test the magnitude.

![Screenshot 2024-08-30 at 11 00 52 PM](https://github.com/user-attachments/assets/dd62eef4-e28b-45a4-9cd4-35efa197678b)

#### Counter-Strike 2
In Counter-Strike 2’s linear regression model, we considered “Rating”, “Final price”, “Final price_shift_4”, “Tournament (INTL)” “Events”, Days of the Week, and “is_historical_low_shift_2” to control for confounding variables. “Peak viewers” and its shift of 1 and 2 days were included to test the association with players. 
The model summary indicates that all three variables are statistically significant with p-values of 0.002, 0.006, and 0.001 respectively. The coefficients of these three variables are greater than 0.01, and their 95% confidence intervals are within the range of positive numbers. Hence, we expect to achieve a positive causal effect estimation of Twitch viewers on players for this game in the following analyses.

![Screenshot 2024-08-30 at 11 01 42 PM](https://github.com/user-attachments/assets/6055bb44-c43a-4fd6-a599-648201d52de6)

#### Rust
We used similar confounding variables for Rust, except we substituted “is_historical_low_shift_2” with “is_historical_low”.  “Peak viewers” with lags of 4, 6, 11, and 13 days were taken into account as variables in this model.
As all five peak viewers variables are statistically significant, “Peak viewers” at time t, t-4, t-6, and t-13 display positive coefficients in predicting players, as well as their corresponding 95% confidence intervals. On the contrary, “Peak viewers” at time t-11 shows a negative coefficient with the magnitude of 0.0124 on players. It is highly likely that “Peak viewer” at different lags can have distinct effects on players. According to the overall outputs we derived from this model, it is more likely that Twitch viewers of Rust will have a positive relationship with players.

![Screenshot 2024-08-30 at 11 02 39 PM](https://github.com/user-attachments/assets/4b7712c5-f02e-4484-8ac8-89f826950323)

### Granger Causality Test
Before proceeding to other more complex algorithms, we applied the Granger Causality Test to determine if there was any evidence implying the relationship between players and Twitch viewers. The Granger Causality Test is a statistical concept used to learn the predictability of one time series on another. It is noteworthy that this test does not provide insights into the true causal relationship, instead it can tell important lags of the predicting variable. 
Since the Granger Causality Test requires data to be stationary and with no seasonality, we used stationary and deseasonalized players and peak viewers' values. In this study, we mainly test the formula

![Screenshot 2024-08-30 at 11 08 29 PM](https://github.com/user-attachments/assets/5a108412-e0b5-41e3-af2e-23508aac37b3)

The maximum lag order of this test was determined by running the vector autoregressive model between “Peak viewers” and “deseasonalized_players” to select orders using AIC. Utilizing the significance level of 0.05, we can conclude from Table 4 that “Peak viewers” granger causes “deseasonalized_players” starting from a lag of 1 for Rainbow Six Siege and Counter-Strike 2 and from a lag of 2 for Rust. This test result helps us choose the optimal lag of variables in the CDMI algorithm.

![Screenshot 2024-08-30 at 11 14 13 PM](https://github.com/user-attachments/assets/6ae4b672-c07e-4e2b-b51f-55522a3d799c)

### Causal Discovery Using Model Invariance (CDMI)
To infer causality between the predictor, Peak Viewers Diff, and the response variable, Peak Players Diff, we leveraged a methodology called Causal Discover Using Model Invariance (CDMI) via knockoffs.
Proposed by Ahmad, Shadaydeh, and Denzler (2022) in their paper, ‘Causal Discovery using Model Invariance through Knockoff Interventions’, the method uses DeepAR to model nonlinear interactions in multivariate time series and employs knockoff variables to create interventional environments, testing for causal relationships by comparing the invariance of the response residuals using statistical tests like the Kolmogorov-Smirnov test. If the residual distribution remains unchanged under intervention, the predictor is deemed non-causal; otherwise, it is considered causal.
In our implementation of the method, we chose to substitute DeepAR with simpler DNNs to model our time series due to the constraints of computational resources and various challenges during implementation attempts. The DNN was trained with the ADAM optimizer with an Exponential Decaying learning rate and its structure is as follows:

![Screenshot 2024-08-30 at 11 15 29 PM](https://github.com/user-attachments/assets/fe0f4c32-7a71-44b2-872e-beea892c8f50)

The other steps remain the same, as Python codes were adapted from the paper’s Github pages.
The identical causal discovery pipeline was applied to all three games. However, input variables were inconsistent across each game as certain variables were dropped due to the mathematical constraint of the knockoff generating algorithm. Among the dropped variables, there were no main predictors of interest (i.e., Peak Viewers Diff and its lagged versions) involved, and dropped variables were highly correlated with other variables with different time lags, which we believe have little effect on our final results. The detailed list of variables could be found below in each game’s section. Through CDMI via Knockoff, we identified several causal links between Peak Viewer Diff and Peak Player Diff with varied time lags across all 3 games.

#### Rainbow Six Siege

A total number of 33 variables were included in the model. The variables are categorized as follows:

Primary Variables:
- Peak viewers
- Final price
- Is_historical_low

![Screenshot 2024-08-30 at 11 19 44 PM](https://github.com/user-attachments/assets/0035453a-8181-45d4-89c3-4b4a6ff57712)

The data was split into training and test sets using TimeSeriesSplit from the scikit-learn library with a 3:1 ratio. The DNN model was then trained on the training set and validated on the test set, resulting in a validation MAPE of 117.40%. After DNN training, knockoff variables were generated using the DeepKnockoff package. Knockoff and original variables were compared using the compute_diagnostics function predefined by Ahmad et al. (2022) to check if they are statistically similar but decorrelated pairwise. The results are as follows:

![Screenshot 2024-08-30 at 11 17 28 PM](https://github.com/user-attachments/assets/077f8ac8-d7cf-4728-9306-60308a32e343)

The causal discovery was conducted using the causal_inference_knockoff function, adapted from the original research paper, by comparing the residual (MAPE) distributions before and after knockoff intervention. Five significant causal predictors were identified at a significance level of 0.05. Specifically, Peak viewers_shift_5 had a p-value of 0.033542, Peak viewers_shift_6 had a p-value of 0.000270, Peak viewers_shift_7 had a p-value of 0.012299, Peak viewers_shift_10 had a p-value of 0.033542, and Peak viewers_shift_11 had a p-value of 0.003967.

![Screenshot 2024-08-30 at 11 17 39 PM](https://github.com/user-attachments/assets/e046c82b-977b-4617-a31d-8b8aa17db84d)

#### Counter-Strike 2

For Counter-Strike 2, a total of 28 variables were included in the model, some variables of rating and is_historical_low were dropped due to the failure to produce knockoffs by the algorithm. The remaining variables are categorized and listed as follows:

Primary Variables:
- Peak viewers
- Final price
- is_historical_low

![Screenshot 2024-08-30 at 11 22 44 PM](https://github.com/user-attachments/assets/5477462d-4247-44a1-9133-dd351460b610)

The data was then split into training and test sets using TimeSeriesSplit from the scikit-learn library in a 3:1 ratio. A new DNN model was trained, producing a validation MAPE of 114.20%. Knockoff variables were generated again using the DeepKnockoff package. After obtaining the knockoff variables, a comparison was made to get the following metrics results:

![Screenshot 2024-08-30 at 11 23 13 PM](https://github.com/user-attachments/assets/31679d04-92d9-4bf8-a7ba-6bc3456c5a7d)

However, Causal discovery using Model Invariance with Counter-Strike 2 data did not identify any significant causal variables. From a business perspective as well as visual observation of the data, this result is understandable in the case of Counter-Strike. As one of the most played games on Steam, Counter-Strike's long-standing popularity predates the emergence of Twitch as a streaming platform. Consequently, the effect of Twitch in bringing new activity to the game is likely very limited.

#### Rust

For Rust, a total of 32 variables were included in the model. These variables are categorized and listed as follows:

Primary Variables:
- Peak viewers
- Final price
- is_historical_low

![Screenshot 2024-08-30 at 11 24 18 PM](https://github.com/user-attachments/assets/a02d2038-5132-45fd-b422-530dc9153ed4)

The same process was applied to the data, comparison metrics between the knockoff and original variables are as follows: 

![Screenshot 2024-08-30 at 11 24 43 PM](https://github.com/user-attachments/assets/a4c8952c-1a71-403a-aeb1-4a91c03d9030)

For the Rust data, Peak viewers_shift_5 (p = 0.033542), Peak viewers_shift_6 (p = 0.012299), and Peak viewers_shift_14 (p = 0.001116) were identified as significant causal predictors for Peak Player Diff. 

![Screenshot 2024-08-30 at 11 25 06 PM](https://github.com/user-attachments/assets/ee127901-224d-40f2-9485-bfda141e689e)

### Latent Peter and Clark Momentary Conditional Independence (LPCMCI)

To cross-validate the CDMI results and further estimate the causal effect, we employed the PCMCI algorithm to identify causal relationships and interactions between multiple time series. Specifically, we used Latent-PCMCI (LPCMCI) to handle unobserved confounders. The algorithm was implemented with the TIGRAMITE library in Python.
The inputs for the algorithm were two time series: Peak Viewer Diff and Peak Player Diff. For the conditional independence test, we used the partial correlation test. We chose tau_max = 14 to look back 14 days (i.e., lagged by 1 to 14 days) and used the default pc_alpha = 0.05 as the significance threshold.
After obtaining the LPCMCI results for causal link identification, we proceeded to estimate the causal effect of our predictor on the response using the CausalEffects class from the TIGRAMITE library with adjustment sets. However, 2 out of the 3 outputs (specifically, the causal graphs from t-14 to t for Rainbow Six Siege and Rust) could not be solved by the estimation algorithm due to the graph’s complexity involving too many nodes and edges.
To quantify the causal effect as much as possible, we simplified the graphs to retain as many causal relationships as possible while allowing the algorithm to solve for an estimation. The simplification was done by pruning links with higher p-values, one by one, until optimal adjustment sets could be found, minimizing potential bias introduced to the model. If links with relatively higher p-values were pruned and the optimal set still could not be found, we further pruned randomly selected links until a solution was found. Throughout this process, causal links of interest (i.e., links from Peak Viewer Diff to Peak Player Diff) remained untouched. 

#### Rainbow Six Siege
Two significant causal links from Peak Viewer Diff to Peak Player Diff were found. Specifically, Peak Viewer Diff at time t-1 was identified to cause Peak Player Diff at time t with a p-value of 0.04696, and Peak Viewer Diff at time t-14 was identified to cause Peak Player Diff at time t with a p-value of 0.00173. The time series DAG plot of the algorithm output is shown below:

![Screenshot 2024-08-30 at 11 29 42 PM](https://github.com/user-attachments/assets/2338bc2e-f20b-4b1f-9ad3-578fbeb5bb3a)

Causal graph pruning was applied to the time series DAG because the algorithm failed to find an optimal adjustment set for causal effect identification and estimation. A total of 15 links were pruned to obtain a solution, leaving only 5 links, including the 2 targeted links. This pruning could heavily impact the estimation validity. The pruned causal DAG is shown below:

![Screenshot 2024-08-30 at 11 30 01 PM](https://github.com/user-attachments/assets/90aab4ed-df80-418b-bff2-f2b68a917f0e)

The causal effect was then estimated using a linear regression approximation by observing the change in the response variable when incrementing the inputs by 1. The combined effect of Peak Viewer Diff lag 1 and Peak Viewer Diff lag 14 is -0.0030. The individual effects of the two variables were 0.0119 and -0.0130, respectively. This unexpected result, which contradicts findings from the other two games, might be attributed to the heavily pruned DAG and the resulting loss of significant links that could otherwise alter the results.

#### Counter-Strike 2

Two significant causal links from Peak Viewer Diff to Peak Player Diff were found. Specifically, Peak Viewer Diff at time t-1 was identified to cause Peak Player Diff at time t with a p-value of 0.00162, and Peak Viewer Diff at time t-2 was identified to cause Peak Player Diff at time t with a p-value of 0.02314. The time series DAG plot of the algorithm output is shown below:

![Screenshot 2024-08-30 at 11 30 35 PM](https://github.com/user-attachments/assets/da354023-2b55-4e25-8684-f978b25f3564)

The causal effect estimation for Counter-Strike 2 ran successfully without any need for pruning. A combined effect of 0.0235 was identified, suggesting a 2% conversion ratio from Twitch viewership to active players for this game.

#### Rust

Again, two significant causal links from Peak Viewer Diff to Peak Player Diff were found. Specifically, Peak Viewer Diff at time t was identified to cause Peak Player Diff at time t with a p-value of 0.00000, and Peak Viewer Diff at time t-4 was identified to cause Peak Player Diff at time t with a p-value of 0.00105. The time series DAG plot of the algorithm output is shown below:

![Screenshot 2024-08-30 at 11 31 27 PM](https://github.com/user-attachments/assets/8bdca381-5be8-4474-a03b-ade8d4769060)

Causal graph pruning was again applied to the time series DAG because the algorithm failed to find an optimal adjustment set for causal effect identification and estimation. A total of 9 links were pruned to get a final solution, leaving 11 links untouched, including the 2 targeted links. The pruned causal DAG is shown below:

![Screenshot 2024-08-30 at 11 31 41 PM](https://github.com/user-attachments/assets/71ac10c7-11b5-4cd4-b4bf-ac1cb310fb59)

The causal effect estimation was conducted, and a combined causal effect of 0.0418 was identified, suggesting a 4% conversion ratio from Twitch viewership to active players for Rust.

### Modeling Results

Based on comprehensive results from four different modeling approaches applied to three selected games, we found evidence of a causal relationship between Twitch peak viewers and Steam peak players. Our analysis suggests that increased viewership on Twitch tends to bring new active players to the games investigated.
We also discovered that this causal relationship usually exhibits a time delay, with the impact from viewership to players typically taking between 1 to 2 weeks to manifest. However, there is also evidence that an immediate impact can occur under certain circumstances, and the duration of these delays can vary between games.
Finally, we obtained estimates of the causal effect for each game. Despite the range of effect sizes spanning from -0.001 to 0.04, estimates between 0.02 and 0.04 seem more reliable, considering the significant pruning of the graph that could potentially lead to biased results. The varying results also suggest between-game variation, as each game possesses unique characteristics and player bases.

## Follow-up Analysis on Twitch Viewership

After confirming the causal relationship between Twitch viewership and player engagement, the business question for companies becomes how to increase their viewership. To address this, we conducted a follow-up analysis based on the recent success of Rainbow Six Siege (RS6). Our case study focused on two key factors: Influence of Top Streamers and Impact of Tournaments.

1. Influence of Top Streamers: To understand the influence of top streamers over the years, we analyzed the annual total viewership from the top channel compared to all channels, calculating the ratio. Among our three target games, Rainbow Six Siege exhibited a significantly increasing trend in top channel influence starting from 2020, rising from 27.2% to 47.5% (corresponding visualizations are attached in the Appendix). This suggests that top streamers have had a notable impact on RS6's Twitch viewership over years. This could potentially explain the huge success of the Six Invitational 2024, which was co-streamed with top streamer Jynxzi.
2. Impact of Tournaments: We examined whether holding tournaments effectively promotes a game.
   2.1 Player and Viewership Comparison:
     a) During tournaments, we observed a 9.78% increase in players and a 319% increase in Twitch viewership compared to non-tournament periods.
   2.2 Monthly Viewership Analysis:
     a) Grouping RS6’s viewership data by month, we observed a noticeable spike in viewership during tournament periods. This pattern was consistent across the two other target games, Counter-Strike 2 and Rust, though not to the same extent.
3. These findings indicate that holding tournaments with top streamers is effective in drawing attention from both new and current players for RS6. However, similar events (co-streaming tournaments with top streamers) were not observed for the other target games, suggesting that the impact of tournaments and top streamers can vary significantly between games.

From our analysis, we learned that Twitch viewership distribution, the streaming environment, and the nature of each game are unique. Therefore, it is not universally applicable to conclude that all gaming companies should collaborate with top Twitch streamers or organize tournaments. Instead, we developed a more general procedure to help companies devise effective marketing strategies tailored to their specific games.

## Recommendations

### General Procedure for Developing Marketing Strategies
![Screenshot 2024-08-30 at 11 42 09 PM](https://github.com/user-attachments/assets/8cb74c29-15d6-4e6b-9f36-a81d59ef0c00)

To develop a game-unique marketing strategy, companies should first examine the relationship between active players and Twitch viewership to determine if there is a causal relationship or correlation, and assess the strength of this relationship. If the relationship is significant, they should compare Twitch with other traditional channels for promoting the game using different KPIs, such as budget and ROI. In researching effective collaboration with Twitch, companies might consider partnering with top streamers who have a significant influence on viewership, as seen in Rainbow Six Siege, or collaborating with multiple channels if top streamer influence is less pronounced.

### Focus on Game Development

When our team was researching the impact of different marketing strategies, we noticed that most events only create short-term effects (appendix). Therefore, alongside these efforts, companies must focus on game development by continuously improving user experience through new game modes, features, and robust anti-cheat systems to keep players engaged. Developing innovative content and ensuring a fair gaming environment are crucial for retaining and attracting players in the long term. Being responsive to user feedback is equally important; companies should actively collect and utilize feedback to guide game development and improvements, ensuring that player concerns and suggestions are addressed promptly.

### Utilize Twitch / Streaming as a Feedback Channel 

Additionally, companies can leverage Twitch and other streaming platforms as feedback channels, treating streamers' reviews and chat interactions as valuable sources of user insights. Compared to the traditional feedback channels, like players’ comments, ratings, and DAU, this approach allows for more direct and real-time feedback and helps identify areas for enhancement. By using these feedbacks to inform game updates and development strategies, companies can create a more engaging and satisfying experience for players, ultimately fostering a loyal and active player base.

## Limitations

During our project, we identified several limitations that we believe could be improved upon with further opportunities. One significant limitation is the need for 'true' experiments to establish causal inference. Our analysis would benefit from more behavioral and demographic data on both viewers and players. This additional data would provide deeper insights into how different factors influence engagement and viewership. Understanding the nuances of player behavior and preferences is crucial for refining marketing strategies and enhancing player experience.

Another area for improvement is the development of more complex and accurate models to quantify the relationship between viewers and players. While our current models provide valuable insights, more sophisticated modeling techniques could yield even more precise predictions and identify key factors influencing this relationship. Advanced models would allow us to better understand trends, predict future behavior, and make more informed decisions about marketing and game development.

Additionally, we recognize the need for more in-depth game information, including updates and features, to understand the impact of these elements on viewership. Analyzing the specific features and updates that drive player engagement and viewership can provide actionable insights for game developers. Furthermore, a detailed study of between-game discrepancies would help us understand why Twitch has a more significant causal influence on some games compared to others. Identifying these differences can help tailor marketing strategies to the unique dynamics of each game.

In conclusion, addressing these limitations would significantly enhance the robustness of our findings. By conducting true experiments for causal inference, gathering more behavioral and demographic data, developing more complex models, and conducting in-depth studies on game-specific factors and between-game discrepancies, we can gain a more comprehensive understanding of the relationship between Twitch viewership and player engagement. These improvements would allow us to provide more actionable recommendations for game developers and marketers, ultimately leading to more effective strategies and better gaming experiences.

## Reference

Ahmad, W., Shadaydeh, M., & Denzler, J. (2022). Causal discovery using model invariance through knockoff interventions. In Proceedings of the ICML 2022 Workshop on Spurious Correlations, Invariance, and Stability, Baltimore, Maryland, USA. arXiv:2207.04055. https://doi.org/10.48550/arXiv.2207.04055

Runge, J. (n.d.). Tigramite: Python package for causal inference in time series. GitHub. Retrieved July 12, 2024, from https://github.com/jakobrunge/tigramite

Via, D. (2024, June 28). Siege reaches its highest player peak on Steam in almost three years. SiegeGG. https://siege.gg/news/siege-reaches-its-highest-player-peak-on-steam-in-almost-three-years 

## Contributors
Chun Yat Cheung\
Ethan Liu\
Sherry Shi

## License
This project is licensed under the MIT License - see the LICENSE file for details.

