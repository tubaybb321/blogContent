Title: Using Hyungtae's Code
Date: 2015-01-17 11:10
Category: Dynamic Belief Fusion

#Background
Hyungtae's code was written to compare various classifiers and methods of data fusion with his Dynamic Belief Fusion technique. It does this by training 4 different classifiers to search images for the presence of a specified object. The four classifiers are:

1. DPM - Deformable Parts Model
2. Exemplar SVM - An Ensemble of SVMs, one for each positive example in the training set
3. HOG SVM - SVM based on Histogram of Gaussian Features
4. SIFT SVM - SVM based upon SIFT features

The identifiable classes come from the ARL office dataset and are:

1. Chair
2. Stairs
3. Poster
4. Window
5. Container

Our interest in Hyungtae's code is to use the results of his fused classifiers to act as a semi-reliable oracle in order to train EEG classifiers. To do this, we needed to write some modified modules, in order to query the fused classifier about individual images, translate its confidence measures into points on a precision-recall curve and to reallocate a dataset into different training, validation and testing sets.

#Creating Files With Fusion Classifier's Responses To Images
The original code relied upon test.m to measure the performance of each classifier and a few different fusing techniques upon the test set. In order to get this information for the validation & testing sets, the functions sg_validate and sg_test were created. Hyungtae's code uses the validation set to determine the best parameters for Dynamic Belief Fusion, as a result the use of the validation set and test set were hard coded in test.m. It was deemed easier, in the immediate term, to make two different functions to get the fusion results for validation and testing sets. 

Both sg_validate and sg_test are used in the same way - e.g.

    ::::Text only
    sg_test('chair')

will produce the file:

    ::::Text only
    chairData.mat

This file will contain the following variables:


1. *ap_all*: Area under Precision/Recall curve for each individual classifier & fused classifier

2. *bestu*: Best value for Hyungtae's u parameter (used to measure uncertainty of classification)

3. *bestn*: Best value for Hyungtae's n parameter (used to estimate best obtainable precision)

4. *bestdets*: An N x 3 cell, where N = # of images used in the set of interest (validation or test set depending upon whether sg_validate or sg_test used). 
    
    a. The first column contains a cell array with the coordinates (upper, left, lower, right) [check with Hyungtae] of each bounding box found in the given image along with the confidence that the bounding box contains a sample of the class of interest. The cell array is sorted by the confidence parameter.

    b. The second column identifies the image file.

    c. The third column names the class of interest.

5. *bestprior*: a 4x3 array of parameters for each of the base classifiers used by Hyungtae to fuse predictions together. Each row contains the parameters for a particular classifier (I don't know which row corresponds to which classifier). 

    a. The first column contains an array 3xM array, where M = # of object detections and the 3 rows respectively contain m(Trgt), m(~Trgt) & m(Uncertain) for various thresholds/confidences. 

    b  The second column contains an array with the corresponding thresholds/confidences.

6. *precRecData*:  I modified Hyungtae's code to give this 1 x 3 cell containing precision and recall measures for various confidence thresholds for the given detector.

    a.  The first column contains a 2 x N array, where N corresponds to the number of p/r measurements with 

        1. precision measures in the first row, 

        2. recall measures in the second row 

    b. The second column of precRecData contains the confidence thresholds that correspond to the precision & recall measurements. 

    c. The third column is empty.

7. *gnd_truth*: An array with ground truth information for each image. If the bounding box (bbox) array is [], then the target object is not in the image, otherwise, bbox contains (leftmost pt, top most pt, right most point, bottom point) for the bounding box [check this]. I don't quite know what det & obj contain, although det=[] seems to indicate no target to detect, but det=0 seems to indicate a target.


This file should then be copied into the TestSetResults directory. In order to create these files for all 5 classes, just run:

    ::::Text only
    sg_tests()

and then move the resulting files to ./TestSetResults. For the validation set, the corresponding sg_validate & sg_validates work in the same way.

 
#Obtaining Fused Classifier's Response To A Particular Image

To see if the fused classifier detected a particular class within a particular image, use the function *sg_oracle*, which has this signature:

    [best_result, precRecCurves, thresholds, gt_bb, ov] = sg_oracle( im_name, class)

For example,

    [best_result, precRecCurves, thresholds, gt_bb, ov] = sg_oracle( '10053.jpg', 'chair');

Here, *best_result* will return a 1x5 array, containing the coordinates of the bounding box most likely to contain the object in question followed by the confidence that it does. *precRecCurves* is a 2xN array with precision values in the first row and recall values in the second row. *thresholds* is a 1xN array containing the confidence values corresponding to those precision and recall values. So, if one decided to accept all responses with a confidence greater or equal to the m^{th} value in *thresholds*, then one should expect precision and recall values that are equal to the m^{th} entries in *precRecCurves*. Finally, *gt_bb* is the ground truth bounding box(es) and *ov* is a list of overlap measurements between the most-likely box found by the fuser and each bounding box. The measure of overlap is (Area of Intersection) / (Area of Union).  

#Creating New Train/Validation/Test Set Divisions

In SG_data_split_test is the function shuffle_data. The arguments to this function are:

1. Name of the output matix
2. Number of parts in training set
3. Number of parts in validation set
4. Number of parts in test set


Example:

    ::::Text only
    shuffleData(4,2,2,'data_4_2_2.mat')

Note: This needs to be made to conform to existing configuration for EEG learner
