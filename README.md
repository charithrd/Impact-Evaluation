![Header Imagery](Assests/ImpactEvaluation.jfif)

> ## Table of Contents

[Research Topic](#Research-Topic)

[1. Executive Summary](#Chapter-1-Executive-Summary)

[2. Project Overview](#Chapter-2-Project-Overview)

[2.1 Problem Statement](#21-Problem-Statement)

[2.2 Solution](#22-Solution)

[2.3 Impact Assessment](#23-Impact-Assessment)

[3. ETL Pipeline](#Chapter-3-ETL-Pipeline-and-Automation)

[4. Data Structure](#Chapter-4-Data-Structure)

[5. Building the Data Model](#Chapter-5-Building-the-Data-Model)

[5.1 Data Selection and Preparation](#51-Data-Selection-and-Preparation)

[5.2 Data Manipulation](#52-Data-Manipulation)

[5.3 Data Joins](#53-Data-Joins)

[5.4 Vessel ETAs’ Calculation Logic](#54-Vessel-ETAs’-Calculation-Logic)

[5.5 Merging Final Data View](#55-Merging-Final-Data-View)

[6. Power BI Reporting](#Chapter-6-Power-BI-Reporting)

[7. Project Reflection](#Chapter-7-Project-Reflection)

[Bibliography](#Bibliography)


> ## Research Topic

How does implementing an automated business solution using a cloud platform impact inventory management efficiency and data accuracy in the global retail sector, particularly in response to international shipping challenges? 



> ## Chapter 1 Executive Summary

Owing to the volatility inherent in international shipping routes, it is imperative for global retailers to accurately monitor their inventory in transit to ensure they can meet customer demand.

KPMG (2023) research states that “global and local retailers may need to review their inventory distribution network”. 

Furthermore, the latest statistics illustrate that “53% of ecommerce retailers find shipping and logistics processes challenging” (Meteor Space, 2022).

This explores Impact evaluation on an automated business solution deployed using Azure Databricks and Power BI.

In a nutshell, the solution calculates stock due weekly with a view of vessel arrival movements. The precursor of this process was manual. With the automation, we have saved 312 business hours per annum ensuring data accuracy and reporting scalability in line with business growth.

Adhering to the Data Protection Act 2018 (Legislation.gov.uk, 2018), business data has not been published and has been anonymized.


> ## Chapter 2 Project Overview
To stay competitive many retailers have moved ‘their manufacturing to low-cost countries’ (Arrigo, 2020). Landing these products on the shop floor on time involves major planning for the logistics team.

Adding more complexity to this context are shipping disruptions in international shipping routes and ports. Hence actively monitoring stock arrivals and adjusting warehouse planning are vital to improve sales and customer satisfaction.

This process is called vessel en-route analysis. Historically this process was completed using Excel spreadsheets retrieving the data from a legacy system and comparing it against another report produced by freight forwarders to identify any late vessel arrivals due weekly.

Recently business has moved from on-premise servers to Azure cloud-based platform. Capitalizing on this infrastructure change and supporting business growth sets the right grounds for automating this workflow and introducing data visualization using Power BI.

### 2.1 Problem Statement
Due to the business growth, there has been substantial use of underlying data usage to complete the process. Multiple users need to join to complete the process. This has resulted in shared Excel workbooks crashing and being unable to save their work. Furthermore, manual user intervention was prone to errors. Hence scalability and reliability were the key concerns.

### 2.2 Solution
ETL framework has been developed using Azure Databricks cloud platform, adopting data security measures. Also created workflows within the Databricks platform and fully automated the ETL pipeline ensuring data accuracy. Finally, the refined dataset is loaded into Power BI, enabling users to access deep insights.

### 2.3 Impact Assessment

#### 2.3.1 Positives
•	Manual user intervention in producing the report has been eliminated. This is estimated to be three hours weekly by two colleagues. Total time saved approximately 156 hours per annum.

•	Shadow deployment strategy has been utilized where the model is compared to “human judgment before putting it into production” (BPP School of Technology, 2024). No deviations from manual records illustrated the model's accuracy and reliability.

•	Ability to scale up reporting times from weekly to daily if required by business.

#### 2.3.2 Negatives
•	Any future improvements to the solution require “dedicated engineers to handle the development” (Palmer, 2024) due to the technical aspects.

#### 2.3.3 Conclusion
In conclusion, implementing this ETL Framework and reporting has yielded significant benefits by eliminating manual user intervention and enhancing reporting accuracy. Thus, maintenance of the data model requires a team member with technical experience.


> ## Chapter 3 ETL Pipeline and Automation

![ETL](Assests/ETL.png)

Above, high level ETL Pipeline is summarized as follows.

### Source: 
 - Structured Azure SQL databases are the primary source.
   
### Data Governance: 
 - Azure Active Directory (AD) is the security layer before extraction.
 - Considering data privacy only the necessary data is processed, and no personal data has been utilized for this project.
   
### ETL Flow:

Azure Databricks
•	Using SQL Notebook key data is extracted from the source.
•	Data is then transformed using deduplication, cleaning, and aggregation techniques enabling further analysis.
•	Azure Notebook is attached to a workflow automating the job run.
•	Notebook output is loaded into Databricks Unity catalogue.
•	Finally, the unity catalogue output is connected to Power BI for visualization.


### Workflow Automation:

 - Databricks notebook is attached to a workflow within a cluster.
 - This job is scheduled to run as per the business's desired time frames.
 - Any workflow failures will get notified to the Admin users via work email.

![Databricks Workflow](Assests/DBWorkflow.PNG)

![Databricks Workflow](Assests/DBAutomation.PNG)


> ## Chapter 4 Data Structure

Below are the metadata characteristics of the final data load to the Power BI model.

![ETL](Assests/Metadata.png)



> ## Chapter 5 Building the Data Model


### 5.1 Data Selection and Preparation

 - The ‘Create or Replace Table’ statement creates a new table called ‘Vessel_En_Route’.
 - Essential fields are added to the select statement.


### 5.2 Data Manipulation

 - Below manipulation steps “includes variable-by-variable transformation, as well as aggregation” (Wickham, 2014).
 - Converting Datetime fields such as v. vy_arrive_date into a date field.
 - A case statement converts warehouse codes in the ‘Planned_dest_code’ field into Warehouse names. 
 - Using an array join statement ContractNo and DeliveryNo fields are concatenated into a single column. This step helps users to search warehouse deliveries easily.
 - Retail Sales Value (RSV) – By multiplying the balance left with the retail sell price we determine the row (option) level retail value.
 - Then Sum () function calculates the total sum of the product across all rows in the dataset. Finally, the data type is cast to an Integer (INT).


```sql
create or replace table Vessel_En_Route

SELECT v.vy_journey_no as Voyage,
       v.vy_vessel_name as Vessel_name,
       v.vy_arrive_loc as Port_Arrival_Code,
       date( v.vy_arrive_date) AS ArrivedDate,
       date(comb.whsdeldate) AS WhseDelDate,
       lu.country_code,
       s.fm_status,
       s.planned_dest_code,
       Case   When trim(s.planned_dest_code) = 'P' then 'Dxxxxxxx'
		      When trim(s.planned_dest_code) = 'D' then 'Dxx'
			  When trim(s.planned_dest_code) = 'DF' then 'Dxxxxxxx'
			  When trim(s.planned_dest_code) = 'H' then 'Sxxxxxxx'
			  When trim(s.planned_dest_code) = 'X' then 'Exxxxxxx'
			  When trim(s.planned_dest_code) = 'V' then 'Dxx'
			  When trim(s.planned_dest_code) = 'XD' then 'Exxxxxxx'
			  When trim(s.planned_dest_code) = 'S' then 'Sxxxxxxx'
			  When trim(s.planned_dest_code) = 'M' then 'Txxxxxxx'
			  When trim(s.planned_dest_code) = '56' then 'Txxxxxxx'
			  When trim(s.planned_dest_code) = '97' then 'Txxxxxxx'
			  else 'Other'
			  end as Depot_Name,
       comb.contractno,
       comb.deliveryno,
       array_join(array(comb.contractno , comb.deliveryno),'') As ContractDel,
       c.vy_partner_cnt_no as Container_No,
       SUM(comb.balleft * i.ret_sell_price) AS RSV
```


### 5.3 Data Joins

 - The primary data source of this project satisfies the third normal form (3NF).
 - Chakraborty et al., 2021 claim that “3NF can reduce redundancies and anomalies from a relational database”.
   
 - The query results are constrained by the ‘WHERE’ clause.
 - To adhere to the 'Uniqueness' dimension of data quality, we have employed the 'GROUP BY' clause.
 - According to Kushtagi (2023), “data deduplication can be achieved using various methods, such as using the DISTINCT keyword, GROUP BY” clause.
 - The code below illustrates the necessary multiple table joins to facilitate the analysis and model building.

```sql
FROM xxxx_dss.dp_itemoption as I
        
   INNER JOIN xxxx_dss.contract_combined as comb
        ON ( comb.itemcode = LEFT(i.itemoption_code,6)
        AND comb.optionno = SUBSTRING(i.itemoption_code,7, 2))
        
   INNER JOIN xxxx_dss.dg_country as lu 
        ON ( comb.countryofdesp = lu.country_code)
            
   LEFT OUTER JOIN xxxx_dss.fm_status as s
        ON ( comb.contractno = s.contract_no
        AND comb.deliveryno = CAST(s.delivery_no AS INTEGER))
            
   LEFT OUTER JOIN xxxx_dss.fm_cnt_copts as cc
        ON ( cc.vy_contract_no = comb.contractno
        AND cc.vy_contract_del = comb.deliveryno
        AND cc.vy_contract_option = comb.optionno
                                  )
   INNER JOIN xxxx_dss.fm_containers as c
        ON ( c.vy_journey_seq_no = cc.vy_journey_seq_no
        AND c.vy_container_no = cc.vy_container_no)
            
   LEFT OUTER JOIN xxxx_dss.fm_voyage as v
        ON ( v.vy_journey_seq_no = c.vy_journey_seq_no )
        
WHERE 
       comb.DtlTransportmode = 'S' --- S = SeaFreight
       AND s.fm_status BETWEEN '40' AND '50'
       AND  v.vy_arrive_loc  LIKE  'GB%'
       
GROUP BY 
        v.vy_journey_no,
        v.vy_vessel_name,
        v.vy_arrive_loc,
        date(v.vy_arrive_date),
        date(comb.whsdeldate),
        lu.country_code,
        s.fm_status,
        s.planned_dest_code,
        comb.contractno,
        comb.deliveryno,
        c.vy_partner_cnt_no
       
HAVING  (SUM(comb.balleft * i.ret_sell_price) > 0 )
```

* Using the ‘Vessel En Route’ table created earlier, we have created an additional table called ‘Vessels’.
* Listing distinct records of vessel Names and respective voyage numbers.


```sql
create or replace table Vessels

select distinct
Vessel_name,
Voyage

from Vessel_En_Route

```

### 5.4 Vessel ETAs’ Calculation Logic.

 - Business is required to identify deviations from original vessel arrival dates (ETA) to current vessel arrival dates to plan warehouse resources and minimize detention and demurrage charges. To achieve this objective we have applied the logic below.

 - We have identified a change data capture (CDC) enabled table called ‘MiniILS’, “which records activity on a database when tables and rows have been modified” (Microsoft.com, 2023). Within this table ‘allocated’ column records a date stamp to track changes.

 - We have adopted the Row_number window function to sequentially organize results in rows where the partition clause reset for each distinct vessel. Then within each partition, the rows will be ordered by the ‘Allocated column in ascending order. We have dropped any records where ‘ArrivedAtPortDate’ is null as a data clean-up measure. 
This data is then joined with the ‘Vessels’ table we created earlier by voyage name and respective first record in ascending order for each voyage number. This logic helps us to identify the original vessel arrived at port date recorded in the database.

 - By reversing the order by criteria in the window function to descending order and joining the data to the ‘Vessels’ table, retrieving only the first record when data is organized in descending order, we can identify the current arrived-at-port date. 

 - These distinct records are then added to a newly created table called ‘Vessel_ETAs’.
 - Listed below is the SQL code for the Vessel ETA logic.


```sql

create or replace table Vessel_ETAs

Select
v.Vessel_name,
v.Voyage,
Orig.ArrivedAtPortDate as OrigArrivedAtPortDate,
Curr.ArrivedAtPortDate as CurrArrivedAtPortDate

From Vessels as V

Left join (select Rec_ID, 
                  Allocated, 
                  Vessel, 
                  ArrivedAtPortDate, 
                  row_number() over (partition by Vessel order by Allocated Asc) as AscOrder

From ils.miniils 
Where
Trim(Vessel) in (Select Distinct trim(Voyage) From vessel_en_route)
and ArrivedAtPortDate is not null) as Original

On Trim(Original.Vessel) = Trim(v.Voyage)
and Original.AscOrder = '1'

Left Join (Select Rec_ID, 
		  Allocated, 
		  Vessel, 
 		  ArrivedAtPortDate, 
                  row_number() over (partition by Vessel order by Allocated desc) as DescOrder

From ils.miniils 
Where
Trim(Vessel) in (Select Distinct trim(Voyage) from vessel_en_route)
and ArrivedAtPortDate is not null) as Current

On Trim(Current.Vessel) = Trim(v.Voyage)
and Current.DescOrder = '1'

```

### 5.5 Merging Final Data View

 - A new table is created called ‘imports. Vessel_En_Route_Master_Current_WK’.
 - Final curated data view is created with the code below.
 
```sql
create or replace table imports.Vessel_En_Route_Master_Current_WK

Select 
v.*,
case When trim(Port_Arrival_Code) = 'GBFXT' then 'Felixstowe'
     When trim(Port_Arrival_Code) = 'GBSOU' then 'Southampton'
     When trim(Port_Arrival_Code) = 'GBTIL' then 'Tilbury'
     When trim(Port_Arrival_Code) = 'GBLGP' then 'London Gateway'
     When trim(Port_Arrival_Code) = 'GBTEE' then 'Teesport'
     When trim(Port_Arrival_Code) = 'GBHUL' then 'Hull'
     When trim(Port_Arrival_Code) = 'GBLIV' then 'Liverpool'
     When trim(Port_Arrival_Code) = 'GBIMM' then 'Immingham'
     When trim(Port_Arrival_Code) = 'GBMID' then 'Middlesbrough'
     End as Arrival_Port_Name,
ETA.OrigArrivedAtPortDate as Original_ETA,
ETA.CurrArrivedAtPortDate as Current_ETA,
datediff(CurrArrivedAtPortDate,OrigArrivedAtPortDate) as Days_Delay
Case When datediff(CurrArrivedAtPortDate,OrigArrivedAtPortDate) in (1,2,3) then '1-3 Days'
     When datediff(CurrArrivedAtPortDate,OrigArrivedAtPortDate) in (4,5,6) then '4-6 Days'
     When datediff(CurrArrivedAtPortDate,OrigArrivedAtPortDate) >=7 then '7+ Days'
     When datediff(CurrArrivedAtPortDate,OrigArrivedAtPortDate) < 0 then 'Early'
     Else 'On Time' 
     End as Days_Delay_Banding,
Case When datediff(CurrArrivedAtPortDate,OrigArrivedAtPortDate) < 0 then 'Early'
     When datediff(CurrArrivedAtPortDate,OrigArrivedAtPortDate) = 0 then 'Not Delayed'
     Else 'Delayed' 
     End as Days_Delayed_Flag,
d.finWeekNo as Week_Number,
d.FinWeekSeqNo as Week_Sequence,
d.CurrentWeekCenteredKey as CurrentWeekKey,
d.dayname as Day_Name,
d.findayofweek
```

 - The ‘Vessel_En_Route’ table created in the first step is joined with the ‘Vessel_ETAs’ table.
 - Additionally, integrating the 'imports.fin_dates_with_keys' table, a calendar table, facilitates the identification of the current financial week for reporting purposes.

```sql
From Vessel_En_Route as v
  Left Join Vessel_ETAs as eta on trim(eta.Voyage) = trim(v.Voyage)
  Left Join imports.fin_dates_with_keys as d on d.date  = eta.CurrArrivedAtPortDate
```

 - Case statements are in operation to enrich the data quality. Arrival port codes are translated into port names.
 - Delays in days are computed using the DATEDIFF function, and the resulting values are subsequently categorized into delay bands.
 - Furthermore, an additional case statement classifies the delays into three categories: early, on-time, and late arrivals.


> ## Chapter 6 Power BI Reporting

 - The reporting page below lets stakeholders see the number of containers due weekly and their respective retail sales value (RSV £).
 - Distinct vessels count by arrival port is presented as a stacked bar chart to easily distinguish the port congestion and avoid unnecessary detention and demurrage costs. 
 - Furthermore, the number of containers due each weekday is listed in a table allowing smooth warehouse workload planning.

![Power BI Landing Report](Assests/PBILanding.PNG)

 - Matrix data tables are used to illustrate the list view of container arrivals.
 - This visual is used per the stakeholder request allowing them to easily extract data into Excel files and share it with the wider business.

![Power BI Report Detail](Assests/PBIInfo.PNG)

> ## Chapter 7 Project Reflection

* In line with Driscoll (2007), the ‘What’ model project work is reflected below.

 ![Driscoll What Analysis](Assests/What.PNG)

#### What:

 - This project entails an automated business solution to calculate stock in transit allowing proactive warehouse resource planning and minimising detention and demurrage costs.

#### So What:
 - The precursor of this project was exercised using Excel spreadsheets with manual user intervention, which was prone to human error and report scalability issues with rapid business growth.
 - An automated Extract, Transformation and Load (ETL) workflow was deployed with Azure Databricks and Power BI platforms.

 - Drawbacks identified from the previous process such as report scalability and accuracy issues were eliminated.
 - This has granted reliability to decision-making and saved approximately 156 hours per annum, assuming reports are only produced once a week. The model can be scheduled for more frequent runs generating further business savings. 

#### Now What:
 - The technical nature of this ETL workflow requires a dedicated data engineer for future developments.
 - This model can be further developed by incorporating a Time series machine learning algorithm to predict future weeks' volume extending the forward forecasting capability.


> ## Bibliography

Arrigo, E., 2020. Global sourcing in fast fashion retailers: Sourcing locations and sustainability considerations. Sustainability, 12(2), p.1.

BPP School of Technology. (2024) ‘Topic 8 Data Engineering Tools’ [Recorded lecture], Module 4: Data Engineering. BPP University London 4 April. Available at: https://learn.bpp.com/mod/scorm/player.php?a=13188&currentorg=articulate_rise&scoid=23976&newattempt=on  (Accessed: 13 June 2024)

Chakraborty, M.S., Bondyopadhyay, A. and Ghosh, T.K., 2021. Towards more efficient 3NF determination using reduced functional dependency sets. Advances and Applications in Mathematical Sciences, 11(1), Available through: https://www.mililink.com/upload/article/297473750aams_vol_2011_september_2021_a50_p2937-2945_m._s._chakraborty,_a._bondyopadhyay_and_t._k._ghosh.pdf [Accessed 15 June 2024].

Dayaratne, C. (2024). ‘Workflow created using Azure Databricks’. Module 5: Data Science Professional Practice. BPP University. Unpublished essay.

Dayaratne, C. (2024). ‘ETL Pipeline designed using draw.io’. Module 5: Data Science Professional Practice. BPP University. Unpublished essay.

Driscoll, J. (2007). Practising clinical supervision: a reflective approach for healthcare professionals. 2nd ed. Edinburgh: Baillière Tindall Elsevie.

KPMG, 2023. The supply chain trends shaking up 2023. [online] Available at: https://kpmg.com/xx/en/home/insights/2022/12/the-supply-chain-trends-shaking-up-2023.html [Accessed 29 June 2024]

Kushtagi, R., (2023). Data Deduplication Strategies. [online]. Medium. Last Updated: 2 August 2023. Available at:https://medium.com/@roopa.kushtagi/data-deduplication-strategies-2256f643066e#:~:text=Deduplicated%20data.,-Method%202%3A%20Using&text=We%20can%20use%20GROUP%20BY,GROUP%20BY%20to%20remove%20duplicates. [Accessed 16 June 2024].

Legislation.gov.uk, 2018. Data Protection Act 2018. [Online] Available at: https://www.legislation.gov.uk/ukpga/2018/12/part/3/chapter/2 [Accessed 12 June 2024].

Meteor Space, 2022. Statistics That Prove How Your Delivery Speed Impacts Your Business. [online] Available at: https://www.meteorspace.com/2022/08/25/statistics-that-prove-how-your-delivery-speed-impacts-your-business/ [Accessed 29 June 2024]

Microsoft.com, 2023. What is change data capture (CDC)?. [online] Available at: https://learn.microsoft.com/en-us/sql/relational-databases/track-changes/about-change-data-capture-sql-server?view=sql-server-ver16 [Accessed 19 June 2024]

Palmer, M., 2024. Understanding ETL. 1st ed. Sebastopol: O’Reilly Media, Inc.

Wickham, H., 2014. Tidy Data. Journal of Statistical Software, 10(13), Available through: http://vita.had.co.nz/papers/tidy-data.html [Accessed 15 June 2024].


