# Impact-Evaluation

## Chapter 1 Executive Summary

Owing to the volatility inherent in international shipping routes, it is imperative for global retailers to accurately monitor their inventory in transit to ensure they can adequately meet customer demand.

KPMG (2023) research states that “global and local retailers may need to review their inventory distribution network”. Furthermore, the latest statistics illustrate that “53% of ecommerce retailers find shipping and logistics processes challenging” (Meteor Space, 2022).

This explores Impact evaluation on an automated business project deployed in 2023, analysing vessels en route to UK ports (stock due weekly). Adhering to the Data Protection Act 2018 (Legislation.gov.uk, 2018), business data are not published and have been anonymized.

## Chapter 2 Project Overview
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
