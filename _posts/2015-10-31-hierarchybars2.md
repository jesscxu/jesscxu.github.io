---
author: Jessica Xu
layout: post-full
title: Another Hierarchical Bar Graph
featimg: hierarchybars2_preview.png
type: visualization
tags: [d3, visualization] 
category: [visualization]
vizlink: hierarchybars2.html
---
In this case, I wanted to look at each frequently occuring term separately in the data set as a whole. For each term, I look at all text answers that included that race/ethnicity and broke it down into all the other races that appeard with it. 


{% highlight python %}
"""
data --> dictionary of all answers/reponses with frequencies as values
frequencies --> a sorted list of most frequently appearing races/ehtnicities
num --> how many of these most frequently appearing races to subdivide

Returns a dictionary put in a specific format to work with the D3 javascript code.
"""
def hierarchybars(data, frequencies, num, bound):
    data_dict = {}
    data_dict["name"] = "DATA"
    data_dict["children"] = [] 
    for i in range(num):
        outer_dict = {}
        outer_dict["name"] = frequencies[i][0]
        outer_dict["children"] = []

        # dw: dict with race frequencies[i][0]
        # dwo: dict without race frequencies[i][0]
        # freq: frequency of all words in dw 
        # not_freq: frequency of all words in dwo 
        # total: all answers/reponses that have only race frequencies[i][0]

        dw, dwo, freq, not_freq, total = find(data, frequencies[i][0])

        if total > 0:
            data_dict["children"].append({"name": outer_dict["name"], "size": total})
        for f in freq:
            if f[1] > bound:
                inner_dict = {}
                inner_dict["name"] = f[0]
                inner_dict["children"] = []
                freq_words = {}
                for key in dw.keys():
                    number = dw[key]
                    words = key.split(" ")
                    
                    if (f[0] in words):         
                        words.remove(f[0])
                        for w in words:
                            if w in freq_words.keys():
                                freq_words[w] += number
                            else:
                                freq_words[w] = number

                for k in freq_words.keys():
                    inner_dict["children"].append({"name": k, "size": freq_words[k]})
                outer_dict["children"].append(inner_dict)
            else:
                outer_dict["children"].append({"name": f[0], "size": f[1]})  

        data_dict["children"].append(outer_dict)

    # Put the remaining most common race/ethnicities in my
    # visualization that are not already included. 

    all_single_data = []
    for d in data_dict["children"]:
        all_single_data.append(d["name"])
    for m in most_common_freqs:
        if m[0] not in all_single_data:
            data_dict["children"].append({"name": m[0], "size": m[1]})

    return data_dict


hierarchy = hierarchybars(DATA, sorted_frequencies, 10, 10)

{% endhighlight %}

<br>

Link to formatted JSON data: [hierarchical bar graph data](/json/hierarchybars2.json)

Link to organized survey data set: [survey data](/json/survey_data.json)

