## Create Cube

We need define dimension combinations and measure types based on existed data model. This process is called Cube design and create. This chapter will introduce Cube creation process with example data coming with KAP.

Open KAP Web UI, select project `KAP_Sample_1` in project list located at upper left corner. Cube creation happens on `Cube` page.

![](images/createcube_1.png)

Step 1: Select `Sample_Model_1` in `Data Model Name`, enter Cube's name `Sample_Cube_1`. Other fields keep as they are. Click `Next` button.

![](images/createcube_2.png)

Step 2: Select some dimension columns from data model as Cube dimensions. The number of selected columns will affect the number of Cuboid generated later, then the size of Cube data.

In table `KYLIN_CATEGORY_GROUPINGS`, three columns (`META_CATEG_NAME`, `CATEG_LVL2_NAME`, `CATEG_LVL3_NAME`) may be filter condition in query. They should be set as dimensions. As normal dimensions can't be created by Auto Generator, we have to add them manually. Here are the steps:

1. Click `Add Dimension` button, then select `Normal Dimension`.
2. For each dimension, enter dimension name in Name filed, select `KYLIN_CATEGORY_GROUPINGS` table in Table Name field and select corresponding column in that table.

Date usually appear in filter and aggregation condition in query, such as filter by week or aggregation by week. Here we take Week as example, column `WEEK_BEG_DT` in table `KYLIN_CAL_DT` is required and determined by column `PART_DT`. And we can always find a value in `WEEK_BEG_DT` for a `PART_DT` value. So column `WEEK_BEG_DT` is set as derived dimension.

For the same reason, columns `USER_DEFINED_FIELD1`, `USER_DEFINED_FIELD3` and `UPD_DATE、UPD_USER` in table `KYLIN_CATEGORY_GROUPINGS` are set as derived dimensions. Finally set column `LSTG_FORMAT_NAME` in fact table as normal dimension.

The result is shown in following figure:

![](images/createcube_3.png)

Step 3: Define Cube measure types according to aggregation requirements in analysis. COUNT measure and SUM measure could be created automatically, which depend on data type, to demonstrate order amount and over all amount of item sold. Of course these defaulted measures can be modified or delete later manually. In this case, `PRICE` is also a important in sales measurement. For example, total sales `SUM(PRICE)`, highest price `MAX(PRICE)` and lowest price `MIN(PRICE)`. They're all added manually as measures in this step.

![](images/createcube_4.png)

Secondly we need count sellers number by `COUNT(DISTINCT SELLER_ID)`. KAP adopts HyperLogLog algorithm, an approximation algorithm, by default in `COUNT_DISTINCT` computing. Low accuracy is enough in this case, so we choose "Error Rate < 9.75%". For the same reason we create another measure `COUNT(DISTINCT LSTG_FORMAT_NAME)`.

![](images/createcube_5.png)

We usually need figure out the best sellers in business case where TOP-N measure is required. In this case, we execute following SQL query to get the best sellers' ids:

```
SELECT SELLER_ID, SUM(PRICE) FROM KYLIN_SALES 
GROUP BY SELLER_ID 
ORDER BY SUM(PRICE)
```

So we create a TOP-N measure, select PRICE column in SUM and ORDERBY and select SELLER_ID in GROUP BY. Select TOPN(100) as the measure accuracy.

![](images/createcube_6.png)

The result is shown in following figure:

![](images/createcube_7.png)
Step 4: We configure cube's building and maintain. Filter and aggregation conditions of a SQL query are usually based on month or weeks. So Cube's set to automatically merge every week and every month, which means cube will be merged every 7 days and every 4 weeks (28 days). The settings are set as bellow:

![](images/createcube_8.png)

As there's need to query orders histories, Cube auto cleanup is not turned on. Please set value of `Retention Threshold` to 0.

In previous section, we mentioned that we want to build Cube incrementally and choose column `PART_DT` as the partition column. The start time of the Cube is required in creation process, we choose "2012-01-01 00:00:00" as the start time in this case.

Step 5: Optimize Cube's storage size and query speed through Cube's advanced settings, including aggregation group and Rowkey. In chapter 6, we mention that Cuboid number can be reduce by adding aggregation groups, leveraging the hierarchy and containing relationships. In this case, three columns （`META_CATEG_NAME`,`CATEG_LVL2_NAME`,`CATEG_LVL3_NAME`）are in hierarchy relationship, such as first level `META_CATEG_NAME` contains multiple second level `CATEG_LVL2_NAME` and second level contains multiple third level `CATEG_LVL3_NAME`. Let's create Hierarchy Dimensions for them. The design result is shown in following figure:

![](images/createcube_9.png)

Because all dimensions are included in Rowkey of Cuboid, they should be added in Rowkey field. In this case, total 7 dimensions are added. We need set encoding type for each Rowkey. In this case, all Rowkeys are dict encoding except of `LSTG_FORMAT_NAME`, which is fixed_length (length 12) encoding. The order of Rowkeys is important for query speed. In general the order of Rowkeys is organized according to its frequency used in filter condition. The first rowkeys has the highest frequency, it's `PART_DT` in this case.

The Rowkey setting result is shown in following figure:

![](images/createcube_10.png)
​	


> **For Plus Version**: Raw Table is new feature in KAP Plus. If enabled, KAP will keep raw table records in additional to cubing result, to support high speed raw record queries. Raw Table is still in beta stage. It can be enabled and disabled, however other related settings are not effective at the moment.

Step 6: Set Cube configuration override. The configuration added here can override the global ones read from file `kylin.properties`. We don't change any configuration in this case.
​	
Step 7: Cube information overview. Please read the information carefully. Click `Save` button if everything is good. Then click `Yes` button in pop-up menu.
​	
Finally Cube creation is done. The new Cube will be shown in Cube list in refreshed Model page. The state of the Cube is disable for that it has not been built.

![](images/createcube_11.png)