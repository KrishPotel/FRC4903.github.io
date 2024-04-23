# Note detection

## Limelight 3 with Google Coral

Limelight allows you to run object detection pipelines when using a [Google Coral USB Accelerator](https://www.coral.ai/products/accelerator). You can either upload a pretrained game piece detection model (which can be found [here](https://limelightvision.io/pages/downloads)) or you can train a tensorflow lite model on your own dataset.

### Training your own model

First, create your dataset. You can take your own images, or you can find a datatset on [Roboflow Universe](https://universe.roboflow.com/). Next, download the dataset in Tensorflow TFRecord format. Now that you have your dataset, you can fine-tune a pre-trained limelight model for your dataset. Follow the instructions in [this notebook](https://colab.research.google.com/github/LimelightVision/notebooks/blob/main/limelight-detector-training-notebook.ipynb#scrollTo=eGEUZYAMEZ6f) to do so. Now you can upload your model to your limelight to start running object detection.
