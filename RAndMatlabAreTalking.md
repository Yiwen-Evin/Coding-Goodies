R and Matlab Are Talking
=================================

As a statistician, having collaboration with scientists and engineers is nice.  
But the coding languages we use are different.  It would be nicer if our code can talk to 
each other. 

Matlab calling R
------------------
Basically, we are calling Rscript in Matlab with a Matlab function 

```
[status] = system('Rscript R\analysis_accuracySimulation.R ');
```
