Title: Theano Scan Function
Date: 2015-01-10 11:10
Category: Theano Notes

#Theano Scan Function(Note: More to come)

Theano's scan function creates a "looping" graph. Regular Theano graphs have a single set of inputs, which are used in to calculate a single set of outputs in a single pass. The scan function makes it possible to "loop" over the graph, using as inputs a combination of:

* Recursively generated values
* Sequences of values, where the nth element of the sequence is used for the nth loop
* Constant input values, which are repeated at each iteration.

This function can improve Theano's efficiency in several ways:

1. Moving more functionality to the symbolic graph
2. Minimizing CPU <-> GPU data transfers
3. Avoiding loops within Python interpreter (e.g. For loop)
4. Lower memory useage

The scan function takes several named arguments, some of which are necessary, some of which are optional and some of which are necessary iff other optional arguments are/are not present. These arguments are:

1. fn = function being looped over. The inputs of this function are given in the following order:
    a. Values being used recursively
    b. Values being passed in a predetermined, sequential order
    c. Values that do not change with each iteration
2. outputs_info = Initial states for any values used recursively - the order of outputs_info will match that of actual outputs
3. sequences = Values used in predetermined sequence
4. non_sequences = Values that are repeated in each loop
5. n_steps = # of times to loop. Presumably, this is only used if there are no sequences to implicitly determine this

The Theano documentation has several examples of scan. The use I found for it was for the train function of a logistic regression learner.

    ::::python
    import numpy
    import theano
    import theano.tensor as T
    rng = numpy.random
    
    #Randomly create 400 pts in a 784 Dim space and randomly
    #assign to two classes
    N = 400
    feats = 784
    D = (rng.randn(N, feats).astype(theano.config.floatX),
         rng.randint(size=N,low=0, high=2).astype(theano.config.floatX))
    training_steps = 10000
    
    # Declare Theano symbolic variables
    x = T.fmatrix("x")
    y = T.fvector("y")
    w = T.fvector("w")
    b = T.fscalar("b")
    
    # Training function
    def oneStep(w,b,x,y):
       # Make graph(s)
       p_1  = 1 / (1 + T.exp(-T.dot(x, w)-b)) # Probability of having a one
       prediction = p_1 > 0.5 # The prediction that is done: 0 or 1
       xent = -y*T.log(p_1) - (1-y)*T.log(1-p_1) # Cross-entropy
       cost = xent.mean() + 0.01*(w**2).sum() # The cost to optimize
       gw,gb = T.grad(cost, [w,b])
       return [w-0.01*gw, b-0.01*gb]
    
    #Set starting values for w and b
    w_orig = rng.randn(feats).astype(theano.config.floatX)
    b_orig = numpy.asarray(0., dtype=theano.config.floatX)
    
    
    #  Create scan/looping function for logistic regression training
    ([ww, bb], updates) = theano.scan(fn=oneStep,
                                      outputs_info=(w_orig, b_orig),
                                      non_sequences = [x,y],
                                      n_steps = training_steps)
    
    
    train = theano.function(inputs=[x,y], outputs=[ww,bb])
    
    
    # Get all sets of weights learned during training
    w_final,b_final = train(D[0],D[1]) 
    
    # Get ultimate set of weights 
    wf = w_final[-1]
    bf = numpy.asarray(b_final[-1], dtype = theano.config.floatX)
    
    #Create predictive function
    p_1 = 1 / (1 + T.exp(-T.dot(x, w)-b)) 
    prediction = p_1 > 0.5
    predict = theano.function(inputs=[x,w,b], outputs=prediction)
      
    #Test predictive function
    print "target values for D:\n", D[1].astype('int8')
    print "\nprediction on D:\n", predict(D[0],wf,bf)
    
    print "diff = ",(numpy.abs(predict(D[0],wf,bf)) - numpy.abs(D[1].astype('int8'))).sum()



