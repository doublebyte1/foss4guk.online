# Processing Large Geospatial Datasets
## <i class="fa fa-rocket" aria-hidden="true"></i>
### Joana Simoes and Christian Rouffaert

--
![corona](/images/teragence-hres-pad.png)<!-- .element  width="50%" -->


- UK based company, operating in the field of network analytics.

<small>https://www.teragence.com/</small>
---
## Goal

- Understanding mobile network usage.

![corona](/images/mobile-users.jpeg)<!-- .element  width="50%" -->

Note:
- Provide mobile phone operators with insights about their customer behaviour
- Spatial problem

---
## Assets

- Crowd-sourced data about mobile usage (e.g.: technology type, frequency bands, location).

![corona](/images/cell_tower_original.png)<!-- .element  width="50%" -->

Note: 
- explain beterhow this is obtained 
- Data for the entire UK: how many points?

---
## How <i class="fa fa-cogs" aria-hidden="true"></i>

- Processing and analysing these large and noisy datasets.

Note:
- Turn raw data into useful information

--
![corona](/images/earthpulse_trans.gif)<!-- .element  width="25%" -->

- Company specialized in Big Data and geospatial analytics.

<small>https://earthpulse.pt</small>
---
## Specific Problem
- Identify areas of coverage (AoC) of telco antenas. <!-- .element: class="fragment fade-in-then-out"-->
- Transform a point cloud into a set of discrete surfaces (e.g.: polygons). <!-- .element: class="fragment fade-in-then-out"-->

![corona](/images/few_clusters_trans.png)<!-- .element  width="55%" -->

Note:
- It basically comes down to delimitate the area covering all the points with the same
cid (A GSM Cell ID) base transceiver station )and lac)

---
## Points to polygons
- Convex Hull

![corona](/images/convex_hull6.png)<!-- .element  width="60%" -->


Note:
-  CH of a set of points X is the smallest convex set that contains X.

---
### Wait, maybe we don't want to use all the points!

- Some points are outliers. <!-- .element : class="fragment"-->
- We are only interested in identifying the high-density areas. <!-- .element: class="fragment"-->

![corona](/images/big_run2.png)<!-- .element  width="80%" -->

---
## Clustering

- In density-based clustering, clusters are defined as areas of higher density than the remainder of the data set. 
- DBSCAN is a cluster algorithm which does not require a pre-defined number of clusters.

![corona](/images/dbscan.png)<!-- .element  width="25%" -->

Note:
- Objects in sparse areas - that are required to separate clusters - are usually considered to be noise and border points. 

---
## Putting it all together

- Some small-scale experiments were run in QGIS.
- Tune algorithm parameters.

![corona](/images/qgis.png)<!-- .element  width="60%" --> 


Note:
- minpts and radius from dbscan
- The next step was to run at this at scale.

--
## Tech Approach <i class="fa fa-database" aria-hidden="true"></i>

- We are dealing with a large volume of data, so it is not really feasible to hold it in memory.
- Databases are capable of storing and managing large amounts of data.<!-- .element: class="fragment"-->

---

PostGIS supports both, the **convex hull** and **DBSCAN** algorithms.

```SQL
SELECT * FROM            
            (SELECT ST_ClusterDBSCAN(geom_, %s, %s)
            OVER()
            AS
            cluster_id, latitude, longitude, geom_, techtype, cid,
            lac 
            FROM
            sample where cid=%s) sq
            WHERE cluster_id IS NOT NULL;
            
    SELECT d.techtype, d.cid, d.lac,
    ST_AsText(ST_ConvexHull(ST_Collect(d.geom_))) As geom_
    FROM clusters As d
    GROUP BY d.techtype, d.cid, d.lac;
```

---
## Architecture <i class="fa fa-sitemap" aria-hidden="true"></i>

- Python application to automate the interaction with the database.
- Wrapped the database and Python app in docker containers and orchestrated them using docker-compose.

![corona](/images/docker-compose_trans.png)<!-- .element  width="20%" --> 

Note:
- Support bulk processing.
- Add a diagram with the architecture and tech stack

---
## Technology Stack <i class="fa fa-cubes" aria-hidden="true"></i>

<table style="border-collapse: collapse; color:black; border: 0px yellow none; font-size: 15px; vertical-align: bottom;">
    <tr>
        <td><img src="/images/python.png" height="150px" style="background:none; border:none; box-shadow:none;"></td> 
        <td><img src="/images/postgis.png" height="150px" style="background:none; border:none; box-shadow:none;"></td>
        <td><img src="/images/docker.png" height="150px" style="background:none; border:none; box-shadow:none;"></td> 
    </tr>
</table>

Note:
- FOSS, FOSS4G
---
## Deployment <i class="fa fa-cloud" aria-hidden="true"></i>

- To run this at scale, the container orchestration was deployed on AWS.
- The use of cloud servers, enabled us to run jobs in parallel.
- Docker made it really easy to deploy the application in different servers.

![corona](/images/aws_logo_179x109.gif)<!-- .element  width="30%" --> 

Note:
- TODO: speak about which machines were created, and how long it took them to run.
- We split the batch jobs for each area code
- Machines 16GB 32GB
- Job structure: process a bulk of files which contains one file, per area code
- When you need to process a large volume of data in a short period of time, cloud is a must
- The hardware was not so much an issue, but the number of servers was.

--
## Results

![corona](/images/minpts_25b.png)<!-- .element  width="80%" --> 

The output was written in OGC WKT format.

Note:
- Say something about the results
- Interoperability
- Say something about the size of the original dataset and how long it took to run

--
## Next Steps


![corona](/images/api.png)<!-- .element  width="80%" --> 

--
## Lessons Learned
- When you process at scale, memory bugs are taken to an unprecedent level.<!-- .element: class="fragment fade-in-then-semi-out"-->
- Small tweaks have a great impact on the performance.<!-- .element: class="fragment fade-in-then-semi-out"-->
- Bulk processing: the art of split and merge.<!-- .element: class="fragment fade-in-then-semi-out"-->
- Monitor everything.<!-- .element: class="fragment fade-in-then-semi-out"-->
- Check the outputs in a GIS.<!-- .element: class="fragment fade-in-then-semi-out"-->
- Try to tune the parameters in small scale experiments.<!-- .element: class="fragment fade-in-then-semi-out"-->

Note:
- processing large amounts of data: it is hard to debug
- Let's say that we have a bug, that at each iteration is eating a small amount of memory. This can grow to a point that will eat all the RAM memory of the machine and cause it fail.
- If we can avoid clustering sets with less than x points
- There is a delicate balance here: we want to run batch jobs to decrease the amount of human intervention, but this also decreases our chance of intervention in case of errors.
- Generate detail logs through the application, in order to understand what happened in case of failure. This allows you to go back to a GIS and check what happened. Also monitor the hardware.
- It is not really easy to understand what is going on here, withour any visual queues.
- It is not always possible to tune the paramaeters in small scale experiments, because you dont have the scale and diversity of situations that you do with the real dataset.
- The use of spatial indexes in the tables is a must.

--

- PostGIS performed really well for this use case, even considering an iterative method such as DBSCAN.

![corona](/images/postgis.png)<!-- .element  width="25%" --> 


--
## Thank you!
### We would love to <i class="fa fa-bullhorn" aria-hidden="true"></i> from you


<table style="border-collapse: collapse; color:black; border: 0px yellow none; font-size: 25px; vertical-align: middle;">
    <tr>
        <td><i class="fa fa-envelope" aria-hidden="true"></i></td> 
        <td> joana@earthpulse.pt</td>
        <td><i class="fa fa-envelope" aria-hidden="true"></i></td> 
        <td>christian.rouffaert@teragence.com</td>        
    </tr>
    <tr>
        <td><i class="fa fa-twitter" aria-hidden="true"></i></td> 
        <td>@doublebyte</td>
    </tr>    

 
</table>


--

## Questions?

![corona](/images/lisa.png) <!-- .element  width="30%" --> 
--

This presentation was created using [Reveal.js](https://revealjs.com/#/), the HTML presentation framework. Fork it at:
[https://github.com/doublebyte1/foss4guk.online](https://github.com/doublebyte1/foss4guk.online)

```
npx reveal.js-online
```


