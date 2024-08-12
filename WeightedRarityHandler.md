
A Utility Module for making an easy rarity system adding up to 100%, while also being very fast at rolling and an extra helper function to get the probabilities. #Utility #Module 

``` lua
local alias_table = {}

alias_table.ReturnNameByDefault = false

local function GetNames(RarityDictionary)
	local weights = {}
	local names = {}
	for name, weight in pairs(RarityDictionary) do
		table.insert(weights, weight)
		table.insert(names, name)
	end
	return weights, names
end

function alias_table:new(Raritys) 
	local weights, names = GetNames(Raritys)
	local total = 0
	for _,v in ipairs(weights) do
		assert(v >= 0, "all weights must be non-negative")
		total = total + v
	end
	assert(total > 0, "total weight must be positive")
	local normalize = #weights / total
	local norm = {}
	local small_stack = {}
	local big_stack = {}
	for i,w in ipairs(weights) do
		norm[i] = w * normalize
		if norm[i] < 1 then
			table.insert(small_stack, i)
		else
			table.insert(big_stack, i)
		end
	end
	local prob = {}
	local alias = {}
	while small_stack[1] and big_stack[1] do
		local small = table.remove(small_stack)
		local large = table.remove(big_stack)
		prob[small] = norm[small]
		alias[small] = large
		norm[large] = norm[large] + norm[small] - 1
		if norm[large] < 1 then
			table.insert(small_stack, large)
		else 
			table.insert(big_stack, large)
		end
	end
	for _, v in ipairs(big_stack) do prob[v] = 1 end
	for _, v in ipairs(small_stack) do prob[v] = 1 end
	self.__index = self
	return setmetatable({alias=alias, prob=prob, n=#weights, ReturnName = alias_table.ReturnNameByDefault, names = names}, self)
end

function alias_table:__call()
	local index = math.random(self.n)
	if self.ReturnName then
		local Selected_Index = math.random() < self.prob[index] and index or self.alias[index]
		return self.names[Selected_Index]	
	end
	return math.random() < self.prob[index] and index or self.alias[index]
end

function alias_table.calculate_probabilities(rarities)
	local total_weight = 0
	for _, weight in pairs(rarities) do
		total_weight = total_weight + weight
	end
	for item, weight in pairs(rarities) do
		local probability = (weight / total_weight) * 100
		print(string.format("%s: %.3f%%", item, probability))
	end
end

return alias_table
```



### How To Use:

*First Create a module and insert the code block above.*

``` lua
local RarityManager = require(script.WeightedRarityHandler)
RarityManager.ReturnNameByDefault = true
```

*Require the module and set the ReturnNameByDefault to true, this will return the Index of the item rolled instead of just its position*.

``` lua
local Rarities = {
	["Sword"] = 75,
	["Shield"] = 20,
	["Wand"] = 2.5,
	["Demon Race"] = 2,
	["DARKNESS"] = 0.5,
}
```

*Make a dictionary of Rarities like the example above, note that the rarities don't have to add up to 100%  because the module will adjust the weights so they do, but as a tip it is easier to understand if you make sure the items add up to 100 because then the probabilities of the items are what you see in the dictionary because no weight adjustment was needed *.

*If you have a lot of rarities and for your use case they don't add up to 100 then you can check the actual adjusted probability of each item by using the helper function: calcuate_probabilities(rarities).*
```lua 
RarityManager.calculate_probabilities(Rarities)
```
*Output:*
```lua
	-- Demon Race: 2.000%  
    -- Shield: 20.000%  
    -- Sword: 75.000%  
    -- DARKNESS: 0.500%  
    -- Wand: 2.500%  	
```

*Now for setting up the main function its simply by passing the list of rarities into the function and calling it to receive the item that was "rolled"*.

```lua 
local rollSample = RarityManager:new(Rarities)
rollSample()
```

Calling the function will return the string of the item that was rolled.

> [!NOTE] Quirks
> Do not use math.randomseed with the same seed in the same invocation cycle or the function will not be random
> 
> Also this module can be used for a varaity of diffrent thing not just for item rarity, it can be used for events and anything that has a chance then this module can be used for it
>  

## Benchmarking

---
1 Million rollSample() calls in 0.14 seconds,.

| Function   | FunctionCalls | Time (seconds) |
| ---------- | ------------- | -------------- |
| rollSample | 1_000_000     | 0.14           |
