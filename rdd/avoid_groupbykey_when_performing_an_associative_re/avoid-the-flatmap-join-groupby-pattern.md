# Avoid the flatMap-join-groupBy pattern

When two datasets are already grouped by key and you want to join them and keep them grouped, you can just use `cogroup`. That avoids all the overhead associated with unpacking and repacking the groups.

