# Kepot - Your Stock-Keeping Manager in Kubernetes (K8s)

Kepot, /ˈkɛpəʊ/, the name is derived from “depot”, “k” stands for “kubernetes”. Also it shares consonant sounds with “keep”, since kepot helps you stock-keeping.

> **Depot**: a place for the storage of large quantities of equipment, food, or goods. *Synonym*: storehouse, warehouse, ...
>
> **Stock-Keeping**: the activity of checking that a store has the right amount of goods available for sale at a particular time.

# To solve what problem

Imagine a professional inventory manager, he/she/they may

- Prepare the resources customer needed **in advance**
- Reduce the **waiting time** after customer placing an order
- Remove **expired or non-compliant** stock from the warehouse
- Adjust warehouse stock level based on **order demand** to reduce costs and increase profits

In Kubernetes world, we have similar problems to solve. 
Imagine a job that takes a long time to complete or a certain resource as a storage unit in our "warehouse". 
We can introduce the above management techniques for these arbitrary "jobs":

- Build an image from source code
- Provision some resources in cloud
- Deploy application to some environment
- Compute a heavy work load with given some parameters

The only requirement for the "job" here is

> Given the same **input** (arguments, environment variables, etc.)
> and the same **operation** (image, artifact, scripts, etc.),
> the **output** is *constant* and *fungible*.

Kepot-controller is the role of inventory manager, who

- Prepare resources / Run long-run jobs **in advance** 
- Reduce the **waiting time** for tasks that are *frequently used* and require *a longer time to complete*
- Clear resources that are **expired or no longer need**
- Adjust the number of resources prepared in advance to **fit the demand**

# Introduce concepts / CRD

Kepot introduces several CRDs (custom resource definition) to your K8s cluster, the three core concepts are

- *Stock* / *Stock-Keeping Unit*
- *Order*
- *Inventory*

## Stock / Stock-Keeping Unit

*Stock*, or actually stock-keeping unit (SKU) is the unit of kepot managing to replenish and deplete. 
A “stock” can be any arbitrary artifact or output, check the following types and examples

- **Replicable Stock (Copy)**: the output can be shared (or copied with a trivial cost) by any number of *Orders*
	- build an image from source code, the input (source code) and operation (build tool chain image) are the same, the output (binary or artifact) is **same** and can be **reused** for any number of *Orders* (each one can copy the artifact)
- **Fungible Stock (Stock)**: each *Order* must hold a unique *Stock*
	- a virtual machine in cloud provider (Azure, AWS, etc...), even though the input (virtual machine spec) and operation (put vm) are the same, we need some "identifier" (name, id, etc...) to distinguish them.
	  User doesn't care what "identifier" the resource acquired, but only have requirements on the spec.

## Order

*Order* describes the need for specific tasks / resources. 
It's the entry point for most kepot users. You place an *Order* (`kubectl create order`), then kepot-controller manages to fulfill the *Order* by

1. Matches *Inventory* by the specification of *Order*
2. Acquire a *Stock* from the stocks tracked by this *Inventory*
3. If no existing *Inventory* is matched, a new *Inventory* and a stack of *Stock* would be created

## Inventory

*Inventory* tracks a full list of *Stocks* and their status, and it connects *Order* and *Stock* demanded.
Reconciling an *Inventory* is stock-keeping job, for instance

- **Replenish** when actual number of *Stocks* is below the expected stock level
- **Deplete** extra *Stocks* when the number is above expected stock level
- **Retire** obsolete *Stocks* that no longer satisfy requirement and not in demand
