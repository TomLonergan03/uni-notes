The coin changing problem is, given a value $v$ and a set of $n$ coins $\{c_1,c_2,...,c_n\}$ how do we calculate the smallest collection of coins possible that sum to that value. We can assume there are an infinite number of each coin.

# Greedy solution
The most obvious solution, is to simply take the largest coin that is smaller than the remaining change and repeat until there is no more change required.

This works in a lot of cases, however is not generalisable to all coin systems, for example if we have coins of value 1, 5, 7 then the greedy solution would suggest that we should use 7+7+1+1+1+1 to represent 18, even though the actual best solution is to use 7+5+5+1.

# Dynamic programming solution
The correct solution is to calculate all possible ways to produce each value, from the value of the lowest value coin up to the desired value. Each time we compute a new target value, we can reuse previously calculated values if they exist.

```py
def coins(value: int, coins: List[int]):
	result = [0] * len(coins)
	c = [999999] * (value + 1)
	p = [999999] * (value + 1)
	
	c[0] = 0
	c[1] = 1
	p[1] = 1

	# determines ways to make each value
	for w in range(2, value + 1):
		for i in range(0, len(coins)):
			if coins[i] <= w and c[w - coins[i]]+1 < c[w]:
				c[w] = 1 + c[w - coins[i]]
				p[w] = i

	# reconstructs used coins
	while value > 0:
		i = p[value]
		result[i] = result[i] + 1
		value = value - coins[i]
	
	print(c[-1], " coins, solution is:")
	for i in range(0, len(coins)):
		if result[i] > 0:
			print(coins[i], ": x", result[i], sep="")
```

