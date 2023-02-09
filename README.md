# Anomaly Detection System Architecture
![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54)
![scikit-learn](https://img.shields.io/badge/scikit--learn-%23F7931E.svg?style=for-the-badge&logo=scikit-learn&logoColor=white)
![NumPy](https://img.shields.io/badge/numpy-%23013243.svg?style=for-the-badge&logo=numpy&logoColor=white)
![Pandas](https://img.shields.io/badge/pandas-%23150458.svg?style=for-the-badge&logo=pandas&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)
***
An architectural overview of an anomaly detection system.

## Description
The document gives an architectural overview of an anomaly detection system that identifies anomalous ETL jobs 
that populate the data warehouse.

The Business Intelligence team is responsible for maintaining the data warehouse.The data warehouse is populated  
by different ETL jobs. The anomaly detection system identifies ETL jobs that ran successfully, but have discrepancy  
in the imported data. Like the number of rows being populated in the data warehouse not being in the expected range.


## Overview
The anomaly detection system uses scikit-learn implementation of isolation forest algorithm to identify anomalous 
ETL jobs. PySpark is used to schedule historical data load. The overall infrastructure is set up using AWS technologies. 
Once the system detects an anomaly in the ETL jobs it raises a ticket to notify relevant stakeholders.


## Architecture Diagram
![alt text](img/Anomaly%20Detection.jpg)


## Explanation
* ETL pipelines are set up using PySpark to load **Historical Data** and the **Recent Job Run Data** from the main data lake 
into S3 Buckets **(S3:Historical, S3:Recent)**. This is achieved using Spark on EMR.
* The model is trained in SageMaker (Jupyter) using the scikit-learn isolation forest algorithm.
* Using Boto3 the Historical Dataset is accessed from S3. The model is trained using selected features: Records Processed, 
Day of Week, and Run Time. The trained model is stored in S3 **(S3:Model)**.
* Lambda Function **(Lambda:Predict)** will load the Model **(S3:Model)** and the Recent Job Run Data **(S3:Recent)** from S3.
* **Lambda:Predict** will use the model to make prediction about the anomalous nature of recent ETL jobs.
* The anomalous ETL jobs detected by the **Lambda:Predict** are written back to S3 **(S3:Predictions)**.
* **Lambda:SIM** will read the data from **S3:Prediction** and generate a ticket to notify the stakeholders about 
anomalous ETL job.

***