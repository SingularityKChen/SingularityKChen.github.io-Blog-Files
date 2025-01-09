---
layout: post
title: "[Tutorial] Understanding Python Variable Assignment and Copying Mechanisms"
description: "In Python, understanding how variables are assigned and how copying mechanisms work is crucial for writing robust and efficient code. This blog post aims to demystify these concepts and provide best practices for handling different variable types to avoid unintended side effects."
categories: [Tutorial]
tags: [Python]
last_updated: 2025-01-09 12:55:00 GMT+8
excerpt: "In Python, understanding how variables are assigned and how copying mechanisms work is crucial for writing robust and efficient code. This blog post aims to demystify these concepts and provide best practices for handling different variable types to avoid unintended side effects."
redirect_from:
  - /2025/01/09/
---

* Kramdown table of contents
{:toc .toc}

In Python, understanding how variables are assigned and how copying mechanisms work is crucial for writing robust and efficient code. This blog post aims to demystify these concepts and provide best practices for handling different variable types to avoid unintended side effects.

# TL; DR

In Python, variables hold references to objects rather than the objects themselves. When dealing with mutable objects like lists or dictionaries, simple assignment can cause multiple variables to reference the same object, leading to unintended side effects. 

To prevent this, shallow copies should be used for simple mutable objects, while deep copies are necessary for complex, nested mutable structures. 

It is crucial to understand the difference between mutable and immutable types and to choose the appropriate copying mechanism based on the situation. 

Additionally, when using deep copies, performance considerations are important, especially with large data structures. 

# Background: The Problem Scenario

Consider a class `A` that returns an instance of a dataclass `D`. If `D` contains a mutable object, such as a list, and this list is shared across multiple instances of `D`, modifications to the list will affect all instances. This can lead to unintended behavior, as seen in the scenario where modifying a shared list alters multiple instances.

```python
from dataclasses import dataclass

@dataclass
class D:
    x: int
    lst: list

class A:
    def __init__(self):
        self.shared_list = [1, 2, 3]
    def m(self):
        return D(10, self.shared_list)

# inst class A
a = A()
e = a.m()
f = a.m()

# Modify a.shared_list
a.shared_list.append(4)
print(e.lst)  # Should be [1, 2, 3, 4]
print(f.lst)  # Should be [1, 2, 3, 4]
```

# Python's Variable Assignment Mechanism

In Python, variables do not hold values directly; they hold references to objects. When you assign a variable to another variable, you are creating a new reference to the same object. This is particularly important when dealing with mutable objects, as changes to the object will be visible through all references.

## Mutable vs Immutable Objects

- **Mutable Objects**: These can be changed after creation. Examples include lists, dictionaries, and sets.
- **Immutable Objects**: These cannot be changed after creation. Examples include integers, strings, and tuples.

# Shallow Copy vs Deep Copy

When dealing with mutable objects, it is often necessary to create copies to avoid unintended modifications.

- **Shallow Copy**: Creates a new container but references the original contained objects. This is suitable when the contained objects are immutable.
- **Deep Copy**: Creates a new container and recursively copies all contained objects. This is necessary when the contained objects are mutable.

# Best Practices

1. **Understand Reference vs Value**: Recognize that variables hold references to objects, not the objects themselves.
2. **Use Shallow Copies for Simple Mutables**: When dealing with simple mutable objects like lists and dictionaries, use shallow copies to avoid shared state.
3. **Use Deep Copies for Complex Mutables**: For objects with nested mutables, use deep copies to ensure complete isolation.
4. **Prefer Immutable Types When Possible**: Immutable objects inherently avoid issues related to shared state.

# Code Example and Explanation

```python
from dataclasses import dataclass
from copy import deepcopy, copy

@dataclass
class MyData:
    int_var: int
    str_var: str
    list_var: list
    dict_var: dict
    set_var: set
    custom_obj: 'MyCustomClass'

class MyCustomClass:
    def __init__(self, value):
        self.value = value
        self.mutable_list = []

def get_my_data():
    original_list = [1, 2, 3]
    original_dict = {'a': 1, 'b': 2}
    original_set = {1, 2, 3}
    custom_instance = MyCustomClass(100)
    
    # Shallow copies for list, dict, and set
    # Deep copy for custom_obj
    return MyData(
        int_var=10,
        str_var="Hello",
        list_var=list(original_list),  # Shallow copy
        dict_var=dict(original_dict),  # Shallow copy
        set_var=set(original_set),     # Shallow copy
        custom_obj=deepcopy(custom_instance)  # Deep copy
    )

# Testing the function
data1 = get_my_data()
data2 = get_my_data()

# Modifying data1's mutable attributes
data1.list_var.append(4)
data1.dict_var['c'] = 3
data1.set_var.add(4)
data1.custom_obj.mutable_list.append(100)

# Checking if data2 is affected
print(data2.list_var)   # Should be [1, 2, 3]
print(data2.dict_var)   # Should be {'a': 1, 'b': 2}
print(data2.set_var)    # Should be {1, 2, 3}
print(data2.custom_obj.mutable_list)  # Should be []
```

In this example, we create shallow copies for simple mutable objects like lists, dictionaries, and sets. For more complex objects with nested mutability, such as `MyCustomClass`, we use a deep copy to ensure complete independence.

# Conclusion

Mastering Python's variable assignment and copying mechanisms is essential for writing clean, efficient, and bug-free code. By understanding the differences between mutable and immutable objects and knowing when to use shallow or deep copies, you can avoid common pitfalls and write more robust Python applications.
