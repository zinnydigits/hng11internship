## HNG 11 INTERNSHIP DATA ANALYST TASK 1 ReadMe

## Overview

The selected state for this Nigerian 2023 Presidential Election Outlier Vote Detection task is Anambra. The task involves analyzing polling unit data by clustering units based on their geolocation proximity and calculating outlier scores for votes received by each party within these clusters. The repository includes clean Anambra state Election data, output data, data with geolocation, and a report summarizing the analysis. The steps to reproduce the analysis are outlined below.

![Outliers Visualization](https://github.com/zinnydigits/hng11internship/blob/main/hngtask1.png)


## Data

### Columns

`State`, `LGA`, `Ward`, `PU-Code`, `PU-Name`, `Accredited_Voters`, `Registered_Voters`, `Results_Found`, `Transcription_Count`, `Result_Sheet_Stamped`, `Result_Sheet_Corrected`, `Result_Sheet_Invalid`, `Result_Sheet_Unclear`, `Result_Sheet_Unsigned`, `APC`, `LP`, `PDP`, `NNPP`, `Results_File`

### Additional Column

`Address`: Concatenation of `PU-Name`, `LGA`, and `State` for geocoding.

## Activities Carried Out

1. **Geolocation Data**: 
   - Concatenated location columns to create addresses needed for geocoding.
   - Used Geocode by Awesome Table to generate geolocations of the polling unit addresses.
   - Exported data for geocode generation using `data.to_csv('find_geo_anambra.csv')`.
   - Imported the CSV file with generated geolocations.

2. **Proximity Clustering**:
   - Calculated the Haversine distance between polling units.
   - Grouped polling units by proximity using the defined radius (e.g., 1 km).
   - Created bins and labels for proximity grouping.

3. **Outlier Score Calculation**:
   - For each cluster, the mean and interquartile range (IQR) for each party's votes were calculated.
   - The deviation and outlier scores based on the deviation of votes from neighboring units within the cluster were as well calculated.

4. **Analysis and Visualization**:
   - Visualized the distribution of polling units by proximity.
   - Merged calculated statistics with the original DataFrame.
   - Identified and highlighted the top 3 outliers and their closest units.
   - Visualized the distribution of outlier scores for each party.

## Code Snippets

### Geolocation and Distance Calculation

```python
data['Address'] = data['PU-Name'] + ", " + data['LGA'] + ", " + data['State']
data.to_csv('find_geo_anambra.csv')
# After geolocation data is generated and imported
first_lat = df.loc[0, 'Latitude']
first_lon = df.loc[0, 'Longitude']

def haversine(lat1, lon1, lat2, lon2):
    R = 6371.0
    dlat = math.radians(lat2 - lat1)
    dlon = math.radians(lon2 - lon1)
    a = math.sin(dlat / 2)**2 + math.cos(math.radians(lat1)) * math.cos(math.radians(lat2)) * math.sin(dlon / 2)**2
    c = 2 * math.atan2(math.sqrt(a), math.sqrt(1 - a))
    return R * c

df['Distance'] = df.apply(lambda row: haversine(first_lat, first_lon, row['Latitude'], row['Longitude']), axis=1)
```

## Clustering and Statistics
```python
ward_stats = df.groupby('New Ward').agg({
    'APC': ['mean', calculate_iqr],
    'LP': ['mean', calculate_iqr],
    'PDP': ['mean', calculate_iqr],
    'NNPP': ['mean', calculate_iqr]
}).reset_index()

df_merged = pd.merge(df, ward_stats, on='New Ward')

coordinates = df[['Latitude', 'Longitude']].values
tree = KDTree(coordinates)
distances, indices = tree.query(coordinates, k=2)
nearest_neighbour = indices[:, 1]
df_merged['Nearest Pooling Unit'] = df_merged['Address'].iloc[nearest_neighbour].to_list()

for party in ['APC', 'LP', 'PDP', 'NNPP']:
    df_merged[f'{party}_deviation'] = df_merged[f'{party}'] - df_merged[f'{party}_mean']
    df_merged[f'{party}_outlier_score'] = (df_merged[f'{party}_deviation'] / df_merged[f'{party}_iqr']).abs()
```

## Conclusion

This task provides a comprehensive analysis of polling unit data by leveraging geolocation to create proximity-based clusters and identifying outlier voting patterns. By following the outlined steps, you can reproduce the analysis and gain insights into the voting behaviors within different regions. The results highlight significant deviations in voting patterns, helping to identify potential anomalies or areas of interest.

The repository includes:
- **Raw Data**: Original data used for the analysis.
- **Output Data**: Results generated from the analysis process.
- **Data with Geolocation**: Polling unit data enriched with geolocation details.
- **Report**: A detailed report summarizing the steps, analysis, and findings.


Thank you for your interest in this project. For any questions or support, please reach out via the contact on the notebook.
