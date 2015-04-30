Title: Theano Scan Function-V2
Date: 2015-01-10 11:10
Category: Theano Notes

#Scan Function Notes

Theano uses a "scan" function to allow for looping in Theano functions. It seems to present a more efficient alternative than repeatedly calling a Theano function from within a Python loop. Consider a function to find the n<sup>th</sup> Fibonacci number:

    ::::Python
    import theano
    from theano import tensor as T
    from theano import function as Tfunc
    import numpy as np

    def fibFnc(iInit, jInit, nthFib):
      #Initialize Theano variables and function
      Ti = T.lscalar()
      Tj = T.lscalar()
      TfibFunc = Tfunc(inputs=[Ti, Tj],outputs = Ti + Tj)
  
      #Initialize output storage array
      outArray = np.zeros(nthFib + 1)
      outArray[0] = jInit
      outArray[1] = iInit

      #Calculate values
      for ctr in range(2, nthFib+1):
          outArray[ctr] = TfibFunc(outArray[ctr-1],outArray[ctr-2])

      return outArray
      
Using the Theano profiler, I found that when calculating the 500<sup>th</sup> Fibonacci number (which gives an erroneous answer due to an overflow), this function takes about 2.36E-02 seconds, on average. However, the following function, which uses 'scan',

    ::::Python
    import theano
    from theano import tensor as T
    from theano.ifelse import ifelse

    nthFib = T.iscalar() #Symbolic variable indicating whicch Fibonaccci
                         #number is wanted
    zero = T.constant(0,dtype='int64') #Note: if unspecified, dtype=int8
    one = T.constant(1,dtype='int64')
  
    def fibo(i,j,ctr,trgt):
        retVals = ifelse(T.eq(ctr,zero),[one,zero,ctr+1],[i+j,i,ctr+1])
        return retVals,theano.scan_module.until(ctr >= trgt)

   
    fibs,_ = theano.scan(fn=fibo,
                         outputs_info=[one,zero,zero],
                         sequences=[],
                         non_sequences=nthFib,
                         n_steps = 100)

    fibFnc = theano.function([nthFib],fibs[1])



averages about 4.41E-03 seconds. If I increase the number of loops to 5000 (i.e. the 5000<sup>th<\sup> Fibonacci number), the first method increases its average time about 10X to 2.03E-01 sec, whereas the second method only increases its average time to 4.70E-03 seconds. So there is a clear advantage to using scan, even for the simplest of functions that do not even use the GPU.

So, how should scan be used? Right now, I'm only using the 5 arguments shown in the 2<sup>nd</sup> Fibonacci code. These arguments are:

1. **fn**: This is the function being looped over. It is given either as the name of some known function or, inline, as a lambda function.

2. **outputs_info**: When looping over a function, some arguments may be modified with each loop. **outputs_info** gives the initial values for those variables. The *n* values in **output_values** are the first *n* arguments for the function.

3. **sequences**: When looping over a function, there may be a predetermined sequence of inputs to be applied. **sequences** passes in these sequences for those variables. The *m* sequences given are assigned to the next *m* input variables. The length of the shortest sequence determines the number of times Theano will loop through the function.

4. **non_sequences**: When looping over a function, there may be some arguments that are passed to the function with exactly the same value over each loop. The *l* values in **non_sequences** are assigned to the final *l* arguments of the function.

5. **n_steps**: In the absence of **sequences**, **n_steps** determines the number of times Theano will loop over a given function. If **n_steps** and **sequences** are given, the scan may crash (need to find out if this is when n_steps > max sequence length, n_steps < min sequence length or in either case)

