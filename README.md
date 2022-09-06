## String metric

The custom string metric algorithm was created in order to have as much flexibility as possible when choosing the criteria for string similarity depending on the project and input. The basis for the creation of the custom metric was that the paper ["A Comparison of String Distance Metrics for Name-Matching Tasks"](https://www.cs.cmu.edu/~wcohen/postscript/ijcai-ws-2003.pdf) concluded the best-performing metric was a 'hybrid method' composed of multiple distances.

For example, a set of input where most company names are missing spaces to delineate substrings, such as ``itotemtechnologies`` and ``calfracwellservices`` would want to have a greater weighting of the token ratio (number of common characters) and the longest common substring ratio, than the Ratcliff/Obershelp ratio and the Regular expression ratio, which both look for common substrings and would be better suited for use cases where the order of the strings and number of strings are important. 

The threshold for a string match has been decided upon to be ``0.8`` (A specific weighting for the CAPP Location Change input has been detailed below)

The custom string metric algorithm takes in two strings and returns a float between ``0.0`` and ``1.0``, ``0.0`` being completely different and ``1.0`` being a complete match. 
```python
string_metric("string one", "string two") 
```

The float is equal to sum of each ratio after their individual weighting is assigned, e.g.:
```python
token_ratio = token_ratio * token_weight
```

```python
final_ratio = token_ratio + sequence_ratio + regEx_ratio + edit_ratio + substring_ratio + ratcliff_obershelp_ratio  
return final_ratio  
```

When comparing strings, it is important to note that metrics are assumed to be case sensitive, so ```iTOTEM Technologies``` and ```itotem technologies``` would not return ```1.0```. Therefore, when using the ```string_metric``` algorithm, both strings are cleaned using the ```clean_string``` function from the module before being compared. As ```string_two``` is being compared to ```string_one```, ```string_one``` is always assumed to be correctly spelt and the default spelling. 

The function makes uses of separate, pre-existing string metric libraries for each type, which are character/edit based, sequence based and token-based distances, in addition to Python's built in [```re```](https://docs.python.org/3/library/re.html) and [```difflib```](https://docs.python.org/3/library/difflib.html) modules. Because overall time for the algorithm was not a consideration (i.e., the script would be left to run overnight), when deciding upon libraries for each individual type of metric, focus was given to fulfilling specific criteria needed to allow the metrics to be as dynamic as possible.  

### Edit metric:

The character/edit based metric chosen was the Damerau-Levenshtein distance and the library used is ```fastDamerauLevenshtein```. The library was chosen because of its ability to assign different weightings to each type of edit operation. Edit distances are traditionally defined by counting the minimum number of operations required to transform one string into the other, and the Damerau–Levenshtein distance includes the most operations, those being insertion, deletion, substitution, and transposition (swapping) which is why it was chosen.  

The [formal mathematic definition of the Damerau–Levenshtein distance](https://en.wikipedia.org/wiki/Damerau%E2%80%93Levenshtein_distance#Definition) is expressed as:  
![](https://wikimedia.org/api/rest_v1/media/math/render/svg/d50fab8cc0233e2b1b5b420f72cb23fdf1d56c59)

The final number returned by the Damerau–Levenshtein distance can be found by [uing the piecewise function formula from above in a matrix](https://medium.com/@ethannam/understanding-the-levenshtein-distance-equation-for-beginners-c4285a5604f0)[ and selecting the value in the bottom right corner](https://www.lemoda.net/text-fuzzy/damerau-levenshtein/index.html):

![](https://miro.medium.com/max/716/1*xyoq20suqByW8wzlKe9O-A.png)



The Damerau-Levenshtein library used is [```fastDamerauLevenshtein```](https://pypi.org/project/fastDamerauLevenshtein/) and was chosen because of it's ability to assign different weightings to each type of edit operation. 

For example, within the edit distance matrix, it’s assumed a multiplier would be assigned depending on the type of operation occurring in each cell, affecting the resulting distance returned by the algorithm. An example of custom weight matrices: 
![](https://upload.wikimedia.org/wikipedia/commons/thumb/d/d1/Levenshtein_distance_animation.gif/1280px-Levenshtein_distance_animation.gif)

Importing this library requires installing the [Microsoft C++ Build Tools](https://visualstudio.microsoft.com/visual-cpp-build-tools/). Once Microsoft Visual Studio Installer is downloaded, [select the C++ build tools like so](https://docs.microsoft.com/en-us/answers/storage/attachments/34873-10262.png).

The ```damerauLevenshtein``` function will return a value between ```0.0``` and ```1.0```, if the ```similarity``` parameter is not set to ```false```, which it is by default.

By default, the value of each edit operation is 1. To change the cost of any operation, simply change the value assigned to the corresponding variable:
```python
deleteWeight = 1
insertWeight = 1
replaceWeight = 1
swapWeight = 1
```

The number returned by ```damerauLevenshtein``` is equal to the ```(length of the longest string - edit distance)/(length of the longest string)```.
This number is then assigned to the variable ```edit_ratio```

Additionally, a variant on the [Jaro–Winkler similarity](https://en.wikipedia.org/wiki/Jaro%E2%80%93Winkler_distance#Jaro%E2%80%93Winkler_similarity),

![](https://wikimedia.org/api/rest_v1/media/math/render/svg/03ce3597b133e80f611220e52ded597ce2ad6fbf)

is applied to the edit ratio if characters within a certain range are equal in both strings and simply increases the score of the edit ratio:

```python
starting_index = 0
ending_index = 3
scaling_factor = 0.10

if string_two[starting_index:ending_index] == string_one[starting_index:ending_index]:
  edit_ratio = edit_ratio + (scaling_factor) * (1 - edit_ratio)
```

The similarity is non-punitive if the characters within the indices do not match, and how much the similarity is worth is based on the ```scaling_factor``` variable. Note that the scaling factor [should not exceed 0.25 and the value commonly used is 0.1](https://en.wikipedia.org/wiki/Jaro%E2%80%93Winkler_distance#Jaro%E2%80%93Winkler_similarity). 

Once ```edit_ratio``` is assigned a final weighting:
```python
edit_weight = 0.2
edit_ratio = edit_ratio * edit_weight
```
___
### NOTE:
As using ```fastDamerauLevenshtein``` requires an extra install, which may indicate the library is becoming deprecated and that I was unable to verify that the weighting was actually implement by assigning a multiplier to each distance in the recursion because the code is not open source, when improving the script I recommend trying to find an alternative for this library. 




## CAPP string metric weighting

## Next steps:
