# Lab 6: Line Detection

We have trained a model that can recognize text in a line, given an image of a single line.
Our next task is to automatically detect line regions in an image of a whole paragraph of text.

Our approach will be to train a model that, when given an image containing lines of text, returns a pixelwise labeling of that image, with each pixel belonging to either background, odd line of handwriting, or even line of handwriting.
Given the output of the model, we can find line regions with an easy image processing operation.

Why the distinction between odd and even lines?
Because adjacent line regions can overlap, and if we don't make this distinction, then we will not be able to know whether a given region is one or multiple lines.

## Data

We are starting from the IAM dataset, which includes not only lines but the original writing sample forms, with each line and word region annotated.

Let's look at the raw data in the file browser.

We want to crop out the region of each page corresponding to the handwritten paragraph as our model input, and generate corresponding ground truth.

Code to do this is in `text_recognizer/datasets/iam_paragraphs_dataset.py`

We can look at the results in `notebooks/04-look-at-iam-paragraphs.ipynb` and by looking at some debug images we output in `data/interim/iam_paragraphs`.

## Training data augmentation

The model code for our new `LineDetector` is in `text_recognizer/models/line_detector_model.py`.

Because we only have about a thousand images to learn this task on, data augmentation will be crucial.
Image augmentations such as streching, slight rotations, offsets, contrast and brightness changes, and potentially even mirror-flipping are tedious to code, and most frameworks provide optimized utility code for the task

We use Keras's `ImageDataGenerator`, and you can see the parameters for it in `text_recognizer/models/line_detector_model.py`.
We can take a look at what the data transformations look like in the same notebook.

## Network description

The network used in this model is `text_recognizer/networks/fcn.py`.

The basic idea is a deep convolutional network with resnet-style blocks (input to block is concatenated to block output).
We call it FCN, as in "Fully Convolutional Network," after the seminal paper that first used convnets for segmentation.

Unlike the original FCN, however, we do not maxpool or upsample, but instead rely on dilated convolutions to rapidly increase the effective receptive field.
[Here](https://fomoro.com/projects/project/receptive-field-calculator) is a very calculator of the effective receptive field size of a convnet.

The crucial thing to understand is that because we are labeling odd and even lines differently, each predicted pixel must have the context of the entire image to correctly label -- otherwise, there is no way to know whether the pixel is on an odd or even line.

## Review training results

## Combining the two models

Now we are ready to combine the new `LineDetector` model and the

## Things to try

- Try adding more data augmentations, or mess with the parameters of the existing ones
- Try the U-Net architecture, that MaxPool's down and then UpSamples back up, with increased conv layer channel dimensions in the middle (https://lmb.informatik.uni-freiburg.de/people/ronneber/u-net/).
