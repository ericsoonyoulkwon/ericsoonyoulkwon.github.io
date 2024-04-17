# Counting the occurrence of words in the sentence

#### Category: Practice Algorithms

---

Sample text is entered.

```python3
sentence = input()

Lorem ipsum dolor sit amet, consectetur adipisci elit, sed eiusmod tempor incidunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrum exercitationem ullam corporis suscipit laboriosam, nisi ut aliquid ex ea commodi consequatur. Quis aute iure reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint obcaecat cupiditat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.
```

Checking if a word is in the list is case-sensitive, so converting all words to lowercase is required.
Then the list of unique words is created.
Then count the occurrences of each word.
Finally, sort the dictionary by the counts in descending order to focus on the top n occurrence.

```python3
sentenceInLower = sentence.lower()
words = sentenceInLower.split()

wordCountDic = {}
for word in words:
    if word in wordCountDic:
        wordCountDic[word] += 1
    else:
        wordCountDic[word] = 1

sorted_wordCountList = sorted(wordCountDic.items(), key=lambda item: item[1], reverse=True)
# sorted_wordCountDic = dict(sorted(wordCountDic.items(), key=lambda item: item[1], reverse=True)) # if dictionary is preferred datatype of the result

print(sorted_wordCountList)
# print(sorted_wordCountDic) # if a dictionary is the preferred data type of the result
```

Result:
```python3
[('ut', 3), ('dolore', 2), ('quis', 2), ('in', 2), ('lorem', 1), ('ipsum', 1), ('dolor', 1), ('sit', 1), ('amet,', 1), ('consectetur', 1), ('adipisci', 1), ('elit,', 1), ('sed', 1), ('eiusmod', 1), ('tempor', 1), ('incidunt', 1), ('labore', 1), ('et', 1), ('magna', 1), ('aliqua.', 1), ('enim', 1), ('ad', 1), ('minim', 1), ('veniam,', 1), ('nostrum', 1), ('exercitationem', 1), ('ullam', 1), ('corporis', 1), ('suscipit', 1), ('laboriosam,', 1), ('nisi', 1), ('aliquid', 1), ('ex', 1), ('ea', 1), ('commodi', 1), ('consequatur.', 1), ('aute', 1), ('iure', 1), ('reprehenderit', 1), ('voluptate', 1), ('velit', 1), ('esse', 1), ('cillum', 1), ('eu', 1), ('fugiat', 1), ('nulla', 1), ('pariatur.', 1), ('excepteur', 1), ('sint', 1), ('obcaecat', 1), ('cupiditat', 1), ('non', 1), ('proident,', 1), ('sunt', 1), ('culpa', 1), ('qui', 1), ('officia', 1), ('deserunt', 1), ('mollit', 1), ('anim', 1), ('id', 1), ('est', 1), ('laborum.', 1)]
```
