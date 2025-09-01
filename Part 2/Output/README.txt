Assumptions:

- Most recent data on the dataset is from 2023, we assume that this data is still relevant.
- We assume that "Monthly Debt Service Amount (Amortizing)" takes into account combined debt of all financial sources, in case two or more loans are in place.
- We assume than migration and emigration from the states is low enough along the years, that it does not affects the calculated "Replacement Rate"
- Reasoning behind each measure is detailed on the EDA. I have included additional metrics to consider not only profitability, but also risk measures and occupancy tendency indicators to provide a more holistic point of view.
- Data quality issues are not going to be directly on the Excel, to simulate a real live situation where we should not modify data directly on a transactional system nor data warehouse/lake



Data Handling And Processing:

The way to handle data is to first understand what business question or pain we are trying to solve. With this starting point we select what data sources may be relevant for our problem and start looking into possible solutions. In this case problem definition, expected solution and data sources were given. 

Once we have this definitions, we take a look into the data sources and give a general description on how these data sources look like, what they represent, their refresh status and reliability (in other cases we would need to check their availability). Then, we select our data sources and mitigate/identify risks, we define which metrics and dimensions we are going to use to solve the business requirements; users feedback is recommended in this stage. We then proceed to select which columns from the dataset are necessary to create measures and dimensions defined previously, as well as determine if their is insufficient data to create one or more of them.

With the selected columns from the datasets, it is necessary to evaluate data quality to assure data is reliable and ready to use. In this case I used python to evaluate selected column and obtain descriptive statistics as well as unique and null values. Once identified data issues, I used Power query to solve them as follow:

- Identify null values, if estimating/research the null value is possible, do it and fix the value by creating a calculated column (or reach out to the data owner to fix it on the source), but if this is not possible consider deleting the record if it largely affects this record analysis. In this case I discovered 6 null values on "cut-off date balanceunit", and solve it by creating a calculated column with the following Power Query formula: 
	if [#"cut-off date balanceunit"] = null then if  [total units] > 0  then [#"cut-off date loan amount"]/[total units] else 0 else [#"cut-off date balanceunit"]

- Identify if null values make sense, for example columns like Renovated Year, 2nd most recent noi and 3rd most recent noi have null values, but since this columns are neither critical nor are required for all properties, we assume null values are right, and therefore we do not exclude them.

- Identify if null values are critical for our analysis. For example a null value in property name, or monthly rent per unit, and others would limit our ability to show relevant information for this record, and if this was the case and we can not fix it we should delete it. However, not all columns are critical for the analysis, in this case latitude and longitude are useful but not critical, therefore, having 20 null values (each) on this columns might affect one visualization, but would not affect the entire dashboard; that is why we did not remove this records.

- Identify data errors, if the error can be fixed using a formula, then create a calculated column to do it. If the error can not be solve delete the record. For example, we had on the "Year built" column a value equal to "Various", since the data was entirely numerical except for this record, I decided to eliminate this record because it causes Power BI to misinterpreted year column sorting as well as the creation of a new dimension.

After dealing with data issues, I calculate the metrics previously defined and compare take a small sample of data to verify that the formula was right.



Data Automatization:

To automatize the report, we can have different approaches, these two should fit multiple scenarios and business requirements:

1) Internal data - partial automatization:
	
	This files can be allocated on a shared folder on OneDrive (with limited access) were users can add, modify and replace records, as well as add new columns and flags that can 	help analyze data faster. Even though this is a quicker solution, it also presents more data quality risks.

2) Internal data - fully automatized:
	
	Data should be allocated on a data warehouse connected to the internal source system. A trigger or a job should be created so that any time data changes (or certain amount of 	time has passed), the data warehouse would store this information in two different tables one that contains most recent data (connected to the dashboard) and a second one that 	retains historic information with snapshots (ideal for time series analysis)

3) External data
	
	If data is not from any internal system, you can implement previous proposals but first you would need to obtain external data by searching/downloading/allocating manually the 	data or by creating a web scraping script / bot to take this information into either One Drive or an FTP for the data warehouse to consume it.



Communicate Findings:

Since the dashboard was created to be interactive and usser friendly the three main ways to comunicate findings would be:

1) Upload the report to Power Service so that they can have real time information and also find their own insights if wanted
2) Create a small dashboard with executives more important KPIs and insights to be automatically send to the stakeholders. This dashboard (unlike the others) would not be interactive, but purerly informative.
3) Create a meeting with a presentation that use the most relevant insights of the report.



Clarifications:

SQL was not needed to create this report because Power Query semantic layer performs all relationships needed. I have included an enriched data set (Column selection is performed in Power Query) because it is one of the requirements. This file was created using the following python script (did not use SQL to avoid defining tables): 

import pandas as pd
demographics_us = pd.read_csv(r"C:\Users\dauribe\OneDrive - ENDAVA\Desktop\WD\Entry Project\Part 2\Input\demographics_US.csv")
properties = pd.read_csv(r"C:\Users\dauribe\OneDrive - ENDAVA\Desktop\WD\Entry Project\Part 2\Input\property_dataset_sample.csv")
states = pd.read_csv(r"C:\Users\dauribe\OneDrive - ENDAVA\Desktop\WD\Entry Project\Part 2\Input\states_codes_mapping.csv")
properties_states = properties.merge(states, left_on="property state", right_on = "adm_1_code_letters" ,how="left")
enriched_dataset = properties_states.merge(demographics_us, left_on="adm_1_code", right_on="state_code", how="left")
enriched_dataset.to_excel(r"C:\Users\dauribe\OneDrive - ENDAVA\Desktop\WD\Entry Project\Part 2\Output\enriched_dataset.xlsx", index=False)




	


