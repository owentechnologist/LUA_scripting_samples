## LUA_scripting_samples
A place to keep LUA scripts to acccomplish various things

###  Using the redis-cli and LUA scripts to test SORTED SET 

Scripts:

#### Example SortedSet write Operations: 1
* populate a single key with several entries:
* in this case the scores are times and values are floats
* we could imagine values as purchase amounts
* args to this script:
* [1] number of keyname args provided
* [2] key prefix to use for routing to shard
* [3] max number of entries to add to key
```
EVAL "redis.call('DEL',KEYS[1]..':counter') local limit = ARGV[1]+1 local counter = 1 while counter < limit do local t = (redis.call('TIME'))[1] local t2 = (redis.call('TIME'))[2] redis.call('ZADD',KEYS[1],t..'.'..t2,counter*1.25) counter = redis.call('INCR',KEYS[1]..':counter') end" 1 z:ss1{1} 1000
```

#### Example SortedSet write Operations: 2
* populate several keys with a single entry:
* in this case the scores are times and values are floats
* we could imagine values as purchase amounts
* args to this script:
* [1] number of keyname args provided
* [2] key prefix to use for routing to shard
* [3] max number of keys to add to Redis
```
EVAL "redis.call('DEL',KEYS[1]..':counter') local limit = ARGV[1]+1 local counter = redis.call('INCR',KEYS[1]..':counter') while counter < limit do local t = (redis.call('TIME'))[1] local t2 = (redis.call('TIME'))[2] redis.call('ZADD',KEYS[1]..':'..counter,t..'.'..t2,(counter%10000*1.25)) counter = redis.call('INCR',KEYS[1]..':counter') end" 1 z:{1} 10
```

#### Example SortedSet write Operations: 3
* populate many keys with varying numbers of entries:
* in this case the scores are times and values are strings
* we could imagine values as IP addresses or terminalIDs
* args to this script:
* [1] number of keyname args provided
* [2] key prefix to use for routing to shard
* [3] max number of entries to add to key
* [4] bit of fluff used to enrich the values assigned   
```
EVAL "redis.call('DEL',KEYS[1]..':counter') local limit = ARGV[1]+1 local counter = 1 while counter < limit do local t1 = (redis.call('TIME'))[1] local t2 = (redis.call('TIME'))[2] for index2 = 1,(counter%15) do redis.call('ZADD',KEYS[1]..':MID:'..counter,t1..'.'..t2,''..ARGV[2]..counter..'-'..counter..index2) end counter = redis.call('INCR',KEYS[1]..':counter') end" 1 z:{9999} 10 46
```

#### Example SortedSet write Operations: 4
* Variation 2
* populate many keys with varying numbers of entries:
* in this case the scores are times and values are floats
* we could imagine values as purchase amounts
* args to this script:
* [1] number of keyname args provided
* [2] key prefix to use for routing to shard
* [3] max number of keys to add to Redis
```
EVAL "redis.call('DEL',KEYS[1]..':counter') local limit = ARGV[1]+1 local counter = 1 while counter < limit do local t1 = (redis.call('TIME'))[1] local t2 = (redis.call('TIME'))[2] for index2 = 1,(counter%15) do redis.call('ZADD',KEYS[1]..':PURCH:'..counter,t1..'.'..t2,index2*1.25) end counter = redis.call('INCR',KEYS[1]..':counter') end" 1 z:{9999} 10
```


#### Example SortedSet read Operation:


#### Example Aggregation operation 
* dynamically gather AVG of all values in the named SortedSet (greedy scan uses criteria provided): 
* returned AVG value is rounded down
* args to this script:
* [1] number of keyname args provided
* [2] key prefix (or full name) to use for searching for key
```
EVAL "local zscoreSum = 0 local kname = (redis.call('SCAN',0,'MATCH',KEYS[1]..'*','COUNT',10000000))[2][1] local zcard = (redis.call('ZCARD',kname)) for index = 1,zcard do zscoreSum = (zscoreSum + (redis.call('ZRANGE',kname,'0','-1'))[index]) end return (zscoreSum/zcard)" 1 z:{9999}:PURCH:2
```


#### This more verbose version replies with the keyname of the SortedSet and more precision
* args to this script:
* [1] number of keyname args provided
* [2] key prefix (or full name) to use for searching for key
```
EVAL "local zscoreSum = 0 local kname = (redis.call('SCAN',0,'MATCH',KEYS[1]..'*','COUNT',10000000))[2][1] local zcard = (redis.call('ZCARD',kname)) for index = 1,zcard do zscoreSum = (zscoreSum + (redis.call('ZRANGE',kname,'0','-1'))[index]) end return 'Avg of Values for Sorted Set named '..kname..' is '..(zscoreSum/zcard)" 1 z:{9999}:PURCH:2
```


###  Using the redis-cli and LUA scripts to test SET Type

Scripts:

#### Example Set write Operations: 1
* populate a single key with several entries:
* values are times
* args to this script:
* [1] number of keyname args provided
* [2] key name to use 
* [3] max number of entries to add to key
```
EVAL "redis.call('DEL',KEYS[1]..':counter') local limit = ARGV[1]+0 local counter = 1 while counter < limit do local t = (redis.call('TIME'))[1] local t2 = (redis.call('TIME'))[2] redis.call('SADD',KEYS[1],''..t..'.'..t2) counter = redis.call('INCR',KEYS[1]..':counter') end" 1 set:s1{1} 10
```

#### Example Set write Operations: 2
* populate several keys with a single entry:
* values are times
* args to this script:
* [1] number of keyname args provided
* [2] key name prefix to use for routing to shard
* [3] max number of keys to add to redis
```
EVAL "redis.call('DEL',KEYS[1]..':counter') local limit = ARGV[1]+0 local counter = 1 while counter < limit do local t = (redis.call('TIME'))[1] local t2 = (redis.call('TIME'))[2] redis.call('SADD',KEYS[1]..':'..counter,''..t..':'..t2) counter = redis.call('INCR',KEYS[1]..':counter') end" 1 set:{1} 100
```

#### Example Set write Operations: 3
* populate many keys with varying numbers of entries:
* args to this script:
* [1] number of keyname args provided
* [2] key prefix to use for routing to shard
* [3] max number of keys to add to redis
* [4] max number of entries to add to any key
```
EVAL "redis.call('DEL',KEYS[1]..':counter') redis.call('DEL',KEYS[1]..':inner_counter') local limit = ARGV[1]+1 local counter = 1 while counter < limit do while redis.call('INCR',KEYS[1]..':inner_counter') < ((redis.call('TIME')[2])+1)%(ARGV[2]+1) do redis.call('SADD',KEYS[1]..':PURCH:'..counter,redis.call('GET',KEYS[1]..':inner_counter')*((redis.call('TIME')[2])%10000)) end redis.call('SET',KEYS[1]..':inner_counter','1') counter = redis.call('INCR',KEYS[1]..':counter')  end return limit" 1 set:{9999} 100 15
```


#### Example Redis Set-type read Operation:


Example Aggregation operation 
* dynamically gather AVG of all values in a Set
* returned value is rounded down to integer
* args to this script:
* [1] number of keyname args provided
* [2] key prefix (or fullname) to use to select key
```
EVAL "local svalueSum = 0 local kname = (redis.call('SCAN',0,'MATCH',KEYS[1]..'*','COUNT',10000000))[2][1] local scard = (redis.call('SCARD',kname)) for index = 1,scard do svalueSum = (svalueSum + (redis.call('SMEMBERS',kname)[index])) end return (svalueSum/scard)" 1 set:{9999}:PURCH:123
```


####  this more verbose version replies with the keyname of the Set and more precision
* args to this script:
* [1] number of keyname args provided
* [2] key prefix (or fullname) to use to select key 
```
EVAL "local svalueSum = 0 local kname = (redis.call('SCAN',0,'MATCH',KEYS[1]..'*','COUNT',10000000))[2][1] local scard = (redis.call('SCARD',kname)) for index = 1,scard do svalueSum = (svalueSum + (redis.call('SMEMBERS',kname)[index])) end return 'Avg score for Set named '..kname..' is '..(svalueSum/scard)" 1 set:{9999}:PURCH:
```

###  Using the redis-cli and LUA scripts to test Hashes 

Scripts:

#### Example Hash write Operations: 1
* populate a single key with several entries:
* args to this script:
* [1] number of keyname args provided
* [2] key prefix to use for routing to shard
* [3] max number of entries to add to key
```
EVAL "for index = 1,ARGV[1] do local t = (redis.call('TIME'))[1] local t2 = (redis.call('TIME'))[2] redis.call('HSET',KEYS[1],'item_'..index..'_cost',(t2%7500*.97)) end" 1 h:h1{1} 10
```

#### Example Hash write Operations: 2
* populate several keys with a single entry:
* args to this script:
* [1] number of keyname args provided
* [2] key prefix to use for routing to shard
* [3] max number of keys to add
```
EVAL "for index = 1,ARGV[1] do local t = (redis.call('TIME'))[1] local t2 = (redis.call('TIME'))[2] redis.call('HSET',KEYS[1]..':'..t2,'item_1_cost',((t2%7500+1)*1.25)) end" 1 h:{1} 10
```

#### Example Hash write Operations: 3
* populate many keys with varying numbers of entries:
* args to this script:
* [1] number of keyname args provided
* [2] key prefix to use for routing to shard
* [3] max number of keys add
* [4] bit of fluff used to enrich the keynames assigned   
```
EVAL "redis.call('DEL',KEYS[1]..':counter') local limit = ARGV[1]+1 while redis.call('INCR',KEYS[1]..':counter') < limit do local t1 = (redis.call('TIME'))[1] local t2 = (redis.call('TIME'))[2] for index2 = 1,((t2%15)+1) do local t3 = (redis.call('TIME'))[2] redis.call('HSET',KEYS[1]..':PURCHASES:'..ARGV[2]..t1..'-'..(redis.call('GET',KEYS[1]..':counter')),'item_'..index2..'_cost',(index2*10.25)) end end" 1 h:{9999} 10 TRGT
```



