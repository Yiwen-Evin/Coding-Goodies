R and Matlab Are Talking
=================================
  	
As a statistician, having collaboration with scientists and engineers is nice.  
But the coding languages we use are different.  It would be nicer if our code can talk to 
each other. 

# Matlab calling R

## Example
Basically, we are calling Rscript in Matlab with a Matlab function 
```
[status] = system('Rscript R\analysis_accuracySimulation.R ');
```
or, 
```
[status] = system('R CMD BATCH --vanilla --quiet R\analysis_accuracySimulation.R output')
```
The syntex is the same as calling Rscript in Command Prompt. Rscript and R CMD
BATCH are two ways of running r scripts.  The differences between the two are in
the default option settings, input arguements and output file.  In general, I
recommend R CMD BATCH.  Please find the list of useful options and a comparison
between Rscript and R CMD BATCH in the appendix. 

About the .R file:
- At the beginning of the R function, need to add libarary 'R.matlab'.
- Source other .R files.  In the example, I sourced 
	1.  'variable.R' which defines the value of the variables will be used.  It
can be easily modified by the users.
	2.  'simulationFun.R' which is the main function doing simulation.
- At the end of the .R file, everything wanted should be saved into a .mat
file with the 'writeMat' function. 

```
outputpath <- "C:/eclipse/workspace/MatlabCodeADC/output/Results.mat"
writeMat(outputpath, allDf=try$plotDf, simDf = try$simData)
```


Then, we need to load the saved .mat file in Matlab:
```
load('C:/eclipse/workspace/MatlabCodeADC/output/Results.mat');
```

See the demo file 'script.m' can be found at <http://ctt.bdx.com:9090/svn/R/Steve/Matlab Code>

For more details, please see <http://rwiki.sciviews.org/doku.php?id=tips:callingr:matlab>


### Things to notice

When calling R in matlab, the program automatically saves the plots created in
the .R file into a pdf called 'Rplots.pdf'.   These are the plots that would have shown in the R Graphics
window if you run R directly, but not those plots that you saved into a pdf file
with function 'pdf()'.

The objects saved to .mat file have different structures from those in .RData
file.  For example, the most commonly used object class 'data.frame' will look
bad in '.mat' file.  Each column in the data frame will be saved as one object
in an 'subOjbect' as a $n \times 1$ matrix.  It's better to save a matrix, when
every elements is numerical, so that Matlab can load that as a matrix. The
following is a comparison between objects saved as matrix, and those saved as
data frame.  'simM' is saved as matrix, the two columns of which are 
`RD' and
`Time'.

```
>> simM(1:5,:)

ans =
1.4149   -8.5833
1.7683   -3.5833
1.7378    1.4167
1.3745    6.4167
1.2732   11.4167

>> simDf

simDf = 
Set: {120x1 cell}
RD: [120x1 double]
GlucoseRegion: {120x1 cell}
LineGroup: {120x1 cell}
Time: [120x1 double]
```

If a data.frame is saved to the '.mat' file, make sure that the column names
doesn't include '.'.  Matlab will problems reading that object. 
		
		
# R calling Matlab

## Get prepared

First, load the needed packages.
```
require(R.matlab)
require(R.utils)
```

Start Matlab server on the local machine. 
```
Matlab$startServer()   
```

Create a Matlab client object used to communicate with Matlab
```
matlab <- Matlab()
```

Connect to the Matlab server, which must already be running 
```
if (!open(matlab))
throw("Matlab server is not running: waited 30 seconds.")
```

## Example 1: a very basic expression.
Run Matlab expressions on the Matlab server
```
evaluate(matlab, "A=1+2;", "B=ones(2,20);")
```

Note the variables 'A' and 'B' are not saved as .mat files.  To get these
temporary Matlab variables
```
data <- getVariable(matlab, c("A", "B"))
cat("Received variables:\n")
str(data)
# List of 2
#  $ A: num [1, 1] 3
#  $ B: num [1:2, 1:20] 1 1 1 1 1 1 1 1 1 1 ...
#  - attr(*, "header")=List of 3
#   ..$ description: chr "MATLAB 5.0 MAT-file, Platform: PCWIN64, Created on: Thu Nov 29 14:47:47 2012                                        "
#   ..$ version    : chr "5"
#   ..$ endian     : chr "little"
```

## Example 2: set variables in Matlab.
```
ABCD <- matrix(rnorm(10), ncol=2)
setVariable(matlab, ABCD=ABCD)
```

However, when reading these variables back into R, it is no longer a nicely
formatted matrix. 

```
data <- getVariable(matlab, "ABCD")
str(data)
# List of 1
#  $ ABCD: num [1:5, 1:2] 0.545 0.51 0.061 0.766 1.273 ...
#  - attr(*, "header")=List of 3
#   ..$ description: chr "MATLAB 5.0 MAT-file, Platform: PCWIN64, Created on: Tue Nov 27 15:39:39 2012                                        "
#   ..$ version    : chr "5"
#   ..$ endian     : chr "little"
```

## Example 3: creating functions

```
setFunction(matlab, "          \
function [win,aver]=dice(B) \
%Play the dice game B times \
gains=[1:B];        		\
plays=rand(B,1);        	\
win=sum(gains*plays);       \
aver=win/B;                 \
");
```

Try the function out. 
```
evaluate(matlab, "[w,a]=dice(6);")
res <- getVariable(matlab, c("w", "a"))
res
# $w
#          [,1]
# [1,] 12.47251
# 
# $a
#          [,1]
# [1,] 2.078752
# 
# attr(,"header")
# attr(,"header")$description
# [1] "MATLAB 5.0 MAT-file, Platform: PCWIN64, Created on: Thu Nov 29 14:54:43 2012                                        "
# 
# attr(,"header")$version
# [1] "5"
# 
# attr(,"header")$endian
# [1] "little"   
```
		
		
## Example 4: general Matlab commands
We've seen the function 'evaluate' several times so far.  The function
			`evaluate' is the key to call Matlab commands.
It works as
```
evaluate(matlab, "%The command line you would put in Matlab%");
```
Here are a few examples.  

Adding path:
```
evaluate(matlab, "addpath('C:/eclipse/workspace/MatlabCodeADC/Matlab')")
```
		
Calling the function 'simple' which is saved in the simple.m file in the above
folder that we just added to the path. 
```
evaluate(matlab, '[output]=simple(1, 0, 1, 2, 5);')
result <- getVariable(matlab, "output")
result
# $output
#      [,1]
# [1,]    8
# 
# attr(,"header")
# attr(,"header")$description
# [1] "MATLAB 5.0 MAT-file, Platform: PCWIN64, Created on: Thu Nov 29 09:46:45 2012                                        "
# 
# attr(,"header")$version
# [1] "5"
# 
# attr(,"header")$endian
# [1] "little"
```



## Read in Matlab data files
Read in the data we simulated and saved in to .mat file. 
```
result <- readMat("output/Results.mat", verbos=TRUE)
```

Again, the data read back into Matlab have different structure than they are in
the RData file.  The above code is the same example as showed in section
[Things to consider].  'result' is a list of $3$.  The first two are saved from the data
		frames, while the third one is saved from the matrix 'simM'.  The numerical
		vectors will be saved as a vector as one element in the list.  The character
		string vector will be saved as a list, and that list, is one element in the
		upper level list.  The following example can explain this better. 
		
```
simDf <- result[[2]]
length(simDf)
# [1] 5
		
v<- simDf[[1]]
		
head(v, n=3)
# $<NA>
#      [,1]       
# [1,] "Simulated"
# 
# $<NA>
#      [,1]       
# [1,] "Simulated"
# 
# $<NA>
#      [,1]       
# [1,] "Simulated"
		
		
simM <- result[[3]]
class(simM)
# [1] "matrix"
		
head(simM, n=3)
#           [,1]      [,2]
# [1,] 1.4149073 -8.583333
# [2,] 1.7683024 -3.583333
# [3,] 1.7378493  1.416667
```
		
		
My suggestion is, always save things in R as a matrix.  That would save a lot of
trouble when transfer things between Matlab and R. 
		
