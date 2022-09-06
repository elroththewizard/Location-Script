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

When comparing strings, it is important to note that metrics are assumed to be case sensitive, so ```iTOTEM Technologies``` and ```itotem technologies``` would not return ```1.0```. Therefore, when using the ```string_metric``` algorithm, both strings are cleaned using the ```clean_string``` function from the module.  

The function makes uses of separate, pre-existing string metric libraries for each type, which are character/edit based, sequence based and token-based distances, in addition to Python's built in ```re``` and ```difflib``` modules. Because overall time for the algorithm was not a consideration (i.e., the script would be left to run overnight), when deciding upon libraries for each individual type of metric, focus was given to fulfilling specific criteria needed to allow the metrics to be as dynamic as possible.  

### Edit metric:

The character/edit based metric chosen was the Damerau-Levenshtein distance and the library used is ```fastDamerauLevenshtein```. The library was chosen because of its ability to assign different weightings to each type of edit operation. Edit distances are traditionally defined by counting the minimum number of operations required to transform one string into the other, and the Damerau–Levenshtein distance includes the operations insertion, deletion, substitution, and transposition (swapping).  

The [formal mathematic definition of the Damerau–Levenshtein distance]([Damerau–Levenshtein distance - Wikipedia](https://en.wikipedia.org/wiki/Damerau%E2%80%93Levenshtein_distance#Definition)) is expressed as:  
![](https://wikimedia.org/api/rest_v1/media/math/render/svg/d50fab8cc0233e2b1b5b420f72cb23fdf1d56c59)


## CAPP string metric weighting

## Next steps:
