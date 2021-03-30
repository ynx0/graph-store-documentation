
## Table of Contents
  * [Advanced Info](#advanced-info)
    + [Misc. Important Notes](#misc-important-notes)
    + [What happens when you add or remove a node?](#what-happens-when-you-add-or-remove-a-node)
    + [Permissions internals](#permissions-internals)



## Advanced Info
This section is not required but does shed light on some implicit assumptions that graph-store makes.


### Misc. Important Notes
Only nodes that successfully typecheck under the validator will be added to the graph
Graphs are validated recursively (see https://github.com/urbit/urbit/blob/5cb6af0433a65fb28b4bc957be10cb436781392d/pkg/arvo/app/graph-store.hoon#L598-L616)
I.e. first, validate all top level nodes of a graph.
Then, recursively validate all children nodes of those nodes.
Base case: empty children is always valid
Nodes are also validated against the graph’s mark when inserted individually (see https://github.com/urbit/urbit/blob/e2ad6e3e9219c8bfad62f27f05c7cac94c9effa8/pkg/arvo/app/graph-store.hoon#L380)
Only the root level graphs get validated with marks. There is only one mark / validator per graph. All child graphs get validated with the same mark as the root/top-level one.
One important note: the way in which Graph Store works is by mirroring all data from a given social media channel. Thus, anything you see is your copy of it, and anything you do is sent as a request to the hosting ship.


### What happens when you add or remove a node?

When adding nodes, graph-store takes in a flat list of nodes that have nodes whose `index`es are arbitrarily deep. It does not allow for a node which has non-existent ancestors to be added (i.e. it doesn’t silently create the node). Thus, it assumes every node up until the 2nd to last index is created, either already existing or included in the graph-update. To ensure consistent behavior, all nodes are added shallowest-index first, which ensures that no child is added before it’s parent (if it exists). This is why any non-leaf node must have its parent exist, either already within the graph or in the update. However, children do not have to be nested within the parent’s data structure in the update. They simply have to exist somewhere in the update, even at the top level is fine. This is because graph-store merges the flat list of nodes from %add-nodes into the fully connected graph structure.

Another constraint to be aware of is that in a map of indexed nodes: for each node, the index in map entry has to be the same index as the in that `node`’s post index.

To see how graph store handles add-nodes, take a look at https://github.com/urbit/urbit/blob/e2ad6e3e9219c8bfad62f27f05c7cac94c9effa8/pkg/arvo/app/graph-store.hoon#L370-L417.


### Permissions internals
Internally, the permissions part of a mark (the grow arm) are built and then watches for changes by graph-store. See https://github.com/urbit/urbit/blob/ac096d85ae847fcfe8786b51039c92c69abc006e/pkg/arvo/app/graph-push-hook.hoon#L154 for more info.


It is important to note that the push hook and permissions system is bespoke, and doesn’t necessarily fit all permissions schemes out there. In addition, the permissions structure is likely to change, so it can make sense to implement your own permissioning structure (if you are not using `graph-push-hook`), especially in the case where you are making your own resource control system. The `graph-push-hook` permissions system is not the definite way to implement permissions, and is simply one pre-existing way to do it. The validator mark doesn’t need any permissions to compile; it just needs a grow arm with at least `noun`. The way to implement your own permissioning structure is in the form of your own grow arm definitions in the validator. There's nothing special about the `graph-permission-add` arms; they are just constants, arms which are known to push-hook. As stated before, Graph Store proper (`app/graph-store.hoon`) doesn't know anything about the permissions.

