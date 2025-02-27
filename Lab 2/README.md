# Lab 2 - Amazon Sagemaker AutoPilot

In this lab, you will get hands-on with Amazon Sagemaker AutoML. An automated machine learning (commonly referred to as AutoML) solution for tabular datasets. You can use SageMaker Autopilot in different ways: on autopilo (hence the name) or with human guidance, without code through SageMaker Studio, or using the AWS SDKs.

Losing customers is costly for any business. Identifying unhappy customers early on gives you a chance to offer them incentives to stay. This lab describes using machine learning (ML) for the automated identification of unhappy customers, also known as customer churn prediction. We will use the same data set as we did in **_Lab 1_** as we are already familiar with it.


In this lab SageMaker Autopilot first inspects your data set, and runs a number of model candidates to figure out the optimal combination of data preprocessing steps, machine learning algorithms and hyperparameters. It then uses this combination to train an Inference Pipeline, which you can easily deploy either on a real-time endpoint, or use it for batch processing.  
  
  
- [_Setup_](https://github.com/lkolodziejek/AmazonSagemakerWorkshop/blob/main/Lab%203/README.md#setup)
- [_Data_](https://github.com/lkolodziejek/AmazonSagemakerWorkshop/blob/main/Lab%203/README.md#data)
- [_Setting up the Amazon Sagemaker Autopilot Job_](https://github.com/lkolodziejek/AmazonSagemakerWorkshop/blob/main/Lab%203/README.md#setting-up-the-amazon-sagemaker-autopilot-job)
- [_Launching the Amazon Sagemaker Autopilot Job_](https://github.com/lkolodziejek/AmazonSagemakerWorkshop/blob/main/Lab%203/README.md#launching-the-amazon-sagemaker-autopilot-job)
- [_Analysis of output artifacts_](https://github.com/lkolodziejek/AmazonSagemakerWorkshop/blob/main/Lab%203/README.md#analysis-of-output-artifacts)
- [_Results_](https://github.com/lkolodziejek/AmazonSagemakerWorkshop/blob/main/Lab%203/README.md#results)
- [_Deploy the best model_](https://github.com/lkolodziejek/AmazonSagemakerWorkshop/blob/main/Lab%203/README.md#deploy-the-best-model)
- [_Inference the model with simple web page_](https://github.com/lkolodziejek/AmazonSagemakerWorkshop/blob/main/Lab%203/README.md#inference-the-model-with-simple-web-page)

**Ok, lest start the lab!!**

# Setup

1. We will start with opening Notebook `autopilot_customer_churn.ipynb` in “AmazonSagemakerWorkshop/Lab 2” folder. We will skip detailed instruction how to procees as you are the expert now.  
  

In case You did not follow this workshop in order, you can find detailed instruction how to complete this step [_here_](https://github.com/lkolodziejek/AmazonSagemakerWorkshop/blob/main/Lab%201/README.md#data-preparation).
  
  
2. In the very first two cells we will start with specifying:  
  
- The S3 bucket and prefix that you want to use for training and model data. Replace the appropriate variable names according to your preferences.  
   
  
![ReplaceMe](https://user-images.githubusercontent.com/36265995/102872824-926aca80-4440-11eb-8998-00b87665ceb3.png)  
  
  
- Next, we'll import the Python libraries we'll need for the remainder of the exercise. Execute the first two cells by pressing `Shift`+`Enter` in each of the cells.  
  
![Import](https://user-images.githubusercontent.com/36265995/102874970-7caad480-4443-11eb-8c62-c6a9de42db1c.png)  
  

# Data

In this section we will dowload the data set. We are going to use the same data set as we did in **_Lab 1_** as we are already familiar with it. Just to briefly recap the dataset we use is publicly available and was mentioned in the book [Discovering Knowledge in Data](https://www.amazon.com/dp/0470908742/) by Daniel T. Larose. It is attributed by the author to the University of California Irvine Repository of Machine Learning Datasets.

1. Now, Let’s download and load the dataset by running the next cell:


![](https://user-images.githubusercontent.com/50714479/112118779-ff673b00-8bbc-11eb-850a-6911d9b26ecf.png)


2. Take a look on data

It is important to keep in mind that it is good practice to check the dataset to make sure that it has no obvious errors. The Autopilot process can take long time to complete, to inspect the dataset before you start a job.   

It shouldn't be a surprise that we end up with the same output as we did in **_Lab 1_**. We will skip the where we analyze the data set. We want Amazon Sagemaker Autopilot to inspect our data set, and runs a number of model candidates to figure out the optimal combination of data preprocessing steps. Still we need to divide the data into training and testing sets.

3. Divide the data into training and testing sets.  
  
  
![Split](https://user-images.githubusercontent.com/36265995/102880525-be3f7d80-444b-11eb-872f-ba86a28060e7.png)
  
  
4. Now we'll upload these files to S3.  
  
  
![SaveToS3](https://user-images.githubusercontent.com/36265995/102887161-b76a3800-4456-11eb-9dd7-89d285657620.png)

# Setting up the Amazon Sagemaker Autopilot Job

After uploading the dataset to Amazon S3, you can invoke Autopilot to find the best ML pipeline to train a model on this dataset.

The required inputs for invoking a Autopilot job are:

- Amazon S3 location for input dataset and for all output artifacts
- Name of the column of the dataset you want to predict (Churn? in this case)
- An IAM role

Currently Autopilot supports only tabular datasets in CSV format. Either all files should have a header row, or the first file of the dataset, when sorted in alphabetical/lexical order by name, is expected to have a header row.

Below you can find Amazon S3 location configuration for input dataset and for all output artifacts:  
  
  
![Config](https://user-images.githubusercontent.com/36265995/102888564-5a23b600-4459-11eb-8c0a-511dde7be70b.png)


Please keep in mind that it is also possible to specify the type of ML problem you want to solve with your dataset (Regression, MulticlassClassification, BinaryClassification). Let's assume that we are not sure. We will allow Amazon Sagemaker Autopilot to infer the problem type based on statistics of the target column (the column we want to predict).

As you remember the target attribute in our data set is: `Churn?`. As it consists of only binary values our model will be performing binary prediction, also known as binary classification.


Always remember that you have the option to limit the running time of a Amazon SageMaker Autopilot job. To achieve such behavior providing either the maximum number of pipeline evaluations or candidates (one pipeline evaluation is called a Candidate because it generates a candidate model) or providing the total time allocated for the overall Autopilot job. Under default settings, this job takes about four hours to run. This varies between runs because of the nature of the exploratory process Autopilot uses to find optimal training parameters.  
  

# Launching the Amazon Sagemaker Autopilot Job

Finally we can now launch the Autopilot job by calling the `create_auto_ml_job` API. We limit the number of candidates to 20 so that the job finishes in a few minutes.

![RunAutoPilotWithOutput](https://user-images.githubusercontent.com/36265995/103557022-e2bb4100-4eb2-11eb-85c8-1718a5c63ff2.png)

The Amazon Sagemaker Autopilot job `automl-churn-22-13-32-04` that we have just started will process through the 3 following steps:
 
- **_Analyzing Data_**, where the dataset is analyzed and Autopilot comes up with a list of ML pipelines that should be tried out on the dataset. The dataset is also split into train and validation sets.
- **_Feature Engineering_**, where Autopilot performs feature transformation on individual features of the dataset as well as at an aggregate level.
- **_Model Tuning_**, where the top performing pipeline is selected along with the optimal hyperparameters for the training algorithm (the last stage of the pipeline). 

With the following code you can monitor the progrss of the Autopilot job:
![Output](https://user-images.githubusercontent.com/36265995/102900651-e390b380-446c-11eb-952b-6d4c0777f834.png)

# Analysis of output artifacts

In this part of the lab we will analyze what the Amazon Sagemaker Autopilot did in the background in each particular pipeline stage:  
- **_Analyzing Data_**  
- **_Feature Engineering_**  
- **_Model Tuning_**  

Let's start with login to AWS Management Console and opening Amazon SageMaker service dashboard.

## “Analyzing Data” pipeline stage

1. Open “_Processing/Processing jobs_” on the left menu. You will find two jobs, that Autopilot has started. First one with prefix "***db-***" and the second one with prefix "***pr-***".  
  
  
If you have more than 2 jobs please refer to “_Creation time_” column to find your jobs. 
  
  
![Preprocess](https://user-images.githubusercontent.com/36265995/102983900-0c688580-450d-11eb-9193-b09282fd7d0a.png)

2. Let's analyse the first processing job with "***db-***" prefix

This is the very first task of Autopilot. In this step, the dataset is verified whether it is suitable for further processing or not. Things like the number of attributes for each row is validated here. Next the dataset is randomly shuffle and split into train and validation sets as well. The artifacts for this step are located in S3 bucket in folder `preprocessed-data` as follow. Keep in mind that in my notebook I've used `customer-churn-autopilot` name for the bucket and `sagemaker/autopilot-churn` as a prefix:
  
  
![db-1](https://user-images.githubusercontent.com/36265995/102987307-6ae43280-4512-11eb-8458-30242ffa3f04.png)


3. Now, let's analyse the second processing job with "***pr***" prefix

In this step Autopilot is doing a huge amount of work for us. So, let's deep dive and figure out what exactly was done here.

At the beginning Autopilot will analyze the data set. As an outcome a ***Amazon SageMaker Autopilot Data Exploration*** report will be generated in form of notebook. You can download `SageMakerAutopilotCandidateDefinitionNotebook.ipynb`here:
> customer-churn-autopilot/sagemaker/autopilot-churn/output/automl-churn-22-13-32-04/sagemaker-automl-candidates/pr-1-413c3523dc0542d6aa408352dc9da401c7df3ed39ad04a8bbf9d50a388/notebooks/

From the raport you can read for example:
> We read **`2666`** rows from the training dataset.  
> The dataset has **`21`** columns and the column named **`Churn?`** is used as the target column.  
> This is identified as a **`BinaryClassification`** problem.  
> Here are **2** examples of labels: `['True.', 'False.']`.  

We will skip the rest of the report as we are the experts now. After the ***Lab 1*** we know everything about this dataset.

When Amazon Sagemaker Autopilot analyzed the dataset it generated **10** machine learning pipeline(s) that use **3** algorithm(s). Each pipeline contains a set of feature transformers and an algorithm. The artifacts for this step are located in S3 bucket in folder `sagemaker-automl-candidates` as follow:  


![pr-1](https://user-images.githubusercontent.com/36265995/102991475-8ef74200-4519-11eb-9d36-6acabc6fabc4.png)

Take a look at what was generated inside this folder. Next we will look at trainable data transformer Python modules like `dpp0.py` and `dpp2.py` in `sagemaker-automl-candidates/pr-1-413c3523dc0542d6aa408352dc9da401c7df3ed39ad04a8bbf9d50a388/generated_module/candidate_data_processors/` folder. 

`dpp0.py`: This data transformation strategy first transforms 'numeric1' features using ***RobustImputer*** (converts missing values to nan), 'categorical1' features using ***ThresholdOneHotEncoder***, 'text1' features using ***MultiColumnTfidfVectorizer***. It merges all the generated features and applies ***RobustStandardScaler***. The transformed data will be used to tune a ***xgboost*** model.

`dpp2.py`: This data transformation strategy first transforms 'numeric1' features using combined ***RobustImputer*** and ***RobustMissingIndicator*** followed by ***QuantileExtremeValuesTransformer***, 'categorical1' features using ***ThresholdOneHotEncoder***, 'text1' features using ***MultiColumnTfidfVectorizer***. It merges all the generated features and applies ***RobustPCA*** followed by ***RobustStandardScaler***. The transformed data will be used to tune a ***linear-learner*** model.

Amazon Sagemaker Autopilot decided that all this **10** data transformers will fit the best for this partucular problem.

**!! Autopilot generated one more very useful notebook that will guide you through the pipelines/candidates/modules that were created at this step**. You can download `SageMakerAutopilotDataExplorationNotebook.ipynb` here:  
> customer-churn-autopilot/sagemaker/autopilot-churn/output/automl-churn-22-13-32-04/sagemaker-automl-candidates/pr-1-413c3523dc0542d6aa408352dc9da401c7df3ed39ad04a8bbf9d50a388/notebooks/

## “Feature Engineering” pipeline stage  

The feature engineering pipeline consists of two SageMaker jobs. At this point Autopilot has already generated trainable data transformer Python modules like `dpp0.py`. The mentioned set of jobs are:

- A **training** job to train the data transformers.
- A **batch transform** job to apply the trained transformation to the dataset to generate the algorithm compatible data. We will use it in **_Model Tuning_** pipeline stage later on.

1. Open “_Training/Training jobs_” on the left menu. You will find 10 jobs, that Autopilot has started in order to train data transformers.

As you can see all of the training jobs have the `automl-chu-dpp` prefix:  
  
  
![TransformTrainer](https://user-images.githubusercontent.com/36265995/103009307-e5bf4480-4536-11eb-930f-9d86700ca932.png)
  
The trained data transformers model were saved in s3 bucker foler `data-processor-models` as follow:

![DataTransformerModel](https://user-images.githubusercontent.com/36265995/103010618-df31cc80-4538-11eb-9493-b9fc20ca54d5.png)
  
2. Now open “_Inference/Batch transform jobs_” on the left menu. You will find 10 batch jobs, that Autopilot has started in order to transform dataset using models trained in previous step.

As you can see all of the batch jobs have the `automl-chu-dpp` prefix:  

![BatchTransform](https://user-images.githubusercontent.com/36265995/103010840-39cb2880-4539-11eb-9222-6e17123e4c11.png)

The transformed data was saved in s3 bucker foler `transformed-data` as follow:

![transformed-data](https://user-images.githubusercontent.com/36265995/103011444-38e6c680-453a-11eb-934b-29a0824d7262.png)


## "Model Tuning" pipeline stage  

  
After each featurized dataset is prepared (“***Feature Engineering***” pipeline stage ), Autopilot executes a multi-algorithm tuning job that will run tuning jobs in parallel for each pipeline. This tuning job will execute training jobs to find the best set of hyper-parameters for each pipeline, as well as finding the overall best performing pipeline.  
  
Algorithm training job configuration for each algorithm is auto-generated by the Autopilot as a part of the recommendation. Here are the parameters that were defined for this dataset:

![HPOparameters](https://user-images.githubusercontent.com/36265995/103017354-c67ae400-4543-11eb-8229-3c1fd6a01f2b.png)


1.  Open “_Training/Hyperparameter tuning jobs_” on the left menu. You will find HPO 1 jobs, that Autopilot has started in order to tune ale the pipelines.  

This HPO job started 20 training jobs in 2 batches (10 jobs per batch). All training jobs within batch will run in paralell. The maximum number of 20 training jobs was configured by us when we launched the Autopilot job by calling the `create_auto_ml_job` API. 10 training per batch it is a default configuration:

![HPOjob](https://user-images.githubusercontent.com/36265995/103017031-4785ab80-4543-11eb-9be4-a4685116271b.png)

2. Open “_Training/Training jobs_” on the left menu. Try to identify jobs started by HPO process.


***Congratulations!!*** You have successfully nalyze Amazon Sagemaker Autopilot artifacts that were generated in the background in each particular pipeline stage.

# Results

Finally, we will find out which model turned out to be the best one. We will use `describe_auto_ml_job` API to look up the best candidate selected by the SageMaker Autopilot job:

![Results](https://user-images.githubusercontent.com/36265995/103529320-bf2dd180-4e85-11eb-88c8-08bac30e4206.png)

The best candidate name is: `tuning-job-1-68b0d889fb234badb4-012-ef007be0` with validation metric: 'F1' and its value: `0.891319990158081`.
Due to some randomness in the algorithms you might received slightly different results. 

# Deploy the best model

Ok. We've trained the algorithm, so now let's create a model and deploy it to a hosted endpoint. I will use ml.m4.xlarge to host the model.

![Deploy-m4](https://user-images.githubusercontent.com/36265995/103531863-73c9f200-4e8a-11eb-96ce-09dc0e6b379f.png)

You can verify if your model is hosted by Amazon Sagemaker. Please go to Amazon Sagemaker service dashboard and in _"Inference/Endpoints"_ check wather the endpoint is in `InService` status.

![endpoint](https://user-images.githubusercontent.com/36265995/103532533-b0e2b400-4e8b-11eb-83d4-297d67840f7b.png)

# Inference the model with simple web page

Now it is the time to test our model. We have already deployed it. So, in this step, you will inference the trained model throught simple web page that you will create. We have already did it in Lab 1, that is why pleass refer to detailed instruction how to do this [_here_](https://github.com/lkolodziejek/AmazonSagemakerWorkshop/blob/main/Lab%201/README.md#inference-the-model-with-simple-web-page). Please use:

- `RoleForInvokingTheModel` as a execution role for lambda function that will inference the model;
- `autopilot-customerchurn-ep-invoker-lambda.py` as a lambda function code that will inference the model;
- `autopilot-customerchurn-website.html` as a html code for www page (interface to the model);

Additionally to steps done in Lab1, increase Lambda Execution Timeout to at least 10 seconds.

In _"Configuration"_ navigate to _"General configuration"_. Click _"Edit"_.

![LambdaTimeoutIncrease](https://user-images.githubusercontent.com/50714479/112128383-ab615400-8bc6-11eb-8d57-939597ae8e7c.png)

Finally, you should be able to access your webpage as described in Lab1, from your s3 bucket.

This should provide following result:

![WWW](https://user-images.githubusercontent.com/36265995/103540324-d37bc980-4e99-11eb-8055-3664e3183024.png)

***Congratulations!!*** You have successfully completed this lab. Now you can go to Lab 3 where you will retrain the model. Please write down the following values as we will need them to complete Lab 3:
-  ___Endpoint name___: In my case it is: `tuning-job-1-68b0d889fb234badb4-012-ef007be004-11-32-13-ep`. Plese refer to [_Deploy the best model_](https://github.com/lkolodziejek/AmazonSagemakerWorkshop/blob/main/Lab%203/README.md#deploy-the-best-model) to find your value.
-  ___AutoMLJobName___: In my case it is: `automl-churn-22-13-32-04`. Plese refer to [_Launching the Amazon Sagemaker Autopilot Job_](https://github.com/lkolodziejek/AmazonSagemakerWorkshop/blob/main/Lab%203/README.md#launching-the-amazon-sagemaker-autopilot-job) to find your value.

___!!! Please don't proceed with Cleanup section of Jupyter's Notebooks of this Lab, unless you have completed or decided to skip Lab3 !!!___