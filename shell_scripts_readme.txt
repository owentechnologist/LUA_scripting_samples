Two shell scripts have been provided
lualoader1.sh
routlooploader.sh

# The shell script lualoader.sh loads 4 LUA scripts and stores their SHA values in keys in redis
# The shell script routlooploader.sh uses the SHA values to repeatedly load SortedSet data into redis and also issue queries

# The following are variables used in the two scripts:

bigzs <-- sha for script that loads many entries into a single SortedSet
varyzs <-- sha for script that loads varying numbers of entries into many SortedSets
avgzsv <-- sha for script that returns the average score for entries contained in a SortedSet
calcrouts <-- sha for script that returns the routing values (for entries used by that run of a LUA script)

Why have these two shell Scripts?

LUA scripts are convenient, but work best when combined with some kind of controller program/script.

Here it is suggested to use a shell script as a wrapper to multiple calls of a LUA script.

It makes sense to first load the interesting scripts into redis so they can be more simply invoked:
SCRIPT LOAD "SCRIPT GOES HERE"
loadLUA.sh loads 4 scripts into redis and stores their SHA values as Strings in redis for later use

Once SHA values are stored, you can run the scripts using the SHA value returned:
EVALSHA "2ac113c2cb2197012e5b21129f9cc2ba1104c714"

-- remember to include any arguments to the script as part of the invocation:
  EVALSHA "2ac113c2cb2197012e5b21129f9cc2ba1104c714" 1 z:routvalues 8
