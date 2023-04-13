# Questions and some Points

### stop point
In the TrackIntel framework, the algorithm for stopping points is based on sliding windows. The specific implementation is to compare two adjacent points, and set the point that satisfies the condition as the stop point. Because of the data itself, there are a large number of stop points. And since this algorithm only calculates the distance and time difference between two points, it is not rigorous enough for stopping points.

### location inference
The calculation process of location lies in a certain number of stop points within a certain range. In my setting, there are more than 3 stop points in a radius of 100 meters, which is set as location
I think this algorithm can combine food data points, so that we can better filter out the food points we need

### home inference
Through the calculation of the location in the previous step, the TrackIntel framework gives the relevant guessing algorithm of the home. In the methods, generate an activity label per user by assigning the most visited location the label “home” and the second most visited location the label “work”. 
Because the data itself is insufficient, I think this algorithm cannot reasonably calculate the location of the monk. More factors need to be considered, such as time