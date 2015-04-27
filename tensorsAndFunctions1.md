Title: Creating Theano Variables and Functions 
Date: 2015-02-05 11:10
Category: Theano Notes


Theano gains its speed by efficient use of memory, efficient compilation of functions and easy usage of GPUs. 

One way it manages to use memory efficiently is through its use of symbolic variables. Symbolic variables are strongly typed, but do not have values associated with them. It is as though they are declarations of input parameters to be used in functions that will be declared later. The most common (?only?) class of symbolic variables is the tensor.

Theano tensors represent numpy N-dim arrays (i.e. numpy.ndarray). By convention, the first letter of a tensor describes the type of variable being stored and the rest of the word describes the dimensionality of the storage area - i.e.

* Variable Type
    * b - int8
    * w - int16
    * i - int32
    * l - int64
    * f - float32
    * d - float64
    * c - complex64
    * z - complex128

* Storage Type
    * scalar - 0 dim array
    * vector - 1 dim array
    * row - 2 dim array with #cols = 1
    * col - 2 dim array with #rows = 1
    * matrix - 2 dim array
    * tensor3 - 3 dim array
    * tensor4 - 4 dim array
    * (other storage types may be created by the user)

Examples of tensors and functions:             

    ::::text only
    >>> import theano.tensor as T
    >>> from theano import function
    >>> x = T.dscalar()
    >>> y = T.dscalar()
    >>> z=x+y #Note: Because z is defined as the sum of 2 float64 scalars, it is also a float64 scalar
    >>> f=function([x,y],z) #Note: f is a function that takes 2 float64 values as inputs and returns
                                #      a value that has been defined as the sum of the two inputs
    >>> f(2,3)
    array(5.0)
    >>> 
    >>> fi = function([x],x) #Note: Here f is a function that takes a float64 scalar value and returns it
    >>> fi(9)
    array(9.0)
    
    # Other Examples
    
    >>> x = T.dvector()
    >>> y = T.dvector()
    >>> z=x+y
    >>> f=function([x,y],z)
    >>> f([2,3],[6,7])
    array([  8.,  10.])
    >>> 
    >>> x = T.dmatrix()
    >>> y = T.dmatrix()
    >>> z=x+y
    >>> f=function([x,y],z)
    >>> f([2,3],[6,7]) #Note: Wrong input type
    Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    File "/usr/local/lib/python2.7/dist-packages/theano/compile/function_module.py", line 497, in __call__
       allow_downcast=s.allow_downcast)
    File "/usr/local/lib/python2.7/dist-packages/theano/tensor/type.py", line 157, in filter
       data.shape))
    TypeError: ('Bad input argument to theano function at index 0(0-based)', 'Wrong number of dimensions: expected 2got 1 with shape (2,).')
    >>>
    >>> f([[2,3]],[[6,7]])
    array([[  8.,  10.]])
