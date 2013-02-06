#Background
A program uses OpenMP to do multi-threading.
  
2N threads are used.
Threads 0, 1, ..., N - 1 are used for processing, and
threads N, N + 1, ..., 2N - 1 are used as ``companion'' threads to load data
for each of the N processing threads.

```cpp
Data** data = new Data*[N];

#pragma omp parallel num_threads(N * 2)
{
  data[threadID] = NULL;

  if (threadID >= N) { // Loading data
    const int index = threadID - N;
    data[index] = new Data(index);
    Send_Data_Ready_Signal();
  } else { // Processing
    Wait_Until_Data_Ready();
    Process(data[threadID]);
    delete[] data[threadID];
    data[threadID] = NULL;
  }
}
```

#Observations

The program seems to work fine when N = 1.
When N = 2, it works most of the time. Sometimes it executes all the logic
correctly, but the program fails to exit with a correct error code 0.
After a compiler upgrade, it crashes when N >= 2 every time.
If the ``delete'' clause is removed, it seems to work fine again.

#Cause

Although the program has many places to improve, the root cause of the bug
is in the line right after entering the openmp environment:

```cpp  
data[threadID] = NULL;
```

This writes out of bound, because threadID is in [0, 2 * N), while data has only N elements allocated.
The error is there for N = 1, 2, ..., but does not cause visible damage when N = 1.

#Lessons learned

+ Whenever an array is accessed, boundary check should be performed either
  implicitly or explicitly. Here the variable threadID should never be used
  as an index to access the array data, even in the following correct but
  confusing case:

```cpp
  if (threadID < N) { // Extra burden for others to read the code
    data[threadID] = NULL;
  }
```
  
+ A function that is longer than 100 lines is evil. This example was extracted
  from a function of more than 1500 lines!!! It would be much easier to spot
  the bug if the code is better structured.
