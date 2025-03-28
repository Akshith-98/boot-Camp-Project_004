
**                                                      **Real-Time Data Streaming Analytics for Fraud Detection
                                                                     Bootcamp Project - 4****

Architecture of Real-Time Data Streaming Analytics for Fraud Detection: 

****![image](https://github.com/user-attachments/assets/6c6acc9a-c2e9-4516-b865-c0f89816b2e4)

 
The following tells you what tools are required for the project.
Tools & Services:
• **Azure Event Hub** (for ingesting streaming data)
•**	**Azure Stream Analytics**(for processing and filtering data)
•	**Azure Data Lake Storage (ADLS)** (for storing clean data)
•	**Azure Cosmos DB or Azure SQL Database** (for unique transactions)
•	**Azure Synapse Analytics** (for further analysis)

Assuming the data collected from various sources are stored in raw container in ADLS Gen 2.

 ![image](https://github.com/user-attachments/assets/867f902e-a9d8-41c1-a48d-8e40d2370f9d)




For ingestion of the o/p, we created a new container called** SILVER** in **ADLS** and **Unique Transactions** in **COSMOS DB**.
 ![image](https://github.com/user-attachments/assets/9ad2dd1e-d8e9-4205-8a24-3f2ba54f1aa8)


**Step1: Data ingestion with Azure Event Hub**
•	Created an Azure Event Hub Namespace
**Aks-event-004** is the name of the event space.
 ![image](https://github.com/user-attachments/assets/16c31de0-4725-4bc2-a48d-32c3bb8da765)


•	Created an Event Hub inside the name space
**Aks-hub-004** is the name of the Event Hub
 
![image](https://github.com/user-attachments/assets/169e3e1d-9120-4b53-b436-e5a55f706150)




•	Configure Event Hub for Data Ingestion

In the Event Hub, a **Shared Access Policy**with **Send** permissions for the data source is created.

![image](https://github.com/user-attachments/assets/777d36a3-d14b-4993-8391-af89ad64b252)


•	 **Sample iteration-1** file is used as source for Event Hub to send the events to Stream analytics. 

 ![image](https://github.com/user-attachments/assets/23188f34-5c73-44fd-9ea2-1b55952649e4)




**Step 2: Stream Analytics Job for Fraud Detection**

Added **Event hub**(aks-hub-004) and **SQL DB** (aks-98) as Input,
 ![image](https://github.com/user-attachments/assets/d5629701-1ffa-4dfc-b494-25e12e4437cd)




Added **ADLS** and **Cosmos DB** as **Outputs**,

![image](https://github.com/user-attachments/assets/df43ce62-45e1-4534-81ea-619b20c9cc07)




Here we have started the job and sent the event from Event Hub to Stream Analytics and was able to get the o/p as designed. 

o/p:
 ![image](https://github.com/user-attachments/assets/917d64a1-0c88-4826-a579-163e74a6fd3f)

 

![image](https://github.com/user-attachments/assets/43ff797d-b1cd-4b0f-869d-01b319fd2736)



 
The Code used within the Stream Analytics is as follows:

SELECT 
    t.transaction_id, 
    t.user_id, 
    t.timestamp, 
    t.amount,
    t.currency,
    t.location, 
    t.device,
    t.ip_address,
    t.merchant_id,
    t.transaction_type,
    t.is_fraud,
    -- Detecting potential fraud reasons
    CASE 
        WHEN t.amount > 5000 THEN 'High Amount Fraud'
        WHEN u.location <> t.location THEN 'Unusual Location Fraud'
        ELSE 'Legitimate Transaction'
    END AS fraud_reason,
    -- Flagging continuous transactions from the same user
    CASE 
        WHEN LAG(t.user_id) OVER (
            PARTITION BY t.user_id 
            LIMIT DURATION(second, 60)  -- Check within 60 seconds window
        ) = t.user_id 
        THEN 'Rapid Transaction Fraud' 
        ELSE 'First Transaction'
    END AS rapid_transaction_flag
INTO 
    [silver]
FROM 
    [aks-hub-004] t TIMESTAMP BY t.timestamp 
LEFT JOIN 
    [aks-98] u
    ON t.user_id = u.user_id
WHERE 
    t.transaction_id IS NOT NULL
    AND t.user_id IS NOT NULL
    AND t.timestamp IS NOT NULL
    AND t.amount IS NOT NULL
    AND t.location IS NOT NULL;
SELECT *
INTO [UniqueTransactions]  -- Cosmos DB Sink
FROM [aks-hub-004] t TIMESTAMP BY t.timestamp


Now we are going to create synapse for further process of reading the data. A serverless SQL Db is created.
**dbo. Transactions** is created as external table.

 ![image](https://github.com/user-attachments/assets/246405d8-4e8f-4321-8861-7fe3668add79)
