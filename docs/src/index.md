
# Trait Simulation Tutorial

Authors: Sarah Ji, Janet Sinsheimer, Kenneth Lange

```@meta
CurrentModule = TraitSimulation
```

```@autodocs
Modules = [TraitSimulation]
```

In this notebook we show how to use the `TraitSimulation.jl` package to simulate traits from genotype data from unrelateds or families with user-specified Generalized Linear Models (GLMs) or Linear Mixed Models (LMMs), respectively. For simulating under either GLM or LMMs, the user can specify the number of repitions for each simulation model. By default, the simulation will return the result of a single simulation. 

The data we will be using are from the Mendel version 16[1] sample files. The data are described in examples under Option 28e in the Mendel Version 16 Manual [Section 28.1,  page 279](http://software.genetics.ucla.edu/download?file=202). It consists of simulated data where the two traits of interest have one contributing SNP and a sex effect.

We use the OpenMendel package [SnpArrays.jl](https://openmendel.github.io/SnpArrays.jl/latest/) to read in the PLINK formatted SNP data. In example 1, we simulate generalized linear models assuming that everyone is unrelated. So in example 1,  the only data used from option 28e is the genotype for a specific locus in the snp file and the sex of the individual. The pedigree structure and relationship matrix are irrelevant.   In example 2 we simulate data under a linear mixed model so that we can model residual dependency among individuals.  In example 2b, we use the same parameters as were used in Mendel Option 28e with the simulation parameters for Trait1 and Trait2 in Ped28e.out as shown below.

In both examples, you can specify your own arbitrary fixed effect sizes, variance components and simulation parameters as desired. You can also specify the number of replicates for each Trait simulation in the `simulate` function.

In the $\mathbf{Generating}$ $\mathbf{Effect}$ $\mathbf{Sizes}$ Section of Example 2), we show how the user can generate effect sizes that depend on the minor allele frequencies from the chisquare distribution. To aid the user when they wish to include a large number of loci in the model, we created a function that automatically writes out the mean components for simulation.

$\textbf{At the end of Examples 1 and 3}$, we demo how to $\textbf{write the results}$ of the simulation to a file on your own machine.

### Double check that you are using Julia version 1.0 or higher by checking the machine information


```julia
versioninfo()
```

    Julia Version 1.1.1
    Commit 55e36cc (2019-05-16 04:10 UTC)
    Platform Info:
      OS: macOS (x86_64-apple-darwin15.6.0)
      CPU: Intel(R) Core(TM) i7-7700HQ CPU @ 2.80GHz
      WORD_SIZE: 64
      LIBM: libopenlibm
      LLVM: libLLVM-6.0.1 (ORCJIT, skylake)


# Add any missing packages needed for this tutorial:

Note: For demonstration purposes, the generation of this Jupyter Notebook requires the use of the following registered packages: `DataFrames.jl`, `SnpArrays.jl`, `StatsModels.jl`, `Random.jl`, `DelimitedFiles.jl`, `StatsBase.jl`, and `StatsFuns.jl`. 

If it is your first time using these registered packages, you will first have to add the registered packages: DataFrames, SnpArrays, StatsModels, Random, LinearAlgebra, DelimitedFiles, Random, StatsBase by running the following code chunk in Julia's package manager:

```{julia}
pkg> add DataFrames
pkg> add SnpArrays
...
pkg> add StatsFuns
```
You can also use the package manager to add the `TraitSimulation.jl` package by running the following link: </br>

```{julia}
pkg> add "https://github.com/sarah-ji/TraitSimulation.jl"
```

Only after all of the necessary packages have been added, load them into your working environment with the `using` command:


```julia
using DataFrames, SnpArrays, StatsModels, Random, LinearAlgebra, DelimitedFiles, StatsBase, TraitSimulation, StatsFuns
using CSV
```

# Reproducibility

For reproducibility, we set a random seed using the `Random.jl` package for each simulation using `Random.seed!(1234)`.  If you wish to end up with different data, you will need to comment out these commands or use another value in Random.seed!().


```julia
Random.seed!(1234);
```

# The notebook is organized as follows:

## Example 1: Generalized Linear Model
In this example we show how to generate multiple traits from GLM's with a genetic variant in the fixed effects, but no residual familial correlation.

### Single Trait:
$$Y âŒ N(\mu, \sigma^{2})$$

In example (1a) we simulate a $\textbf{SINGLE INDEPENDENT NORMAL TRAIT}$, with simulation parameters: $\mu = 20 + 3*sex - 1.5*locus$, $\sigma^{2} = 2$. By default, without specifying a number of replicates for the user specified GLM (like this example), the `simulate` function returns a single simulated trait.

## Example 2: Linear Mixed Model
In this example we show how to generate data so that the related individuals have correlated trait values even after we account for the effect of a snp, a combination of snps or other fixed effects.

For convenience we use the common assumption that the residual covariance among two relatives can be captured by the additive genetic variance times twice the kinship coefficient. However, if you like you can specify your own variance components and their design matrices as long as they are positive semi definite using the `@vc` macro demonstrated in this example. We run this simulation 1000 times, and store the simulation results in a vector of DataFrames.

### (a) Multiple Independent Traits:
$$Y âŒ N(\mu, 4* 2GRM + 2I, n_{reps} = 10)$$

In example (2a) we simulate $\textbf{MULTIPLE 2 INDEPENDENT TRAITS CONTROLLING FOR FAMILY STRUCTURE}$, with the corresponding Mendel Example 28e Simulation parameters, location : $\mu = 40 + 3*sex - 1.5*locus$, scale : $V$ = 4* 2GRM + 2I$.$ We run this simulation 1000 times.

## (b) Multiple Correlated Traits: (Mendel Example 28e Simulation)

$$ Y = 
\begin{bmatrix}
Y_{1}\\
Y_{2}
\end{bmatrix}, Y_{1} \not\!\perp\!\!\!\perp Y_{2}
$$

$$Y \sim N(\mathbf{\mu},\Sigma  = V_{a} \otimes (2GRM) + V_{e} \otimes I_{n}) ,$$


$$
\mathbf{\mu} = \begin{bmatrix}
\mu_1 \\
\mu_2 \\
\end{bmatrix}
= \begin{bmatrix}
40 + 3(sex) - 1.5(locus)\\
20 + 2(sex) - 1.5(locus)\\
\end{bmatrix} , V_{a} = \begin{bmatrix}
4 & 1\\
1 & 4\\
\end{bmatrix} , V_e 
= \begin{bmatrix}
2 & 0\\
0 & 2\\
\end{bmatrix}
$$


We simulate $\textbf{TWO CORRELATED TRAITS CONTROLLING FOR FAMILY STRUCTURE}$ with simulation parameters, location = $\mu$ and scale = $\Sigma$. By default, without specifying a number of replicates for the user specified LMM (like this example), the `simulate` function returns a single set of simulated traits.


## Example 3: Rare Variant Linear Mixed Model 

This example is meant to simulate data in a scenario in which a number of rare mutations in a single gene can change a trait value.  In this example we model the residual variation among relatives with the additive genetic variance component and we include 20 rare variants in the mean portion of the model, defined as loci with minor allele frequencies greater than 0.002 but less than 0.02.  In practice rare variants have smaller minor allele frequencies, but we are limited in this tutorial by the relatively small size of the data set. Note also that our modeling these effects as part of the mean is not meant to imply that the best way to detect them would be a standard association analysis. Instead we recommend a burden or SKAT test. <br>

Specifically we are generating a single normal trait controlling for family structure with residual heritabiity of 67%, and effect sizes for the variants generated as a function of the minor allele frequencies. The rarer the variant the greater its effect size.

We run this simulation 1000 times, and store the simulation results in a vector of DataFrames. At the end of this example we write the results of the first of the 1000 replicates to a file on your own machine.

$$ Y âŒ N(\mu_{rare20}, 4* 2GRM + 2I, n_{reps} = 1000)
$$

# Reading the Mendel 28a data using SnpArrays

First use `SnpArrays.jl` to read in the genotype data. The value 212 is the number of individuals in the genotype data set. 



```julia
snpdata = SnpArray("traitsim28e.bed", 212)
```




    212Ã253141 SnpArray:
     0x03  0x03  0x00  0x03  0x03  0x03     0x02  0x02  0x00  0x00  0x03  0x00
     0x03  0x03  0x00  0x02  0x02  0x03     0x00  0x03  0x00  0x00  0x03  0x00
     0x03  0x03  0x00  0x03  0x03  0x03     0x03  0x02  0x00  0x00  0x03  0x00
     0x03  0x03  0x00  0x03  0x03  0x03     0x02  0x03  0x00  0x00  0x03  0x00
     0x03  0x03  0x00  0x03  0x03  0x03     0x00  0x03  0x00  0x00  0x03  0x00
     0x03  0x03  0x00  0x03  0x03  0x03     0x00  0x00  0x00  0x00  0x00  0x03
     0x03  0x02  0x00  0x03  0x03  0x03     0x02  0x03  0x00  0x03  0x00  0x03
     0x03  0x03  0x00  0x03  0x03  0x03     0x02  0x03  0x00  0x03  0x00  0x03
     0x03  0x02  0x00  0x03  0x03  0x03     0x02  0x02  0x00  0x02  0x00  0x03
     0x03  0x02  0x00  0x03  0x03  0x03     0x03  0x03  0x00  0x03  0x00  0x03
     0x03  0x02  0x00  0x03  0x03  0x03     0x00  0x02  0x00  0x02  0x00  0x03
     0x03  0x03  0x00  0x03  0x03  0x03     0x00  0x02  0x00  0x02  0x00  0x03
     0x03  0x02  0x00  0x03  0x03  0x03     0x02  0x02  0x00  0x02  0x00  0x03
     0x03  0x03  0x00  0x03  0x03  0x03     0x00  0x03  0x00  0x00  0x03  0x00
     0x03  0x03  0x00  0x03  0x03  0x03     0x00  0x03  0x00  0x02  0x02  0x02
     0x03  0x03  0x00  0x03  0x03  0x03     0x00  0x02  0x00  0x00  0x03  0x00
     0x03  0x02  0x00  0x02  0x02  0x03     0x02  0x03  0x00  0x03  0x00  0x03
     0x03  0x03  0x00  0x02  0x02  0x03     0x02  0x03  0x00  0x00  0x03  0x00
     0x03  0x03  0x00  0x03  0x03  0x03     0x02  0x03  0x00  0x02  0x02  0x00
     0x03  0x03  0x00  0x02  0x02  0x03     0x02  0x03  0x00  0x00  0x02  0x02
     0x03  0x03  0x00  0x03  0x03  0x03     0x00  0x03  0x00  0x00  0x03  0x00
     0x03  0x02  0x00  0x03  0x03  0x03     0x02  0x03  0x00  0x00  0x02  0x02
     0x03  0x03  0x00  0x03  0x03  0x03     0x00  0x03  0x00  0x00  0x03  0x00
     0x03  0x03  0x00  0x03  0x03  0x03     0x02  0x03  0x00  0x00  0x03  0x00
     0x03  0x03  0x00  0x03  0x03  0x03     0x00  0x03  0x00  0x02  0x02  0x02



The first line gives the size of the matrix. 212 individuals by 253141, the number of loci.

The binary codes correspond to genotypes, A1,A1=0x00, missing=0x01, A1,A2=0x02 and A2,A2=0x03

SnpArrays is a very useful utility and can do a lot more than just read in the data. More information about all the functionality of SnpArrays can be found at:
https://openmendel.github.io/SnpArrays.jl/latest/

Store the FamID and PersonID of Individuals in Mendel 28e data


```julia
famfile = readdlm("traitsim28e.fam", ',')
Fam_Person_id = DataFrame(FamID = famfile[:, 1], PID = famfile[:, 2])
```


Note: Because later we will want to compare our results to Mendel 28e results,  we subset `Traits_Mendel28e` 


```julia
Traits_Mendel28e = DataFrame(Trait1 = famfile[:, 7], Trait2 = famfile[:, 8])
```


Transform sex variable from M/F to 1/-1 as is done in the older version of Mendel.  If you prefer you can use the more common convention of making one of the sexes the reference sex (coding it as zero) and make the other sex have the value 1 but then you will have to work a little harder to compare the results to the older version of Mendel. 


```julia
sex = map(x -> strip(x) == "F" ? -1.0 : 1.0, famfile[:, 5]) # note julia's ternary operator '?'
```

    212-element Array{Float64,1}:
     -1.0
     -1.0
      1.0
      1.0
     -1.0
     -1.0
      1.0
      1.0
     -1.0
      1.0
     -1.0
      1.0
     -1.0
      1.0
      1.0
      1.0
     -1.0
      1.0
      1.0
      1.0
      1.0
      1.0
      1.0
      1.0
      1.0



### Names of Variants:

We will use snp rs10412915 as a covariate in our model.  We want to find the index of this causal locus in the snp_definition file and then subset that locus from the genetic marker data above. 
We first subset the names of all the loci into a vector called `snpid`


```julia
snpdef28_1 = readdlm("traitsim28e.bim", Any; header = false)
snpid = map(x -> strip(string(x)), snpdef28_1[:, 1]) # strip mining in the data 
```




    253141-element Array{SubString{String},1}:
     "rs3020701"  
     "rs56343121" 
     "rs143501051"
     "rs56182540" 
     "rs7260412"  
     "rs11669393" 
     "rs181646587"
     "rs8106297"  
     "rs8106302"  
     "rs183568620"
     "rs186451972"
     "rs189699222"
     "rs182902214"        
     "rs188169422"
     "rs144587467"
     "rs139879509"
     "rs143250448"
     "rs145384750"
     "rs149215836"
     "rs139221927"
     "rs181848453"
     "rs138318162"
     "rs186913222"
     "rs141816674"
     "rs150801216"



We next need to find the position of the snp rs10412915.  If you wish to use another snp as the causal locus just change the rs number to another one that is found in the available genotype data, for example rs186913222.


```julia
ind_rs10412915 = findall(x -> x == "rs10412915", snpid)[1]
```

    236074



We see that rs10412915, is the 236074th locus in the dataset.

Let's create a design matrix for the model that includes sex and locus rs10412915.


```julia
locus = convert(Vector{Float64}, @view(snpdata[:, ind_rs10412915]))
X = DataFrame(sex = sex, locus = locus)
```

# Example 1 Generalized Linear Model:

This example simulates a case where three snps have fixed effects on the trait. Any apparent genetic correlation between relatives for the trait is due to the effect of these snps, so once these effects of these snps are modelled there should be no residual correlation among relatives. Note that by default, individuals with missing genotype values will have missing phenotype values, unless the user specifies the argument `impute = true` in the convert function above.
Be sure to change Random.seed!(1234) to something else (or comment out) if you want to generate a new data set. 


### Example 1a: Single Trait
$$Y âŒ N(\mu, \sigma^{2})$$

In example (1a) we simulate a $\textbf{SINGLE INDEPENDENT NORMAL TRAIT}$, with simulation parameters: $\mu = 20 + 3*sex - 1.5*locus$, $\sigma^{2} = 2$


```julia
mean_formula = "20 + 3(sex) - 1.5(locus)"
GLM_trait_model = GLMTrait(mean_formula, X, NormalResponse(2), IdentityLink())
Simulated_GLM_trait = simulate(GLM_trait_model)
```



```julia
describe(Simulated_GLM_trait, stats = [:mean, :std, :min, :q25, :median, :q75, :max, :eltype])
```

## Saving Simulation Results to Local Machine

Write the newly simulated trait into a comma separated (csv) file for later use. Note that the user can specify the separator to '\t' for tab separated, or another separator of choice. 

Here we output the simulated trait and covariates for each of the 212 individuals, labeled by their pedigree ID and person ID.


```julia
Trait1_GLM = hcat(Fam_Person_id, Simulated_GLM_trait, X)
```



```julia
#cd("/Users") #change to home directory
CSV.write("Trait1_GLM.csv", Trait1_GLM)
```




    "Trait1_GLM.csv"



# Example 2: Linear Mixed Model (with additive genetic variance component).
Examples 2a simulates a single trait, while Example 2b simulates two correlated traits.

Note you can scale the function to simulate the trait multiple times by specifying the argument, `n_reps`. 
Also, you can extend the model in Example 2b to include more than 2 variance components using the `@vc` macro.


## The Variance Covariance Matrix

Recall : $E(\mathbf{GRM}) = \Phi$

We use the [SnpArrays.jl](https://github.com/OpenMendel/SnpArrays.jl) package to find an estimate of the Kinship ($\Phi$), the Genetic Relationship Matrix (GRM). 

We will use the same values of $\textbf{GRM, V_a, and V_e}$ in the bivariate covariance matrix for both the mixed effect example and for the rare variant example.

Note that the residual covariance among two relatives is the additive genetic variance, $\textbf{V_a}$, times twice the kinship coefficient, $\Phi$. The kinship matrix is derived from the genetic relationship matrix $\textbf{GRM}$ across the common SNPs with minor allele frequency at least 0.05.


```julia
GRM = grm(snpdata, minmaf = 0.05)
```




    212Ã212 Array{Float64,2}:
      0.498264     0.0080878    0.0164327   âŠ   0.0246825    0.00181856
      0.0080878    0.498054    -0.0212599      -0.0285927   -0.0226525 
      0.0164327   -0.0212599    0.499442       -0.0219661   -0.00748536
      0.253627    -0.00160532   0.282542        0.00612693  -0.00339125
      0.126098     0.253365     0.128931       -0.0158446   -0.00633959
     -0.014971    -0.00266073  -0.00243384  âŠ   0.00384757   0.0145936 
     -0.0221357    0.0100492   -0.0107012      -0.0148443   -0.00127783
     -0.01629     -0.00749253  -0.015372       -0.0163305   -0.00258392
     -0.016679     0.00353587  -0.0128844      -0.0332489   -0.00707839
     -0.0176101   -0.00996912  -0.0158473      -0.00675875  -0.0122339 
     -0.0162558    0.00938592   0.0064231   âŠ  -0.00510882   0.0168778 
     -0.0167487    0.00414544  -0.00936538     -0.0134863    0.0020952 
     -0.031148     0.00112387  -0.010794        0.00383105   0.0198635 
      â®                                     â±   â®                      
     -0.00865735  -0.00335548  -0.0148433   âŠ   0.00806601  -0.0211537 
      0.00296028   0.0043655   -0.0183683       0.0012496    0.00898193
     -0.0204601   -0.0270898   -0.00194048     -0.0185883   -0.0116621 
     -0.0174561   -0.0128509   -0.0155773      -0.0274183   -0.0063823 
     -0.00170995   0.0154211   -0.00168146     -0.00684865  -0.0067438 
      0.00718047  -0.00525265  -0.00283975  âŠ   0.0309601    0.0261103 
     -0.0170218   -0.00661916   0.0020924      -0.022858     0.0037451 
      0.0142551    0.0208073    0.0096287       0.00598877   0.0094809 
     -0.00586031  -0.00733706   0.0339257       0.0109116   -0.0177771 
      0.00299024  -0.0134027    0.0150825       0.00799507   0.0150077 
      0.0246825   -0.0285927   -0.0219661   âŠ   0.593999     0.0497083 
      0.00181856  -0.0226525   -0.00748536      0.0497083    0.491743  



### Example 2a: Single Trait 
$$
Y âŒ N(ÎŒ, 4* 2GRM + 2I, n_{reps} = 1000)$$

We simulate a Normal Trait controlling for family structure, location = $\mu = 40 + 3(sex) - 1.5(locus)$ and scale =  $\mathbf{V} = 2*V_a \Phi + V_e I = 4* 2GRM + 2I$. 



```julia
mean_formula = ["40 + 3(sex) - 1.5(locus)"]
```



    1-element Array{String,1}:
     "40 + 3(sex) - 1.5(locus)"




```julia
I_n = Matrix{Float64}(I, size(GRM));
LMM_trait_model = LMMTrait(mean_formula, X, 4*(2*GRM) + 2*(I_n))
Simulated_LMM_Trait = simulate(LMM_trait_model, 1000)
```

Let's look at summary statistics of just the first of the 1000 simulation results.


```julia
describe(Simulated_LMM_Trait[1], stats = [:mean, :std, :min, :q25, :median, :q75, :max, :eltype])
```


###  Example 2b: Multiple Correlated Traits (Mendel Example 28e Simulation)

We simulate two correlated Normal Traits controlling for family structure, location = $ÎŒ$ and scale = $\mathbf\Sigma$. 
The corresponding bivariate variance covariance matrix as specified Mendel Option 28e, $\mathbf{Î£}$, is generated here.

$$
Y âŒ N(ÎŒ, \mathbf\Sigma)
$$ 

$$
\mathbf{\mu} = \begin{vmatrix}
\mu_1 \\
\mu_2 \\
\end{vmatrix}
= \begin{vmatrix}
40 + 3(sex) - 1.5(locus)\\
20 + 2(sex) - 1.5(locus)\\
\end{vmatrix}
\\
$$

$$
\mathbf\Sigma  = V_a \otimes (2GRM) + V_e \otimes I_n
$$


&nbsp; $FYI$: To create a trait with different variance components change the elements of $\mathbf\Sigma$. We create the variance component object `variance_formula` below, to simulate our traits in example 2b. While this tutorial only uses 2 variance components, we make note that the `@vc` macro is designed to handle as many variance components as needed. 

As long as each Variance Component is specified correctly, we can create a `VarianceComponent` Julia object for Trait Simulation:

&nbsp; 
Example) Specifying more than 2 variance components (let V_H indicate an additional Household Variance component and V_D indicate a dominance genetic effect) 

```{julia}
    multiple_variance_formula = @vc V_A â 2GRM + V_E â I_n + V_D â Î + V_H â H;
```

V_E is multiplies a 212 by 212 identity matrix, which we creat along with the V_E and V_A matrices. 


```julia
V_A = [4 1; 1 4]
V_E = [2.0 0.0; 0.0 2.0];
```


```julia
# @vc is a macro that creates a 'VarianceComponent' Type for simulation
variance_formula = @vc V_A â 2GRM + V_E â I_n;
```

These are the formulas for the fixed effects, as specified by Mendel Option 28e.


```julia
mean_formulas = ["40 + 3(sex) - 1.5(locus)", "20 + 2(sex) - 1.5(locus)"]
```




    2-element Array{String,1}:
     "40 + 3(sex) - 1.5(locus)"
     "20 + 2(sex) - 1.5(locus)"




```julia
Multiple_LMM_traits_model = LMMTrait(mean_formulas, X, variance_formula)
Simulated_LMM_Traits = simulate(Multiple_LMM_traits_model)
```


### Summary Statistics of Our Simulated Traits


```julia
describe(Simulated_LMM_Traits, stats = [:mean, :std, :min, :max, :eltype])
```



### Summary Statistics of the Original Mendel 28e dataset Traits:

Note we want to see similar values from our simulated traits!


```julia
describe(Traits_Mendel28e, stats = [:mean, :std, :min, :max, :eltype])
```



# Example 3: Rare Variant Linear Mixed Model


$$
Y âŒ N(\mu_{rare20}, 4* 2GRM + 2I)
$$

In this example we first subset only the rare SNP's with minor allele frequency greater than 0.002 but less than 0.02, then we simulate traits on 20 of the rare SNP's as fixed effects. For this demo, the indexing `snpid[rare_index][1:2:40]` allows us to subset every other rare snp in the first 40 SNPs, to get our list of 20 rare SNPs. Change the range and number of SNPs to simulate with more or less SNPs and from different regions of the genome. The number 20 is arbitrary and you can use more or less than 20 if you desire by changing the final number. You can change the spacing of the snps by changing the second number. 
For example, `snpid[rare_index][1:5:500]` would give you 100 snps.

Here are the 20 SNP's that will be used for trait simulation in this example.  

In this demo, we run this simulation 1000 times. You can change the number of repitions by changing the second argument in the `simulate(rare_20_snp_model, 1000)` function to any integer.


```julia
# filter out rare SNPS, get a subset of uncommon SNPs with 0.002 < MAF â€ 0.02
minor_allele_frequency = maf(snpdata)
rare_index = (0.002 .< minor_allele_frequency .â€ 0.02)
data_rare = @view(snpdata[:, rare_index]);
```


```julia
maf_20_rare_snps = minor_allele_frequency[rare_index][1:2:40]
rare_snps_for_simulation = snpid[rare_index][1:2:40]
```




    20-element Array{SubString{String},1}:
     "rs3020701"  
     "rs181646587"
     "rs182902214"
     "rs184527030"
     "rs10409990" 
     "rs185166611"
     "rs181637538"
     "rs186213888"
     "rs184010370"
     "rs11667161" 
     "rs188819713"
     "rs182378235"
     "rs146361744"
     "rs190575937"
     "rs149949827"
     "rs117671630"
     "rs149171388"
     "rs188520640"
     "rs142722885"
     "rs146938393"



## Generating Effect Sizes (Based on MAF)

In practice rare SNPs have smaller minor allele frequencies but we are limited in this tutorial by the number of individuals in the data set. We use generated effect sizes to evaluate $\mu_{rare20}$ on the following Dataframe: <br> 


```julia
geno_rare_converted = convert(Matrix{Float64}, data_rare);
```


```julia
geno_rare20_converted = convert(DataFrame, geno_rare_converted[:, 1:2:40])
names!(geno_rare20_converted, Symbol.(rare_snps_for_simulation))
```

### Chisquared Distribution (df = 1)

For demonstration purposes, we simulate effect sizes from the Chi-squared(df = 1) distribution, where we use the minor allele frequency (maf) as x and find f(x) where f is the pdf for the Chi-squared (df = 1) density, so that the rarest SNP's have the biggest effect sizes. The effect sizes are rounded to the second digit, throughout this example. Notice there is a random +1 or -1, so that there are effects that both increase and decrease the simulated trait value.


```julia
# Generating Effect Sizes from Chisquared(df = 1) density
n = length(maf_20_rare_snps)
chisq_coeff = zeros(n)

for i in 1:n
    chisq_coeff[i] = rand([-1, 1]) .* (0.1 / sqrt.(maf_20_rare_snps[i] .* (1 - maf_20_rare_snps[i])))
end
```

Take a look at the simulated coefficients on the left, next to the corresponding minor allele frequency. Notice the rarer SNPs have the largest absolute values for their effect sizes.


```julia
Ex3_rare = round.([chisq_coeff maf_20_rare_snps], digits = 3)
Ex3_rare = DataFrame(Chisq_Coefficient = Ex3_rare[:, 1] , MAF_rare = Ex3_rare[:, 2] )
```


```julia
simulated_effectsizes_chisq = Ex3_rare[:, 1]
```




    20-element Array{Float64,1}:
     -0.785
     -0.847
      1.034
     -0.735
      1.034
     -1.459
      1.193
     -1.034
     -1.193
      2.062
      0.847
      1.459
      2.062
     -1.459
      0.735
      2.062
      2.062
     -2.062
      0.735
     -1.459



## Function for Mean Model Expression

In some cases a large number of variants may be used for simulation. Thus, in this example we create a function where the user inputs a vector of coefficients and a vector of variants for simulation, then the function outputs the mean model expression. 

The function `FixedEffectTerms`, creates the proper evaluated expression for the simulation process, using the specified vectors of coefficients and snp names. The function outputs `evaluated_fixed_expression` which will be used to estimate the mean effect, `ÎŒ` in our mixed effects model. We make use of this function in this example, instead of having to write out all 20 of the coefficients and variant locus names.


```julia
rare_snps_for_simulation
```




    20-element Array{SubString{String},1}:
     "rs3020701"  
     "rs181646587"
     "rs182902214"
     "rs184527030"
     "rs10409990" 
     "rs185166611"
     "rs181637538"
     "rs186213888"
     "rs184010370"
     "rs11667161" 
     "rs188819713"
     "rs182378235"
     "rs146361744"
     "rs190575937"
     "rs149949827"
     "rs117671630"
     "rs149171388"
     "rs188520640"
     "rs142722885"
     "rs146938393"




```julia
function FixedEffectTerms(effectsizes::AbstractVecOrMat, snps::AbstractVecOrMat)
 # implementation
    fixed_terms = ""
for i in 1:length(simulated_effectsizes_chisq) - 1
expression = " + " * string(simulated_effectsizes_chisq[i]) * "(" * rare_snps_for_simulation[i] * ")"
    fixed_terms = fixed_terms * expression
end
    return String(fixed_terms)
end

```




    FixedEffectTerms (generic function with 1 method)



Example 3: Single Trait, Rare Variants

We look at just the first of the 1000 simulation results below.

$$
Y âŒ N(ÎŒ_{20raresnps}, 4* 2GRM + 2I, n_{reps} = 1000)$$


```julia
mean_formula_rare = FixedEffectTerms(simulated_effectsizes_chisq, rare_snps_for_simulation);
rare_20_snp_model = LMMTrait([mean_formula_rare], geno_rare20_converted, 4*(2*GRM) + 2*(I_n))
trait_rare_20_snps = simulate(rare_20_snp_model, 1000)[1]
```


Some summary statistics of just the first of the 1000 simulation results.


```julia
describe(trait_rare_20_snps[1], stats = [:mean, :std, :min, :max, :eltype])
```


## Saving Simulation Results to Local Machine

Here we output the simulated trait values and corresponding genotypes for each of the 212 individuals, labeled by their pedigree ID and person ID for the first iteration of the 1000 simulations. 


```julia
Trait3_rare = hcat(Fam_Person_id, trait_rare_20_snps[1], geno_rare20_converted)
```


In addition, we output the simulation parameters (generated effect sizes and SNP names) used to simulate this trait.


```julia
Coefficients = DataFrame(Coefficients = simulated_effectsizes_chisq)
SNPs_rare = DataFrame(SNPs = rare_snps_for_simulation)
Trait3_rare_sim = hcat(Coefficients, SNPs_rare)
```

```julia
#cd("/Users") #change to home directory
CSV.write("Trait3_rare.csv", Trait3_rare)
CSV.write("Trait3_rare_sim.csv", Trait3_rare_sim);
```

## Citations: 

[1] Lange K, Papp JC, Sinsheimer JS, Sripracha R, Zhou H, Sobel EM (2013) Mendel: The Swiss army knife of genetic analysis programs. Bioinformatics 29:1568-1570.`


[2] OPENMENDEL: a cooperative programming project for statistical genetics.
[Hum Genet. 2019 Mar 26. doi: 10.1007/s00439-019-02001-z](https://www.ncbi.nlm.nih.gov/pubmed/?term=OPENMENDEL).
