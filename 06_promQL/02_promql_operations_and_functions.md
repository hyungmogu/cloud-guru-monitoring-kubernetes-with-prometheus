# PromQL Operations and Functions

## Arthimetic Operators
1. `+`
    - addition
2. `-`
    - subtraction
3. `*`
    - multiplication
4. `/` 
    - division
5. `%`
    - module
6. `^`
    - power / exponentiation


## Logical / Binary Operators

1. and (intersection)
2. or (union)
3. unless (complement)


## Aggregation Operators
1. sum
    - calculates sum over dimension
2. min
    - selects minimum over dimension
3. max
    - selects maximum over dimension 
4. avg
    - calculate the average over dimension
5. stddev
    - calculate population standard deviation over dimensions
6. stdvar
    - calculate population standard variance over dimensions
7. count
    - count number of elements in the vector
8. count_values
    - count number of elements with the same value
9. bottomk
    - smallest k elements by the sample value
10. topk
    - top k elements by the sample value
11. quantile
    - calculate quantile over dimension

## Functions

- List can be found [here](https://prometheus.io/docs/prometheus/latest/querying/functions/)

## Example

1. Total percentage of RAM

```
(((sum(node_memory_MemTotal_bytes) - sum(node_memory_MemFree_bytes) - sum(node_memory_buffer_bytes) - sum(node_memory_Cached_bytes))/sum(node_memory_MemTotal_bytes)) * 100
```

<img src="https://user-images.githubusercontent.com/6856382/223031070-05af626b-b3b3-4989-a028-d7de9c45d8b1.png">

#