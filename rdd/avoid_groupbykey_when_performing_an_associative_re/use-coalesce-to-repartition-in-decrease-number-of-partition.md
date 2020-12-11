# Use coalesce to repartition in decrease number of partition

Use **coalesce** if you decrease number of partition of the RDD instead of **repartition**. **coalesce** is usefull because avoids a **full** shuffle, It uses existing partitions to minimize the amount of data that's shuffled.

