# iTOTEM Location Module Documentation

The module contains functions for various location validation use cases.

## Google Search

The ```google_search``` function takes in a ```company name```, ```location```, ```region``` and Google Places API Text Search ```next_page_token```, all as strings. The function makes use of the [Google Maps Platform Text Search](https://developers.google.com/maps/documentation/places/web-service/search-text) from the Places API, and requires an API key to be generated from the Google Cloud console. (Note that Google recommends storing the API on the cloud)

Each string the function takes in as a parameter is used within the HTTP URL like so: 
```https://maps.googleapis.com/maps/api/place/textsearch/output?parameters```

```location```, ```company_name``` and ```region``` are all used within the initial request like so:
```python
user_input = company_name + "%20" + location + location_bias + "&key=" + KEY
url = "https://maps.googleapis.com/maps/api/place/textsearch/json?query=" + user_input
```
Any ```&``` characters within ```company_name``` are converted to the string ```and```, as ```&``` is used to separate query elements. Alternatively, the encoding ```%24``` could also be used as an escape character. 

The location bias variable passes in a set of longitude and latitude coordiantes and and a radius expressed in meters, which is accessed if the ```region``` string passed in exists within the ```region_lat_long``` dictionary, for example:
```python
region_lat_long['bc'] = "&location=53.7267%2C-127.6476&radius=1200000"
region = 'bc'
if region in region_lat_long:
  location_bias = region_lat_long[region]
```
Spaces within ```company_name``` and ```location``` are replaced with the encoding ```%20``` for the URL. 

When using the ```google_search``` function, in order to obtain the maximum results possible from the Places API, the function must be called multiple times, with the ```location``` parameter modified each call to include new and different information. 

For example with:

```python
company_name = "iTOTEM"
postal_code = "V7M 3N3"
location = "North Vancouver"
region = "bc"
```

```location``` would have to once be run as just ```North Vancouver```, then ```North Vancouver bc```, then ```North Vancouver bc V7M 3N3```, and then this would be repeated with ```bc``` as ```british columbia``` instead.

It is also important to note that the registrey containing the result from the Google Places API is different than the registrey containing the results of a manual Google search via a web browser. 

The API results are formatted and returned as a json request:
```python
response = requests.get(url)
json_response = response.json()
```
```python
{'html_attributions': [], 'results': [{'business_status': 'OPERATIONAL', 'formatted_address': '124 W 1st St #102, North Vancouver, BC V7M 3N3, Canada', 'geometry': {'location': {'lat': 49.31251229999999, 'lng': -123.0796238}, 'viewport': {'northeast': {'lat': 49.31378947989271, 'lng': -123.0783167701073}, 'southwest': {'lat': 49.31108982010727, 'lng': -123.0810164298927}}}, 'icon': 'https://maps.gstatic.com/mapfiles/place_api/icons/v1/png_71/generic_business-71.png', 'icon_background_color': '#7B9EB0', 'icon_mask_base_uri': 'https://maps.gstatic.com/mapfiles/place_api/icons/v2/generic_pinlet', 'name': 'Itotem Technologies', 'place_id': 'ChIJqw6BiUxwhlQRnqbXPeGRucM', 'plus_code': {'compound_code': '8W7C+25 North Vancouver, British Columbia', 'global_code': '84XR8W7C+25'}, 'rating': 0, 'reference': 'ChIJqw6BiUxwhlQRnqbXPeGRucM', 'types': ['point_of_interest', 'establishment'], 'user_ratings_total': 0}], 'status': 'OK'}
```
And the results are accessed with the key:
```python
results = json_response['results']
```
```python
[{'business_status': 'OPERATIONAL', 'formatted_address': '124 W 1st St #102, North Vancouver, BC V7M 3N3, Canada', 'geometry': {'location': {'lat': 49.31251229999999, 'lng': -123.0796238}, 'viewport': {'northeast': {'lat': 49.31378947989271, 'lng': -123.0783167701073}, 'southwest': {'lat': 49.31108982010727, 'lng': -123.0810164298927}}}, 'icon': 'https://maps.gstatic.com/mapfiles/place_api/icons/v1/png_71/generic_business-71.png', 'icon_background_color': '#7B9EB0', 'icon_mask_base_uri': 'https://maps.gstatic.com/mapfiles/place_api/icons/v2/generic_pinlet', 'name': 'Itotem Technologies', 'place_id': 'ChIJqw6BiUxwhlQRnqbXPeGRucM', 'plus_code': {'compound_code': '8W7C+25 North Vancouver, British Columbia', 'global_code': '84XR8W7C+25'}, 'rating': 0, 'reference': 'ChIJqw6BiUxwhlQRnqbXPeGRucM', 'types': ['point_of_interest', 'establishment'], 'user_ratings_total': 0}]
```

The results are then iterated through and numbered and saved in a list ```search_matches``` 
```python
for i in enumerate(results):
  iteration = i[0]
  name = (results[iteration])['name']
  formatted_address = (results[iteration])['formatted_address']
  formatted_address = str(formatted_address)
  formatted_address = formatted_address.replace('{', '')
  formatted_address = formatted_address.replace('}', '')
  search_matches.append("Result " + str(iteration + 1) + ". " + name + "," + formatted_address)
```
For API searches where there are more than 20 results, the json request returns a ```next_page_token```, which looks like:
```python
'next_page_token': 'AeJbb3cFAPZ--HJgV37WCas2vzMoz_z0MZKNldkBUm1-_dPBGGwY2urEoWBIE0Y0xSfBqa1TPMnbUaDQ0jbGMLN3vsbke2PqFF7lrmMjSbI0_iE97rusKWxuvRMhZdzHEJalmE9_RBaYZ--nfyf3SnibbJ2Ad93fHh9fpCbb7EUwy_nc47qmDUG1KnElzFKI8e47cFK9g6uf0L83wLz9_RfplBaYaQWOKaNyewewoqHdVYjh5NJCZJKekN_VBdDY7q-HyKX_lptluyrBm9sldk0Q2mzd9NS0mTFvo9rEFXKLzPCsjf8JkLRmaPjWL6lKD2O-KwR5yb7_kY8JBYoKstKPG6ugMmQSLr411Pe3UcO7zOJsNK5pgu8EDZ7Vl-FSqxxokY3zaMHRV6tA3m6UsBfn8LOvOiIIXoBrT0Bky2QV09CBHs7ehnGDWS5JrHc5hT8FCTig644biFwWCxdP3vYISQlWgQ'
```
And is used in the url as:
```python
url = "https://maps.googleapis.com/maps/api/place/textsearch/json?pagetoken=" + next_page_token + "&key=" + KEY
```

The ```try``` block continues to check for the ```next_page_token``` is it exists and appends the additional results to the original list:
```python
    try:
        next_page_token = json_response['next_page_token']
        token_status = True
        result_num = len(search_matches)
        while token_status is True:
            if next_page_token is not None: 
                time.sleep(5)  # Needed for API search cool down
                additional_results = google_search('', '', '', next_page_token)
                search_matches.append({'next_page_token': next_page_token})
                for x in additional_results:
                    if 'Result' in x:
                        period_index = x.find('.')
                        result_num = result_num + 1
                        x = x.replace(x[7:period_index], str(result_num))
                        search_matches.append(x)
                    else:
                        search_matches.append(x)
                next_page_token = (search_matches[-1])['next_page_token']
            else:
                token_status = False
        return search_matches
```

The ```time.sleep(5)``` is needed for the API to have time to generate the additional results

If the ```next_page_token``` is ```None``` during the first check, a ```KeyError``` is given, which we catch with the ```except``` block:
```python
    except KeyError:
        next_page_token = None
        search_matches.append({'next_page_token': next_page_token})
        return search_matches
```



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
___

### Sequence based metrics:
Sequence based similarities search for the longest common sequence within both strings and typically return a greater value the greater the number of common sequences found. The difference between a substring and a subsequence is that the characters in a substring need to be contiguous, whereas the characters within a subsequence do not. However, in both the character order does matter, differentiating the similarity from token-based similarities.  


The Regular Expression module used is Python's built-in [re](https://www.w3schools.com/python/python_regex.asp) package. The module is used to check how many common substrings from the input string exist in the comparison string. The module is used in combination with the [SequenceMatcher class from the difflib library](https://docs.python.org/3/library/difflib.html), which itself states that it functions differently than the Ratcliff/Obershelp algorithm. The intention behind combining the re module and the SequenceMatcher class was to create a variation the Ratcliff/Obershelp algorithm that does not [check exclusively for matches on the right or left side of the common substring](https://en.wikipedia.org/wiki/Gestalt_Pattern_Matching#Algorithm).

```seqMatch``` is created as ```SequenceMatcher``` object, with ```None``` passed in for the ```isjunk``` parameter, and copies of ```string_one``` and ```string_two``` are passed as ```a``` and ```b```. While the official documentation say that the argument is optional, omitting it resulted in an ```IndexError: string index out of range``` error.  This is based off code from [SequenceMatcher in Python for Longest Common Substring](https://www.geeksforgeeks.org/sequencematcher-in-python-for-longest-common-substring/)

```python
seqMatch = SequenceMatcher(None, regEx_string_one, regEx_string_two)
```

A match is then searched for and stored in ```match``` using the ```find_longest_match``` function. While a match exists within both strings, the search continues and the number of characters within the second string that belong to the matching substring are stored in ```len_regEx_character_matches```, and the matching substring is then removed so the search can continue:

```python
while (match.size != 0):
  #  .a is the starting index of the match and .size is the length of the matching string
  sub_string_match = regEx_string_one[match.a: match.a + match.size]
  num_regEx_matches = len(re.findall(sub_string_match, regEx_string_two))
  len_regEx_character_matches = (len_regEx_character_matches + num_regEx_matches * len(sub_string_match))
  regEx_string_one = regEx_string_one.replace(sub_string_match, '')
  regEx_string_two = regEx_string_two.replace(sub_string_match, '')
  match = seqMatch.find_longest_match(0, len(regEx_string_one), 0, len(regEx_string_two))
```

```regEx_ratio``` is then just set to the number of matching substring characters over the total length of the second string and is assigned a final weighting: 

```python
regEx_ratio = (len_regEx_character_matches) / len(string_two)
regEx_weight = 0.2
regEx_ratio = regEx_ratio * regEx_weight
```

The other library used for sequence similarity is [TextDistance](https://pypi.org/project/textdistance/), which contains algorithms for various string metrics.  

All three sequece based algorithms from the library can be used, however the Regular Expression and SequenceMatcher algorithm may already cover sequence-based distances. Regardless, code implementing the algorithms is included and the weighting given is just set to zero. 

```python
sequence_ratio = textdistance.lcsseq.normalized_similarity(string_one, string_two)
``` 

Returns a value equal to ```1 - (the length of the longest input string - length of the longest common subsequence)/length of the common subsequence```

Similarly:

```python
substring_ratio = textdistance.lcsstr.normalized_similarity(string_one, string_two)
```

Returns a value equal to ```1 - (the length of the longest input string - length of the longest common substring)/length of the common substring```

The final sequence-based algorithm from the ```textdistance``` module is the [```ratcliff_obershelp``` similarity](https://en.wikipedia.org/wiki/Gestalt_Pattern_Matching). As mentioned above, the ```ratcliff_obershelp``` similarity [checks exclusively for matches on the right or left side of the common substring after removing the common substring from the string](https://en.wikipedia.org/wiki/Gestalt_Pattern_Matching#Algorithm) based on the formula:

![](https://wikimedia.org/api/rest_v1/media/math/render/svg/97592a38a8687e4a86309ad559f31bc0369cf977)

And returns a value between 0 and 1 equal to: ```(2 * the number of matching characters)/(total number of characters within both strings)```, where ```number of matching characters``` is equal to ```the longest common substring, plus recursively the number of matching characters in the non-matching regions on both sides of the longest common substring (matching characters on the left and right)```.

This means that ```string_one = technolgies itotem``` and ```string_two = itotem technolgies``` would only return ```0.6111111111111112```. 

Much like the ```Jaro–Winkler similarity``` applied to the ```edit ratio```, the Ratcliff-Obershelp similarity should be given a higher weighting if we are prioritizing company names that have a common prefix whose substrings are in the same order.  

### Token based metrics:
The final type of string metric used is a token-based algorithm from the ```textdistance``` module, the [Jaccard index](https://en.wikipedia.org/wiki/Jaccard_index). Token-based distances use a set of tokens as input instead of complete strings and find the number of similar tokens within both sets. The greater the number of common tokens, the greater the similarity. The Jaccard index works using this formula: 

![](https://miro.medium.com/max/406/1*wDOEGSMvUMzHGC45tgLAcw.png)

Where the number of common tokens is divided by the number of unique tokens, which is also equal to the number of common tokens divided by the length of the two strings minus the common tokens. Essentially, the way the Jaccard index is being used here is just finding the number of common characters between two strings, so: 

```python
textdistance.jaccard("technolgies itotem","itotem technolgies")
```
Returns ```1.0```.

The reason why the Jaccard index was chosen as the token based metric is due to its popularity and accessible formula, and because according to the article [String similarity — the basic know your algorithms guide!](https://itnext.io/string-similarity-the-basic-know-your-algorithms-guide-3de3d7346227), the other popular token-based metric, the Sørensen–Dice coefficient, will always overestimate the similarity between two strings because it's denominator is the total number of tokens, not the total number of unique tokens. 

### A quick summary of the different use cases for each string metric when determining the weighting is as follows:

**Edit distance**:  Higher weighting when we want the strings to be nearly identical 

**Regular expression**: Higher weighting when we want the strings to be nearly identical, i.e., more common substrings **

**Longest common subsequence/substring**: Higher weighting when we want to prioritize the longest common subsequence/substring and don’t necessarily need the strings to be completely identical 

**Ratcliff/Obershelp**: Higher weighting when we want the strings to be identical, including order  

**Token**: Higher weighting when we want to prioritize the length of the strings matching and don’t necessarily need the strings to be completely identical 

## Example CAPP string metric weighting

Sources:
https://www.w3schools.com/python/python_regex.asp  
https://www.baeldung.com/cs/fuzzy-search-algorithm
https://www.baeldung.com/cs/string-similarity-edit-distance 
https://www.kdnuggets.com/2019/01/comparison-text-distance-metrics.html 
https://www.baseclass.io/newsletter/jaro-winkler  
https://www.learndatasci.com/glossary/jaccard-similarity/#:~:text=The%20Jaccard%20similarity%20measures%20the,of%20observations%20in%20either%20set.
https://en.wikipedia.org/wiki/Tf%E2%80%93idf
https://en.wikipedia.org/wiki/Cosine_similarity
https://en.wikipedia.org/wiki/Jensen%E2%80%93Shannon_divergence
https://en.wikipedia.org/wiki/String_metric
https://pypi.org/project/python-Levenshtein/ 
https://maxbachmann.github.io/Levenshtein
https://en.wikipedia.org/wiki/Edit_distance  
https://en.wikipedia.org/wiki/String_metric  
https://www.baeldung.com/cs/string-similarity-sequence-based   
https://www.baeldung.com/cs/string-similarity-token-methods 
https://en.wikipedia.org/wiki/Damerau%E2%80%93Levenshtein_distance  
## Next steps:
1. "Hunter's rules" - common abbreviations and database clean up - see clean string
2. Access longitude and latidue from json results and implement more libraries: https://pypi.org/project/pgeocode https://developers.google.com/maps/documentation/geocoding/start#:~:text=The%20Geocoding%20API%20is%20a,Client%20for%20Google%20Maps%20Services (separate API key)
3. Develop list of weights for use case and keep testing the string metric function
4. Use mapping table as PostalCodeDatabase.csv input for verify_location
5. Create an accurate price estimate: https://developers.google.com/maps/billing-and-pricing/billing#billing-overview
6. Create UI for QAin process - ideally get to the point where the weighting could be changed from the UI by non programmers
