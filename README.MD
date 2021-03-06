[![Codacy Badge](https://app.codacy.com/project/badge/Grade/e263d84692974d38a0678f3090a09187)](https://www.codacy.com/manual/nielstenboom/recurring-content-detector?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=nielstenboom/recurring-content-detector&amp;utm_campaign=Badge_Grade)

# Recurring content detector (credits, recaps and previews detection)

This repository contains the code that was used to conduct experiments for a [master's thesis](https://github.com/nielstenboom/masterthesis/raw/master/main.pdf). The goal was to detect recaps, opening credits, closing credits and previews from video files in an unsupervised manner. This can be used to automate the labeling for the skip functionality of a VOD streaming service.

The experiments done in the master's thesis were done in the jupyter notebooks, but as the code in these got quite messy. I packed the used code in a python package so that it can be used more easily.

## Quickstart with Docker

To try this out with Docker on your own data, copy the video files of one season to a folder in the current directory named `videos`, then create a script `detect.py` with the contents:

```python
import recurring_content_detector as rcd

results = rcd.detect("videos")

print(results)
```
Now you can run the detector by running the docker command:
```
docker run -it -v $(pwd):/opt/recurring-content-detector nielstenboom/recurring-content-detector:latest python detect.py
```
It'll first downsize the videos using ffmpeg, then it will convert the videos to feature vectors and then the algorithm for detecting the recurring content is applied, the `results` variable contains the intervals of all the recurring parts.

## Local Installation

To install the package, do the following steps (assuming you have an anaconda setup with python 3.6) if you want to run the program with the default settings:

```bash
git clone https://github.com/nielstenboom/recurring-content-detector.git
cd recurring-content-detector
conda install faiss-cpu -c pytorch
pip install mkl
# optional step: change parameters in recurring_content_detector/config.py
pip install .
```

It is also possible to build a docker container that does all the steps for you with the [Dockerfile](Dockerfile) in the directory.

Make sure [ffmpeg](https://ffmpeg.org/) is in the PATH variable and that [tensorflow](https://www.tensorflow.org/install/pip) (GPU version preferably) is installed.

Run `pip uninstall recurring-content-detector` to uninstall the package.

## Recurring content detection

With this code it is possible to detect recaps, opening credits, closing credits and previews in video files from a TV-show unsupervised up to a certain extent. The following image gives a schematic overview of how it works: 

<p align="center">
<img src="images/thesisdiagram.png?raw=true">
</p>

You can run the detector in a python program in the following way:

```python
import recurring_content_detector as rcd
rcd.detect("/directory/with/season/videofiles")
```
This will run the detection by building the color histogram feature vectors. The feature vector function can also be changed:
```python
# options for the function are ["CNN", "CH", "CTM"]
rcd.detect("/directory/with/season/videofiles", feature_vector_function="CNN")
```
This will CNN vectors, which are a bit more accurate but take much longer to build.

Because the videos need to be resized and the feature vectors saved in files, some artifacts will be created. On default they will be saved in the same directory as the video files, if you want them saved in a different directory:
```python
rcd.detect("/directory/with/season/videofiles", feature_vector_function="CH", artifacts_dir="/tmp")
```

Make sure the video files you used can be sorted in the right alphabetical order similar as to when they play in the season! So episode_1 -> episode_2 -> episode_3 -> etc.. You'll get weird results otherwise.

It will take some time as video processing takes quite some resources. An example application in production should run detections in parallel.


## Annotations

If you want to quantitively test out how well this works on your own data, fill in the [annotations](annotations_example.csv) file and supply it as the second parameter.
```python
rcd.detect("directory/with/season/videofiles", annotations = "path/to/annotations.csv")
```

When the program is done, it will give an example with the same outline as the example below:

 ```
Detections for: episode1.mp4
0:01:21.600000           -               0:02:20.880000
0:02:49.320000           -               0:03:15.480000

Detections for: episode2.mp4
0:00:00                  -               0:01:16.920000
0:01:38.040000           -               0:02:37.440000

Detections for: episode3.mp4
0:00:00                  -               0:01:27.840000
0:01:51.120000           -               0:02:50.760000
0:42:26.400000           -               0:42:54

Total precision = 0.862
Total recall = 0.853
 ```
 
## Credits
- https://github.com/noagarcia/keras_rmac for the CNN vectors 
- https://github.com/facebookresearch/faiss for the efficient matching of the feature vectors 
