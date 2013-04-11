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
or, 
```
[status] = system('R CMD BATCH --vanilla --quiet R\analysis_accuracySimulation.R output');
```
The syntex is the same as calling Rscript in Command Prompt. Rscript and R CMD
BATCH are two ways of running r scripts.  The differences between the two are in
the default option settings, input arguements and output file.  In general, I
recommend R CMD BATCH.  Please find the list of useful options and a comparison
between Rscript and R CMD BATCH in the appendix. 

About the .R file:
- At the beginning of the R function, need to add libarary `R.matlab'.
- Source other .R files.  In the example, I sourced 
  1. `variable.R' which defines the value of the variables will be used.  It
  can be easily modified by the users.
  2. `simulationFun.R' which is the main function doing simulation.
- At the end of the .R file, everything wanted should be saved into a .mat
  file with the `writeMat' function. 

```{r, eval=FALSE}
outputpath <- "C:/eclipse/workspace/MatlabCodeADC/output/Results.mat"
writeMat(outputpath, allDf=try$plotDf, simDf = try$simData)
```


