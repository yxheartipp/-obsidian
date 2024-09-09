---
alias: []
Type: 
tags: ["#review"]
sr-due: 2024-09-10
sr-interval: 1
sr-ease: 250
---

```
./redis-server --port 9999 --cluster-enabled yes
```

```
redis_cli='./redis-cli -p 9999 -c'
cluster_node=`$redis_cli cluster nodes | awk '{print $1}'`

for i in {0..16383}
do
    $redis_cli CLUSTER SETSLOT $i node $cluster_node &>/dev/null
done
```