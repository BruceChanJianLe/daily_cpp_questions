> Some common C++ questions as quick refresher or daily reading

## Topic: auto and type deduction
1. Explain auto type deduction
2. When can `auto` deduce undesired types?
3. What are the advantages of using `auto`?
4. What is the type of `myCollection` after the following declaration?
5. What are trailing return types?
6. Explain `decltype`!
7. When to use `decltype(auto)`?
8. Which data type do you get when you add two `bool`s?

## Topic: static keyword
9. What does a `static` member variable mean?
10. What does a `static` member function mean?
11. What is the `static` initialization order fiasco?
12. How to solve the static initialization order fiasco?

## Topic: Polymorphism, inheritance, virtual functions
13. Difference between function overloading and overriding?
14. What is a `virtual` function?
15. What is the `override` keyword and its advantages?
16. Explain covariant return types and use-cases
17. What is virtual inheritance and when to use it?
18. Should we always use virtual inheritance?
19. Output and expectations of a sample program?
20. Can you access public/protected members with private inheritance?
21. What is private inheritance used for?
22. Can you call a `virtual` function from a constructor/destructor?
23. What role does a `virtual` destructor play?
24. Can we inherit from standard containers like `std::vector`?
25. What does a strong type mean and its advantages?
26. Explain short-circuit evaluation
27. What is a destructor and how can we overload it?
28. Output of a code sample and why?
29. How to use the `= delete` specifier?

## Topic: Lambda functions
30. What are immediately invoked lambda functions?
31. What kind of captures are available for lambdas?

## Topic: const qualifier
32. Output of code sample and why?
33. Advantages of using `const` local variables?
34. Is it good to have `const` members in a class?
35. Does it make sense to return `const` objects by value?
36. How should you return `const` pointers from functions?
37. Should functions return `const` references?
38. Should you take plain old data types by `const` reference?
39. Should you pass objects by `const` reference?
40. Does function declaration signature match definition?
41. Explain `consteval` and `constinit`

## Topic: Best practices in modern C++
42. What is aggregate initialization?
43. What are explicit constructors and their advantages?
44. What are user-defined literals?
45. Why use `nullptr` instead of `NULL` or `0`?
46. What advantages does `alias` have over `typedef`?
47. Advantages of scoped `enum`s over unscoped?
48. Should you explicitly delete unused special functions?
49. How to use the `= delete` specifier?
50. What is a trivial class?

## Topic: Smart pointers
51. Explain the RAII idiom
52. When should we use unique pointers?
53. Reasons to use shared pointers?
54. When to use a weak pointer?
55. Advantages of `std::make_shared` and `std::make_unique`?
56. Should you use smart pointers over raw pointers always?
57. When and why initialize pointers to `nullptr`?

## Topic: References and move semantics
58. What does `std::move` move?
59. What does `std::forward` forward?
60. Difference between universal and rvalue references?
61. What is reference collapsing?
62. When are `constexpr` functions evaluated?
63. When should you declare functions as `noexcept`?

## Topic: C++20
64. What are concepts in C++?
65. What are the available standard attributes?
66. What is 3-way comparison?
67. Explain `consteval` and `constinit`
68. What are modules and their advantages?

## Topic: Special functions
69. Explain the rule of three
70. Explain the rule of five
71. Explain the rule of zero
72. What does `std::move` move?
73. What is a destructor and how can we overload it?
74. Should you explicitly delete unused special functions?
75. What is a trivial class?
76. Advantages of having a default constructor?

## Topic: Object-oriented design
77. Differences between a class and a struct?
78. What is constructor delegation?
79. Explain covariant return types
80. Difference between overloading and overriding?
81. What is the `override` keyword?
82. Explain friend classes or functions
83. What are default arguments?
84. What is `this` pointer and can we delete it?
85. What is virtual inheritance?
86. Should we always use virtual inheritance?
87. What does a strong type mean?
88. What are user-defined literals?
89. Why shouldn't we use boolean arguments?
90. Distinguish between shallow and deep copy
91. Are class functions part of object size?
92. What does dynamic dispatch mean?
93. What are vtable and vpointer?
94. Should base class destructors be virtual?
95. What is an abstract class?
96. Is polymorphism possible without virtual functions?
97. How to use the Curiously Recurring Template Pattern (CRTP)?
98. Good reasons to use init() functions?

## Topic: Observable behaviors
99. What is observable behavior of code?
100. Characteristics of an ill-formed C++ program?
101. What is unspecified behavior?
102. What is implementation-defined behavior?
103. What is undefined behavior?
104. Reasons behind undefined behavior's existence?
105. Approaches to avoid undefined behavior?
106. What is iterator invalidation?

## Topic: Standard Template Library
107. What is the STL?
108. Advantages of algorithms over raw loops?
109. Do algorithms validate ranges?
110. Can you combine containers of different sizes?
111. How is a `vector`'s memory layout organized?
112. Can we inherit from standard containers?
113. What is the type of myCollection after declaration?
114. Advantages of `const_iterator`s over iterators?
115. Binary search an element with algorithms
116. What is an Iterator class?

## Topic: Miscellaneous
117. Can you call a `virtual` function from constructor/destructor?
118. What are default arguments?
119. Can virtual functions have default arguments?
120. Should base class destructors be virtual?
121. Function of the `mutable` keyword?
122. Function of the `volatile` keyword?
123. What is an inline function?
124. What do we catch?
125. Differences between references and pointers?
126. Which variable declarations compile?
127. What will the code print out and why?
128. Difference between pre- and post-increment/decrement?
129. Final values of variables?
130. Does this string declaration compile?
131. What are Default Member Initializers?
132. What is the most vexing parse?
133. Does this code compile?
134. What is `std::string_view`?
135. How to check if string starts or ends with substring?
136. What is RVO?
137. How to ensure compiler performs RVO?
138. Primary and mixed value categories in C++?
139. Can you safely compare signed and unsigned integers?
140. Return value of main and available signatures?
141. Prefer default arguments or overloading?
142. How many variables should you declare on a line?
143. Prefer switch statement or chained if?
144. What are include guards?
145. Use angle brackets or double quotes for includes?
146. How many return statements should a function have?

## Topic: C++ and algorithmic complexities
147. Differences between `std::map` and `std::unordered_map`?
148. When to use a list over a vector?
149. Algorithmic complexities of important algorithms?

# Reference

Below are the reference links to where questions are consolidated.

- [Daily C++ Interview Questions](https://leanpub.com/cppinterview)
