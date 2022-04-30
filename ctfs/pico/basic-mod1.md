# basic-mod1
## Cryptography challenge from picoCTF2022
### Solution

```python
import string 


map = dict()

for char in range(len(string.ascii_uppercase)):
	map[char] = string.ascii_uppercase[char]

count = 0 
for num in range(26,36):
	map[num] = count
	count += 1

map[36] = "_"

with open("message.txt","r") as f:
	text = f.read()
	for num in text.strip().split(" "):
		print(map[int(num)%37],end="")
```
