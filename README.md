# Object Detection in an Urban Environment

## Data

For this project, we will be using data from the [Waymo Open dataset](https://waymo.com/open/).

[OPTIONAL] - The files can be downloaded directly from the website as tar files or from the [Google Cloud Bucket](https://console.cloud.google.com/storage/browser/waymo_open_dataset_v_1_2_0_individual_files/) as individual tf records. We have already provided the data required to finish this project in the workspace, so you don't need to download it separately.

## Structure

### Data

The data you will use for training, validation and testing is organized as follow:
```
/home/workspace/data/waymo
	- training_and_validation - contains 97 files to train and validate your models
    - train: contain the train data (empty to start)
    - val: contain the val data (empty to start)
    - test - contains 3 files to test your model and create inference videos
```
The `training_and_validation` folder contains file that have been downsampled: we have selected one every 10 frames from 10 fps videos. The `testing` folder contains frames from the 10 fps video without downsampling.
```
You will split this `training_and_validation` data into `train`, and `val` sets by completing and executing the `create_splits.py` file.


### Experiments
The experiments folder will be organized as follow:
```
experiments/
    - pretrained_model/
    - exporter_main_v2.py - to create an inference model
    - model_main_tf2.py - to launch training
    - reference/ - reference training with the unchanged config file
    - experiment0/ - create a new folder for each experiment you run
    - experiment1/ - create a new folder for each experiment you run
    - experiment2/ - create a new folder for each experiment you run
    - label_map.pbtxt
    ...
```

## Prerequisites

### Local Setup

For local setup if you have your own Nvidia GPU, you can use the provided Dockerfile and requirements in the [build directory](./build).

Follow [the README therein](./build/README.md) to create a docker container and install all prerequisites.

### Download and process the data

**Note:** ”If you are using the classroom workspace, we have already completed the steps in the section for you. You can find the downloaded and processed files within the `/home/workspace/data/preprocessed_data/` directory. Check this out then proceed to the **Exploratory Data Analysis** part.

The first goal of this project is to download the data from the Waymo's Google Cloud bucket to your local machine. For this project, we only need a subset of the data provided (for example, we do not need to use the Lidar data). Therefore, we are going to download and trim immediately each file. In `download_process.py`, you can view the `create_tf_example` function, which will perform this processing. This function takes the components of a Waymo Tf record and saves them in the Tf Object Detection api format. An example of such function is described [here](https://tensorflow-object-detection-api-tutorial.readthedocs.io/en/latest/training.html#create-tensorflow-records). We are already providing the `label_map.pbtxt` file.

You can run the script using the following command:
```
python download_process.py --data_dir {processed_file_location} --size {number of files you want to download}
```

You are downloading 100 files (unless you changed the `size` parameter) so be patient! Once the script is done, you can look inside your `data_dir` folder to see if the files have been downloaded and processed correctly.

### Classroom Workspace

In the classroom workspace, every library and package should already be installed in your environment. You will NOT need to make use of `gcloud` to download the images.

## Instructions

### Exploratory Data Analysis

You should use the data already present in `/home/workspace/data/waymo` directory to explore the dataset! This is the most important task of any machine learning project. To do so, open the `Exploratory Data Analysis` notebook. In this notebook, your first task will be to implement a `display_instances` function to display images and annotations using `matplotlib`. This should be very similar to the function you created during the course. Once you are done, feel free to spend more time exploring the data and report your findings. Report anything relevant about the dataset in the writeup.

Keep in mind that you should refer to this analysis to create the different spits (training, testing and validation).


### Create the training - validation splits
In the class, we talked about cross-validation and the importance of creating meaningful training and validation splits. For this project, you will have to create your own training and validation sets using the files located in `/home/workspace/data/waymo`. The `split` function in the `create_splits.py` file does the following:
* create three subfolders: `/home/workspace/data/train/`, `/home/workspace/data/val/`, and `/home/workspace/data/test/`
* split the tf records files between these three folders by symbolically linking the files from `/home/workspace/data/waymo/` to `/home/workspace/data/train/`, `/home/workspace/data/val/`, and `/home/workspace/data/test/`

Use the following command to run the script once your function is implemented:
```
python create_splits.py --data-dir /home/workspace/data
```

### Edit the config file

Now you are ready for training. As we explain during the course, the Tf Object Detection API relies on **config files**. The config that we will use for this project is `pipeline.config`, which is the config for a SSD Resnet 50 640x640 model. You can learn more about the Single Shot Detector [here](https://arxiv.org/pdf/1512.02325.pdf).

First, let's download the [pretrained model](http://download.tensorflow.org/models/object_detection/tf2/20200711/ssd_resnet50_v1_fpn_640x640_coco17_tpu-8.tar.gz) and move it to `/home/workspace/experiments/pretrained_model/`.

We need to edit the config files to change the location of the training and validation files, as well as the location of the label_map file, pretrained weights. We also need to adjust the batch size. To do so, run the following:
```
python edit_config.py --train_dir /home/workspace/data/train/ --eval_dir /home/workspace/data/val/ --batch_size 2 --checkpoint /home/workspace/experiments/pretrained_model/ssd_resnet50_v1_fpn_640x640_coco17_tpu-8/checkpoint/ckpt-0 --label_map /home/workspace/experiments/label_map.pbtxt
```
A new config file has been created, `pipeline_new.config`.

### Training

You will now launch your very first experiment with the Tensorflow object detection API. Move the `pipeline_new.config` to the `/home/workspace/experiments/reference` folder. Now launch the training process:
* a training process:
```
python experiments/model_main_tf2.py --model_dir=experiments/reference/ --pipeline_config_path=experiments/reference/pipeline_new.config
```
Once the training is finished, launch the evaluation process:
* an evaluation process:
```
python experiments/model_main_tf2.py --model_dir=experiments/reference/ --pipeline_config_path=experiments/reference/pipeline_new.config --checkpoint_dir=experiments/reference/
```

**Note**: Both processes will display some Tensorflow warnings, which can be ignored. You may have to kill the evaluation script manually using
`CTRL+C`.

To monitor the training, you can launch a tensorboard instance by running `python -m tensorboard.main --logdir experiments/reference/`. You will report your findings in the writeup.

### Improve the performances

Most likely, this initial experiment did not yield optimal results. However, you can make multiple changes to the config file to improve this model. One obvious change consists in improving the data augmentation strategy. The [`preprocessor.proto`](https://github.com/tensorflow/models/blob/master/research/object_detection/protos/preprocessor.proto) file contains the different data augmentation method available in the Tf Object Detection API. To help you visualize these augmentations, we are providing a notebook: `Explore augmentations.ipynb`. Using this notebook, try different data augmentation combinations and select the one you think is optimal for our dataset. Justify your choices in the writeup.

Keep in mind that the following are also available:
* experiment with the optimizer: type of optimizer, learning rate, scheduler etc
* experiment with the architecture. The Tf Object Detection API [model zoo](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/tf2_detection_zoo.md) offers many architectures. Keep in mind that the `pipeline.config` file is unique for each architecture and you will have to edit it.

**Important:** If you are working on the workspace, your storage is limited. You may to delete the checkpoints files after each experiment. You should however keep the `tf.events` files located in the `train` and `eval` folder of your experiments. You can also keep the `saved_model` folder to create your videos.


### Creating an animation
#### Export the trained model
Modify the arguments of the following function to adjust it to your models:

```
python experiments/exporter_main_v2.py --input_type image_tensor --pipeline_config_path experiments/reference/pipeline_new.config --trained_checkpoint_dir experiments/reference/ --output_directory experiments/reference/exported/
```

This should create a new folder `experiments/reference/exported/saved_model`. You can read more about the Tensorflow SavedModel format [here](https://www.tensorflow.org/guide/saved_model).

Finally, you can create a video of your model's inferences for any tf record file. To do so, run the following command (modify it to your files):
```
python inference_video.py --labelmap_path label_map.pbtxt --model_path experiments/reference/exported/saved_model --tf_record_path /data/waymo/testing/segment-12200383401366682847_2552_140_2572_140_with_camera_labels.tfrecord --config_path experiments/reference/pipeline_new.config --output_path animation.gif

### Project overview
This is Udacity Self Driving Nanodegree Object Detection project, which uses Tensorflow to create ML model to detect objects Cars, Pedestrians and Cyclist in Urban Environment. A sample Waymo dataset contains tfrecords file which will then be modified and split into training, validation and testing sets using "create_splits.py"
Waymo dataset can be downloaded from this link:https://console.cloud.google.com/storage/browser/waymo_open_dataset_v_1_2_0_individual_files/

### Set up
For setting the environment, a remote VM is provided by Udacity with GPU capabilities and utilize CUDA cores for models training process.
The environment has already installed all of the necessary libraries and tools:
Python
Tensorflow
Numpy
Pandas
matplotlib 

### Dataset
#### Dataset analysis
With this dataset, rectangular bounding boxes are used and apply to images that contain objects(cars, pedestrians, cyclist). Exploring the dataset,images are taken in multiple places and time as day or night, sunny or rainy.

<img width="274" alt="Blurry_Dark_Cars" src="https://user-images.githubusercontent.com/36104217/175806885-033b65fc-b9dd-49f3-8dc5-9fc55c06aa7f.png">

<img width="278" alt="Complex_Cars_Peds" src="https://user-images.githubusercontent.com/36104217/175806891-3a617e6e-9e36-4b80-b285-aec1e3a3af00.png">

As can be seen in above images, sample images are embedded with color coded bounding boxes in red, blue and green associated with Cars, pedestrians and cyclist.
By using Exploratory Data Analysis notebook, it can be noticed that there are large samples of cars and pedestrians while it's quite small size for cyclist. It might be an issue for training process and could cause the model to be overfitting, this will be discussed in later sections.
Here are the distribution for each labels in 5000 randoms images:

<img width="396" alt="ObjectCounts" src="https://user-images.githubusercontent.com/36104217/175806906-10f04dd0-01ae-4e17-a9eb-a2580f732b7c.png">

***Distribution of Cars***

<img width="364" alt="Dist_cars" src="https://user-images.githubusercontent.com/36104217/175806919-6881e7d7-2cd2-4bff-a321-b7988d326a4a.png">

The distribution shows around 20000 labels of around 10 cars appeared in the same image.

<img width="367" alt="Dist_Peds" src="https://user-images.githubusercontent.com/36104217/175806925-50f72f5b-c1fe-4a47-bf0b-1feb311ad0ca.png">

It's similar case for pedestrian distribution, less than 10 pedestrians are presented in around 4000 to 8000 labels.

<img width="356" alt="Dist_Cyclist" src="https://user-images.githubusercontent.com/36104217/175806927-857a85b8-cacb-42eb-9f7e-e6047d1fb1d4.png">

As mentioned above, number of cyclist is significantly lesser than other objects and even in each image.

#### Cross validation
To counter the imbalance classes in sample data, 100 tfrecords files are being shuffled and randomly split into training, testing and validation sets. With a ratio 0.8:0.2 for training and validation data to minimize the occurence of overfitting.

### Training
#### Reference experiment
In this project, Residual network model (RESNET) is recommended to use, and the results can be seen as below:

<img width="619" alt="Exp_1_Loss" src="https://user-images.githubusercontent.com/36104217/175806943-1dc9c871-b00c-4656-b7a8-dd6580c41270.png">

In this result, training loss is in orange and validation loss is in blue. As predicted, the model got overfitted since a significant error rate during validation phase.

In the other hand, precision and recall graphs show a slow and steady performance increase.

<img width="619" alt="Exp_1_Dectect_Precision" src="https://user-images.githubusercontent.com/36104217/175806951-26a2f2fc-7174-409f-b327-aff2ce06c607.png">

<img width="625" alt="Exp_1_Dectect_Recall" src="https://user-images.githubusercontent.com/36104217/175806952-46017075-fd14-4e96-a0bc-0787fcb4ea9e.png">

The results are hard to analyse since remote VM has a limited disk space, so only a small amount of validation set was used.

#### Improve on the reference
To increase further the model performance, there is a jupyter notebook provided: "Explore Augmentation". There are four types of augmentations: random horizontal flip, random crop, random convert to grey scale and random adjust brightness. These augmentations are configured in :"solutions/exp3/pipeline_new.config". Since sample images are also taken in dark environment so adjust brightness can help to increase the accuracy of object detection.
Greyscale:

<img width="301" alt="Greyscale" src="https://user-images.githubusercontent.com/36104217/175806957-fa4133e1-d283-42dd-a92a-cb99455fad54.png">

Brightness:

<img width="356" alt="Brightness" src="https://user-images.githubusercontent.com/36104217/175806965-6323d6a7-36a2-4e5f-bb26-619735529e87.png">

Here is the result with Augmentation:
Loss:

<img width="616" alt="Exp_3_Loss" src="https://user-images.githubusercontent.com/36104217/175806970-04044ab3-26b9-4c07-a358-d93e3d1b945e.png">

Recall:

<img width="622" alt="Exp_3_Dectect_Recall" src="https://user-images.githubusercontent.com/36104217/175806972-b49f3c50-6cf5-45b7-8871-351194e58382.png">

Precision:

<img width="620" alt="Exp_3_Dectect_Precision" src="https://user-images.githubusercontent.com/36104217/175806977-a690562e-f327-44a5-a2d1-77d5f96e4a75.png">

As can be seen in the result with augmentation, there is a significant improvement, training and validation loss gap has been reduced. This is understandable since with augmentation, more sample images are added to the training set, meaning more images for cyclist and pedestrians, minimizing the chances of overfitting. 
Conclusion, it can be proven that data augmentation is very helpful when there's small sample set for training, and also reduce the imbalances of labels. Further improvements can be done by collecting more data and optimizing the training parameters.


