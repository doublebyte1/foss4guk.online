# Processing Large Geospatial Datasets

### Joana Simoes and Christian Rouffaert

--
![corona](/images/teragence-hres-pad.png)<!-- .element  width="50%" -->

- UK based company, operating in the field of network analytics.


---
## Goal

- Understanding mobile network usage.

Note:
- Provide mobile phone operators with insights about their customer behaviour
- Spatial problem

---
## Assets

- Crowd-sourced data about mobile usage (e.g.: technology type, frequency bands, location).

Note: 
- explain beterhow this is obtained 

---
## How

- Processing and analysing these large and noisy datasets.

Note:
- Turn raw data into useful information

--
![corona](/images/earthpulse_trans.gif)<!-- .element  width="25%" -->

- Company specialized in Big Data and geospatial analytics.

---
## Specific Problem
- Identify areas of coverage (AoC) of telco antenas.
- Transform a point cloud into a set of discrete surfaces (e.g.: polygons).

---
## Points to polygons
- Convex Hull

![corona](/images/ch.png)<!-- .element  width="50%" -->


Note:
-  CH of a set of points X is the smallest convex set that contains X.

---
### Wait, maybe we don't want to use all the points!

- Some points are outliers.
- We are only interested in identifying the high-density areas.

---
## Clustering

- In density-based clustering, clusters are defined as areas of higher density than the remainder of the data set. 
- DBSCAN is a cluster algorithm which does not require a pre-defined number of clusters.

Note:
- Objects in sparse areas - that are required to separate clusters - are usually considered to be noise and border points. 

---
## Putting it all together

- Some small-scale experiments were run in QGIS.
- Tune algorithm parameters.

Note:
- The next step was to run at this at scale.

---
## Tech Approach

- We are dealing with a large volume of data, so it is not really feasible to hold it in memory.
- Databases are capable of storing and managing large amounts of data.
- PostGIS supports both, convex hull and DBSCAN algorithms.

---
## Architecture

- We wrote a Python application to automate the interaction with the database, and to support bulk processing.
- We wrapped the database and Python app in docker containers and orchestrated them using docker-compose.

Note:
- Add a diagram with the architecture and tech stack

---
## Deployment

- To run this at scale, the container orchestration was deployed on AWS.

Note:
- TODO: speak about which machines were created, and how long it took them to run.

--
## Results


TODO: discuss some results; show some images

--
## Next Steps

TODO: API

--
## Final Thoughts

TODO: Some lessons learned here

--
## Questions?


--

This presentation was created using [Reveal.js](https://revealjs.com/#/), the HTML presentation framework. Fork it at:
[https://github.com/doublebyte1/foss4guk.online](https://github.com/doublebyte1/foss4guk.online)

```
npx reveal.js-online
```


