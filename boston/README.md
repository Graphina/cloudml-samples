<h1>Overview</h1>
This code implements a Regression model using the Google Cloud Platform. It includes code to process data, train a TensorFlow model and assess model performance.
This guide trains a neural network model to predict house prices based on different features.

#
* **Data description**

We'll use the
[Boston Housing dataset](https://www.cs.toronto.edu/~delve/data/boston/bostonDetail.html)
This dataset contains information collected by the U.S Census Service concerning
housing in the area of Boston MA. It was obtained from the StatLib archive
(http://lib.stat.cmu.edu/datasets/boston), and has been used extensively
throughout the literature to benchmark algorithms. The dataset
is small in size with only 506 cases.

The dataset contains 13 different features:

1.  Per capita crime rate.
2.  The proportion of residential land zoned for lots over 25,000 square feet.
3.  The proportion of non-retail business acres per town.
4.  Charles River dummy variable (= 1 if tract bounds river; 0 otherwise).
5.  Nitric oxides concentration (parts per 10 million).
6.  The average number of rooms per dwelling.
7.  The proportion of owner-occupied units built before 1940.
8.  Weighted distances to five Boston employment centers.
9.  Index of accessibility to radial highways.
10. Full-value property-tax rate per $10,000.
11. Pupil-teacher ratio by town.
12. 1000 * (Bk - 0.63) ** 2 where Bk is the proportion of African-american people by town.
13. Percentage lower status of the population.

* **Set up and test your GCP environment**

The best way to setup your GCP project is to use this section in this
[tutorial](https://cloud.google.com/ml-engine/docs/tensorflow/getting-started-training-prediction#set-up-your-gcp-project).

* **Environment set-up:**

Virtual environments are strongly suggested, but not required. Installing this
sample's dependencies in a new virtual environment allows you to run the sample
without changing global python packages on your system.

There are two options for the virtual environments:

*   Install [Virtualenv](https://virtualenv.pypa.io/en/stable/) 
    *   Create virtual environment `virtualenv myvirtualenv`
    *   Activate env `source myvirtualenv/bin/activate`
*   Install [Miniconda](https://conda.io/miniconda.html)
    *   Create conda environment `conda create --name myvirtualenv python=2.7`
    *   Activate env `source activate myvirtualenv`

* **Install dependencies**

Install the python dependencies. `pip install --upgrade -r requirements.txt`

#

* **How to satisfy Cloud ML Engine project structure requirements**

The basic project structure will look something like this:

```shell
.
├── README.md
├── requirements.txt
├── setup.py
└── trainer
    ├── __init__.py
    ├── model.py
    └── task.py
```


<h1>Data processing</h1>

The code from the Tensorflow website
[Predict house prices: regression](https://www.tensorflow.org/tutorials/keras/basic_regression)
downloads the Boston data *every time* it is run. The Boston dataset comes
packaged with TensorFlow. Downloading the data is impractical/expensive for
large datasets, so we will get the original files to illustrate a [more general
data preparation process] you might follow in your own projects.

If you want to use local files directly, you can use the following commands:

```shell
mkdir data && cd data
curl -O https://storage.googleapis.com/tensorflow/tf-keras-datasets/boston_housing.npz
cd ..
```
* **Upload the data to a Google Cloud Storage bucket**

Cloud ML Engine works by using resources available in the cloud, so the training
data needs to be placed in such a resource. For this example, we'll use [Google
Cloud Storage], but it's possible to use other resources like [BigQuery]. Make a
bucket (names must be globally unique) and place the data in there:

```shell
gsutil mb gs://your-bucket-name
gsutil cp -r data/boston_housing.npz gs://your-bucket-name/boston_housing.npz
```

<h1>Training</h1>

* **GCloud configuration:**

```
export DATA_FOLDER=data
mkdir $DATA_FOLDER
DATE=`date '+%Y%m%d_%H%M%S'`
export JOB_NAME="boston_keras_$DATE"
export JOB_DIR=boston_$DATE
export TRAIN_FILE=$DATA_FOLDER/boston_housing.npz
export JOB_DIR=/tmp/$JOB_NAME
export REGION=us-central1
rm -rf $JOB_DIR
```

* **Test locally:**

```
python -m trainer.task \
 --train-file=$TRAIN_FILE \
 --job-dir=$JOB_DIR
```

* **Google Cloud ML Engine:**

* **GCloud configuration:**

```
export BUCKET_NAME=your-bucket-name
DATE=`date '+%Y%m%d_%H%M%S'`
export JOB_NAME="boston_keras_$DATE"
export JOB_DIR=gs://$BUCKET_NAME/$JOB_NAME
export REGION=us-central1
export TRAIN_FILE=gs://cloud-samples-data/ml-engine/boston/boston_housing.npz
```
* **Run locally:**

```
gcloud ml-engine local train --module-name=trainer.task \
 --package-path=trainer -- \
 --train-file=$TRAIN_FILE \
 --job-dir=$JOB_DIR
```

*Feel free to modify the destination file for in utils.py

* **Run in Google Cloud ML Engine:**

You can train the model on Cloud ML Engine:

```
gcloud ml-engine jobs submit training $JOB_NAME \
 --stream-logs \
 --runtime-version 1.10 \
 --job-dir=$JOB_DIR \
 --package-path=trainer \
 --module-name trainer.task \
 --region $REGION -- \
 --train-file=$TRAIN_FILE
```

* **Monitor with TensorBoard:**

```
tensorboard --logdir=$JOB_DIR
```

## References

[Tensorflow tutorial](https://www.tensorflow.org/tutorials/keras/basic_regression)
