 Hospital Length of Stay Prediction Template in SQL Server with R sevices
--------------------------
 * **Introduction**
 * **System Requirements**
 * **Workflow Automation (4 Scenarios)**
 * **Step 0: Creating Tables**
 * **Step 1: Pre-Processing and Cleaning**
 * **Step 2: Feature Engineering**
 * **Step 3a: Splitting the data set**
 * **Step 3b: Training**
 * **Step 3c: Scoring**
 * **Step 3d: Evaluating**
 * **The Production Pipeline (Scenarios 2 and 4)**
 
### Introduction
-------------------------

In order for hospitals to optimize resource allocation, it is important to predict accurately how long a newly admitted patient will stay in the hospital.

For businesses that prefers an on-prem solution, the implementation with SQL Server R Services is a great option, which takes advantage of the power of SQL Server and RevoScaleR (Microsoft R Server). In this template, we implemented all steps in SQL stored procedures: data preprocessing, and feature engineering are implemented in pure SQL, while models training, scoring and evaluation steps are implemented with SQL stored procedures with embedded R (Microsoft R Server) code. 

All the steps can be executed on SQL Server client environment (such as SQL Server Management Studio). We provide a Windows PowerShell script, Length_Of_Stay.ps1, which invokes the SQL scripts and demonstrates the end-to-end modeling process.

### System Requirements
-----------------------

To run the scripts, it requires the following:
 * SQL server 2016 with Microsoft R server (version 9.0.1) installed and configured;
 * The SQL user name and password, and the user is configured properly to execute R scripts in-memory;
 * SQL Database for which the user has write permission and can execute stored procedures (see create_user.sql);
 * Implied authentification is enabled so a connection string can be automatically created in R codes embedded into SQL Stored Procedures (see create_user.sql).
 * For more information about SQL server 2016 and R service, please visit: https://msdn.microsoft.com/en-us/library/mt604847.aspx


### Workflow Automation (4 Scenarios)
-------------------

We provide a Windows PowerShell script to demonstrate the end-to-end workflow. To learn how to run the script, open a PowerShell command prompt, navigate to the directory storing the PowerShell script and type:

    Get-Help .\SQLR-Length_Of_Stay.ps1

To invoke the PowerShell script, type:
(
    .\SQLR-Length_Of_Stay.ps1 -is_production "Y/N" -ServerName "Server Name" -DBName "Database Name" -username "" -password "" -uninterrupted "Y/N" -dataPath         

You can also type .\SQLR-Length_Of_Stay.ps1, and PowerShell will prompt you for the different parameters. 

-ServerName: a good practice is to write "localhost". 
-username and -password: use rdemo and D@tascience2016 unless you changed them in create_user.sql. 
-dataPath: it is an optional argument that lets the user specify the path for the folder containing the data files. If not specified, the default path links to the Data folder in the parent directory. 

The possible values of is_production and -uninterrupted correspond to 4 different scenarios: 

**Scenario 1: Modeling/Development without prompts (is_production = "N" and -uninterrupted = "Y")**
In this scenario, the end-to-end process from loading the data LengthOfStay.csv, to cleaning it, performing feature engineering, modeling, scoring and evaluating is performed. 
Tables such as Stats, Models and ColInfo are created and stored to be used in a Production pipeline (is_production = Y).
Note that in this Uninterrupted Mode, only the Boosted Trees model (rxFastTrees implementation) is trained, scored and evaluated in this scenario. 

**Scenario 2: Production without prompts (is_production = "Y" and -uninterrupted = "Y")**
In this scenario, a new data set is scored, using models built in a previously performed Modeling/Development pipeline. This data set is cleaned, new features are computed, and it is finally scored.
The predictions are stored in a SQL table. 
Note that in this Uninterrupted Mode, the tables Stats, Models and ColInfo are assumed to be in a development database called "Hospital", and only the Boosted Trees Model is used. If you would like to use another development database, go to Scenario 4. 

**Scenario 3: Modeling/Development with prompts (is_production = "N" and -uninterrupted = "N")**
The difference with Scenario 1, is that the user will be prompted before every step, and will be able to give non default names to tables. 
Also, the user will have the opportunity to build a Random Forest model (rxDForest implementation)

**Scenario 4: Production with prompts (is_production = "Y" and -uninterrupted = "N")**
The difference with Scenario 2, is that the user will be prompted before every step, and will be able to give non default names to tables. 
Also, the user will have the opportunity to specify the name of the database holding the development tables, and use a previously built Random Forest model (rxDForest implementation) for scoring.

**The end-to-end process for the Modeling/Development pipeline (Scenarios 1 and 3) is presented in what follows.**

### Step 0: Creating Tables
-------------------------

The data set LengthOfStay.csv is provided in the Data directory.

In this step, we create a table “LengthOfStay” in a SQL Server database, and the data is uploaded to these tables using bcp command in PowerShell. This is done through either load_data.ps1 or through running the beginning of Length_Of_Stay.ps1. 

Input:

* Raw data: LengthOfStay.csv 

Output:

* 1 Table filled with the raw data: "LengthOfStay"(filled through PowerShell).

Related files:
* step0_create_table.sql

### Step 1: Pre-Processing and Cleaning
-------------------------

In this step, statistics (mode, mean, standard deviation - when applicable -) of LengthOfStay are computed and stored into a table called Stats. This table will be used for the Production pipeline (Scenarios 2 and 4). 
This is done through the [dbo].[compute_stats] stored procedure. 

Then, the raw data is cleaned. This assumes that the ID variable (eid) and the label (lengthofstay) do not contain blanks. 
There are two ways to replace missing values:

The first provided stored procedure, [fill_NA_explicit], will replace the missing values with "missing" (character variables) or -1 (numeric variables). It should be used if it is important to know where the missing values were.

The second stored procedure, [fill_NA_mode_mean], will replace the missing values with the mode (categorical variables) or mean (float variables).

If running the stored procedures yourself, or if running Length_Of_Stay.ps1 with uninterrupted = "N", you will have the opportunity to choose between the two stored procedures. 
If running Length_Of_Stay.ps1 with uninterrupted = "Y", [fill_NA_mode_mean] will be automatically used.

Input:
* 1 Table filled with the raw data: "LengthOfStay"(filled through PowerShell).

Output:
* A view, "LoS0" with the cleaned LengthOfStay data.
* "Stats" table with statistics on the raw data set. 

Related files:
* step1_data_processing.sql

### Step 2: Feature Engineering
-------------------------

In this step, we create a stored procedure [dbo].[feature_engineering] that designs new features:  

* The continuous laboratory measurements (e.g. hemo, hematocritic, sodium, glucose etc.) are standardized: we substract the mean and divide by the standard deviation. 
* number_of_issues: the total number of preidentified medical conditions.

Variables names and types (and levels for factors) of the raw data set are then stored in a table called ColInfo through the stored procedure [dbo].[get_column_info]. It will be used during Production (Scenario 2 and 4) in order to ensure we have the same data types and levels of factors as the development model.

Input:

* "LoS0" View.

Output:

* "LoS" View containing new features.
* "ColInfo" table with variables names and types (and levels for factors) of the raw data set.

Related files:

* step2_feature_engineering.sql

### Step 3a: Splitting the data set
-------------------------

In this step, we create a stored procedure [dbo].[splitting] that splits the data into a training set and a testing set. The user has to specify a splitting percentage. For example, if the splitting percentage is 70, 70% of the data will be put in the training set, while the other 30% will be assigned to the testing set. The eid that will end in the training set, are stored in the table “Train_Id”.


Input:

* "LoS" View.

Output:

* "Train_Id" table containing the eid that will end in the training set.

Related files:

* step3a_splitting.sql


### Step 3b: Training 
-------------------------

In this step, we create a stored procedure [dbo].[train_model] that trains a regression Random Forest (rxDForest implementation) or a Boosted Trees model (rxFastTrees implementation) on the training set. The trained models are serialized then stored in a table called “Models”. The PowerShell script automatically trains only rxFastTrees during deployment (Scenario 1). The other model can be trained by the user with PowerShell (parameter uninterrupted set to 'N' Scenario 3) or from SSMS or R IDE. 


Input:

* "LoS" and "Train_Id" tables.

Output:

* "Models" table containing the trained regression model(s). 

Related files:

* step3b_training.sql

### Step 3c: Scoring  
-------------------------

In this step, we create a stored procedure [dbo].[score] that scores a trained model on the testing set. The Predictions are stored in a SQL table. 


Input:

* "LoS","Train_Id", and "Models" tables.

Output:

* Table(s) storing the predictions from the tested model(s).


Related files:

* step3c_scoring.sql

### Step 3d: Evaluating 
-------------------------

In this step, we create a stored procedure [dbo].[evaluate] that computes regression performance metrics written in “Metrics”.


Input:

* Table(s) storing the predictions from the tested model(s).

Output:

* "Metrics" table containing the performance metrics of the model(s).


Related files:

* step3d_evaluating.sql


Finally, a table LoS_Predictions, stores data from the testing set as well as predicted discharge dates from the rxFastTrees model, and will be used for PowerBI. 
The stored procedure that creates it can be found in the step4_full_table.sql file.  

### The Production Pipeline (Scenarios 2 and 4)
-------------------------

In the Production pipeline, the data from the file LengthOfStay_Prod is uploaded through PowerShell to the LengthOfStay_Prod table.
The tables Stats, ColInfo and Models, created during the development pipeline are then moved to the Production database through the stored procedure [dbo].[copy_modeling_tables] located in the file create_tables_prod.sql .

LengthOfStay_Prod is then cleaned like in Step 1, and a feature engineered view is created like in Step 2 (both using the Stats table). Finally, the view is scored on the model(s) stored in the Models table, using the ColInfo table information.
The predictions are stored in a SQL table.  