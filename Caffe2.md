Title: Using A Trained Caffe Net
Date: 2015-04-28
Category: Caffe 


#Using A Trained Net To Classify ImageNet images

In order for a trained net to be useful, we need to setup a workflow,
with a function that takes an input image and produces an output of
possible classifications and their confidences. To do this, I've setup
an iPython notebook with the following cells:

Cell 1: Initial imports and path setting


    ::::Python
    import numpy as np
    import matplotlib.pyplot as plt
    %matplotlib inline #Here to allow matplotlib to place 
                       #images in notebook
    import os
    import sys
    import random
    
    caffe_home='/home/me/Caffe/'
    caffe_image_dir = os.path.join(caffe_home,'images')
    
    sys.path.insert(0, caffe_home + 'caffe/python') #Caffe's Python
                                                    #installation is a
                                                    #little rough
                                                    #Sometimes this
                                                    #insert is needed   
    import caffe
    
    # Set the right path to your model definition file, pretrained model weights,
    # and the image you would like to classify.
    MODEL_FILE = os.path.join(caffe_home,'caffe/models/bvlc_reference_caffenet/deploy.prototxt')
    PRETRAINED = os.path.join(caffe_home,'caffe/models/bvlc_reference_caffenet/bvlc_reference_caffenet.caffemodel')

Cell 2: Load dictionaries

The preloaded machine gave classes as integers between 0 and 999. The
dictionary, classDict, translates these integers to names. Similarly,
the image files are tagged with code strings, which nLabelDict maps to
names.

    ::::Python
    import pickle
    classDictFile = open(os.path.join(caffe_home,'caffe/data/ilsvrc12/classDict.pkl'),'rb')
    classDict = pickle.load(classDictFile)
    nLabelDictFile = open(os.path.join(caffe_home,'caffe/data/ilsvrc12/nLabelDict.pkl'),'rb')
    nLabelDict = pickle.load(nLabelDictFile)


Cell 3: Set up trained machine

The next step was to tell Caffe to use the GPU, and then create a
classifier with a specified architecture (MODEL_FILE) and weights that
came from prior training (PREDETERMINED).

    ::::Python
    caffe.set_mode_gpu()
    net = caffe.Classifier(MODEL_FILE, PRETRAINED,
                           mean=np.load(caffe_home + 'caffe/python/caffe/imagenet/ilsvrc_2012_mean.npy').mean(1).mean(1),
                           channel_swap=(2,1,0),
                           raw_scale=255,
                           image_dims=(256, 256))



I don't entirely understand the named arguments, but conjecture that

1. The "mean" argument is meant to preprocess the image by giving the
mean intensity of the training set.

2. The "channel_swap" argument is meant to compensate for the fact
that Caffe uses BGR images (like OpenCV?), but the input images are
RGB.

3. The "raw_scale" argument gives the max value in the image array

4. The "image_dims" argument gives either (cols, rows) or (rows, cols)

  
