```python
def mergeSort(array):
    if len(array) > 1:
        middle_index = len(array) // 2
        left_part = array[:middle_index]
        right_part = array[middle_index:]
        left_part = mergeSort(left_part)
        right_part = mergeSort(right_part)
        return merge(left_part, right_part)
    else:
        return array[:]
    
def merge(left_part, right_part):
    merged_result = []
    i, j = 0, 0
    while i < len(left_part) and j < len(right_part):
        if left_part[i] < right_part[j]:
            merged_result.append(left_part[i])
            i += 1
        else:
            merged_result.append(right_part[j])
            j += 1
    while i < len(left_part):
        merged_result.append(left_part[i])
        i += 1
    while j < len(right_part):
        merged_result.append(right_part[j])
        j += 1
    return merged_result
        
        
n = int(input())
a = [int(i) for i in input().split(" ")]
print(*mergeSort(a))
```
![](Pasted%20image%2020240119170557.png)