# How to estimate the size of a Dataset

An approximated calculation for the size of a dataset is:

```text
number Of Megabytes = M = (N*V*W) / 1024^2
```

where:

```text
    N  =  number of records

    V  =  number of variables

    W  =  average width in bytes of a variable
```

In approximating **W**, remember:

| **Type of variable**  |         **Width**             |
| :--- | :--- |
| Integers,  −127 &lt;= x &lt;=  100 | 1 |
| Integers, 32,767 &lt;= x &lt;=  32,740 | 2 |
| Integers, -2,147,483,647 &lt;= x &lt;= 2,147,483,620 | 4 |
| Floats single precision | 4 |
| Floats double precision | 8 |
| Strings | maximum lenght |

Say that you have a 20,000-observation dataset. That dataset contains

```text
    1  string identifier of length 20                     20

    10  small integers (1 byte each)                      10

    4  standard integers (2 bytes each)                    8

    5  floating-point numbers (4 bytes each)              20

    --------------------------------------------------------

    20  variables total                                   58
```

Thus the average width of a variable is:

```text
W = 58/20 = 2.9  bytes
```

The size of your dataset is:

```text
M = 20000*20*2.9/1024^2 = 1.13 megabytes
```

This result slightly understates the size of the dataset because we have not included any variable labels, value labels, or notes that you might add to the data. That does not amount to much. For instance, imagine that you added variable labels to all 20 variables and that the average length of the text of the labels was 22 characters.

That would amount to a total of 20\*22=440 bytes or 440/10242=.00042 megabytes.

### **Explanation of formula**

```text
M = 20000*20*2.9/1024^2 = 1.13 megabytes
```

N\*V\*W is, of course, the total size of the data. The 1,0242 in the denominator rescales the results to megabytes.

Yes, the result is divided by 1,0242 even though 1,0002 = a million. Computer memory comes in binary increments. Although we think of k as standing for kilo, in the computer business, k is really a “binary” thousand, 210 = 1,024. A megabyte is a binary million—a binary k squared:

```text
1 MB = 1024 KB = 1024*1024 = 1,048,576 bytes
```

With cheap memory, we sometimes talk about a gigabyte. Here is how a binary gig works:

```text
1 GB = 1024 MB = 10243 = 1,073,741,824 bytes
```

