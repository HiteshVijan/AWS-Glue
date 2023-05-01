# Query Data in Amazon S3 with Amazon Athena and AWS Glue

Big data problems often involve a large number of heterogeneous data sources. As a data analyst, you might not know the schema for some data sources. This is the *variety* aspect of the five Vs of big data (Volume, Variety, Velocity, Veracity, and Value). In this lab, you will work with AWS Glue. You can direct AWS Glue to a data source, and it can infer a schema based on the data types that it discovers. AWS Glue builds a catalog that contains metadata about the various data sources. AWS Glue is similar to Amazon Athena in that the actual data you analyze remains in the data source. The key difference is that you can build a *crawler* with AWS Glue to discover the schema.

AWS Glue integrates with the following data sources:

- Amazon Athena
- Amazon Redshift Spectrum
- Amazon EMR

# Objectives

After completing this lab, you will be able to:

- Access AWS Glue in the AWS Management Console
- Create a crawler with AWS Glue
- Create tables and a schema with AWS Glue
- Query data in Amazon Simple Storage Service (Amazon S3) from Amazon Athena with the AWS Glue data catalog

**Prerequisites**

This lab requires:

- Access to a notebook computer with Wi-Fi and Microsoft Windows, macOs, or Linux (Ubuntu, SuSE, or Red Hat)
- For Microsoft Windows users: Administrator access to the computer
- An internet browser such as Chrome, Firefox, or IE9 (previous versions of Internet Explorer are not supported)

**Duration**

This lab requires **60** minutes to complete. The lab will remain active for **120** minutes to allow extra time to complete the challenge question.

## Accessing the AWS Management Console

1. At the top of these instructions, choose <span id="ssb_voc_grey">Start Lab</span> to launch your lab.

  A **Start Lab** panel opens, which displays the lab status.

2. Wait until you see the message *Lab status: ready*, then close the **Start Lab** panel by choosing the **X**.

3. At the top of these instructions, choose <span id="ssb_voc_grey">AWS</span>

  This will open the AWS Management Console in a new browser tab. The system will automatically log you in.

  **Tip**: If a new browser tab does not open, there will typically be a banner or icon at the top of your browser that indicates that your browser is preventing the website from opening pop-up windows. Choose the banner or icon and then choose **Allow pop ups**.

4. Arrange the **AWS Management Console** browser tab so that it displays next to these instructions. Ideally, you should be able to see both browser tabs at the same time, which can make it easier to follow the lab steps.

## Scenario summary ##

In this lab, you are a data analyst who works for an international development agency. The agency focuses on drought relief. You are asked to look for weather patterns since 1950.

Fortunately for you, global weather data is already stored in Amazon S3. The National Centers for Environmental Information (NCEI), which is in the US, maintains a dataset of climate data. This dataset includes observations from weather stations around the globe. The Global Historical Climatology Network Daily (GHCN-D) contains daily weather summaries from ground-based stations. The dataset contains data that goes back to 1763, and it is updated daily.

The most common recorded parameters are daily temperatures, rainfall, and snowfall. These parameters are useful for assessing risks for drought, flooding, and extreme weather. The data definitions are publicly available on the <a href="https://docs.opendata.aws/noaa-ghcn-pds/readme.html" target="_blank">AWS Open Data Website</a>.

The following diagram illustrates the architecture of the solution you will develop in this lab:

<img src="images/lab-3-overview.png" alt="Overview of Lab 3" width="400">

## Task 1: Create a crawler for the GHCN-D dataset

As a data analyst, you might not always know the schema of the data that you need to analyze. AWS Glue is designed for this situation. You can direct AWS Glue to your data stored on AWS, and it will discover your data. AWS Glue will then store the associated metadata (for example, the table definition and schema) in the AWS Glue Data Catalog. You accomplish this by creating a crawler that will inspect the data source and infer a schema based on the data. The account that you use to log in when you run AWS Glue must have permissions to access the data source. In this lab, you will work with publicly available data, so there's no need to create a specific AWS Identity and Access Management (IAM) account. However, this will not always be the case. To read more about how AWS Glue and IAM work together, see <a href="https://docs.aws.amazon.com/glue/latest/dg/authentication-and-access-control.html" target="_blank">Authentication and Access Control for AWS Glue</a>.

The first task is to create a crawler that will discover the schema for the GHCN-D dataset.

5. On the AWS Management Console, on the **Services** , choose **AWS Glue**.

6. Choose **Get started**.

   **Note:** If you do not see the **Get started** window, then proceed to the next step.

7. Choose **Add tables using a crawler**.

8. For the crawler name, enter `Weather`.

9. Choose **Next**.

10. On the **Specify crawler source type** page, choose **Data stores**.

11. Choose **Next**.

12. Choose **Specified path in another account**.

13. For the **Include path**, enter the following S3 bucket location:

  ```
  s3://noaa-ghcn-pds/csv/
  ```

14. Choose **Next**.

15. When prompted to add another data store, choose **No**.

16. Choose **Next**.

17. Choose **Choose an existing IAM role**.

18. Choose *gluelab*.

19. Choose **Next**.

20. Accept the default frequency of **Run on demand**.

21. Choose **Next**.

22. Choose **Add database**.

23. In the **Database name** box, enter `weatherdata`.

24. Choose **Create**.

25. Choose **Next** .

26. Review the summary of the crawler and then choose **Finish**.

### Task 1.1: Run the crawler ###

You can create AWS Glue crawlers to either run on demand, or on a set schedule. To read more about scheduling crawlers, see <a href="https://docs.aws.amazon.com/glue/latest/dg/schedule-crawler.html" target="_blank">Scheduling an AWS Glue Crawler</a>. Because you created your crawler to run on demand, you must run the crawler to generate the metadata.

27. From the window with the message that the *weather* crawler was created, choose **Run it now?**

  You will see the status for the crawler change to *Starting* and then *Running crawler*. After approximately 1 minute, the status will change to *Ready*, and the **Tables added** column will indicate that one table was added.

28. AWS Glue creates a table to store the metadata about the GHCN-D dataset. Inspect the data that AWS Glue captured about the data source.

### Task 1.2: Review the metadata created by AWS Glue

29. In the navigation pane, choose **Databases**.

30. Choose the **weatherdata** database.

31. Choose **Tables in weatherdata**.

32. Choose the **csv** table.

33. Review the metadata that the weather crawler captured. You should see a list of the columns the crawler discovered. The following screenshot illustrates some of the columns:

<img src="images/crawler.png" alt="AWS Glue crawler output" width="600">

Notice that the columns are named *col0* through *col6*. In the next step, you will give the columns more descriptive names.


### Task 1.3: Edit the schema ###

34. In the upper-right corner of the window, choose **Edit schema**.

35. Change the column names by selecting them and entering the new names. The following table lists the new column names to use.

<img src="images/Lab3task1.png" alt="New column names" width="600">

36. Choose **Save**.


## Task 2: Query the table using the AWS Glue Data Catalog

Now that you created the AWS Glue Data Catalog, you can use the metadata that is stored in the AWS Glue Data Catalog to query the data in Amazon Athena.

37. From the navigation pane, choose **Tables**.

38. Select the **csv** table check box.

39. From the **Action** menu, choose **view data**.

  You will see a warning that Athena is going to open and that you will be charged for Athena usage. The Athena console will open.

40. Choose **Preview data**.

41. Choose **Get started**.

42. If the tutorial window opens, close it.

You see the following message at the top of the console:

<img src="images/athenas3message.png" alt="Message for adding an S3 bucket to hold query results" width="800">

You must specify an Amazon Simple Storage Service (Amazon S3) bucket to hold the results from any queries that you run.

43. On the AWS Management Console, on the **Services** menu, choose **S3**.

Amazon S3 opens and you see the bucket that was created for you in the lab environment. 

44. Select the bucket name.

45. In the bucket properties window, choose **Copy bucket ARN**.

46. Paste the bucket Amazon Resource Name (ARN) into a text editor.

47. To return to Athena, go back the AWS Management Console and on the **Services** menu, choose **Athena**.

48. Choose **setup a query result location in Amazon S3**.

49. In the **Query Result Location** box, enter the name of the bucket. The name of the bucket is the long string of characters at the end of the bucket ARN. Before the name of the bucket, make sure to specify `s3://` and terminate the bucket name with a forward slash (`/`), as shown in the example.

<img src="images/athenaqueryresultlocation.png" alt="Dialog box for adding the S3 bucket for storing query results" width="800">

50. Choose **Save**.

51. From the list of databases, choose the **weatherdata** database.

52. Choose the **csv** table.

53. Choose the vertical ellipsis (three dots) and then choose **Preview table**.

You will see the first 10 records of the weather table.

Athena ran a structure query language (SQL) query to get first 10 rows from the table. Data is not yet loaded at this stage, but by using AWS Glue, you inferred and edited the schema to suit your needs. Also, note the resources that were consumed by the Athena query (the run time and the amount of data that was scanned). As you develop more complex applications, minimizing resource consumption will play an important role in optimizing costs.

### Task 2.1: Create a table for data after 1950 ###

In this step, you will create an external table that only includes data since 1950. To optimize your use of Athena, you will store data in the Apache Parquet format. Apache Parquet is an open source, columnar data format that is optimized for performance and storage. To read more about Apache Parquet, see <a href="https://parquet.apache.org/" target="_blank">Apache Parquet</a>.

54. Create an S3 bucket to store the external table. The bucket should be in the same Region that your lab is running in. Follow the bucket naming conventions that are described in the <a href="https://docs.aws.amazon.com/AmazonS3/latest/gsg/CreatingABucket.html" target="_blank">Amazon S3 documentation</a>.

55. Copy the following query and in the Athena command window, paste the query. Remember to replace *&lt;yourbucket&gt;* with the name of the bucket that you created:

  ```sql
  CREATE table weatherdata.late20th
  WITH (
    format='PARQUET', external_location='s3://<yourbucket>/lab3/'
  ) AS SELECT date, type, observation  FROM csv
  WHERE date/10000 between 1950 and 2015;
  ```

56. Choose **Run query**.

57. Preview the query by going to the *late20th* table, choosing the vertical ellipsis (three dots) next to the table, and then choosing **Preview table**.


### Task 2.2: Run a query from the selected data ###

Now that you isolated the data that you are interested in, you can write queries for further analysis. Start by creating a view that only includes the maximum temperature reading, or TMAX, value. To create this view:

58. Copy the following query and in the Athena command window, paste the query:

  ```sql
  CREATE VIEW TMAX AS
  SELECT date, observation, type
  FROM late20th
  WHERE type = 'TMAX'
  ```

59. Choose **Run query**.

60. Preview the data in the view by going to **tmax**, choosing the vertical ellipsis (three dots) next to it, and then choosing **Preview**.

61. Copy the following query and in the Athena command window, paste the query:

  ```sql
  SELECT date/10000 as Year, avg(observation)/10 as Max
  FROM tmax
  GROUP BY date/10000 ORDER BY date/10000;
  ```

62. Choose **Run query**.

You should see a table of data results from 1950–2018, with the average maximum temperature for each year.

## Challenge question

Now that you know how AWS Glue and Athena work together, try the following exercise to test your knowledge. Your lab instructor has a model solution. However, there is more than one way to solve the challenge.

The Global Historical Climatology Network Daily receives data from around the world. You can download data that describes these stations at the following location: <a href="https://noaa-ghcn-pds.s3.amazonaws.com/ghcnd-stations.txt" target="_blank">ghcnd-stations.txt</a>. The data dictionary for the observation and stations data is available at the following location: <a href="https://docs.opendata.aws/noaa-ghcn-pds/readme.html" target="_blank">Readme</a>.

**Note** The ghcnd-stations.txt file is a fixed-width text file. Before you use it with AWS Glue, you must convert it to a comma-separated values (CSV) file, or one of the other file formats that AWS Glue supports. One easy way to convert a text file to a .csv file is to open the text file with a spreadsheet program and then save the file in .csv format. You can also find free utility programs on the internet that can help with this process.


For this challenge, do the following tasks:

- Use AWS Glue to create a table for the weather stations.
- Write a query in Athena to count the number of stations that are not in the US or Canada. The first two characters of the station ID field indicate the country where the station is located. The country codes for the United States is *US* and the country code for Canada is *CA*.


## Lab complete

<i class="icon-flag-checkered"></i> Congratulations! You have completed the lab.

63. At the top of this page, choose <span id="ssb_voc_grey">End Lab</span> and then confirm that you want to end the lab by choosing <span id="ssb_blue">Yes</span>

   A panel will appear, with this message: *DELETE has been initiated... You may close this message box now.*

64. To close the panel, go to the top-right corner and choose the **X**.

