---
title: How to create YAML files
published: 2022-09-12
description: I'm not going to explain what YAML is, nor its uses, if you want to learn how to create one, I'm pretty sure you already have that information.
tags: [Docker, Server, Linux, Hosting]
category: Server
draft: false
---
So let's get straight to the heart of the matter.

Let's start with basic syntax:

```
# Remember "Key: Value". This key and value pair is called a map.

# String variables
vegetable: "Lettuce"
fruit: "apple"
animal: 'dog'

# String in multiple lines
dutch: | 
 I have a plan, Arthur,
 and this one is a good one.
# Write a single line in multiple lines
Multiline-message: >
 this will 
 all be 
 in one single line.
# Same as
message: !!str this will all be in one single line
#Numbers
number: 5473
marks: 98.76
# Boolean
booleanValue: !!bool No # n, N, false, False, FALSE
# Same for true -> yes, y, Y

# Specify the type
zero: !!int 0
positiveNum: !!int 45
negativeNum: !!int -45
binaryNum: !!int 0b11001
octalNum: !!int 06574
hexa: !!int 0x45
commaValue: !!int +540_000 # 540,000
exponential numbers: 6.023E56
---
# Floating point numbers
marks: !!float 56.89
infinite: !!float .inf
not a num: .nan

# Null
surname: !!null Null # or null NULL ~
~: this is a null key

# Dates and time
date: !!timestamp 2002-12-14
random time: 2001-12-15T02:59:43.10 +4:00
no time zone: 2001-12-15T02:59:43.10
```

Now, the good stuff, sequence and maps:

```
 cities: !!seq
 - new york
 - madrid
# Same as
cities: [new york, madrid]
# If some of the keys of the seq will be empty, it is a sparse sequence
sparse seq:
 - hey
 - how
 - 
 - Null
 - sup
# Example of a nested sequence
- 
 - mango
 - apple
 - banana
-
 - marks
 - roll num
 - date
# nested mappings: a map within a map
name: Michael De Santa
role:
  age: 45
  job: bank robber
  
# Same as
name: Michael De Santa
role: { age: 45, job: bank robber}

# Pairs: keys may have duplicate values
# !!pairs

pair example: !!pairs
 - job: student
 - job: teacher

# same as
pair example: !!pairs [job: student, job: teacher]
# this will be an array of hashtables

# !!set will allow you to have unique values
names: !!set
 ? Franklin
 ? Michael
 ? Trevor

# Dictionary !!omap
people: !!omap
  - person1:
     name: Franklin
     age: 25
     height: 1.83
  - person2:
     name: Trevor
     age: 46
     height: 1.86
# I already cover yaml archor in my Docker-Compose guide, 
# but I will be covering it here again
likings: &likes
  fav fruit: mango
  dislikes: grapes

person1:
  name: Michael
  <<: *likes

person2:
  name: Trevor
  <<: *likes
  dislikes: berries # This overrides the value of "dislikes" in the archor

# This will look like
person1:
  name: Michael
  fav fruit: mango
  dislikes: grapes

person2:
  name: Trevor
  fav fruit: mango
  dislikes: berries
```

That's all. If you want something more complex... do it! This guide is supposed to be simple because it only focuses on syntax, like all "Learn this language" guides on the Internet.   

## Credits

[https://www.youtube.com/watch?v=IA90BTozdow&list=WL&index=17&t=3054s](https://www.youtube.com/watch?v=IA90BTozdow&list=WL&index=17&t=3054s)