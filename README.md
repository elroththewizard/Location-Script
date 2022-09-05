## String metric

The custom string metric algorithm was created in order to have as much flexibility as possible when choosing the criteria for string similarity depending on the project and input. The basis for the creation of the custom metric was that the paper ["A Comparison of String Distance Metrics for Name-Matching Tasks"](https://www.cs.cmu.edu/~wcohen/postscript/ijcai-ws-2003.pdf) concluded the best-performing metric was a 'hybrid method' composed of multiple distances.

For example, a set of input where most company names are missing spaces to delineate substrings, such as `itotemtechnologies”, and “calfracwellservices”, would want to have a greater weighting of the token ratio (number of common characters) and the longest common substring ratio, than the Ratcliff/Obershelp ratio and the Regular expression ratio, which both look for common substrings and would be better suited for use cases where the order of the strings and number of strings are important. 



## Next steps:
