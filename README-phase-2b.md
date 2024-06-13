0. The goal of phase 2b is to perform benchmarking/scalability tests of sample three-tier lakehouse solution.

1. In main.tf, change machine_type at: ✅

   ```
   module "dataproc" {
     depends_on   = [module.vpc]
     source       = "github.com/bdg-tbd/tbd-workshop-1.git?ref=v1.0.36/modules/dataproc"
     project_name = var.project_name
     region       = var.region
     subnet       = module.vpc.subnets[local.notebook_subnet_id].id
     machine_type = "e2-standard-2"
   }
   ```

   and subsititute "e2-standard-2" with "e2-standard-4".

2. If needed request to increase cpu quotas (e.g. to 30 CPUs): ✅
   https://console.cloud.google.com/apis/api/compute.googleapis.com/quotas?project=tbd-2023z-9918

3. Using tbd-tpc-di notebook perform dbt run with different number of executors, i.e., 1, 2, and 5, by changing: ✅
   ```
    "spark.executor.instances": "2"
   ```

   in profiles.yml.

4. In the notebook, collect console output from dbt run, then parse it and retrieve total execution time and execution times of processing each model. Save the results from each number of executors. ✅

5. Analyze the performance and scalability of execution times of each model. Visualize and discucss the final results. ✅

   Total execution time (in minutes):  
   ![all](https://github.com/Dove6/tbd-workshop-1/assets/24943032/5837040a-3bf7-4db3-bde2-5bda5a8433bf)

   Execution time per layer:  
   ![per_layer](https://github.com/Dove6/tbd-workshop-1/assets/24943032/c12fea7d-a05a-4f5e-8aa7-cae6cfd48d95)

   Execution time per model (in seconds, logarithmic scale):  
   ![bronze](https://github.com/Dove6/tbd-workshop-1/assets/24943032/321f5920-f119-42ba-a310-5ad76302c3d4)  
   ![silver](https://github.com/Dove6/tbd-workshop-1/assets/24943032/dcd5cf3d-91cf-410c-908f-347e70f165a9)  
   ![gold](https://github.com/Dove6/tbd-workshop-1/assets/24943032/be64d680-f3bd-4ef6-b518-93ea98d45045)



   The benchmark shows that the best number of executors for the task is 2.  
   It seems the task does not scale well, quickly hitting the point where increasing the number of processing nodes no longer decreases the processing time:
   - increasing the number of executors from 1 to 2 yields significant time improvements,
   - increasing the number of executors from 2 to 5 extends the total time by a few minutes. The task is using up more resources, but the performance does not improve.  
   The performance seems to depend on the number of JOIN clauses used in models. The longest-executing model in the silver layer contains two such clauses and the longest-executing one in the gold layer has four of them.
   The processing of data for 2 executors took about 10x more time than generating them (66 minutes vs. 7 minutes).  
   In comparison to silver and gold layer, filling bronze layer takes the least amount of time as the data undergoes little to no processing.  
