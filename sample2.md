Title: Initial Theano Notes - Scan
Date: 2015-01-10 11:10
Category: Theano Notes


    ::::Text only
    me@Bedrock1:~$ python
    Python 2.7.6 (default, Mar 22 2014, 22:59:56) 
    [GCC 4.8.2] on linux2
    Type "help", "copyright", "credits" or "license" for more information.
    >>> import theano.tensor as T
    >>> from theano import function
    >>> from theano import shared
    >>> import theano
    >>> X = T.ivector('X')
    >>> res, ups = theano.scan(lambda x: x + 1, sequences=X)
    >>> inc=function([X],res)
    /usr/local/lib/python2.7/dist-packages/theano/tensor/subtensor.py:110: FutureWarning: comparison to `None` will result in an elementwise object comparison in the future.
      start in [None, 0] or
    >>> inc([1,2,3])
    array([2, 3, 4], dtype=int32)
    >>> Y = shared(0)
    >>> Y.dtype
    'int64'
    >>> resy, upsy = theano.scan(lambda x:{Y:(Y+x)}, sequences=X)
    >>> acc=function([X],[],updates=upsy)
    /usr/local/lib/python2.7/dist-packages/theano/tensor/opt.py:2165: FutureWarning: comparison to `None` will result in an elementwise object comparison in the future.
      if (replace_x == replace_y and
    /usr/local/lib/python2.7/dist-packages/theano/scan_module/scan_perform_ext.py:85: RuntimeWarning: numpy.ndarray size changed, may indicate binary incompatibility
      from scan_perform.scan_perform import *
    >>> Y.get_value()
    array(0)
    >>> acc([1,2,3,4])
    []
    >>> Y.get_value()
    array(10)
    >>> acc2 =function([X],[resy],updates=upsy)
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
      File "/usr/local/lib/python2.7/dist-packages/theano/compile/function.py", line 223, in function
	profile=profile)
      File "/usr/local/lib/python2.7/dist-packages/theano/compile/pfunc.py", line 490, in pfunc
	no_default_updates=no_default_updates)
      File "/usr/local/lib/python2.7/dist-packages/theano/compile/pfunc.py", line 237, in rebuild_collect_shared
	+ ' of type ' + str(type(v)))
    TypeError: Outputs must be theano Variable or Out instances. Received None of type <type 'NoneType'>
    >>> resy
    >>> type(resy)
    <type 'NoneType'>
    >>> X2 = T.imatrix('X2')
    >>> res2, ups2 = theano.scan(lambda x: x + 1, sequences=X2)
    >>> inc2=function([X2],res2)

    >>> import numpy as np
    >>> inc2(np.array([[1,2],[4,5]]))
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
      File "/usr/local/lib/python2.7/dist-packages/theano/compile/function_module.py", line 497, in __call__
	allow_downcast=s.allow_downcast)
      File "/usr/local/lib/python2.7/dist-packages/theano/tensor/type.py", line 119, in filter
	raise TypeError(err_msg, data)
    TypeError: ('Bad input argument to theano function at index 0(0-based)', 'TensorType(int32, matrix) cannot store a value of dtype int64 without risking loss of precision. If you do not mind this loss, you can: 1) explicitly cast your data to int32, or 2) set "allow_input_downcast=True" when calling "function".', array([[1, 2],
	   [4, 5]]))
    >>> inc2=function([X2],res2,allow_input_downcast=True)
    >>> inc2(np.array([[1,2],[4,5]]))
    array([[2, 3],
	   [5, 6]], dtype=int32)
    >>> resy2, upsy2 = theano.scan(lambda x:{Y:(Y+x)}, sequences=X2)
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
      File "/usr/local/lib/python2.7/dist-packages/theano/scan_module/scan.py", line 803, in scan
	profile=False)
      File "/usr/local/lib/python2.7/dist-packages/theano/compile/function.py", line 223, in function
	profile=profile)
      File "/usr/local/lib/python2.7/dist-packages/theano/compile/pfunc.py", line 490, in pfunc
	no_default_updates=no_default_updates)
      File "/usr/local/lib/python2.7/dist-packages/theano/compile/pfunc.py", line 217, in rebuild_collect_shared
	raise TypeError(err_msg, err_sug)
    TypeError: ('An update must have the same type as the original shared variable (shared_var=<TensorType(int64, scalar)>, shared_var.type=TensorType(int64, scalar), update_val=Elemwise{add,no_inplace}.0, update_val.type=TensorType(int64, vector)).', 'If the difference is related to the broadcast pattern, you can call the tensor.unbroadcast(var, axis_to_unbroadcast[, ...]) function to remove broadcastable dimensions.')
    >>> X = T.matrix('X')
    >>> floatX='float32'
    >>> results, updates = theano.scan(lambda i, j, t_f: T.cast(X[i, j] + t_f, floatX),
    sequences=[T.arange(X.shape[0]), T.arange(X.shape[1])],
    outputs_info=np.asarray(0., dtype=floatX))
    ... ... >>> 
    >>> compute_trace = theano.function(inputs=[X], outputs=[results])
    >>> x = np.eye(5, dtype=theano.config.floatX)
    >>> x[0] = np.arange(5, dtype=theano.config.floatX)
    >>> x
    array([[ 0.,  1.,  2.,  3.,  4.],
	   [ 0.,  1.,  0.,  0.,  0.],
	   [ 0.,  0.,  1.,  0.,  0.],
	   [ 0.,  0.,  0.,  1.,  0.],
	   [ 0.,  0.,  0.,  0.,  1.]], dtype=float32)
    >>> compute_trace(x)
    [array([ 0.,  1.,  2.,  3.,  4.], dtype=float32)]
    >>> x[0][0]
    0.0
    >>> x[0][0]=77
    >>> compute_trace(x)
    [array([ 77.,  78.,  79.,  80.,  81.], dtype=float32)]
    >>> results2, updates2 = theano.scan(lambda i, j, huh: T.cast(X[i, j] + huh, floatX),
    sequences=[T.arange(X.shape[0]), T.arange(X.shape[1])],
    outputs_info=np.asarray(0., dtype=floatX))
    ... ... >>> 
    >>> compute_trace2 = theano.function(inputs=[X], outputs=[results2])
    >>> x2 = np.eye(5, dtype=theano.config.floatX)
    >>> x2[0][0]=55
    >>> compute_trace(x2)
    [array([ 55.,  56.,  57.,  58.,  59.], dtype=float32)]
    >>> 
    >>> 
    >>> results3, updates3 = theano.scan(lambda i, j: T.cast(X[i, j], floatX),
    sequences=[T.arange(X.shape[0]), T.arange(X.shape[1])],
    outputs_info=np.asarray(0., dtype=floatX))
    ... ... Traceback (most recent call last):
      File "<stdin>", line 4, in <module>
      File "/usr/local/lib/python2.7/dist-packages/theano/scan_module/scan.py", line 732, in scan
	condition, outputs, updates = scan_utils.get_updates_and_outputs(fn(*args))
    TypeError: <lambda>() takes exactly 2 arguments (3 given)
    >>> results3, updates3 = theano.scan(lambda i, j: T.cast(X[i, j], floatX),
    sequences=[T.arange(X.shape[0]), T.arange(X.shape[1])],
    outputs_info=np.asarray(0., dtype=floatX))
    ... ... Traceback (most recent call last):
      File "<stdin>", line 3, in <module>
      File "/usr/local/lib/python2.7/dist-packages/theano/scan_module/scan.py", line 732, in scan
	condition, outputs, updates = scan_utils.get_updates_and_outputs(fn(*args))
    TypeError: <lambda>() takes exactly 2 arguments (3 given)
    >>> results3, updates3 = theano.scan(lambda i, j: T.cast(X[i, j], floatX), sequences=[T.arange(X.shape[0]), T.arange(X.shape[1])])
      File "<stdin>", line 1
	results3, updates3 = theano.scan(lambda i, j: T.cast(X[i, j], floatX), sequences=[T.arange(X.shape[0]), T.arange(X.shape[1])])
	^
    IndentationError: unexpected indent
    >>> results3, updates3 = theano.scan(lambda i, j: T.cast(X[i, j], floatX), sequences=[T.arange(X.shape[0]), T.arange(X.shape[1])])
    >>> np.asarray(0., dtype=floatX)
    array(0.0, dtype=float32)
    >>> results3, updates3 = theano.scan(lambda i, j, huh: T.cast(X[i, j] + huh, floatX),
    sequences=[T.arange(X.shape[0]), T.arange(X.shape[1])],
    outputs_info=np.asarray(20., dtype=floatX))
    ... ... >>> 
    >>> compute_trace3 = theano.function(inputs=[X], outputs=[results3])
    >>> x3 = np.eye(5, dtype=theano.config.floatX)
    >>> x3[0][0]=87
    >>> x3
    array([[ 87.,   0.,   0.,   0.,   0.],
	   [  0.,   1.,   0.,   0.,   0.],
	   [  0.,   0.,   1.,   0.,   0.],
	   [  0.,   0.,   0.,   1.,   0.],
	   [  0.,   0.,   0.,   0.,   1.]], dtype=float32)
    >>> compute_trace3(x3)
    [array([ 107.,  108.,  109.,  110.,  111.], dtype=float32)]
    >>> results3, updates3 = theano.scan(lambda i, j, uh, huh: T.cast(X[i, j] + huh + 2*uh, floatX), sequences=[T.arange(X.shape[0]), T.arange(X.shape[1])],outputs_info=[np.asarray(1., dtype=floatX)), np.asarray(2., dtype=floatX))
      File "<stdin>", line 1
	results3, updates3 = theano.scan(lambda i, j, uh, huh: T.cast(X[i, j] + huh + 2*uh, floatX), sequences=[T.arange(X.shape[0]), T.arange(X.shape[1])],outputs_info=[np.asarray(1., dtype=floatX)), np.asarray(2., dtype=floatX))
																								      ^
    SyntaxError: invalid syntax
    >>> results3, updates3 = theano.scan(lambda i, j, uh, huh: T.cast(X[i, j] + huh + 2*uh, floatX), sequences=[T.arange(X.shape[0]), T.arange(X.shape[1])],outputs_info=[ np.asarray(1., dtype=floatX), np.asarray(2., dtype=floatX)])
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
      File "/usr/local/lib/python2.7/dist-packages/theano/scan_module/scan.py", line 820, in scan
	raise ValueError('Please provide None as outputs_info for '
    ValueError: Please provide None as outputs_info for any output that does not feed back into scan (i.e. it behaves like a map) 
    >>> 
