# Decommissioning a datacenter

Steps to properly remove a datacenter so no information is lost.

1. Make sure no clients are still writing to any nodes in the datacenter.
2. Run a full repair with `nodetool repair`. This ensures that all data is
propagated from the datacenter being decommissioned. It needs to be executed
in all nodes. (`nodetool repair -full`)
3. Change all keyspaces so they no longer reference the datacenter being
removed.
4. Run `nodetool decommission` on every node in the datacenter being removed.

Alter keyspaces in a NetworkTopologyStrategy

```sql
ALTER  KEYSPACE keyspace_name 
   WITH REPLICATION = { 'class' : 'NetworkTopologyStrategy', 'dc1_name' : N [, ...]
   }
```

For example:

```sql
ALTER  KEYSPACE hello_keyspace 
   WITH REPLICATION = { 'class' : 'NetworkTopologyStrategy', 'datacenter1' : 3, 'datacenter2': 3, 'datacenter3': 3 
   }
```
