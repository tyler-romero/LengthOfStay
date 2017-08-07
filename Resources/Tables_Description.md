
# Length of Stay Tables Description
--------------------------
 This table describes the different data sets that you will find in your database after deployment. 

<table style="width:85%">
   <tr>
    <th>File</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>LengthOfStay</td>
    <td>Raw data about each hospital admission </td>
  </tr>
    <tr>
    <td>Stats</td>
    <td>Variable names, types, mode, mean and standard deviation of the raw data set (when applicable) </td>
  </tr>
    <tr>
    <td>LoS0 (View)</td>
    <td>Cleaned data set </td>
  </tr>
  <tr>
    <td>LoS (View) </td>
    <td>Analytical data set: cleaned data with new features</td>
  </tr>
  <tr>
    <td>Train_Id</td>
    <td>List of encounter IDs that will go to the training set</td>
  </tr>
  <tr>
    <td>Models</td>
    <td>Table storing the trained models</td>
  </tr>
    <tr>
    <td>Boosted_Prediction</td>
    <td>Prediction results when testing the trained boosted trees (rxFastTrees)</td>
  </tr>
    <tr>
    <td>Metrics</td>
    <td>Performance metrics of the models tested</td>
  </tr>
    <tr>
    <td>LoS_Predictions</td>
    <td>Testing set with the predicted dates of the trained models for discharge</td>
  </tr>
</table>
