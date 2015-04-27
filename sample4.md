Title: Theano Unittesting
Date: 2015-01-10 11:10
Category: Theano Notes

#Theano Unittesting

Unittesting for Theano is fairly straightforward, with 2 (that I know of caveats).To unittest, after installing a new version, just open a terminal and from the command line type (provided that one is not in theano's parent directory):

    ::::Text only
    me@Bedrock1:~$ python -c "import theano; theano.test()"

Then, wait about 60-90 minutes (add info about my machine & NVIDIA card). The two caveats are:

1. Do not do this from theano's parent directory. Otherwise you'll very quickly get an error message telling you to try again from a different directory.

2. Make certain that in your .theanorc file that device = cpu. If it is set to gpu, then the unittest will run with all sorts of errors and problems.

## Selective Unittesting 
### The Right Way - with theano-nose
If you are no further from your installation than Theano's parent directory, then one may use theano-nose for unit testing.

* <code>theano-nose --theano</code> or `theano-nose` will run all theano tests
* `theano-nose --batch` will run all theano tests in batches of 100. It gives prettier output, but there may be additional failed tests due to errors in copying memory.
* `theano-nose folder_name` will run every test in that folder. (Must the folder be a package? What about sub-folders?)
* `theano-nose test_file.py` will run every test in that file.
* `theano-nose test_file.py:test_TestClass` will run every test in that class
* `theano-nose test_file.py:test_method` will run that test


### My Clumsy Hacks To Selectively Unittest (before I knew about theano-nose methods)
###(Note: All of my selective testing was in a leaf directory. Some changes may be necessary when used for non-leaf directories)

Running all the unittests is time-consuming. In order to run the tests in just one leaf subdirectory/package, make the following additions to the __init__ file of that package:

    ::::python
    import theano.tests
    if hasattr(theano.tests, "TheanoNoseTester"):
      test = theano.tests.TheanoNoseTester().test
    else:
      def test()
        raise ImportError("The nose module is not installed. It is needed for Theano tests.")

(This snippet was stolen from theano.__init__).

Then, assuming that the theano.sandbox.cuda.tests package is the one being tested, one can test it in this way (provided that one is not in theano's parent directory):
    ::::Text only
    me@ARL-M6800:~/Projects/Theano/Feb05-Theano-git$ python -c "import theano; theano.sandbox.cuda.tests.test()"

Sometimes running all unittests of a package is also fairly time consuming. In order to run just one test in one package, add the following code to the __init__ file of the package with the test to be run:

    ::::python
    import os
    import re
    import theano.tests
    if hasattr(theano.tests, "TheanoNoseTester"):
       test = theano.tests.TheanoNoseTester().test
    else:
       def test():
	 raise ImportError("The nose module is not installed."
			   " It is needed for Theano tests.")

    localDir = os.path.dirname(os.path.abspath(__file__))

    #Get all tests in desired module
    def _getTests(module_name):
      f = open(os.path.join(localDir,module_name),'r')
      fl = f.readlines()
      f.close()

      testList = [x.strip()[4:] for x in fl if 'def ' in x and 'test_' in x]
      testList = [x for x in testList if x[0:5] == 'test_']
      testList = [x.split('(')[0].strip() for x in testList]

      return testList

    def buildExcludeList(keepModule, keepTest):
      #Get all test files in package
      testFiles = [x for x in os.listdir(localDir) if x[0:5] == 'test_' and x[-3:] == '.py']
      tests = _getTests(keepModule)

      excludeFiles = [re.compile(z) for z in testFiles if z != keepModule]
      excludeTests = [re.compile(z) for z in tests if z != keepTest]
      excludeList = excludeFiles + excludeTests

      return excludeList

Additionally, in the module theano.tests.main.py, change the function test, so that it now has _excludeList_ as an input parameter, with a default value of _None_:

    ::::python
    def test(self, verbose=1, extra_argv=None, coverage=False, capture=True,
             knownFailure=True, excludeList=None):

Then, make certain the _excludeList_ value gets placed in the configure variable (i.e. _cfg_) by adding the following line:

    ::::python
    if excludeList:
      cfg.exclude = excludeList #Note: It might be better to make this += excludeList, 
                                #      in case cfg somehow already has a list for exclude

This line should come just after _cfg_ is created.

Then, assuming the test to be run is test_gemm_subsample from test_conv_cuda_ndarray.py in the package theano.sandbox.cuda.tests, that single test can be run as follows:

    ::::Text only
    me@ARL-M6800:~/Projects/Theano/Feb05-Theano-git$ python
    Python 2.7.6 (default, Mar 22 2014, 22:59:56) 
    [GCC 4.8.2] on linux2
    Type "help", "copyright", "credits" or "license" for more information.
    >>> import theano.sandbox.cuda.tests
    >>> from theano.sandbox.cuda.tests import buildExcludeList
    >>> el = buildExcludeList('test_conv_cuda_ndarray.py', 'test_gemm_subsample')
    >>> theano.sandbox.cuda.tests.test(capture = False, excludeList = el)



NOTE: These were all only used and tested on the examples shown. Problems may arise if

1. The package involved is not a 'leaf' package.
2. The chosen test belongs to a class and is not a stand alone function.
