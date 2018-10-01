---
layout: post
title: "Developing Unit Tests for my Python Applications"
date: 2018-10-01
---

I built many of my applications out of desires for experimentation rather than implementation. That does not reduce their need for unit testing, however. Therefore, I am going through and adding tests to several of them. Given the variation in complexity, some tests will get implemented faster than others (I am getting to the simpler functions first). Read on if you are interested in how I add tests to my applications. 

## Mimic Master
The tests I am making for my <a href="/2018/08/24/mimic-master-pitchcurve-vs-fingerprint.html">Mimic Game</a> can be referenced <a href="https://github.com/a-n-rose/mimic-master-how-well-can-you-mimic/tree/master/pitch_curve">here</a>.

Note, in order to run the tests easily in the commandline, e.g.:
```
$ python3 test_compare_signals.py
```
I include the following in all of my test scripts.:
```
if __name__=="__main__":
    unittest.main()
```

### test_analyse_audio.py

analyse_audio.py is a script that contains functions which allow me to analyse sound signals. Below, I will cover compare_signals.py, which runs a series of those functions to compare two signals. 

I have generated one test so far, for analyse_audio.py, a test that verifies a function takes the power of two of a matrix. I still have some exception cases to add, but here is the test so far.

```
import numpy as np
import unittest
import analyse_audio

class TestCalc(unittest.TestCase):
    
    def test_stft2power(self):
        #test positive floats
        matrix = np.random.rand(10,4)
        self.assertEqual(analyse_audio.stft2power(matrix).all(),(np.abs(matrix)**2).all())
        #test positive and negative floats
        matrix = np.random.randn(10,4)
        self.assertEqual(analyse_audio.stft2power(matrix).all(),(np.abs(matrix)**2).all())
        #test high and low positive and negative floats
        matrix = np.random.uniform(low=-100.0,high=100.0,size=(10,4))
        self.assertEqual(analyse_audio.stft2power(matrix).all(),(np.abs(matrix)**2).all())
        #test high and low positive and negative integers:
        matrix = np.random.randint(low=-100.0,high=100.0,size=(10,4))
        
        
        #test for empty or None input
        #self.assertRaises(TypeError,analyse_audio.stft2power,[])
        #self.assertRaises(TypeError,analyse_audio.stft2power,None)
        
        #or via context manager
        with self.assertRaises(TypeError):
            analyse_audio.stft2power([])
        with self.assertRaises(TypeError):
            analyse_audio.stft2power(None)
        
if __name__ == '__main__':
    unittest.main()
```

### test_compare_signals.py

The function I started testing first in compare_signals.py was the match_len function. This function is currently necessary for comparing signals' pitch curves. The pitch values of the two signals compared need to have the same length in order to apply the Hermes Weighted Coefficient. (For reference to this coefficient and how I compared similarity in general, see <a href="https://a-n-rose.github.io/2018/08/29/comparing-prosody.html">this post</a>).

This test ensures that matrices match the length to the matrix with the shortest length. Furthermore, if the two matrices have variable column length, it asserts a raised ValueError.

```
import numpy as np
import unittest
import compare_signals

class TestCalc(unittest.TestCase):
    
    def test_match_len(self):
        matrix_1 = np.random.rand(20,4)
        matrix_2 = np.random.rand(18,4)
        matrix_3 = np.random.rand(3,4)
        matrix_4 = np.random.rand(20,2)

        matrix_list1 = [matrix_1,matrix_2,matrix_3]
        matrix_list2 = [matrix_1,matrix_2,matrix_3,matrix_4]
        
        matrix_matched = [3,3,3]
        _, lens_list = compare_signals.match_len(matrix_list1)
        
        self.assertEqual(lens_list,matrix_matched)
        
        #test for not matching columns
        #self.assertRaises(ValueError,compare_signals.match_len,matrix_list2)
        
        #or via context manager
        with self.assertRaises(ValueError):
            compare_signals.match_len(matrix_list2)

        
if __name__ == '__main__':
    unittest.main()

```

## Next steps

The implementaton of more tests is a work in progress and I will steadily add to these tests as well as tests for my other applications. 
