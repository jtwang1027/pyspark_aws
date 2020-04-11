# PySpark in AWS for predicting small molecule IC50

## Configuring AWS elastic MapReduce for distributed computing
AWS EMR
### Software options

<img width="639" alt="aws-cluster_setup-software" src="https://user-images.githubusercontent.com/46359281/78968102-dfd12280-7ad1-11ea-97b5-6f3d4e8bb542.png">

### Bootstrapping / custom AMIs
In addition to the software listed above, additional software and packges are likely going to be needed. These can be installed while the cluster is being provisioned via boostrapping actions (which can be a series of bash scripts). As another option, you can select/create your own AMIs (Amazon machine images) from The AWS marketplace that already contains operating system & software you need--such as miniconda, rstudio with bioconductor, tensorflow 2+ and pytorch. Running instances of these marketplace AMIs typically costs extra.

From  my own experience, the bootstrapping can add an unusually long amount of time. (Instead of a normal 5-7 min startup, bootstrapping can add 10-20min). While you're testing bootstrapping scripts, I would pay close attention to the logs that are being generated, and remember to supply -yes flags to proceed through installations when needed. If you're unsure the programs are being installed correctly, it's worthwhile to ssh into the master node and install the packages from the command line.

### Using spot instances to save on cost
If amenable to your application, spot instances are great way to save on running these clusters. You can set the max percentage of the original price that you're willing to pay, but it will kick you off with a 1min notice if the resource is needed.
<img width="260" alt="aws-cluster_setup-spot_inst" src="https://user-images.githubusercontent.com/46359281/78968180-f5dee300-7ad1-11ea-8da4-b2c732dd3c26.png">

## Background on IC50 prediction

MoleculeNet [(paper,](https://arxiv.org/abs/1703.00564)[website)](http://moleculenet.ai/) is a benchmark established by Vijay Pande's group at Stanford for molecular machine learning. MoleculeNet curates data from multiple public sources, such as ChEMBL, to establish 4 categories for molecular-based machine learning: quantum mechanics, physical chemistry, biophysics, and physiology. The benchmark showcases the performance of various types of models and chemical featurizations. Here, the BACE dataset will be used, which contains IC50 values for chemical inhibitors of human beta-secretase 1 (BACE-1).

### Featurization and ML in cheminformatics
Rdkit and deepchem are great packages for cheminformtaics and machine learning, and they incorporate functions for featurizing chemical data (such as using extended connectivity fingerprints)as well as generating machine learning models, including graph convolutional networks. Here, we explore an approach for representing molecules as vectors (mol2vec)[(paper)](https://pubs.acs.org/doi/abs/10.1021/acs.jcim.7b00616). Mol2vec is inspired by NLP techniques like word2vec, and learns vector representations of molecular substructures. Here,the pre-trained model was applied to the BACE dataset to see if it could improve IC50 prediction.

## PySpark General workflow

The Pyspark notebook run in AWS EMR is available [(here)](https://github.com/jtwang1027/pyspark_aws/blob/master/aws-pyspark.ipynb). I've also done the analysis using PySpark run in a google colab environment [(here)](https://github.com/jtwang1027/pyspark_aws/blob/master/colab_pyspark_bace.ipynb).

I'll break the PySpark workflow down into a few components.

1) Data ingestion

2) Data preprocessing using PySpark's *Pipeline*

3) Train/test/validation split

4) Model fitting:
- Linear Regression (OLS)
- Elastic Net with hyperparameter tuning
- Gradient boosted trees

5) Visualization  
![gbtree](https://user-images.githubusercontent.com/46359281/78973310-a1d9fb80-7add-11ea-8261-9f8b34b41b41.png)

