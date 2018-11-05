---
layout: post
category : lessons
tagline: "Supporting tagline"
tags : [intro, beginner, jekyll, tutorial]
---
{% include JB/setup %}

{% highlight c %} int i = 5; {% endhighlight %}

Example with Markdoown redcarpet:

``` python
import logging

class Sort(object):
    """ Sorting algorithms """

    def __init__(self, logger=None):
        self.logger = logger or logging.getLogger(__name__)

    def bubbleSort(self, values, enhanced=False):
        """ Executing bubble sort algorithm on the list 'values'. 'values' is passed by reference! '"""
        self.logger.info("Executing bubble sort with enhancements=%s" % enhanced)
        swaps = True
        for pas in range(1,len(values)):
            self.logger.debug("[Bubble Sort] Passage %d" % pas)
            if enhanced:
                maxpas = len(values) - pas
                if swaps is not True:
                    return
                else:
                    swaps = False
            else:
                maxpas = len(values) - 1
            for n in range(0, maxpas):
                if(values[n] > values[n+1]):
                    values[n], values[n+1] = values[n+1], values[n] # swap
                    self.logger.debug("[Bubble Sort] Swap %d" % (n+1))
                    swaps = True
```
