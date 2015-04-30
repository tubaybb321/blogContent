Title: Reading and Writing Datasets for Caffe
Date: 2015-04-26
Category: Caffe 



#Reading and Writing datasets for Caffe:

Caffe stores each data element in a Datum object. This object has the following fields:

1. *channels*: int - specifies # of data channels (e.g. 1 for a B/W image, 3 for an RGB image etc.)
2. *height*: int - specifies # of rows in data
3. *width*: int - specifies # of cols in data
4. *data*: string - seems to represent uint8 type data values
5. *label*: int - Number assigned to class label of data (e.g. 1 = 'dog', 2 = 'cat' ...)
6. *float_data*: float / int - Values of non-uint8 data
7. *encoded*: ???

These elements are stored in either an lmdb or a leveldb database. Since I've only examined the lmdb database, this write-up will be limited to it. 

#Lightining Memory Mapped Database - lmdb

lmdb (Lightining Memory-Mapped Database) is an interface to a high-performance embedded transactional database in the form of a key-value store. In Caffe, the keys are all integers and the values are Datum objects. For reasons I don't understand (keeping various transactions in synch with each other?) lmdb databases come in pairs and are stored in a common directory. So, a database referred to as CaffeData is really a directory containing (at least) 2 files - data.mdb and lock.mdb. However, to gain access to this database, one just needs to reference the parent directory. 

#Reading Databases
## Quick Stats
To get a quick overview of what is in an lmdb database, I wrote the following code, which will give the number of samples of each class, along with database statistics:

    ::::python
    import caffe
    from collections import defaultdict
    import lmdb
    import matplotlib.cm as cm
    import matplotlib.pyplot as plt
    import numpy as np
    import pprint
    import sys
    
    def getDB(DBname):
      '''Gets variables needed to iterate through LMDB database
         I don't really understand this. All I think I "strictly"
         need is the iterator, but I am passing along others to
         eventually close the interaction in a clean manner'''
    
      env = lmdb.open(DBname)
      txn = env.begin()
      curs = txn.cursor()
      cursIter = curs.iternext()
    
      return [env, txn, curs, cursIter]
    
    def getStats(cursIter):
        outDict = defaultdict(int)
        done = False
        ctr = 0
        while not done:
          try:
            currItem = cursIter.next()
            keyStr = currItem[0] # Key within the DB
            dataStr = currItem[1] #Actual data as a Serialized String
    
            #Unserialize Datum object (Note: As near as I can tell, Datum is
            #based on a C++ pair object. The first field of this object is the
            #ground truth label. The second field is the raw data to be 
            #trained/validated/tested
            data = caffe.proto.caffe_pb2.Datum()
            data.ParseFromString(dataStr)
            labelStr = data.label
     
            outDict[labelStr] +=1
            ctr += 1
          except StopIteration:
            done = True
            print "Number of items:", ctr
            print "Number of classes:", len(outDict.keys())
            for currClass in sorted(outDict.keys()):
                print currClass,":", outDict[currClass], "elements"
            print "\n"
            pprint.pprint (env.stat())
            
    if __name__ == '__main__':
      dbDir = sys.argv[1] #Note: LMDB databases seem to come in pairs
                          # A data db and a lock db. So, to open one, it
                          # is necessary to refer to the common directory
                          # where the two db's are stored.
      env, txn, curs, cursIter = getDB(dbDir)
      getStats(cursIter)

Running this code, will give results similar to this:

    ::::Text only
    $ python getLMDBstats.py ../examples/mnist/mnist_train_lmdb/
    Number of items: 60000
    Number of classes: 10
    0 : 5923 elements
    1 : 6742 elements
    2 : 5958 elements
    3 : 6131 elements
    4 : 5842 elements
    5 : 5421 elements
    6 : 5918 elements
    7 : 6265 elements
    8 : 5851 elements
    9 : 5949 elements
     
    
    {'branch_pages': 68L,
     'depth': 3L,
     'entries': 60000L,
     'leaf_pages': 15000L,
     'overflow_pages': 0L,
     'psize': 4096L}


## Viewing Images
To observe the images in an existing lmdb Caffe database, I wrote the following code:

    ::::python
    import caffe
    import lmdb
    import matplotlib.cm as cm
    import matplotlib.pyplot as plt
    import numpy as np
    import sys

    def getDB(DBname):
      '''Gets variables needed to iterate through LMDB database
         I don't really understand this. All I think I "strictly"
         need is the iterator, but I am passing along others to
         eventually close the interaction in a clean manner'''

      env = lmdb.open(DBname)
      txn = env.begin()
      curs = txn.cursor()
      cursIter = curs.iternext()
    
      return [env, txn, curs, cursIter]
    
    def showNextImage(cursIter):
      '''Shows next iage. Right now, there's no safeguard
         for hitting the end of the database.'''
    
      currItem = cursIter.next()
      keyStr = currItem[0] # Key within the DB
      dataStr = currItem[1] #Actual data as a Serialized String
    
      #Unserialize Datum object (Note: As near as I can tell, Datum is
      #based on a C++ pair object. The first field of this object is the
      #ground truth label. The second field is the raw data to be 
      #trained/validated/tested
      data = caffe.proto.caffe_pb2.Datum()
      data.ParseFromString(dataStr)
      labelStr = data.label
      image = caffe.io.datum_to_array(data)
      origShape = image.shape
    
      #Adjust for Color and Greyscale images
      if image.shape[0] == 1:
        #Greyscale images
        image = image[0,:,:]
        colorMap = cm.Greys_r
      else:
        #Color images - the ilsvrc images seemed to be BGR instead of RGB
        b,g,r = image
        image = np.array([r,g,b])
        image = np.transpose(image,(1,2,0))
        colorMap = None
    
      #Show db Key, Ground Truth label and Image
      plt.imshow(image, cmap = colorMap)
      plt.ion()
      print "DB Key =", keyStr
      print "Label =", labelStr
      print "Shape =", origShape
      print "Dtype =", image.dtype
      plt.show()
    
    
    if __name__ == '__main__':
      dbDir = sys.argv[1] #Note: LMDB databases seem to come in pairs
                          # A data db and a lock db. So, to open one, it
                          # is necessary to refer to the common directory
                          # where the two db's are stored.
      env, txn, curs, cursIter = getDB(dbDir)

      done = False
      dummy = ''
      while done == False:
        if dummy.lower() != 'q':
           try:
             showNextImage(cursIter)
             dummy = raw_input("\nEnter 'q' to quit; enter any other key to see the next image\n")
           except StopIteration:
             done = True
             print "No more images"
        else:
           done = True

Although the UI has a certain paleolithic quality, it is serviceable. At some point, I might make it more user friendly. To run this code (i.e. viewLMDB_images2.py) from the command line to look at the mnist_train database, for example:

    ::::Text only
    $ python viewLMDB_images2.py ../examples/mnist/mnist_train_lmdb/
    

    

Gives:

<img src="/CaffeFive.png" alt="/CaffeFive.png" style="width: 750px; height: 800px;"/>

with a text output of:

    ::::Text only
    DB Key = 00000000
    Label = 5
    Shape = (1, 28, 28)
    Dtype = uint8

    Enter 'q' to quit; enter any other key to see the next image    


#Writing Databases

In order to train a Caffe net on non-prepackaged data, it will be necessary to create an lmdb database of Caffe datum objects. So, I wrote a small program to go through a 2-level directory of images, where the main level is a 'grandparent' on the path to each image and each secondary level contains a set of same-class images.


    ::::python
    import sys
    import caffe
    import Image
    import lmdb
    import matplotlib.cm as cm
    import matplotlib.pyplot as plt
    import numpy as np
    import os
    from pgmWrapper import pgmWrapper
    from random import shuffle

    #Needed so that PIL will read pgm images
    Image.open = pgmWrapper(Image.open)
    
    #Grandparent & Parent directoried for all images (Note: Only want digits)
    mainImageDir = '/media/smgutstein/OSDisk/newNist/pgmProcImages/'
    subImageDirs = [x for x in os.listdir(mainImageDir)
                    if x.isdigit() and os.path.isdir(os.path.join(mainImageDir,x))]
                
    #Number samples per class in data set 
    numClassSamples = 1000
    
    def getImageLists():
       '''Get list of all images for each class'''
       outDict = {}
       for currIm in subImageDirs:
           currDir = os.path.join(mainImageDir,currIm)
           currIms = os.listdir(currDir)
           shuffle(currIms)
           outDict[currIm] = [currDir,currIms[0:min(numClassSamples, len(currIms))]]
       return outDict

    def makeLMDB(lmdbName, dataDict):
       '''Make lmdb database for Caffe'''
       newDB = lmdb.open(lmdbName,map_size=int(1e12))
       with newDB.begin(write=True) as txn:
           
        for batchCtr in range(numClassSamples):
           for ctr, currClass in enumerate(subImageDirs):
               if dataDict[currClass][1] == []:
                   # Class with fewer samples than desired
                   print "Class ", currClass, " is empty"
                   continue
    
                #Get next data sample and prepare to make Datum object
               currElem = dataDict[currClass][1].pop()
               currLabel = int(currClass)
               currImage = np.array(
                           Image.open(os.path.join
                                         (dataDict[currClass][0],
                                          currElem)))
               currImage = currImage.astype('uint8') #Pixel values 0-255
               
               #Adjust for Color and Greyscale images
               if currImage.ndim == 2:
                  #Greyscale images
                  currImage = np.expand_dims(currImage,0)   
               else:
                  #Color images - the ilsvrc images seemed to be BGR instead of RGB
                  currImage = np.array(currImage)
                  currImage = np.transpose(currImage,(2,0,1))
                  r, g, b = currImage
                  currImage = np.array([b, g, r])

               #Create Datum object and store to lmdb
               currImageDatum = caffe.io.array_to_datum(currImage, currLabel)
               txn.put('{:0>10d}'.format(batchCtr*10 + ctr + 1),
                       currImageDatum.SerializeToString())

       newDB.close()
       print "Wrote data to", os.path.join(os.getcwd(), lmdbName)
       
    if __name__ == '__main__':
        outDict = getImageLists()
        if len(sys.argv) < 2:
            outDB = 'default_LMDB'
        else:
            outDB = sys.argv[1]
        makeLMDB(outDB, outDict)
    
