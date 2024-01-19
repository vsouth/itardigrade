![](Pasted%20image%2020240119164515.png)
balanced search tree has height of O(log n)

## red-black tree
1. a node is either black or red
2. the root and leaves (NIL) are black
3. if a node is red then its children are black
4. **all paths from a node to its NIL descendants contain the same number of black nodes**
5. **the longest path (root to farthest NIL) is no more than twice the length of the shortest path (root to nearest NIL)**

O(log n):
- search
- insert (requires rotation)
- remove (requires rotation)
![](Pasted%20image%2020240119164846.png)
