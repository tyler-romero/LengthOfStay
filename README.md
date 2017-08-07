# Healthcare Solution Template 
### *Predicting Length of Stay in Hospitals*
---

### Summary
This solution enables a predictive model for Length of Stay for in-hospital admissions. Length of Stay (LOS) is defined in number of days from the initial admit date to the date that the patient is discharged from any given hospital facility. There can be significant variation of LOS across various facilities and across disease conditions and specialties even within the same healthcare system. Advanced LOS prediction at the time of admission can greatly enhance the quality of care as well as operational workload efficiency and help with accurate planning for discharges resulting in lowering of various other quality measures such as readmissions.

### Solution Details
The resources included in this solution are as follow:
- **Deployment and Configuration Scripts** - Powershell scripts for deploying and configuring a Microsoft Azure VM with SQL Server 2016 installed, SQL R services set up, and external script execution enabled.
- **Sample data** - Synthetic data modeled after real world hospital inpatient records. The data included contains variables such as demographics and disease conditions.
- **R Scripts** - Scripts for loading data, training a model, and scoring new data within SQL Server.
- **Visualizations** - A Power BI dashboard for visualizing length of stay results.

## Installation & Setup
We structure the installation and setup procedures as applying to one of three personas:
- **Developer** - Interested in deploying the solution end to end and feeding the output into BI tools.
- **Business manager** - Interested in analyzing the results of LOS scoring in order to find insights.
- **Data scientist** - Interested in making modifications to the underlying models in order to satisfy particular business needs.

### Developer Instructions
*Prerequisites*
- Install [Azure Powershell](https://azure.microsoft.com/en-us/documentation/articles/powershell-install-configure/) where deployment script will be run 
- Download and install Power BI desktop from [here](https://powerbi.microsoft.com/en-us/desktop/?gated=0&number=1).

*1) Deployment*
- Navigate to \deploy\ directory and run deploy.ps1. You will be prompted to enter:
	- subscriptionId: The ID of the Azure subscription you will deploy to. This ID can be found under Subscriptions in the Azure Portal.
	- resourceGroupName: The name of the resource group you will deploy to. Can be a new group. If new, you will later be prompted to enter a location (e.g. West US, East US, etc.).
	- storageAccount1: A name for the standard storage account you will create. May not be an existing storage account name.
	- storageAccount2: A name for the premium storage account you will create. May not be an existing storage account name.
	- password: The admin password for the VM you will create.
- Log in to the VM you created (VM name will be 'lengthofstayvm', username 'lengthofstayvmadmin', password will be the password you provided in the last step.
- Copy or clone this repo on the VM. 
- Run VMSetup.ps1 in powershell (handles configuring SQL Server).

*2) Data Loading and Model Training* 

- Navigate to the \data\ directory on the VM and run LoadTrainDataCreateModel.ps1. This file takes an optional parameter to a .csv file to be used for training. The default parameter points to a sample file included in this solution.
    
*3) Scoring and results*

- Run LoadAndScoreTestData.ps1. This file takes an optional parameter to a .csv file to be used for scoring. The default parameter points to a sample file included in this solution.
  
- Navigate to the \visualize\ directory and open MS_SA_direct in Power BI.

### Business Administrator Instructions

For business administrators, simply navigate to the \score\ directory and open MS_SA_loaded in Power BI.

### Data Scientist Instructions

All above instructions apply to the Data Scientist, with the addition of the \data\TrainLOSModel.R file. This file should be opened using an R IDE of choice (RStudio, RTVS, RGui). The file contains code for conducting addtional exploration of the data in order to perform addtional model training / tuning.

## Removing the Resources
- Simply go the portal.azure.com and delete the resource group you created, this will delete all resources created