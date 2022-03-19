---
layout: post
title: Python Programming language Quick Reference Guide (beginner guide) 
# author: "Himanshu Aggarwal"
# description: "Python Programming language Quick Reference Guide (beginner 30 mins guide)"
date: 2022-03-01
categories: python tutorials
---

Python Programming language Quick Reference Guide (beginner guide). <br>
A popular programming language and becoming a de-facto language in the field of Data Science. <br>
Having 2nd Highest TIOBE index as of May, 2021.
<br>

**Some KeyPoints:**
- It is an interpreted Language.
- A dynamically typed language ( type is associated with run-time values).
- Easy to read and write (English like syntax)
- Usecases : Web Applications, Workflows, Games and Apps, Data Science, Database systems, Scripting
- Uses identation as the block of code.


```python


#**** Comments ****# 

#Single Line comment 
"""
This is a multiline comment
"""


#To check the version of Python   
#on cmd
#!python --version
#in console
import sys
sys.version


#**** Print ****# 

print("Hello World!")
print("Hello", "World")


#**** Variables ****# 

#Variables (dynamically typed laguage i.e datatype is detected automatically)
#Naming convention : alphanumeric and underscores, are case sensitive (cannot start with numberic)
x = 5
x = "Hello"
X = "Himanshu!"
p, q = 1, 2
print(x , X)
print(p, q)


#**** Numbers ****#

#Python Numbers
x = 1    # int
y = 2.8  # float
Y = 2e4  #float can use scientific notation for 10^ power notation
z = 4 + 1j   # complex

#Can use type function to return the type of the variable
print(x , type(x))
print(y , type(y))
print(Y , type(Y))
print(z , type(z))

#**** Strings ****#

#Strings are immutable.
#In python, string is array of bytes representing unicode characters

#Single line String
x = "Hello World!"
print(x) #prints the string

#multiline string
x = """ Hello 
World!"""
print(x)

#In python, string is array of bytes representing unicode characters
#Can be accessed using indexes 
x= "python"
print(x[1])
#Substring
print(x[0:5])  
print(x[:5])  #empty y default starts from initial i.e. 0th index
#Negative indexing starts from the last like,
print(x[-1])

#Length of string
print("Length Of String : " , len(x))

print("python".upper())                 # Converts whole string to Uppercase
print("Python".lower())                 # Converts whole string to lowercase
print("python is eAsy".title())         # Converts each word first letter to uppercase (camel case)

print(" python ".strip())               # Removes the whitespace from start and end 
print("*python***#*".strip("*"))        # Removes the specified character from the start and end till the other occurence of any other character

print("Pythommm".replace("m", "n"))     #str.replace(a, b) replaces the every occurence of a with b in str 
print("Pythommm".replace("m", "n", 2))  #str.replace(a, b, n) replaces the a's first n occurences each with b in str.

print("Python,R,C,C++".split(","))      #Split the string into list on the specified string Return's ['Python', 'R', 'C', 'C++']


#**** Formatting of Strings ****#

#Printing using .format()
x = "Python"
version = "3"
print("You are learning {} {}!".format(x, version))

# Positional Formatting 
x , y , z = "Python", "cool", "is"
print("{0} {2} {1}!".format(x, y, z))

# String alignment 
print("\nLeft, center and right alignment with Formatting: ") 
print("|{:<10}|{:^20}|{:>10}|".format('Python','is','Cool'))

#Rounding off floats, changing representation
print("Rounding off float to 2 decimal places : {:.2f}".format(322.5657), "\n Exponential representation of float : {:e}".format(2000000000000000))


#**** Operators ****#

#** Arithmetic operators : +, -, *, /, **, //  **#
x = 12
y = 5
print("Addition      : ", x+y)
print("Subtraction   : ", x-y)
print("Multplication : ", x*y)
print("Division      : ", x/y)
print("Exponentiation: ", x**y)
print("Floor Division: ", x//y)

#** Comparison Operators : ==, !=, <, >, <=, >=   Result can be True, False **#
print("\nComparison operators : ")
print("Equal Comparator                : ", x == y)
print("Not EqualTo Comparator          : ", x != y)
print("Greator Than Comparator         : ", x > y)
print("Smaller Than Comparator         : ", x < y)
print("Greater Than Equalto Comparator : ", x >= y)
print("Smaller Than Equalto Comparator : ", x <= y)

#** Logical Operators : and, or, not **#
print("\nLogical Operators : ")
print("and", x < 12 and y < 12)
print("or ", x < 12 and y < 12)
print("not", not x < 12)

#** Identity Operators : is, is not **#
print("\nIdentity Operators : ")
#is operator returns True When both variables are same object
x = ["python"]
y = ["python"]
print(x is y)      # This returns False because both objects are different.
print(x is not y)  # This returns True because both objects are different.

x = y
print(x is y)      # This returns true because now both variables are the same objects.
print(x is not y)  # This can be considered as negation of is operator.

#** Membership Operators : in, not in **#
#Returns true if specified value in object
print("\nMembership Operators : ")
x = "python"
y = ["python", "programming", "language"]
print("in operator example     : ", x in y)      # Returns True as x is present in the iterable(a object that can be iterate over) y 
print("not in operator example : ", x not in y)  # Returns False (negation of above)

#** BitWise Operators **#
#Complement(~) , AND(&), OR(|), XOR(^), LEFT SHIFT(<<), Right Shift(>>)
print("\nBitWise Operators : ")
x = 12
print("COMPLEMENT(~) Operator : ", ~x)  # Returns -13S
#binary representation of 12 :  00001100 
#now, negation will of 1100 is  11110011 
#binary representation of 13 :  00001101
#i.e. 2's complement of a number represents a negative number  
#2's complement of 13 L 11110011 i.e. -13 
print("AND(&) Operator : ", 12 & 13)
print("OR (|) Operator : ", 12 | 13)
print("XOR(^) Operator : ", 12 ^ 13)
print("LEFT SHIFT Operator  : ", 12 << 13)
print("RIGHT SHIFT Operator : ", 12 >> 13)


#**** Lists ****#

#It is a ordered and mutable collection of values(items).
#Written as a list of comma-separated list of values, between square[] brackets and Can contain values of different types

l = [1, 2, 3, 4]             # A list of integers
l = [1, 2, 3, 4, "Python"]   # Can contain values(items) of different types
print(l)
l[0] = 'Hello'               # List can be updated by using index (list starts from 0th index)
print(l[0])
print(l)

l = [1, 2, 3, 4]
l.append(10)                 #Appends an element at the end of the list
print(l)
l.insert(1,4)                #Appends an elemnet at the specified location, shift the element to the right
print(l)
l.remove(4)                  #Removes the first occurence of the value
print(l)
l.pop(2)                     #Removes the element from the specified location(index).
print(l)
l.clear()                    #Empties the list
print(l)

l.append(2)                  #inserting value 2 at the end.
l.append(3)
print(l)
del l[0]                     #Removes the specfied element.
print(l)
del l                        #Deletes the entire list


l = ["Hello", "World"]
l2 = l                       #(l2 means List2) This does not copies a list l to l2, instead l2 is just a refrence to l
#i.e. changes made to any list will also be reflected to another
print(l)
l2[0] = 1
print(l)


#**** Tuples ****#
#It is an ordered and unmutable collection of items.

t = (1, "Hello")      # like list it can also contains items of different datatypes
#t[0] = 2             # throws error as, it is unmutable

l = tuple([1, 2, 3])  # converts a list to a tuple 


#**** Sets ****#
#Unordered Collection of elements, mutable , unindexed and contain no duplicates.

s = {1, 2, 3, 3}      #Initializing a set
print(s)              #It will remove the duplicate items and only retain one occurence of each item(value).
#s[0] = 4             #Cannot change the existing items also, cannot be accessed as index.

#** Set Methods **#
s.add(5)              #adds element to the last

s.remove(5)           #Removes the item from the set, if exists otherwise throws an error 
print(s)

s.discard(6)          #Same as remove method, but will not throw error if the item is not present in the set
#s.remove(6)          #this will throw an KeyError exception
print(s)

s.pop()               #Removes an item from the set.


#**** Dictionaries *****#
#Unordered collection of items. It consists of key value pairs.

d = {"Language" : "Python", "Type" : "Interpreted", "TIOBE_RANKING" : 2 }

print(d.keys())              # Returns the keys of the dictionary
print(d.values())            # Returns the values of the dictionary
print(d.items())             # Returns a items in dictionary as(key,value) in a list.

#Looping through a dictionary
for i, j in d.items():
    print("Key : {}  Value : {}".format(i,j))
    
#Accessing a dictionary
print(d["Language"])         # Returns the value for the specified key
#d["Gender"]                 # This will throw KeyError exception, if key is not present in the dictionary
v_gender = d.get("Gender")   # This will not throw an Exception.
print(v_gender)

#Value Can be changed
d["Language"] = "C++"
d["Type"] = "Compiled"
d["TIOBE_RANKING"] = None

print(d)

#Removing a specified key from the dictionary
d.pop("TIOBE_RANKING")      # Removes the specified key from the dictionary
#del d["TIOBE_RANKING"]     # Removes specified key from the dictionary
#del d                      #Deletes the complete dictionary
d.popitem()                 # Removes the last added key from the dictionary, and also returns the removed key

print("Language" in d)      # Check's if the key is present in the dictionary or not 

d1 = d.copy()               # Creates copy of a dictionary
print(d1)
d1.clear()                  # Empties the dictionary
print(d1)

dict(Language='Java', Type='Compiled')


#**** if else ****#

a, b = 1, 2           # multiple variable assignment in a single line
if a > b:
    print("A is greater")
elif a == b:
    print("Both are equal")
else:
    print("B is greater")


print("A is greater") if a > b else print("B is greater") if b > a else print("Both are equal") #Single line if..elif..else


#**** Loops ****#

#** While Loop Example... **#
#While Loop, executes til the condition satisfies

a = 0
while a < 10:
    a += 1
    if a == 5: continue  #skip the statements after this , when the condition is true
    if a == 7: break     #exits the loop
    print(a)
    
#** For Loop Example... **#

a = 10
for each in range(a):
    print(each)
#print(list(range(a)))

#** For Loop With else **#

a = 5
for i in range(a):
    print(i)
else:
    print("Loop Finished")
    
#** For loop over a list... **#

letters = ["A", "B", "C", "D"]
for each in letters:
    print(each, end=" ")     #bonus : print function also have end parameter, by default end = newline character


#**** Functions ****#

#Functions are declared by using def keyword
def func():                   #Simple Function
    print("Function Called")

func()                        #Calling the function

def func(a):                  #function with argument(s)
    print("Value Passed : {}".format(a)) 
    
func(10)                      #Calling the function by passing the value

#func() #this will throw an error , because it excepts a single parameter

def func(a = 10):             #Argument with default value
    print("Value Passed : {}".format(a))

func()

#Recursive function (A function that calls itself)

def recurr_func(a = 0): # Here, we are increasing the value of a with each call to the function itself. 
    print(a)
    a += 1
    if a < 10: recurr_func(a)

recurr_func()           # Note, it is necessary to provide a terminate condition when using recursion otherwise, the function will kept calling itself recursively indefinitely.


#**** Lambda Function ****#
#An Anonymous Function (also known as single line functions)
#Syntax 
#lambda Arguments : expression

twice_of = lambda x : x * 2
twice_of(10)

#It can take any number of arguments 
multipy_three_no = lambda x, y, z : x*y*z
multipy_three_no(1,2,3)

#Let's learn a new feature of list upacking and packing of arguments
#Suppose we don't know the number of arguments, then prefixing * will pack all the arguments into a tuple 
def func(*args):
    print(args)    #This will return all the arguments in a tuple asigned to a varibale args.
func(1, 2, 3, 4, 5)

#Let's use this in lambda function with list comprehension
#A function that will multiple all the numbers by 2 and return the result in a list.

multiply_by_2 = lambda *x : [each*2 for each in x]
multiply_by_2(1, 2, 3, 4)  #we can pass any number of arguments
multiply_by_2(1, 2, 3, 4, 5, 6)


#**** Type Conversion ****#
#Python is an object-orientated language, and as such it uses classes to define data types, including its primitive types.
#Can use int() , float(), str() , list(), tuple(), dict() , str()

x, y = 1, 1
c = complex(x,y)
print(c)

```