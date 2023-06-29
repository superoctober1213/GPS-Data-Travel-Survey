# GPS Data Travel Survey

### Framework & Package
- pandas
- numpy
- trackintel
- shapely
- pyproj
- datetime
For more information about trackintel framework, in https://trackintel.readthedocs.io/en/latest/index.html

###  Data processing
Filter out drift GPS data according to Forensicflag. Transfer the filtered GPS points to the trackintel framework data, extracts longitude and latitude and build a geopandas GeoDataFrame (POINT)

##### Algorithm
``` python
trackintel.io.file.read_positionfixes_csv('data.csv', columns={'user_id':'user_id', 'latitude':'latitude', 'longitude':'longitude', 'time':'tracked_at'},tz= 'America/New_York',crs ='EPSG:4326',index_col = 0)
```
Parameters
-  tz= 'America/New_York' --- change original time zone "New York Time" to UTC
-  crs ='EPSG:4326' --- Set coordinate reference system as EPSG:4326

Returns:pfs – A GeoDataFrame containing the positionfixes.
Return type: GeoDataFrame (as trackintel positionfixes)

###  Stop Inference
Infer the stay point based on the number of GPS points and the stop time. When a user stays within a radius of 100 meters for more than 5 minutes, then we will infer that the user has stayed in this area. The center point of this stop point is the geometric center of all GPS points in this area and time.
##### Algorithm
``` python
pfs, sp = pfs.as_positionfixes.generate_staypoints(method='sliding', distance_metric='haversine', dist_threshold=100, time_threshold=5.0, gap_threshold=720.0)
```
Parameters
- dist_threshold=100：when the user stays at one location and moves from that location to another, the distance between the two locations must be less than or equal to dist_threshold in order to place them considered as the same stop.
- time_threshold=5.0：It measures the time difference between two temporally adjacent location points.
- gap_threshold=720.0：The gap_threshold parameter specifies the time interval between two location points in the trajectory. If this time interval exceeds gap_threshold, the two location points will be considered discontinuous.

Returns:
pfs (GeoDataFrame) – The original positionfixes with a new column [`staypoint_id`].
sp (GeoDataFrame) – The generated staypoints.

### Trip Inference
Infer the path based on the stop point in the previous step. For all GPS points between two adjacent stay points, if the interval between the points is not greater than the set 60 minutes, then the GPS points can be connected into a trip.
```python 
pfs, tpls = pfs.as_positionfixes.generate_triplegs(sp, method='between_staypoints', gap_threshold = 60)
```
Parameters
gap_threshold = 60: the same meaning as stop inference

Returns:
pfs (GeoDataFrame) – The original positionfixes with a new column [`tripleg_id`].
tpls (GeoDataFrame) – The generated triplegs.

### Home Inference
The location of the user's home is inferred from the GPS data of each user.
In the first step, we count the GPS position and data number of each user from 10:00 p.m. to 6:00 a.m. the next day. The location with the most data is set as the potential home location. After that, the potential location with the most number of days is estimated as the home location.
In the second step, users who did not find their home location in the previous step, we use the GPS data of these users on weekends to infer their home location. We set the location where the user spends the longest weekend as the potential home location. Afterwards, the potential location with the most days is estimated as the home location.

### Food Inference
From all stay points, we find out the stay points within 200 meters of the food retail area. We infer that the user has visited the food retails.

### Food Tour Inference
##### Algorithm
In the previous step, we inferred the stay points that visited the retail store.We use these points as the end point to find out the previous trip and stay point. Connect these trips and stops into a whole tour. Based on this algorithm, we deduce the process and starting point of each trip to the retail store.

The algorithm starts by identifying a stay point and then searches for any trip or stay points that occurred before it within a specific time window. If such points are found, they are temporally connected to the current point, they are considered in the same tour. This process is repeated until there are no more trip or stay points that occurred before the current point within the time window.

```python
find_trip_before(stay_point_id, time_window = 15):
```
Parameters
- time_window = 15： the time_window parameter refers to a specific time interval used to identify candidate trip and stay points that occurred before a given stay point.



