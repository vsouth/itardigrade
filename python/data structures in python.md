hashset:
```
# set

set.add(2)
set.remove(2)
i in set
{1,2}
```

queue (deque):
```
from collections import deque
deque.append()
deque.appendleft()
deque.popleft()
deque.pop()
```

hashmap:
```
# dict

d["key"] = num
d.pop("key")
key in d

for key in d
for val in d.values()
for key, val in d.items()
```
tuples (1,2) can be used as keys for dicts, as items for sets
lists cannot

heap:
```
import heapq

heapq.heapify(data)
heapq.heappop(data)
heapq.heappush(data, num)
# it uses minheap
# for maxheap:
# make data negative
# or
# heapq._heapify_max(data)
# but it is undocumented
# heapq._heappop_max, etc.
heapq.merge(l1, l2)
```