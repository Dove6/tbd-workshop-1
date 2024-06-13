IMPORTANT ❗ ❗ ❗ Please remember to destroy all the resources after each work session. You can recreate infrastructure by creating new PR and merging it to master.

![img.png](doc/figures/destroy.png)

0. The goal of this phase is to create infrastructure, perform benchmarking/scalability tests of sample three-tier lakehouse solution and analyze the results using:
* [TPC-DI benchmark](https://www.tpc.org/tpcdi/)
* [dbt - data transformation tool](https://www.getdbt.com/)
* [GCP Composer - managed Apache Airflow](https://cloud.google.com/composer?hl=pl)
* [GCP Dataproc - managed Apache Spark](https://spark.apache.org/)
* [GCP Vertex AI Workbench - managed JupyterLab](https://cloud.google.com/vertex-ai-notebooks?hl=pl)

Worth to read:
* https://docs.getdbt.com/docs/introduction
* https://airflow.apache.org/docs/apache-airflow/stable/index.html
* https://spark.apache.org/docs/latest/api/python/index.html
* https://medium.com/snowflake/loading-the-tpc-di-benchmark-dataset-into-snowflake-96011e2c26cf
* https://www.databricks.com/blog/2023/04/14/how-we-performed-etl-one-billion-records-under-1-delta-live-tables.html

2. Authors: ✅

   **Group no. 1**

   **The forked repo is available here: https://github.com/Dove6/tbd-workshop-1**

3. Sync your repo with https://github.com/bdg-tbd/tbd-workshop-1. ✅

4. Provision your infrastructure. ✅

    a) setup Vertex AI Workbench `pyspark` kernel as described in point [8](https://github.com/Dove6/tbd-workshop-1/tree/master#project-setup) ✅

    b) upload [tpc-di-setup.ipynb](https://github.com/Dove6/tbd-workshop-1/blob/master/notebooks/tpc-di-setup.ipynb) to 
the running instance of your Vertex AI Workbench ✅

5. In `tpc-di-setup.ipynb` modify cell under section ***Clone tbd-tpc-di repo***: ✅

   **The forked repo is available here: https://github.com/Dove6/tbd-tpc-di**

   a) first, fork https://github.com/mwiewior/tbd-tpc-di.git to your github organization. ✅

   b) create new branch (e.g. 'notebook') in your fork of tbd-tpc-di and modify profiles.yaml by commenting following lines: ✅
   ```  
        #"spark.driver.port": "30000"
        #"spark.blockManager.port": "30001"
        #"spark.driver.host": "10.11.0.5"  #FIXME: Result of the command (kubectl get nodes -o json |  jq -r '.items[0].status.addresses[0].address')
        #"spark.driver.bindAddress": "0.0.0.0"
   ```
   This lines are required to run dbt on airflow but have to be commented while running dbt in notebook.

   c) update git clone command to point to ***your fork***. ✅

6. Access Vertex AI Workbench and run cell by cell notebook `tpc-di-setup.ipynb`. ✅

    a) in the first cell of the notebook replace: `%env DATA_BUCKET=tbd-2023z-9910-data` with your data bucket. ✅

    b) in the cell: ✅
         ```%%bash
         mkdir -p git && cd git
         git clone https://github.com/mwiewior/tbd-tpc-di.git
         cd tbd-tpc-di
         git pull
         ```
      replace repo with your fork. Next checkout to 'notebook' branch.
   
    c) after running first cells your fork of `tbd-tpc-di` repository will be cloned into Vertex AI  enviroment (see git folder). ✅

    d) take a look on `git/tbd-tpc-di/profiles.yaml`. This file includes Spark parameters that can be changed if you need to increase the number of executors and ✅
        ```
         server_side_parameters:
             "spark.driver.memory": "2g"
             "spark.executor.memory": "4g"
             "spark.executor.instances": "2"
             "spark.hadoop.hive.metastore.warehouse.dir": "hdfs:///user/hive/warehouse/"
        ```


7. Explore files created by generator and describe them, including format, content, total size. ✅

   The generator outputs files representing data from a fictional On-Line Transaction Processing system in the form of three batches: historical load and two incremental updates.

   **Format**:
   - the data:
     - mainly CSV files without a header,
       - with `,` as a delimiter (.csv files),
       - with `|` as a delimiter (.txt files),
       - fixed-width lines (files without an extension),
     - a single XML file without a schema,
   - audit data: CSV files with `,` as a delimiter,
   - generation metadata:
     - a TXT file summarizing the number of generated records (digen_report.txt),
     - BatchDate.txt (one in each of Batch1, Batch2, Batch3 dirs) with the date when the batch was created.

   **Content**: retail brokerage data including
   - data from a financial newswire split into multiple FINWIRE files (for each quarter) in Batch1,
   - customer and account information (including prospective customers).

   **Total size**:  
   - 9.6 GiB,
   - 162228471 rows (160873381 in Batch1, 677582 in Batch2, 677508 in Batch3).

8. Analyze tpcdi.py. What happened in the loading stage? ✅

   The loading stage loads files of various formats: XML, CSV, TXT (as CSV).  
   The data is first saved in GCS buckets, then loaded as Spark Dataframes from that bucket.  
   At last, the dataframes are either displayed or written to a table, controlled by one of the script parameters.
    
   Two kinds of files are loaded: FINWIRE and everything else.  
   FINWIRE files are spread between many files, so they are gathered using `glob`.  
   They all use the same file format, all lines are constant width.  
   `rec_type` column dictates the schema of each record, there are 3 kinds: CMP, SEC, FIN.
    
   Other files are processed one by one, each using a different loading method depending on their format.  
   CSV and TXT files are processed as CSV and need to be augmented with a schema for the data.  
   The XML file is processed differently, the schema is not provided and we use a `select` to get only a subset of available data.

9. Using SparkSQL answer: how many table were created in each layer? ✅

   ```python
   print(f'layer       | table count')
   dbs = spark.sql('show databases').collect()
   for db in dbs:
       layer = db.namespace
       spark.sql(f'use {layer}')
       count = spark.sql(f'show tables').count()
       print(f'{layer:11} | {count}')
   ```

   Output:
   ```
   layer       | table count
   -------------------------
   bronze      | 0
   default     | 0
   demo_bronze | 17
   demo_gold   | 12
   demo_silver | 14
   digen       | 17
   gold        | 0
   silver      | 0
   ```

   We are interested only in databases with the `demo_` prefix. To sum up, there are:
   - 17 tables in the bronze layer,
   - 14 tables in the silver layer,
   - 12 tables in the gold layer.

10. Add some 3 more [dbt tests](https://docs.getdbt.com/docs/build/tests) and explain what you are testing. ***Add new tests to your repository.*** ✅

    A test for ensuring there is no negative bid price:
    ```sql
    select
        sk_trade_id,
        bid_price
    from {{ ref('fact_trade') }} 
    where bid_price < 0
    ```

    A test for ensuring the correct order of placing and removing a watch on a customer:
    ```sql
    select *
    from {{ ref('fact_watches') }} 
    where sk_date_removed < sk_date_placed
    ```

    sprawdzenie czy daty są w dobrej relacji

    A test for ensuring every company has a CEO:
    ```sql
    select 
        sk_company_id,
        name,
        ceo
    from {{ ref('dim_company') }}
    where ceo is null
    ```

    The results of running the tests:  
    ![image](https://github.com/Dove6/tbd-workshop-1/assets/24943032/43aa67e7-80a1-4d3d-9f83-ac75cd2c49cb)

11. In main.tf update 🔄
    ```
    dbt_git_repo            = "https://github.com/mwiewior/tbd-tpc-di.git"
    dbt_git_repo_branch     = "main"
    ```
    so dbt_git_repo points to your fork of tbd-tpc-di. 

12. Redeploy infrastructure and check if the DAG finished with no errors: ✅

    There are no new errors after re-deploying the infrastructure:  
    ![image](https://github.com/Dove6/tbd-workshop-1/assets/24943032/61e8b7fd-f3f8-47d2-9ecb-eee0abef62bf)
