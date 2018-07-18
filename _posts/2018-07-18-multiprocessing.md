---
layout : post
title : Simple Code in Python 2 for multiprocessing
tags : code
---

Recently, I have the need to parallelize my code, especially for cross validation process in machine learning as I could not use sklearn's built in CV functions due to specific reason. I have no prior knowledge about multiprocessing and would gracefully admit that my coding skills are not splendid. However, due to the need of finishing my research project, I recently bumped into a post on stackoverflow which taught me how to parallelize nested loops in a pretty simple way. Thanks to Chris Arndt who posted his solution [here] (https://stackoverflow.com/questions/6974695/python-process-pool-non-daemonic). Here, I will briefly explain the code in a cross validation setting.

Due to some reason related to `daemon` (explained in the post), we couldn't set up `Pool()` twice for each loop. Hence, we will need to define a class which overcomes that.

```
import multiprocessing
import multiprocessing.pool as pool

#These are directly copied from the post, credits to Chris!
class NoDaemonProcess(multiprocessing.Process):
    # make 'daemon' attribute always return False
    def _get_daemon(self):
        return False
    def _set_daemon(self, value):
        pass
    daemon = property(_get_daemon, _set_daemon)

# We sub-class multiprocessing.pool.Pool instead of multiprocessing.Pool
# because the latter is only a wrapper function, not a proper class.
class MyPool(pool.Pool):
    Process = NoDaemonProcess
```

Say you want to parallelize two loops, first loop iterates over possible parameters you would like to experiment for your model, and 2nd loop iterates over different folds. You could do the following:

```
import multiprocessing
import itertools
from multiprocessing import Pool

#Assume possible_parameters is the list of parameters you would 
#like to try and tt_indices are the indices for each 
#fold e.g. return by StratifiedKFold()

def inner_loop(idx, model, X,y):
    #Do something...

    return accuracy

#For multiprocessing inner_loop
def inner_loop_multirun(args)
    return inner_loop(*args)

def outer_loop(param, X,y,model):
    tt_indices = StratifiedKFold(5).split(X,y)   #train test split indices

    #multiprocess inner loop
    n_jobs=None
    pool = Pool(n_jobs) #Default is None -> all cores

    #This returns a list where each element has the format of 
    #the object returned by inner_loop i.e. accuracy in this example
    inner_loop_returned = pool.map(inner_loop_multirun, 
            itertools.product([idx for idx in tt_indices], [model], [X], [y]))
        
    #Do something
    
    return accuracy_list

#For multiprocessing outer_loop
def outer_loop_multirun(args):
    return outer_loop(*args)

def main():
    #Do something

    n_jobs=None
    myPool = MyPool(n_jobs)
    
    #This returns a list where each element has the format of 
    #the object returned by outer_loop i.e. accuracy_list in this example
    outer_loop_returned = myPool.map(outer_loop_multirun, 
        itertools.product([param for param in possible_parameters], [X], [y], [model]))


if __name__ == "__main__":
    main()
 
```

Although it doesn't look nice and a little hacky due to this part `itertools.product(...)` of the code, but it works for me and I am able to achieve what I wanted. :)
