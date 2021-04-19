# Wish.com-Online-Sales-Analysis

**I. Company Overview**

*Background*

Wish is a San Francisco-based e-commerce platform that supports over 100 million monthly active users worldwide and hosts over 300 million merchant products. Founded in 2011, Wish’s mission “is to provide everyone access to the most affordable and convenient shopping experience on the planet.” Wish was the most downloaded app in 2018 and, according to Forbes, ranks as the third-largest e-commerce marketplace by sales (Olson).

*Key Objectives*

Wish’s business strategy involves heavily marketing across digital and physical mediums to drive exposure, attract users to the site, and ultimately increase the number of items sold. The items sold on the Wish site are incredibly inexpensive and oftentimes eclectic; thus, the business model revolves around selling high quantities of goods and catering to frequent, repeat business by offering a diverse array of products through a large number of merchants.

**II. Data Description** 

*Our Key Metric*

As mentioned above, Wish’s core business is centered around high inventory turnover because the prices of the goods on the site are so low (for our dataset, the average price of the 1,574 clothing articles is only $8.33). As a result, our team determined that maximizing the number of units sold is the most critical metric to assess for Wish to identify product performance on the site (the more units sold, the better Wish does - Wish collects a 15% revenue share for each product sold).

Because each line item in our dataset represents an individual item listed on Wish, our team leveraged the attribute “Unit_Sold” to use as our metric, which, according to the data dictionary, captures the number of each item sold during July 2020.
Variable Selection

To examine the factors that influence the number of units sold, we used causal analysis. First, we established a few hypotheses (using attributes of the dataset) to test based on our domain knowledge of the e-commerce landscape and initial exploration of the data:
- Price: We believe that lowering prices may result in higher unit sales.

-	Merchant Rating: Higher merchant ratings (1-5) may increase the number of units sold.

-	Rating: The higher the mean product rating (1-5), the more the products may sell.

-	Uses_ad_boosts: We would assume that merchants who purchase boosted ads would report increased item sales. However, the barplot of average units sold among the products with boosted ads seems to give us the opposite story. We’d like to dig in and see if we can get something out of this feature. 

-	Merchant_has_profile_picture: Although the number of merchants with profile pictures is relatively small, we noticed that the average units sold of these merchants are higher than those without a profile picture.

**IV. Analysis**

*Causal Analysis*

After identifying the variables to conduct causal analysis with, we checked the distributions of all independent variables (Figure 3, Appendix). We performed a min-max normalization for three numeric variables (price, merchant rating, and rating) to scale them to a fixed range.
We then examined the distribution, mean, and variance of our target - units_sold, and found there was overdispersion. We considered mainly using negative binomial/linear regression to treat the data instead of Poisson.
We built a linear regression and a negative binomial regression with the five selected variables: price, merchant rating, product rating, ad boosts, and merchant profile pictures (Figure 4 & 5, Appendix).  

The formulas were:

**Units_Sold=𝛽0 + 𝛽1 * price + 𝛽2 * rating + 𝛽3 * merchant_rating + 𝛽4 * uses_ad_boosts + 𝛽5 * merchant_has_profile_pictures +𝜀**

**ln(Units_Sold)=𝛽0 + 𝛽1 * price + 𝛽2 * rating + 𝛽3 * merchant_rating + 𝛽4 * uses_ad_boosts + 𝛽5 * merchant_has_profile_pictures +𝜀**

The result of the likelihood ratio test suggests that the negative binomial regression model performs better than the linear regression model (Figure 4, Appendix). Furthermore, the negative binomial regression outputs were in line with our initial hypotheses, except for the ad boost variable. Because the outputs conflicted with our initial hypothesis (boosted ads were NOT significant), we performed Propensity Score Matching (PSM) on the ad boost to verify that boosted ads truly have no impact on the number of units sold. 
We included the price and rating for the ad boost variable to match the records (after checking if the other independent variables are significantly different), showing that we have 152 records for each group (Figure 6, Appendix). We ran a single negative binomial regression of this match (Figure 7, Appendix); However, the output revealed that the impact of ad boosting on units sold is statistically insignificant. We created a density plot of units sold to assess why, which portrayed a similar trend for each group from our dataset (Figure 8, Appendix). Thus, we cannot conclude that the ad has a significant impact on units sold. 
We followed the same PSM process to evaluate the merchant profile picture attribute to confirm our model’s result.  While there were only five matching records, we determined that merchant pictures positively impact units sold (Figures 9 &10, Appendix). 
Finally, while it makes sense that customers would purchase products with higher ratings from merchants with higher ratings, we wanted to test if the product and merchant ratings have the same impact on units sold. Hence, we ran a linear hypothesis test. The result (Figure 11, Appendix) shows the p-value is low enough to reject the null hypothesis (that product rating and merchant ratings have the same effect on units sold), and, because the merchant rating has a higher coefficient, we can conclude that the merchant rating has a greater impact on units sold. 

**V. Suggestions**

Based on our analysis, we formed two suggestions to improve the number of units sold. Firstly,  according to Forbes, one of the primary complaints customers have is poor merchant response time, resulting in customers giving merchants low ratings. From our analysis, we know that the merchant rating directly impacts unit sales (and is even more important than the actual product rating). Like other e-commerce companies, Wish could tag merchant profiles with a banner designating the merchant’s average response time and shipping time (much like eBay). Secondly, we would encourage every merchant to provide a profile picture by sending out a notification to vendors to upload a photo when they sign up for an account. Based on our PSM analysis, we know that merchant profile pictures have a direct impact on units sold. In fact, merchants sell 6x more units when they have a profile picture (Figure 2, Appendix)!

**VI. Experiment Design**
We propose Wish conduct a before-after quasi-experiment to test if adding merchant profile pictures can bring more sales. When we conducted PSM on the merchant's profile variable, only five groups matched, which may conflict with our causal analysis. Also, this experiment will provide plenty of candidates for both groups. The experiment will be conducted as follows:
1.	Randomly assign 50,000 merchants that have been on Wish for more than 6 months and never used profile pictures to 2 groups (25000 for control and 25000 for test) and extract their monthly units sold for the past 3 months. 
2.	Calculate the difference in the sum of the monthly units sold between the test and control groups (also known as the inherent difference).
3.	Request the merchants in the test group to upload profile pictures. 
4.	Record the monthly units sold for the following 3 months for two groups and calculate the difference of the sum of monthly units sold between the two groups again.
5.	Compare the difference in Step(2) and Step(4).
6.	With 6-month units sold numbers (3 months before treatment, 3 months after treatment) for each group in hand, we can perform a Difference in Differences (DID) analysis to assess how units sold changes after merchants in the test group post profile pictures compared to the control group (Figure 12, Appendix).
7.	The DID analysis: Units_Sold = 𝛽0 + 𝛽1*𝑇𝑟𝑒𝑎𝑡𝑒𝑑 + 𝛽2*𝐴𝑓𝑡𝑒𝑟 + 𝛽3*𝐴𝑓𝑡𝑒𝑟*𝑇𝑟𝑒𝑎𝑡𝑒𝑑 + 𝜀
8.	Outcome evaluation: We expect to see a significant effect of After*Treated, and all the coefficients for this variable should be positive, showing that adding a merchant profile picture can genuinely boost the units sold.

