Should I be embarrassed to say that I still do things in Windows system?  Anyway, here it goes


## Compile CUDA code:
### Windows
To get *.exe file:
```
nvcc --machine 32 myfile.cu
nvcc --machine 32 *.cu
```
To get *.opi file:
```
nvcc -O2 --machine 32 *.cu
```
My advisor also used an option `-c`, which I do not quite understand. 

### Linux server 
``` 
nvcc *.cu
nvcc -O2 *.cu
```
The GPU server in the department doesn't work.  It keep saying that can not find command `opencuda', something like that. 

## Compile c++ code:
### Windows
```
g++ -O2 -c "-I/C:/Program Files/NVIDIA GPU Computing Toolkit/include" *.cpp
```
### Linux server
``` 
g++ -O2 -c -I/user/local/cuda-5.0/include *.cpp
```
Do not need the "" when there is no spaces in the path. 

## Link the *.o files
```
g++ -O2 OutputProg -L/user/local/cuda-5.0/include -lcudart *.o
```
Tried this out on my Windows machine, but it tells me some authority issue stopped it from compiling. 
Haven't get to try it out on the linux server, because the cuda file hasn't been compiled yet. 
