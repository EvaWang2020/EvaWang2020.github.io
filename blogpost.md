---
layout: default
---

![](https://static.wixstatic.com/media/456b92_1d30506d006e41ea81bdea3f909c0bf6~mv2.jpg/v1/fill/w_360,h_408,al_c,q_90/456b92_1d30506d006e41ea81bdea3f909c0bf6~mv2.webp)

## **Overview of Project**

The purpose of this project is to analyze the data quality of the 2019 Vancouver Business License. The dataset was downloaded from the official website City of Vancouver Open Data Portal ([https://opendata.vancouver.ca/explore/dataset/business-licences/information/?disjunctive.status&disjunctive.businesssubtype&refine.folderyear=19&refine.issueddate=2019](https://opendata.vancouver.ca/explore/dataset/business-licences/information/?disjunctive.status&disjunctive.businesssubtype&refine.folderyear=19&refine.issueddate=2019)). The tool I used is DQ Analyzer, which is specialized in data qualify analysis and data profiling. The first step I did was to profile the dataset to get a baseline analysis. Through data profiling, I found below issues:

![](https://static.wixstatic.com/media/456b92_8a34a963299142cf84892df331d989a7~mv2.png/v1/fill/w_740,h_127,al_c,q_95/456b92_8a34a963299142cf84892df331d989a7~mv2.webp)

Afterward, I invited a subject matter expert (SME) to rank the priority of the identified column issues and provide suggestions. My SME thought the issues with BusinessName, Status, and Street are significant. My SME suggested investigating whether the missing business name and missing street issues are under control. He also suggested investigating why there is an issued date when the status is “Pending”.

My SME is a financial risk control director. His main job is to manage the risks of bank investments such as stock and mortgage products. Data quality is crucial for his work as he is responsible for risk analysis and reporting. Recently, he found that his organization mistakenly calculated an extra profit of around 500 million due to a data quality issue.

Based on SME’s suggestions, I did statistical process control analysis on the missing business name and missing street name issue, separately. I found that the issues are under control. I was not able to find out the exact reason why “Pending” status and issued data co-exist. This becomes a mystery.

My SME suggested adding a non-unique constraint on the primary key. He suggested implementing a constraint to reject the null value for BusinessName column if the status is “Issued”. He also suggested implementing non-null value constrain on PostalCode and Street columns on the user interface to reject empty input. He suggested adding validation on Status, IssuedData, and ExpiredDate column.

## **Initial Assessment**

-   **LicenceRSN: Non-Unique Primary Key**
    

The LicenceRSN column is described in the documentation as the primary key. Primary keys must contain unique values to maintain data integrity in a rational database. In the basic analysis, I can see that around 0.01% of primary key values (9 records) are not unique.

![](https://static.wixstatic.com/media/456b92_03bc4c2cedb844f7b5d617cb1e6695a9~mv2.png/v1/fill/w_360,h_184,al_c,lg_1,q_95/456b92_03bc4c2cedb844f7b5d617cb1e6695a9~mv2.webp)

By checking the Frequency Analysis, I found that below 9 LicenceRSN records appear twice.

![](https://static.wixstatic.com/media/456b92_8f6b88eb1da44f69928496d29e359cc3~mv2.png/v1/fill/w_346,h_399,al_c,lg_1,q_95/456b92_8f6b88eb1da44f69928496d29e359cc3~mv2.webp)

After drilling through each LicenceRSN value, I found that the pair of records sharing the same LicenceRSN values are all very similar to each other.

For example, below 2 rows both have LicenceRSN 3235769. The only difference between the two records is the value of BusinessTradeName column: one is Cho Construction, while another is Cho’s Construction. Both records have a “Pending” status.

![](https://static.wixstatic.com/media/456b92_798e743d6151438aa0563c352064df8b~mv2.png/v1/fill/w_740,h_60,al_c,lg_1,q_95/456b92_798e743d6151438aa0563c352064df8b~mv2.webp)

LicenceRSN 3235770’s situation is the same as LicenceRSN 3235769. And, both records have “Pending” status.

![](https://static.wixstatic.com/media/456b92_25b04a4938d6442db777140a5167db2d~mv2.png/v1/fill/w_740,h_59,al_c,lg_1,q_95/456b92_25b04a4938d6442db777140a5167db2d~mv2.webp)

LicenceRSN 3249850’s two records are the same except the value in the Unit column. Both records have canceled status.

![](https://static.wixstatic.com/media/456b92_a14d9f95895c4000ac93beba884699c7~mv2.png/v1/fill/w_360,h_65,al_c,lg_1,q_95/456b92_a14d9f95895c4000ac93beba884699c7~mv2.webp)

LicenceRSN 3254886’s two records are the same except the values in Street and PostalCode columns. Both records have canceled status.

![](https://static.wixstatic.com/media/456b92_dbacee2330954ca2b2d4dbb95521c4b0~mv2.png/v1/fill/w_360,h_42,al_c,q_95/456b92_dbacee2330954ca2b2d4dbb95521c4b0~mv2.webp)

LicenceRSN 3336640’s two records are the same except the values in the BusinessTradeName column. Both records have “Issued” status.

![](https://static.wixstatic.com/media/456b92_ac75f4560e9b4d38a1da07f3759ba3d6~mv2.png/v1/fill/w_740,h_48,al_c,lg_1,q_95/456b92_ac75f4560e9b4d38a1da07f3759ba3d6~mv2.webp)

From the above, we can see that the status of the record is not the reason for having a non-unique LicenceRSN number.

-   **BusinessName: Missing Business Name**
    

From the basic analyses of BusinessName column, I found that 9% of records, which means 6319 rows, have no business name.

![](https://static.wixstatic.com/media/456b92_4d7573071bfa4f18825f7ca4bc05158f~mv2.png/v1/fill/w_360,h_199,al_c,q_95/456b92_4d7573071bfa4f18825f7ca4bc05158f~mv2.webp)

To verify whether business status is related to the issue, I compared the null business name ratio among different statuses. From the below result, we can see that no matter whether the business is having an Issued status or not, there are records with no business name.

![](https://static.wixstatic.com/media/456b92_bb3b4a64206848d7928466350ea45c8a~mv2.png/v1/fill/w_360,h_98,al_c,q_95/456b92_bb3b4a64206848d7928466350ea45c8a~mv2.webp)

If a business had been issued a license before, but there is no business name record, we might want to know whether the business name was deleted after the license was issued. I, therefore, checked the LicenceRivisionNumber column, which should indicate the times of changes, to verify this assumption. For example, LicenceRSN 3293281 has a status of “Issued” and has no BusinessName. LicenseRevisionNumber indicates that the license had never changed. This might suggest that business could be issued a license even if there is no business name?

![](https://static.wixstatic.com/media/456b92_cabd388ea7b843aa97860d9203649a59~mv2.png/v1/fill/w_360,h_36,al_c,q_95/456b92_cabd388ea7b843aa97860d9203649a59~mv2.webp)

-   **Status: Issued Date and “Pending” Status Co-Existence**
    

The frequency analysis of the Status column indicates that 10% of recording has a “Pending” status.

![](https://static.wixstatic.com/media/456b92_5cd23a076d7b42aa8d1be020a0e438df~mv2.png/v1/fill/w_360,h_211,al_c,lg_1,q_95/456b92_5cd23a076d7b42aa8d1be020a0e438df~mv2.webp)

To check the detail of records with the “Pending” status, I separated them from the total 2019 data set and created a sub data set. After profiling the sub data set, I found that that 2.7% (see below Mask Analysis) of the records with “Pending” status has an issued date.

![](https://static.wixstatic.com/media/456b92_b51fb3537a0b42dc82fa0e8650dabff8~mv2.png/v1/fill/w_360,h_154,al_c,lg_1,q_95/456b92_b51fb3537a0b42dc82fa0e8650dabff8~mv2.webp)

For example, LicenceRSN 3228782 has a “Pending” status, but its issued data is “2019-01-16”. One thing we need to think about is: if a business registration is still pending, should it have an issued data? Or is the issued data from last year and the pending status suggests the business is asking to renew the license?

![](https://static.wixstatic.com/media/456b92_d63e9415228844acb78b192d0f766761~mv2.png/v1/fill/w_360,h_43,al_c,q_95/456b92_d63e9415228844acb78b192d0f766761~mv2.webp)

To verify this assumption, I drilled through the IssuedDate column and found that among all the records (with a “Pending” status and an issued date) have an expired date of Dec 31, 2019. This might explain why “Pending” status and issued data co-exist: the business is renewing its license. But, the column arrangement of the data set is confusing.

![](https://static.wixstatic.com/media/456b92_79332146ccf54177bc58400594efb198~mv2.png/v1/fill/w_360,h_209,al_c,q_95/456b92_79332146ccf54177bc58400594efb198~mv2.webp)

-   **PostalCode: Various Patterns**
    

From Mask analysis, I found that there are many different types of postal code patterns.

![](https://static.wixstatic.com/media/456b92_a15aaa2035aa4b02abcd02e8b46e7553~mv2.png/v1/fill/w_360,h_511,al_c,lg_1,q_95/456b92_a15aaa2035aa4b02abcd02e8b46e7553~mv2.webp)

As the license is for the Vancouver area, the major pattern should be “LDL DLD”. To verify this assumption, I went to check the Masks of PostalCode column and found that only 52% has the pattern of “LDL DLD”.

![](https://static.wixstatic.com/media/456b92_38807ec313a54848897c4588730bfe09~mv2.png/v1/fill/w_360,h_228,al_c,lg_1,q_95/456b92_38807ec313a54848897c4588730bfe09~mv2.webp)

To further investigating where these postal codes below to geographically, I went to check the Frequency of the Country column. I found that around 92% of the rows have the value of “CA” in this column. This suggests that 92% of the registration records are Canadian businesses. Here “CA” represents Canada. This percentage conflicts the 52% of recording having an “LDL DLD” pattern.

![](https://static.wixstatic.com/media/456b92_c6b15df27c974c2c9171f36bdc7e3961~mv2.png/v1/fill/w_360,h_154,al_c,lg_1,q_95/456b92_c6b15df27c974c2c9171f36bdc7e3961~mv2.webp)

-   **Street: Empty Value**
    

In the Street column, I found that around 46% of records have no value. We might think that street input is not mandatory before the license gets issued.

![](https://static.wixstatic.com/media/456b92_3c1f642f5e70440e9fa24e6c183c70d4~mv2.png/v1/fill/w_360,h_202,al_c,q_95/456b92_3c1f642f5e70440e9fa24e6c183c70d4~mv2.webp)

To verify this, I compared the null street name ratio among different statuses. From the below result, we can see that no matter whether the business is having an Issued status or not, there are records with no street name.

![](https://static.wixstatic.com/media/456b92_e42f193a5a8549ee8bf628f1aa1ac40c~mv2.png/v1/fill/w_360,h_107,al_c,q_95/456b92_e42f193a5a8549ee8bf628f1aa1ac40c~mv2.webp)

Further analyzing the street column, I found below the top 5 business types that have no street name. Apartment House, for example, has 2302 records, but no street name. I was wondering whether this business needs a street name.

![](https://static.wixstatic.com/media/456b92_2d20cd12bcbd41c3b4a8424df2963372~mv2.png/v1/fill/w_360,h_95,al_c,q_95/456b92_2d20cd12bcbd41c3b4a8424df2963372~mv2.webp)

## **SME Review of Initial Assessment**

-   **LicenceRSN: Non-Unique Primary Key**
    

My SME thinks that the non-unique value issue in LicenceRSN column is not very significant as there is only 0.01% of records having non-unique values.

-   **BusinessName: Missing Business Name**
    

My SME thinks that missing a business name is a significant issue. Though the percentage of missing business name, which is 9%, is not surprisingly high, this is still a big issue. “This is not an era of pen and paper. The dataset should not have such imperfection”, he said. He believed that recent year business registration must have a business name. He suggested investigating the potential reason for missing the business name. “Is it because the license is expired, the LicenceRevisionNumber column did not record changes properly?”, he asked?

-   **Status: Issued Date and “Pending” Status Co-Existence**
    

My SME thinks that this issue is significant. Although there are 2.7% of records having both a “Pending” status and an issued date, the issue is significant as it is very confusing. “The business might be renewing their license, which caused the pending status”, he said. He suggested checking the LicenceRevisionNumber column to see whether there was an update.

-   **PostalCode: Various Patterns**
    

My SME thinks that the significance of this issue is low. Though there are many types of patterns, most patterns have a low frequency. For example, the pattern “LDLDLD” only appears in 0.51% of records. Other Patterns such as “LDL DLD” and “LDL DLL” appear less than 0.1% of times. Besides, the pattern “LDLDLD” and “LDL DLD” can be easily converted into the correct pattern format “LDL DLD”.

-   **Street: Empty Value**
    

My SME thinks that this issue is significant. He said, “Around half of the registration records do not have a street name, this is an issue. If a business does not have a street name, it might be very suspicious. Is it a legal business?”. He suggested investigating whether this situation is under control.

## **Further Research for the SME**

-   **BusinessName: Missing Business Name**
    

To further research why there are records with no business name but still have an issued date, I went to investigate the Status column. I found that all those records have an expired date before 2020. Could this suggest that the business name got deleted once the licenses get expired?

![](https://static.wixstatic.com/media/456b92_06378ecc13014bc49d3236b7c41aeb4a~mv2.png/v1/fill/w_360,h_320,al_c,q_95/456b92_06378ecc13014bc49d3236b7c41aeb4a~mv2.webp)

To further investigate whether no business name issue is under control, I did a statistical process control (SPC) analysis to analyze the ratio of records with no business name. The data set chosen are those records with Issued status. The time chosen is from January to Dec 2019.

![](https://static.wixstatic.com/media/456b92_35f2e7634a464233a6e8f71c6114ee1a~mv2.png/v1/fill/w_360,h_186,al_c,q_95/456b92_35f2e7634a464233a6e8f71c6114ee1a~mv2.webp)

From the above SPC chart, we can see that ratios of missing business names are always between the upper control line (UCL) and LCL (lower control line). Therefore, the situation is under control

-   **Status: Issued Date and “Pending” Status Co-Existence**
    

To verify whether the “Pending” status was caused by license renew, I drilled through the sub data set ( the one with only “Pending” status) again, and checked the LicenceRevisionNumber column, I found that 99% of the records show zero types of change, according to LicenceRevisionNumber column. By now, the reason why Issued Date and “Pending” Status co-exist becomes a puzzle.

![](https://static.wixstatic.com/media/456b92_c5c0cd34b83c41a8a10cbfd6aae44edd~mv2.png/v1/fill/w_360,h_168,al_c,q_95/456b92_c5c0cd34b83c41a8a10cbfd6aae44edd~mv2.webp)

-   **Street: Empty Value**
    

As per the SME’s suggestion, I did an SPC analysis of the ratio of records with no street name. The data set chosen are those records with Issued status. The period chosen is from January to Dec 2019.

![](https://static.wixstatic.com/media/456b92_89c7563f1d1546a0aceae77656243e54~mv2.png/v1/fill/w_360,h_185,al_c,q_95/456b92_89c7563f1d1546a0aceae77656243e54~mv2.webp)

From the above SPC chart, we can see that ratios of missing street names are always between the upper control line (UCL) and LCL (lower control line). Therefore, the situation is under control.

## **SME Suggests Some DQ Rules**

My SME suggested putting a unique constraint on LicenceRSN column to avoid the non-unique primary key issue.

He suggested adding a conditional constraint on the BusinessName column. If the status of a record is “Issued”, there must have a business name.

He suggested to set up validation rules on Status, IssuedDate, and ExpiredDate columns. For example, if a status is not “Issued”, the system should not generate an issued date. Through this change, it could avoid confusion when analyzing the data. Another thing that the system designers can do is to add another column called “Previous Status” to keep a record of previous status and how users understand the changes.

He suggested checking LicenceRevisionNumber column to see why it failed to record the changes.

He suggested adding a non-null constrain on Street column. If the users want to save the record, they must input a street name.

Finally, my SME suggested to set up a validation rule on the PostalCode column. For example, if a user inputs a postal code that is not in “LDL DLD” pattern and Country represents Canada, the web page should pop out a warning, which might stop the user the save the input.