---
layout: post
title: "Fuzzy Matching For Double Data Entry"
date: 2019-08-29
---

<par> In my past position at [Charity Science Health](https://www.charitysciencehealth.com/) I was given the problem to solve related to managing and reconciling the double entry system for our cold renrollment program. </par> 

<par> We had been given access to records of patients in a large number of hospitals and for the sake of ensuring data accuracy used two data entry operators per register. They recorded the entires using the google drive google sheets application. The idea was that in doing double entry on separate sheets we could catch mismatches (which cropped up from entering in hospital records on paper) easily and reconcile the difference for the sake of accuracy. </par>

<par> The *problem* was there was hundreds of thousands of entries to be made, a limited time schedule and limited number of data entry operators. Often times name fields, for the patients name or village name the entry was not entered in exactly by both operators causing the operator to spend more time on reconciling the data. The *solution* I developed was to use a fuzzy matching system in JavaScript to ignore differences in the double entry sheets at a threshold. This was to be used on columns that contain names, for example if one operator entered in “Christopher” and another “Kristopher” given a threshold of 2 or more we would not highlight it as a mistake. For other columns such as contact numbers this was not used but typically these problems did not arise because many of the names were written in Hindi script and then entered in Latin script. The data entry operators may have spelt the same name phonetically but with different letters. The solution was DBLENTRYMATCH a google sheets JavaScript function, the reason it was written in JavaScript was because the entry operators were using google sheets to enter data live. If this were doe without JavaScript it would require manually exporting and reimporting the data, something that would increase the amount of time wasted by the entry operators. </par>

<par> All of the following functions use the Levenshtein distance which is the number of character changes a string must go through to become the target string. This is a relatively useful method for fuzzy matching for typos. </par>



### Descriptions:

 ##### =DBLENTRYIGNORE(cell1, cell2, distance, “separator”)

```javascript
function DBLENTRYIGNORE(str1, str2, distance, seperator) {
  if (str1 === null && str2 === null) return 'Not entered';
  if (str1 === null || str2 === null) return 'Only one entry';
  str1 = String(str1); str2 = String(str2);

  var lv = levenshtein(str1, str2)
   
   if (distance >= lv) {
     return ""
   } else {
    return str1 + " " + seperator + " "  + str2
   }
  
}

```

<par> Given cell1 and cell2 cell the comparison entries and a numerical string distance (I set this to 3 but you can change it in function). It will generate an empty cell if the strings are no more than the specified numerical distance of characters different. If not it will output the two cells side by side of with the chosen separator between them.

 ##### =DBLENTRYMATCH(cell1, celldistance, “separator”)

```javascript
function DBLENTRYMATCH(str1, str2, distance, seperator) {
  if (str1 === null && str2 === null) return 'Not entered';
  if (str1 === null || str2 === null) return 'Only one entry';
  str1 = String(str1); str2 = String(str2);

  var lv = levenshtein(str1, str2)
   
   if (distance >= lv) {
     return str1
   } else {
    return str1 + " " + seperator + " "  + str2
   }
   
}
```


<par> This is the same as above except it will fill the cell with cell1 if the cells are no more than the specified number of character distance apart. </par>


<par> The second problem that came up was that running a function on each cell was time consuming and crashed the scripts in google sheets. Google put a limit on 1000 custom functions in a second which limited our ability to use DBLENTRYMATCH on all the necessary cells. Instead I generated a function which ran on an array of values and outputted the descrepencies. </par>

###### DBLENTRYARRAY(str1, str2, distance, seperator)

<par> The str1 and str2 inputs can handle vectors of any length and will compare each vector and input in a column below the function. </par> 

```javascript
function DBLENTRYARRAY(str1, str2, distance, seperator) {
  var output = [];
  var index = 0
  for  (index = 0; index < str1.length; ++index) {
    if (str1[index] === null && str2[index] === null) return 'Not entered';
  if (str1[index] === null || str2[index] === null) return 'Only one entry';
  var input1 = String(str1[index]); 
    var input2 = String(str2[index]);

  var lv = levenshtein(input1, input2)
   
   if (distance >= lv) {
     output.push("");
   } else {
    output.push(input1 + " " + seperator + " "  + input2)
   }
  }
  return output
}
```
