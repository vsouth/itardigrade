---
tags:
  - task
date: 2024-01-19T16:24
---
## тема:
куча
## задача:
find top K largest elements
## решение: 
min-heap of K elements
## сложность:
O(N logK)


- solution 1
sort + take K first elements
Complexity: O(N logN) - bc of sorting
до миллиона значений работает супер

- solution 2
min-heap of K elements
Complexity: O(N logK)
после миллиона значений работает супер