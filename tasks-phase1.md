IMPORTANT ❗ ❗ ❗ Please remember to destroy all the resources after each work session. You can recreate infrastructure by creating new PR and merging it to master.
  
![img.png](doc/figures/destroy.png)

1. Authors: ✅

   **Group no. 1**

   **The forked repo is available here: https://github.com/Dove6/tbd-workshop-1**
   
3. Follow all steps in README.md. ✅

4. Select your project and set budget alerts on 5%, 25%, 50%, 80% of 50$ (in cloud console -> billing -> budget & alerts -> create buget; unclick discounts and promotions&others while creating budget). ✅

  ![img.png](doc/figures/discounts.png)

5. From avaialble Github Actions select and run destroy on main branch. ✅
   
7. Create new git branch and: ✅
    1. Modify tasks-phase1.md file. ✅
    
    2. Create PR from this branch to **YOUR** master and merge it to make new release. ✅

       **Merged PR: https://github.com/Dove6/tbd-workshop-1/pull/3**

       ![image](https://github.com/Dove6/tbd-workshop-1/assets/24943032/2b30ab02-2458-44f1-b706-8b55daa83508)

8. Analyze terraform code. Play with terraform plan, terraform graph to investigate different modules. 🔄

    ***describe one selected module and put the output of terraform graph for this module here***
   
9. Reach YARN UI ✅
   
   ***The command used for setting up the tunnel:***
   ```sh
   gcloud compute --project "tbd-2024l-304108" ssh --zone "europe-west1-b" "tbd-cluster-m" -- -L 8088:localhost:8088
   ```

   ***The port: 8088*** (according to [Google](https://cloud.google.com/dataproc/docs/concepts/accessing/cluster-web-interfaces#available_interfaces_))

   ***The screenshot of YARN UI:***  
   ![image](https://github.com/Dove6/tbd-workshop-1/assets/24943032/27dda782-ece0-411a-96c4-6b3b5fe8e867)

   ***Additional notes:***
   UI of particular applications can be accessed after additionally tunneling port 18080 (`-L 18080:localhost:18080`) and putting the following entry in the `hosts` file:  
   `127.0.0.1 tbd-cluster-m.c.tbd-2024l-304108.internal. tbd-cluster-m`
   
10. Draw an architecture diagram (e.g. in draw.io) that includes: ✅
    1. VPC topology with service assignment to subnets ✅

    ![network](https://github.com/Dove6/tbd-workshop-1/assets/13040204/158bbfba-9de0-425d-8451-9eb0b38ffef5)

    ![firewall](https://github.com/Dove6/tbd-workshop-1/assets/13040204/508803b6-28f1-43a2-bbc9-230c6ba637a3)

    2. Description of the components of service accounts ✅

    ![accounts](https://github.com/Dove6/tbd-workshop-1/assets/13040204/cd08455c-3247-4943-bfdb-85d14e98f101)

    3. List of buckets for disposal ✅

    ![buckets](https://github.com/Dove6/tbd-workshop-1/assets/13040204/2da1026b-67ac-446b-97f5-6688cc6bb214)

    4. Description of network communication (ports, why it is necessary to specify the host for the driver) of Apache Spark running from Vertex AI Workbech ✅

    The host for the Apache Driver is necessary because it's the command center of Apache Spark application and all other parts of the architecture need to communicate with it.
    The driver address is `10.11.0.3:30000`, which belongs to the previously mentioned `composer_subnet`.
    [The address is currently hardcoded](https://github.com/mwiewior/tbd-tpc-di/blob/892927064ab9246340f8b22910081372cd2520b9/profiles.yml#L15).
    The block manager port is `30001`.
    The same ports are statically assigned to the host node using a Kubernetes `NodePort`.
    Apache Spark Web UI is available at port `18080`.

11. Create a new PR and add costs by entering the expected consumption into Infracost ✅  
For all the resources of type: `google_artifact_registry`, `google_storage_bucket`, `google_service_networking_connection`
create a sample usage profiles and add it to the Infracost task in CI/CD pipeline. Usage file [example](https://github.com/infracost/infracost/blob/master/infracost-usage-example.yml)

    <details>
    <summary><b>The expected consumption entered:</b></summary>

    ```yml
    resource_usage:
    module.gcr.google_artifact_registry_repository.registry:
        storage_gb: 150                  # Total data stored in the repository in GB

    google_service_networking_connection.private_vpc_connection:
        monthly_egress_data_transfer_gb: # Monthly VM-VM data transfer from VPN gateway to the following, in GB:
        same_region: 250
        europe: 70
        worldwide: 200

    module.data-pipelines.google_storage_bucket.tbd-code-bucket:
        storage_gb: 50                    # Total size of bucket in GB.
        monthly_class_a_operations: 10000 # Monthly number of class A operations (object adds, bucket/object list).
        monthly_class_b_operations: 20000 # Monthly number of class B operations (object gets, retrieve bucket/object metadata).
        monthly_egress_data_transfer_gb:  # Monthly data transfer from Cloud Storage to the following, in GB:
        same_continent: 150
        worldwide: 50
        asia: 1
        china: 1
        australia: 1

    module.data-pipelines.google_storage_bucket.tbd-data-bucket:
        storage_gb: 150                   # Total size of bucket in GB.
        monthly_class_a_operations: 40000 # Monthly number of class A operations (object adds, bucket/object list).
        monthly_class_b_operations: 20000 # Monthly number of class B operations (object gets, retrieve bucket/object metadata).
        monthly_egress_data_transfer_gb:  # Monthly data transfer from Cloud Storage to the following, in GB:
        same_continent: 450
        worldwide: 150
        asia: 1
        china: 1
        australia: 1

    google_storage_bucket.tbd-state-bucket:
        storage_gb: 25                    # Total size of bucket in GB.
        monthly_class_a_operations: 20000 # Monthly number of class A operations (object adds, bucket/object list).
        monthly_class_b_operations: 40000 # Monthly number of class B operations (object gets, retrieve bucket/object metadata).
        monthly_egress_data_transfer_gb:  # Monthly data transfer from Cloud Storage to the following, in GB:
        same_continent: 75
        worldwide: 25
        asia: 1
        china: 1
        australia: 1

    module.vertex_ai_workbench.google_storage_bucket.notebook-conf-bucket:
        storage_gb: 50                    # Total size of bucket in GB.
        monthly_class_a_operations: 20000 # Monthly number of class A operations (object adds, bucket/object list).
        monthly_class_b_operations: 10000 # Monthly number of class B operations (object gets, retrieve bucket/object metadata).
        monthly_egress_data_transfer_gb:  # Monthly data transfer from Cloud Storage to the following, in GB:
    ```
    </details>

    **The screenshots of `infracost` output:**  
    ![image](https://github.com/Dove6/tbd-workshop-1/assets/24943032/c8b8b8e0-f7d7-4d04-a467-f088d5e65f57)  
    ![image](https://github.com/Dove6/tbd-workshop-1/assets/24943032/61c80390-d4bc-41c3-8725-1ebf63ceb390)

11. Create a BigQuery dataset and an external table using SQL ✅
    
    **The code (based on README.md):**
    ```sql
    CREATE SCHEMA IF NOT EXISTS demo OPTIONS(location = 'europe-west1');
    CREATE OR REPLACE EXTERNAL TABLE demo.shakespeare
      OPTIONS (
      format = 'ORC',
      uris = ['gs://tbd-2024l-304108-data/data/shakespeare/*.orc']);
    SELECT * FROM demo.shakespeare ORDER BY sum_word_count DESC LIMIT 5;
    ```

    **Obtained output:**
    ![image](https://github.com/Dove6/tbd-workshop-1/assets/24943032/66abbd72-f6c0-4fc5-a31d-717a13a9573a)
   
    **Why does ORC not require a table schema?**  
    There is no need to define a table schema using SQL as OCR stores its own schema definition at the end of each file. It is "a self-describing type-aware columnar file format".
  
12. Start an interactive session from Vertex AI workbench: ✅

    ***The screenshot of notebook:***  
    ![image](https://github.com/Dove6/tbd-workshop-1/assets/24943032/a384b220-5e9e-426a-a0cb-74f51ea3d5c4)
   
13. Find and correct the error in spark-job.py ✅

    Modified files:
    - [modules/data-pipeline/resources/spark-job.py](https://github.com/Dove6/tbd-workshop-1/pull/13/files#diff-8bc902d147d96130dd3c0f57b09f9e9cc2e431438a9f6f85189ed998752d6b3e),
    - [modules/data-pipeline/main.tf](https://github.com/Dove6/tbd-workshop-1/pull/13/files#diff-44c683c0db0603206df91e5810276d8a4e641908e3659f4084270b73e331d0d1).

    The error was located in line 21 of the file. The URL of data bucket had been hardcoded, so it wouldn't work when the script is executed as a part of another project. The issue has been fixed by referencing the variable containing the current name of data bucket:  
    ```diff
    - DATA_BUCKET = "gs://tbd-2024l-9910-data/data/shakespeare/"
    + DATA_BUCKET = "gs://${data_bucket_name}/data/shakespeare/"
    ```  
    and filling it in using `templatefile` Terraform function.

    The error was reported by Dataproc. Information about the error could be found in the "Status" column in the Jobs view in Dataproc GCP module:  
    ![image](https://github.com/Dove6/tbd-workshop-1/assets/24943032/2b4fe322-b51d-4c60-a394-b860f3d54293)  
    A detailed stacktrace was present in the output of each job:  
    ![image](https://github.com/Dove6/tbd-workshop-1/assets/24943032/51f00771-146a-4548-96d8-c027915c742a)

14. Additional tasks using Terraform: 🔄

    1. Add support for arbitrary machine types and worker nodes for a Dataproc cluster and JupyterLab instance 🔄

    ***place the link to the modified file and inserted terraform code***
    
    3. Add support for preemptible/spot instances in a Dataproc cluster 🔄

    ***place the link to the modified file and inserted terraform code***
    
    3. Perform additional hardening of Jupyterlab environment, i.e. disable sudo access and enable secure boot 🔄
    
    ***place the link to the modified file and inserted terraform code***

    4. (Optional) Get access to Apache Spark WebUI

    ***place the link to the modified file and inserted terraform code***
