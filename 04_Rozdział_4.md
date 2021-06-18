---
layout: page
mathjax: true
title:  "Rozdział 4"
date:   2021-06-06 20:18:11 +0200
---

### 4.1.1. Wprowadzenie do generatorów liczb pseudolosowych

```julia
using Random, Distributions

Random.seed!(2305);
rand(Normal(0,1),6)
6-element Vector{Float64}:
 -1.1344364851189617
  0.3059407752133852
  0.23721940231011668
 -1.9463410058376256
 -1.5605769739603836
  0.6624700033213337
```
```julia
Random.seed!(2305);
rand(Normal(),6)
6-element Vector{Float64}:
 -1.1344364851189617
  0.3059407752133852
  0.23721940231011668
 -1.9463410058376256
 -1.5605769739603836
  0.6624700033213337
```

```julia
Random.seed!(1313);
rand(Uniform(),10)
10-element Vector{Float64}:
 0.7671316271344586
 0.27942702262234076
 0.09836449048362628
 0.40083683737378006
 0.09424398487826502
 0.9735010202063734
 0.6095683917137007
 0.5643348689905758
 0.6162205256628577
 0.09049210702210386
```
```julia
Random.seed!(1313);
rand(Uniform(0,1),10)
10-element Vector{Float64}:
 0.7671316271344586
 0.27942702262234076
 0.09836449048362628
 0.40083683737378006
 0.09424398487826502
 0.9735010202063734
 0.6095683917137007
 0.5643348689905758
 0.6162205256628577
 0.09049210702210386
```

### 4.1.2. Popularne rozkłady zmiennych losowych

```julia
rand(Uniform(),1)
1-element Vector{Float64}:
 0.41702365985754386
cdf(Uniform(),0.5)
0.5
pdf(Uniform(),0.5)
1.0
quantile(Uniform(),0.1)
0.1
```

```julia
using Plots

x = -4:0.01:4
plot(x,pdf(Normal(),x), lc="blue",legend = :topleft,ylabel="pdf()",label="pdf", size=(500,300))
plot!(twinx(),x, cdf(Normal(),x), lc="red", ls=:dash, legend = :topright,
      grid = :off, xaxis = false, xtickfont=font(1),ylabel="cdf()",label="cdf")
savefig("/home/.../assets/p412.png")
```
![](/assets/p412.png)

```julia
cdf(Normal(),-3)+(1-cdf(Normal(),3))
0.0026997960632601965
```
```julia
pdf(Normal(),-1:1)
3-element Vector{Float64}:
 0.24197072451914337
 0.3989422804014327
 0.24197072451914337
```
```julia
quantile(Normal(),[0.001,0.025,0.05,0.5,0.95,0.975,0.999])
7-element Vector{Float64}:
 -3.090232306167818
 -1.9599639845400592
 -1.6448536269514724
  0.0
  1.6448536269514717
  1.9599639845400576
  3.090232306167818
```
```julia
rand(Normal(2,1),10)
10-element Vector{Float64}:
 1.0975865340521673
 1.960510542721741
 2.618464863709529
 0.9357464997167511
 1.8565943221414611
 1.782642422711376
 0.9957240098168698
 0.44515676161698203
 2.1210216596289344
 0.9087856635902114
```
```julia
v = [rand(Normal(i,i),1) for i in 1:10]
 [-0.5686869384994986]
 [-1.6506538852245916]
 [3.800832239508454]
 [-6.013671068316878]
 [-0.8398128611045248]
 [11.913428487895864]
 [-2.380574030114838]
 [17.851646069846804]
 [6.766513241246967]
 [19.836425422978937]

Float64.(Iterators.flatten(v))
10-element Vector{Float64}:
 -0.5686869384994986
 -1.6506538852245916
  3.800832239508454
 -6.013671068316878
 -0.8398128611045248
 11.913428487895864
 -2.380574030114838
 17.851646069846804
  6.766513241246967
 19.836425422978937

```

#### 4.1.3.1 Proces Poissona

```julia
function rPoisProces(;T=1, lambda=0.2)
     N = rand(Poisson(T*lamb),1)[1]
     u = sort(rand(Uniform(0,T),N))
     return u
end

rPoisProces(T=10,lambda=0.2)
2-element Vector{Float64}:
 2.8348389265407503
 7.317893981845014
```

```julia
using DataFrames

function rRandomField(;T1=1, T2=1, lambda=1)
     N = rand(Poisson(T1*T2*lambda),1)[1]
     u = DataFrame(x=rand(Uniform(0,T1),N),y=rand(Uniform(0,T2),N))
     return u
end

rRandomField(T1=10, T2=10, lambda=0.02)
3×2 DataFrame
 Row │ x           y        
     │ Float64     Float64  
─────┼──────────────────────
   1 │ 1.32083     8.88948
   2 │ 0.00536225  0.525562
   3 │ 5.1095      6.76599
```

#### 4.1.3.2. Wielowymiarowy rozkład normalny

```julia
using LinearAlgebra, Random

function Test(N)
    n = 100
    d = 100
    Sigma = cov(Matrix(rand(MersenneTwister(1313),Normal(), n,d)))
    X = Matrix(rand(Normal(), n,d))
    @time begin
      for i in 1:N 
        Q = cholesky(Sigma).U; X*Q
       end 
    end
    @time begin
      for i in 1:N 
        tmp = svd(Sigma); X*( tmp.U *Diagonal(sqrt.(tmp.S))*transpose(tmp.V) ) 
       end 
    end
    @time begin
      for i in 1:N 
        tmp = eigen(Symmetric(Sigma, :L)); X*( tmp.vectors *Diagonal(sqrt.(tmp.values))*transpose(tmp.vectors) ) 
       end 
    end
end

Test(10000)
  1.445759 seconds (90.00 k allocations: 1.493 GiB, 2.27% gc time)
 28.442879 seconds (190.00 k allocations: 6.813 GiB, 0.49% gc time)
 16.078795 seconds (220.00 k allocations: 4.796 GiB, 0.60% gc time)
```

#### 4.1.3.3. Kopule / funkcje łączące

```julia
using DatagenCopulaBased

x = simulate_copula(1000, Gaussian_cop([1. 0.5; 0.5 1.]));
x[1:3,:]
3×2 Matrix{Float64}:
 0.670963  0.447983
 0.191617  0.117943
 0.935061  0.654589
```

```julia
N = 40
x = simulate_copula(N, Clayton_cop(2, 0.5));
y = [quantile(Exponential(2),x[:,1]) quantile(Exponential(2),x[:,2])];
y[1:6,:]
6×2 Matrix{Float64}:
 0.243109   4.37217
 0.0165588  0.238914
 1.27849    2.65173
 0.433793   0.0424227
 5.4762     3.40724
 0.289804   0.11797
```

```julia
using StatsBase

function cor_test(x,y)
    p = cdf(Normal(0,sqrt(1/(length(y)-3))),atanh(cor(x,y)))
    return 2*minimum([p,1-p])
end

function PV(N,B)
           z = zeros(B,2)
           for i in 1:B 
            x = simulate_copula(N, Clayton_cop(2, 0.5))
            y = [quantile(Exponential(2),x[:,1]) quantile(Exponential(2),x[:,2])]
            @inbounds z[i,1] = cor_test(y[:,1],y[:,2])
            @inbounds z[i,2] = cor_test(tiedrank(y[:,1]),tiedrank(y[:,2]))
         end
         return z
       end

pv = PV(40,1000);
Dict(zip(["Pearson","Spearman"], mean(pv .<0.05, dims=1)))
  "Pearson"  => 0.198
  "Spearman" => 0.461
```

### 4.1.4. Estymacja parametrów rozkładu

```julia
wek = rand(LogNormal(), 100);
f = fit(LogNormal, wek)
LogNormal{Float64}(μ=-0.04680492612411163, σ=1.0008730975679103)
params(f)
(-0.04680492612411163, 1.0008730975679103)
```
```julia
fit(Gamma, wek)
Gamma{Float64}(α=1.0158077118640898, θ=1.656495552687473)
```

### 4.2.1. Brakujące obserwacje

Pakiet [`dprep`](https://cran.r-project.org/src/contrib/Archive/dprep/) przeniesiony do archiwum.

```julia
using RData

h = load("/home/.../dprep/data/hepatitis.rda")
Dict{String, Any} with 1 entry:
  "hepatitis" => 155×20 DataFrame…
hepatitis = DataFrame(h["hepatitis"]);
```
```julia
size(hepatitis)
(155, 20)
```

```julia
t = describe(hepatitis, :nmissing)
20×2 DataFrame
 Row │ variable  nmissing 
     │ Symbol    Int64    
─────┼────────────────────
   1 │ V1               0
   2 │ V2               0
   3 │ V3               0
   4 │ V4               1
   5 │ V5               0
   6 │ V6               1
  ⋮  │    ⋮         ⋮
  16 │ V16             29
  17 │ V17              4
  18 │ V18             16
  19 │ V19             67
  20 │ V20              0
            9 rows omitted

sum(t.nmissing)
167
```

```julia
sum([maximum(ismissing,i) for i in eachrow(hepatitis)])
75
```

```julia
nrow(dropmissing(hepatitis))
80
nrow(dropmissing(hepatitis))/nrow(hepatitis)
0.5161290322580645
```

```julia
using StatsBase

summarystats(hepatitis[:,19])
Summary Stats:
Length:         155
Missing Count:  67
Mean:           61.852273
Minimum:        0.000000
1st Quartile:   46.000000
Median:         61.000000
3rd Quartile:   76.250000
Maximum:        100.000000
```

```julia
summarystats(dropmissing(df, 19).V19)
Summary Stats:
Length:         88
Missing Count:  0
Mean:           61.852273
Minimum:        0.000000
1st Quartile:   46.000000
Median:         61.000000
3rd Quartile:   76.250000
Maximum:        100.000000
```

```julia
wektor = [1,5,missing,2,8,8,3,missing,10]
coalesce.(wektor,2.5)
9-element Vector{Real}:
  1
  5
  2.5
  2
  8
  8
  3
  2.5
 10
```

```julia
coalesce.(wektor,mean(skipmissing(wektor)))
9-element Vector{Real}:
  1
  5
  5.285714285714286
  2
  8
  8
  3
  5.285714285714286
 10
```

```julia
using Impute: Substitute, impute

imputacjemedian = impute(Array(hepatitis), Substitute(; statistic=median); dims=:cols)
imputacjemean = impute(Array(hepatitis), Substitute(; statistic=mean); dims=:cols)
imputacjeknn = Impute.knn(Array(hepatitis); dims=:cols)
```

```julia
df = DataFrame(imputacjemean,:auto)
modelRepl = lm(@formula(x18 ~ x6+x12), df)
StatsModels.TableRegressionModel{LinearModel{GLM.LmResp{Vector{Float64}}, GLM.DensePredChol{Float64, LinearAlgebra.CholeskyPivoted{Float64, Matrix{Float64}}}}, Matrix{Float64}}

x18 ~ 1 + x6 + x12

Coefficients:
────────────────────────────────────────────────────────────────────────
                Coef.  Std. Error      t  Pr(>|t|)  Lower 95%  Upper 95%
────────────────────────────────────────────────────────────────────────
(Intercept)  2.95259     0.187127  15.78    <1e-33  2.58289     3.32229
x6           0.30125     0.105366   2.86    0.0048  0.0930801   0.50942
x12          0.275779    0.107542   2.56    0.0113  0.0633088   0.488249
────────────────────────────────────────────────────────────────────────
```
```julia
modelBezRep = lm(@formula(V18 ~ V6+V12), hepatitis)
StatsModels.TableRegressionModel{LinearModel{GLM.LmResp{Vector{Float64}}, GLM.DensePredChol{Float64, LinearAlgebra.CholeskyPivoted{Float64, Matrix{Float64}}}}, Matrix{Float64}}

V18 ~ 1 + V6 + V12

Coefficients:
────────────────────────────────────────────────────────────────────────
                Coef.  Std. Error      t  Pr(>|t|)  Lower 95%  Upper 95%
────────────────────────────────────────────────────────────────────────
(Intercept)  2.88703     0.204694  14.10    <1e-27  2.48212     3.29193
V6           0.306896    0.116699   2.63    0.0096  0.0760537   0.537738
V12          0.313853    0.117262   2.68    0.0084  0.0818974   0.545809
────────────────────────────────────────────────────────────────────────
```

```julia
using StatsBase

map(r2,[modelRepl,modelBezRepl])
2-element Vector{Float64}:
 0.12677718779619374
 0.14686463436054453

[map(i,[modelRepl,modelBezRepl]) for i in [aic,loglikelihood]]
2-element Vector{Vector{Float64}}:
 [287.9258718973875, 250.27119680422607]
 [-139.96293594869374, -121.13559840211303]
```

### 4.2.2. Normalizacja, skalowanie i transformacje nieliniowe

```julia
using RCall

@rlibrary Przewodnik
daneSoc = rcopy(R"Przewodnik::daneSoc");
```

```julia
cov(Matrix(daneSoc[:,[1,6,7]]))
3×3 Matrix{Float64}:
 191.742     -6.01666  -8.94125
  -6.01666  246.904    82.8092
  -8.94125   82.8092   60.3246
```

```julia
using Statistics, DataFrames

d = [col/std(col) for col = eachcol(daneSoc[:,[1,6,7]])]
w = DataFrame(d,:auto)
cov(Matrix(w))
3×3 Matrix{Float64}:
  1.0        -0.0276524  -0.0831366
 -0.0276524   1.0         0.678527
 -0.0831366   0.678527    1.0
```

```julia
using BoxCoxTrans

BoxCoxTrans.lambda(daneO.VEGF).value
-0.01170644585814663

BoxCoxTrans.lambda(daneO.Wiek).value
1.9999999557536448
```

```julia
using CategoricalArrays

zmOryginalna = [0.1,0.8,0.5,0.2,0.9,0.7];
zmZmieniona = cut(zmOryginalna,[0,0.33,0.66,1],labels=["niski","sredni","wysoki"])
6-element CategoricalArray{String,1,UInt32}:
 "niski"
 "wysoki"
 "sredni"
 "niski"
 "wysoki"
 "wysoki"
```

```julia
tDaneO = transform(daneO, :VEGF => ByRow(log) => :logVEGF,
                          :Nowotwor => CategoricalArray => :fNowotwor,
                          :Wiek => (x -> cut(x,[29,36,43,50,58])) => :cutWiek);
tDaneO[:,7:end]
97×6 DataFrame
 Row │ Niepowodzenia  Okres_bez_wznowy  VEGF   logVEGF   fNowotwor  cutWiek      
     │ Categorical…   Int64?            Int64  Float64   Cat…?      Categorical… 
─────┼───────────────────────────────────────────────────────────────────────────
   1 │ brak                         22    914   6.81783  2          [29, 36)
   2 │ brak                         53   1118   7.0193   2          [29, 36)
   3 │ brak                         38    630   6.44572  2          [29, 36)
   4 │ brak                         26   1793   7.49165  3          [29, 36)
   5 │ brak                         19    963   6.87005  missing    [29, 36)
   6 │ wznowa                       36   2776   7.92877  3          [29, 36)
  ⋮  │       ⋮               ⋮            ⋮       ⋮          ⋮           ⋮
  93 │ brak                         30    624   6.43615  2          [50, 58)
  94 │ brak                         36   1354   7.21082  1          [50, 58)
  95 │ brak                         29    373   5.92158  2          [50, 58)
  96 │ brak                    missing   1255   7.13489  missing    [50, 58)
  97 │ brak                         46    380   5.94017  2          [50, 58)
                                                                  86 rows omitted
```


### 4.3.2. Analiza jednoczynnikowa

```julia
using RCall

@rlibrary Przewodnik
mieszkania = rcopy(R"Przewodnik::mieszkania");
describe(mieszkania)
5×7 DataFrame
 Row │ variable      mean      min        median    max          nmissing  eltype       ⋯
     │ Symbol        Union…    Any        Union…    Any          Int64     DataType     ⋯
─────┼───────────────────────────────────────────────────────────────────────────────────
   1 │ cena          175934.0  83279.9    174935.0  2.95762e5           0  Float64      ⋯
   2 │ pokoi         2.55      1          3.0       4                   0  Int64
   3 │ powierzchnia  46.197    17.0       43.7      87.7                0  Float64
   4 │ dzielnica               Biskupin             Srodmiescie         0  CategoricalV
   5 │ typ_budynku             kamienica            wiezowiec           0  CategoricalV ⋯
                                                                         1 column omitted
```

```julia
using GLM, Distributions

function aov(ols)
 R2 = r2(ols);
 k = dof(ols)-2; n = nrow(mieszkania);
 F = (R2/k)/((1-R2)/(n-k-1))
 df1 = k; df2 = n-k-1;
 p_value = 1 - cdf(FDist(df1,df2),F)
 return Dict(:F=>F, :df1=>df1, :df2=>df2, :p=>p_value)
end

ols = lm(@formula(cena ~ dzielnica), mieszkania);
res = aov(ols)
Dict{Symbol, Real} with 4 entries:
  :F   => 5.04563
  :p   => 0.00729437
  :df2 => 197
  :df1 => 2

ols0 = lm(@formula(cena ~ 1), mieszkania);
ftest(ols0.model,ols.model)
F-test: 2 models fitted on 200 observations
────────────────────────────────────────────────────────────────────────────────────
     DOF  ΔDOF                SSR               ΔSSR      R²     ΔR²      F*   p(>F)
────────────────────────────────────────────────────────────────────────────────────
[1]    2        369298265315.1666                     0.0000                        
[2]    4     2  351302882089.9193  -17995383225.2473  0.0487  0.0487  5.0456  0.0073
────────────────────────────────────────────────────────────────────────────────────
```
```julia
MSE = deviance(ols)/dof_residual(ols)
1.7832633608625343e9
```
```julia
ols = lm(@formula(cena ~ typ_budynku), mieszkania);
res = aov(ols)
Dict{Symbol, Real} with 4 entries:
  :F   => 6.47246
  :p   => 0.00189471
  :df2 => 197
  :df1 => 2

ftest(ols0.model,ols.model)
F-test: 2 models fitted on 200 observations
────────────────────────────────────────────────────────────────────────────────────
     DOF  ΔDOF                SSR               ΔSSR      R²     ΔR²      F*   p(>F)
────────────────────────────────────────────────────────────────────────────────────
[1]    2        369298265315.1666                     0.0000                        
[2]    4     2  346527817008.6356  -22770448306.5309  0.0617  0.0617  6.4725  0.0019
────────────────────────────────────────────────────────────────────────────────────
```

```julia
using Combinatorics

u = unique(mieszkania.dzielnica);
res = collect(combinations(u,2));
mu = combine(groupby(mieszkania, [:dzielnica]), [:cena => mean]);
DF = [mu[mu.dzielnica .== res[i][1], :cena_mean] - mu[mu.dzielnica .== res[i][2], :cena_mean]
      for i in 1:3 ];
mu.mean_diff = collect(Iterators.flatten(DF));
mu.a = [res[i][1] for i in 1:3];
mu.b = [res[i][2] for i in 1:3];
select!(mu, [:a,:b] => ByRow((x,y) -> string(x,"-",y)) => :dzielnica,
        Not([:dzielnica,:cena_mean,:a,:b]))
3×2 DataFrame
 Row │ dzielnica             mean_diff 
     │ String                Float64   
─────┼─────────────────────────────────
   1 │ Krzyki-Biskupin       -21321.0
   2 │ Krzyki-Srodmiescie     -2970.48
   3 │ Biskupin-Srodmiescie   18350.5
```

```julia
using StatsPlots

pt = combine(groupby(mieszkania, [:dzielnica]), :cena => mean=> :s, :cena => (x->var(x)^0.5) =>:v);
groupedbar(pt.dzielnica, pt.s, yerr = pt.v, group = repeat(["srednia_cena"],outer=3),
           fillalpha= 0.4, legend= false)
savefig("/home/.../assets/p432.png")
```
![](/assets/p432.png)

#### 4.3.2.1. Kontrasty w analizie wariancji 

```julia
using StatsModels

StatsModels.ContrastsMatrix(HelmertCoding(), ["a", "b", "c", "d","e"]).matrix'
4×5 adjoint(::Matrix{Float64}) with eltype Float64:
 -1.0   1.0   0.0   0.0  0.0
 -1.0  -1.0   2.0   0.0  0.0
 -1.0  -1.0  -1.0   3.0  0.0
 -1.0  -1.0  -1.0  -1.0  4.0
```
```julia
kontr = Dict(:dzielnica => HelmertCoding());
model = lm(@formula(cena ~ dzielnica), contrasts = kontr, mieszkania)
StatsModels.TableRegressionModel{LinearModel{GLM.LmResp{Vector{Float64}}, GLM.DensePredChol{Float64, LinearAlgebra.CholeskyPivoted{Float64, Matrix{Float64}}}}, Matrix{Float64}}

cena ~ 1 + dzielnica

Coefficients:
─────────────────────────────────────────────────────────────────────────────────────────────────
                                Coef.  Std. Error      t  Pr(>|t|)       Lower 95%      Upper 95%
─────────────────────────────────────────────────────────────────────────────────────────────────
(Intercept)                  1.7627e5     3015.73  58.45    <1e-99       1.70323e5      1.82217e5
dzielnica: Krzyki       -10660.5          3535.81  -3.02    0.0029  -17633.4        -3687.62
dzielnica: Srodmiescie   -2563.34         2219.76  -1.15    0.2496   -6940.88        1814.19
─────────────────────────────────────────────────────────────────────────────────────────────────
```

```julia
contrast_matrix = [2 0; -1 1; -1 -1];
kontr = Dict(:dzielnica => ContrastsCoding(contrast_matrix));
model = lm(@formula(cena ~ dzielnica), contrasts = kontr, mieszkania)
StatsModels.TableRegressionModel{LinearModel{GLM.LmResp{Vector{Float64}}, GLM.DensePredChol{Float64, LinearAlgebra.CholeskyPivoted{Float64, Matrix{Float64}}}}, Matrix{Float64}}

cena ~ 1 + dzielnica

Coefficients:
───────────────────────────────────────────────────────────────────────────────────────────────
                               Coef.  Std. Error      t  Pr(>|t|)      Lower 95%      Upper 95%
───────────────────────────────────────────────────────────────────────────────────────────────
(Intercept)                 1.7627e5     3015.73  58.45    <1e-99      1.70323e5      1.82217e5
dzielnica: Krzyki        6611.93         2135.39   3.10    0.0022   2400.77       10823.1
dzielnica: Srodmiescie  -1485.24         3688.39  -0.40    0.6876  -8759.04        5788.56
───────────────────────────────────────────────────────────────────────────────────────────────
```

#### 4.3.3.1. Model addytywny w analizie wariancji

```julia
ab = lm(@formula(cena ~ typ_budynku), mieszkania);
a2 = lm(@formula(cena ~ dzielnica+typ_budynku), mieszkania);
ftest(ab.model,a2.model)
F-test: 2 models fitted on 200 observations
────────────────────────────────────────────────────────────────────────────────────
     DOF  ΔDOF                SSR               ΔSSR      R²     ΔR²      F*   p(>F)
────────────────────────────────────────────────────────────────────────────────────
[1]    4        346527817008.6356                     0.0617                        
[2]    6     2  328583954796.3170  -17943862212.3187  0.1102  0.0486  5.3244  0.0056
────────────────────────────────────────────────────────────────────────────────────

ad = lm(@formula(cena ~ dzielnica), mieszkania);
ftest(ad.model,a2.model)
F-test: 2 models fitted on 200 observations
────────────────────────────────────────────────────────────────────────────────────
     DOF  ΔDOF                SSR               ΔSSR      R²     ΔR²      F*   p(>F)
────────────────────────────────────────────────────────────────────────────────────
[1]    4        351302882089.9193                     0.0487                        
[2]    6     2  328583954796.3170  -22718927293.6023  0.1102  0.0615  6.7413  0.0015
────────────────────────────────────────────────────────────────────────────────────
```

```julia
mucd = combine(groupby(mieszkania, [:dzielnica]), [:cena => mean]);
mucb = combine(groupby(mieszkania, [:typ_budynku]), [:cena => mean]);
```
```julia
using Plots

d = unique(mucd.dzielnica);
b = unique(mucb.typ_budynku);
plot([1,1,1],mucd.cena_mean[1:3],shape=:hline,markersize=6,legend=false,xlim=(0,3),size=(600,300))
annotate!([1],mucd.cena_mean[1],text(d[1], :gray, :bottom, 8))
annotate!([1],mucd.cena_mean[2],text(d[2], :gray, :bottom, 8))
annotate!([1],mucd.cena_mean[3],text(d[3], :gray, :bottom, 8))
plot!([2,2,2],mucb.cena_mean[1:3],shape=:hline,markersize=6)
annotate!([2],mucb.cena_mean[1],text(b[1], :gray, :bottom, 8))
annotate!([2],mucb.cena_mean[2],text(b[2], :gray, :bottom, 8))
annotate!([2],mucb.cena_mean[3],text(b[3], :gray, :bottom, 8))
plot!(x-> mean(mieszkania.cena),ls=:dash);
xticks!([1:1:2;], [names(mucd)[1],names(mucb)[1]])
savefig("/home/.../assets/p4331.png")
```
![](/assets/p4331.png)

#### 4.3.3.2. Model z interakcją w analizie wariancji

```julia
muc = combine(groupby(mieszkania, [:typ_budynku,:dzielnica]), [:cena => mean])
9×3 DataFrame
 Row │ typ_budynku  dzielnica    cena_mean 
     │ Cat…         Cat…         Float64   
─────┼─────────────────────────────────────
   1 │ kamienica    Biskupin     1.90804e5
   2 │ kamienica    Krzyki       1.7053e5
   3 │ kamienica    Srodmiescie  1.66809e5
   4 │ niski blok   Biskupin     2.06699e5
   5 │ niski blok   Krzyki       1.82189e5
   6 │ niski blok   Srodmiescie  1.82155e5
   7 │ wiezowiec    Biskupin     1.74651e5
   8 │ wiezowiec    Krzyki       1.56823e5
   9 │ wiezowiec    Srodmiescie  1.62065e5
```
```julia
using Plots

plot(1:3,muc.cena_mean[1:3],label=b[1],ls=:dot,lw=2,xlim=(0.95,4),size=(600,300));
plot!(1:3,muc.cena_mean[4:6],label=b[2],ls=:dash,lw=2);
plot!(1:3,muc.cena_mean[7:9],label=b[3],lw=2,xlabel="dzielnica",ylabel="średnia ceny");
xticks!([1:1:3;], unique(muc.dzielnica))
savefig("/home/.../assets/p4332.png")
```
![](/assets/p4332.png)

```julia
mu1 = combine(groupby(daneSoc, [:st_cywilny,:plec]), [:wiek => mean])
sc = unique(mu1.st_cywilny);
pl = unique(mu1.plec);

plot(1:2,mu1[1:2,3],label=sc[1],ls=:dash,lw=2,xlim=(0.95,2.5),size=(600,300))
plot!(1:2,mu1[3:4,3],label=sc[2],lw=2,xlabel="płeć",ylabel="średnia wieku")
xticks!([1:1:2;], pl)
savefig("/home/.../assets/p4332a.png")
```
![](/assets/p4332a.png)
```julia
mu2 = combine(groupby(daneO, [:Wezly_chlonne,:Niepowodzenia]), [:Rozmiar_guza => mean])
wc = unique(mu2.Wezly_chlonne);
np = unique(mu2.Niepowodzenia);

plot(1:2,mu2[1:2,3],label=wc[1],ls=:dash,lw=2,xlim=(0.95,2.5),size=(600,300))
plot!(1:2,mu2[3:4,3],label=wc[2],lw=2,xlabel="niepowodzenia",ylabel="średnia rozmiaru guza")
xticks!([1:1:2;], np)
savefig("/home/.../assets/p4332b.png")
```
![](/assets/p4332b.png)

```julia
am = lm(@formula(cena ~ dzielnica*typ_budynku), mieszkania);
ad = lm(@formula(cena ~ dzielnica+typ_budynku), mieszkania);
ftest(am.model,ad.model)
F-test: 2 models fitted on 200 observations
──────────────────────────────────────────────────────────────────────────────────
     DOF  ΔDOF                SSR            ΔSSR      R²      ΔR²      F*   p(>F)
──────────────────────────────────────────────────────────────────────────────────
[1]   10        327588675136.4390                  0.1129                         
[2]    6    -4  328583954796.3170  995279659.8779  0.1102  -0.0027  0.1451  0.9650
──────────────────────────────────────────────────────────────────────────────────
```

```julia
b = lm(@formula(cena ~ typ_budynku), mieszkania);
ftest(ad.model,b.model)
F-test: 2 models fitted on 200 observations
────────────────────────────────────────────────────────────────────────────────────
     DOF  ΔDOF                SSR              ΔSSR      R²      ΔR²      F*   p(>F)
────────────────────────────────────────────────────────────────────────────────────
[1]    6        328583954796.3170                    0.1102                         
[2]    4    -2  346527817008.6356  17943862212.3187  0.0617  -0.0486  5.3244  0.0056
────────────────────────────────────────────────────────────────────────────────────
```
```julia
d = lm(@formula(cena ~ dzielnica), mieszkania);
ftest(ad.model,d.model)
F-test: 2 models fitted on 200 observations
────────────────────────────────────────────────────────────────────────────────────
     DOF  ΔDOF                SSR              ΔSSR      R²      ΔR²      F*   p(>F)
────────────────────────────────────────────────────────────────────────────────────
[1]    6        328583954796.3170                    0.1102                         
[2]    4    -2  351302882089.9193  22718927293.6023  0.0487  -0.0615  6.7413  0.0015
────────────────────────────────────────────────────────────────────────────────────
```
```julia
a0 = lm(@formula(cena ~ 1), mieszkania);
ftest(a0.model,d.model)
F-test: 2 models fitted on 200 observations
────────────────────────────────────────────────────────────────────────────────────
     DOF  ΔDOF                SSR               ΔSSR      R²     ΔR²      F*   p(>F)
────────────────────────────────────────────────────────────────────────────────────
[1]    2        369298265315.1666                     0.0000                        
[2]    4     2  351302882089.9193  -17995383225.2473  0.0487  0.0487  5.0456  0.0073
────────────────────────────────────────────────────────────────────────────────────
```

#### 4.3.3.3. Wielowymiarowa analiza wariancji

```julia
R"m = with(manova(cbind(cena,powierzchnia)~dzielnica+typ.budynku),data=Przewodnik::mieszkania);
      s = summary(m,test='Hotelling-Lawley')"
RObject{VecSxp}
             Df Hotelling-Lawley approx F num Df den Df    Pr(>F)    
dzielnica     2          0.86337   41.658      4    386 < 2.2e-16 ***
typ.budynku   2          0.32249   15.560      4    386 8.198e-12 ***
Residuals   195                                                      
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
```
```julia
manova = @rget s
OrderedCollections.OrderedDict{Symbol, Any} with 4 entries:
  :row_names   => ["dzielnica", "typ.budynku", "Residuals"]
  :SS          => OrderedCollections.OrderedDict{Symbol, Any}(:dzielnica=>[1.79954e10 8.62897e5; 8.62897e5 289.691…
  :Eigenvalues => [0.859707 0.00366137; 0.305188 0.0173024]
  :stats       => Union{Missing, Float64}[2.0 0.863369 … 386.0 4.94026e-29; 2.0 0.32249 … 386.0 8.19828e-12; 195.0…

manova[:stats][16:17]
2-element Vector{Union{Missing, Float64}}:
 4.9402586222266735e-29
 8.198284856354054e-12
```

#### 4.3.4.1. Wprowadzenie do regresji liniowej

```julia
modelPP = lm(@formula(cena ~ powierzchnia+pokoi), mieszkania);
coef(modelPP)
3-element Vector{Float64}:
 82407.0882807293
  2070.89656420296
  -840.1007675346755
```
```julia
modelPP
StatsModels.TableRegressionModel{LinearModel{GLM.LmResp{Vector{Float64}}, GLM.DensePredChol{Float64, LinearAlgebra.CholeskyPivoted{Float64, Matrix{Float64}}}}, Matrix{Float64}}

cena ~ 1 + powierzchnia + pokoi

Coefficients:
──────────────────────────────────────────────────────────────────────────
                  Coef.  Std. Error      t  Pr(>|t|)  Lower 95%  Upper 95%
──────────────────────────────────────────────────────────────────────────
(Intercept)   82407.1       2569.91  32.07    <1e-79   77339.0    87475.1
powierzchnia   2070.9        149.17  13.88    <1e-30    1776.72    2365.07
pokoi          -840.101     2765.1   -0.30    0.7616   -6293.1     4612.9
──────────────────────────────────────────────────────────────────────────

modelPP0 = lm(@formula(cena ~ 1), mieszkania);
ftest(modelPP.model,modelPP0.model)
F-test: 2 models fitted on 200 observations
───────────────────────────────────────────────────────────────────────────────────────
     DOF  ΔDOF                SSR               ΔSSR      R²      ΔR²        F*   p(>F)
───────────────────────────────────────────────────────────────────────────────────────
[1]    4         39248025340.0797                     0.8937                           
[2]    2    -2  369298265315.1666  330050239975.0869  0.0000  -0.8937  828.3206  <1e-95
───────────────────────────────────────────────────────────────────────────────────────
```
```julia
r2(modelPP)
0.8937226923972019
deviance(modelPP)
3.924802534007968e10
dof_residual(modelPP)
197.0
```

```julia
stderror(modelPP)
3-element Vector{Float64}:
 2569.9056785985204
  149.1704767576413
 2765.101822884913
```

#### 4.3.4.2. Wykresy diagnostyczne dla modelu

```julia
model = lm(@formula(cena ~ powierzchnia+pokoi+dzielnica), mieszkania)
StatsModels.TableRegressionModel{LinearModel{GLM.LmResp{Vector{Float64}}, GLM.DensePredChol{Float64, LinearAlgebra.CholeskyPivoted{Float64, Matrix{Float64}}}}, Matrix{Float64}}

cena ~ 1 + powierzchnia + pokoi + dzielnica

Coefficients:
───────────────────────────────────────────────────────────────────────────────────────
                              Coef.  Std. Error       t  Pr(>|t|)  Lower 95%  Upper 95%
───────────────────────────────────────────────────────────────────────────────────────
(Intercept)              94222.0       2320.36    40.61    <1e-96   89645.8    98798.2
powierzchnia              2022.99       116.312   17.39    <1e-40    1793.6     2252.38
pokoi                       34.3596    2157.52     0.02    0.9873   -4220.72    4289.44
dzielnica: Krzyki       -20934.9       1842.79   -11.36    <1e-22  -24569.2   -17300.5
dzielnica: Srodmiescie  -12722.6       2008.03    -6.34    <1e-08  -16682.8    -8762.37
───────────────────────────────────────────────────────────────────────────────────────
```
```julia
using LinearAlgebra

model =  lm(@formula(cena ~ powierzchnia), mieszkania);
fitted = predict(model);
ei = fitted-mieszkania.cena;

x = DataFrame(x1=mieszkania[:,:powierzchnia],x2=ones(200));
X = Matrix(x);
hi = diag(X*inv(transpose(X)*X)*transpose(X));
sei = ei ./ (std(ei).*sqrt.(1 .- hi));
cdi = (ei.^2 .*hi)./([2].*[198315231].*(([1].-hi).^2))
```
```julia
p1 = scatter(fitted,ei,legend=false)
x = sort(ei)
i = 1:length(x)
fi = (i .- 0.5)/length(x)
x_norm = quantile(Normal(),fi)
b = quantile(Normal(),[0.75])-quantile(Normal(),[0.25])
a = iqr(x)/b
p2 = scatter(x_norm,x,xlabel="kwantyle teoretyczne",ylabel="kwantyle empiryczne",legend=false)
p2 = plot!(x -> a[1]*x+b[1],lw=2)
plot(p1, p2, layout = grid(1, 2), size=(800,300));
savefig("/home/.../assets/p4342a.png")
```
![](/assets/p4342a.png)

```julia
s1 = scatter(fitted,sqrt.(abs.(sei)),legend=false)
s2 = bar(cdi,bar_width=0.1,legend=false)
plot(s1, s2, layout = grid(1, 2), size=(800,300));
savefig("/home/.../assets/p4342b.png")
```
![](/assets/p4342b.png)

```julia
DataFrame(imax = findall(>(maximum(cdi)/2), cdi), cdi_max=cdi[imax])
4×2 DataFrame
 Row │ imax   cdi_max   
     │ Int64  Float64   
─────┼──────────────────
   1 │    49  0.0895013
   2 │   125  0.0548537
   3 │   156  0.04615
   4 │   198  0.0681202
```

```julia
g1 = scatter(hi,sei,legend=false)
g2 = scatter(hi,cdi,legend=false)
plot(g1, g2, layout = grid(1, 2), size=(800,300));
savefig("/home/.../assets/p4342c.png")
```
![](/assets/p4342c.png)

```julia
using CategoricalArrays

modelDP = lm(@formula(cena ~ powierzchnia+dzielnica), mieszkania);
ndane = DataFrame(powierzchnia=[48,68],dzielnica=CategoricalArray(["Biskupin","Krzyki"]));
predict(modelDP,ndane)
2-element Vector{Union{Missing, Float64}}:
 191415.93384291366
 210976.88441278608
```

#### 4.3.4.3. Współliniowość zmiennych objaśniających

```julia
m1 = lm(@formula(cena ~ pokoi+powierzchnia), mieszkania);
m2 = lm(@formula(powierzchnia ~ pokoi+cena), mieszkania);
m3 = lm(@formula(pokoi ~ powierzchnia+cena), mieszkania);
m = map(x -> 1/(1-r2(x)), [m1,m2,m3]);
Dict("cena"=>m[1], "powierzchnia"=>m[2], "pokoi"=>m[3])
Dict{String, Float64} with 3 entries:
  "cena"         => 9.40935
  "pokoi"        => 8.96522
  "powierzchnia" => 17.7278
```

### 4.3.5. Wprowadzenie do regresji logistycznej

```julia
z = String.(daneO[:,:Niepowodzenia]);
z1 = replace.(z, "brak"=> "0");
z2 = replace.(z1, "wznowa"=> "1");
daneO.N = map(x -> parse(Int64, x), z2);
modelNV = glm(@formula(N ~ Nowotwor+log(VEGF)), daneO, Binomial(), LogitLink())
StatsModels.TableRegressionModel{GeneralizedLinearModel{GLM.GlmResp{Vector{Float64}, Binomial{Float64}, LogitLink}, GLM.DensePredChol{Float64, Cholesky{Float64, Matrix{Float64}}}}, Matrix{Float64}}

N ~ 1 + Nowotwor + :(log(VEGF))

Coefficients:
─────────────────────────────────────────────────────────────────────────
                 Coef.  Std. Error      z  Pr(>|z|)  Lower 95%  Upper 95%
─────────────────────────────────────────────────────────────────────────
(Intercept)  -17.3914     4.37567   -3.97    <1e-04  -25.9676    -8.81527
Nowotwor       2.25859    0.765705   2.95    0.0032    0.75784    3.75935
log(VEGF)      1.32928    0.424998   3.13    0.0018    0.4963     2.16226
─────────────────────────────────────────────────────────────────────────
```

```julia
modelN = glm(@formula(N ~ Nowotwor), daneO, Binomial(), LogitLink());
ndaneO = DataFrame(Nowotwor=[1,2,3]);
predict(modelN,ndaneO)
3-element Vector{Union{Missing, Float64}}:
 0.011259045389590293
 0.07249761087083298
 0.349185127566951
```

#### 4.3.5.1. Uogólniony model liniowy

```julia
modelN0 = glm(@formula(N ~ 1), daneO, Binomial(), LogitLink());
loglikelihood(modelN)
-31.035190851511924
loglikelihood(modelN0)
-38.21401216502366
```
```julia
1-deviance(modelN)/deviance(modelN0)
0.18785835108103022
1-loglikelihood(modelN)/loglikelihood(modelN0)
0.18785835108102922
```

#### 4.3.5.2. Regresja nieliniowa

```julia
using LsqFit, Random

model(x,a) = x.^a[1].-a[2];
x = rand(MersenneTwister(2305),Uniform(),100)*3;
y = x .^(2.5) .-5 + rand(MersenneTwister(2305),Normal(0,3),100);
fit = curve_fit(model, x, y, [2.0, 2.0]);
DataFrame(beta=fit.param, error=stderror(fit))
2×2 DataFrame
 Row │ beta     error     
     │ Float64  Float64   
─────┼────────────────────
   1 │ 2.43104  0.0681388
   2 │ 4.79015  0.418114

confidence_interval(fit, 0.95)
2-element Vector{Tuple{Float64, Float64}}:
 (2.426755729470465, 2.4353231666026187)
 (4.76386740579981, 4.816438997067286)
std(fit.resid)
3.2067363840032215
```

```julia
mojModel(x,a) = sin.(a[1].+a[2].*x).+a[3];
x1 = rand(MersenneTwister(2305),Uniform(),100)*6;
y1 = mojModel(x1,[pi/2,2.5,1])+rand(MersenneTwister(2305),Normal(),100);
fit1 = curve_fit(mojModel, x1, y1, [2.0, 2.0, 1.0]);
DataFrame(beta=fit1.param, error=stderror(fit1))
3×2 DataFrame
 Row │ beta      error     
     │ Float64   Float64   
─────┼─────────────────────
   1 │ 1.58315   0.326482
   2 │ 1.9717    0.0863691
   3 │ 0.986002  0.121859
```

```julia
xn = minimum(x):0.01:maximum(x);
yn = model(xn,fit.param);
f = scatter(x,y,label="points",legend=:topleft)
f = plot!(xn,yn,label="fit",lw=3)
f = plot!(x -> x^2.5-5,label="function",ls=:dash)

xn = minimum(x1):0.01:maximum(x1);
yn = mojModel(xn,fit1.param);
f1 = scatter(x1,y1,label="points",legend=:topleft)
f1 = plot!(xn,yn,label="fit",lw=3)
f1 = plot!(x -> sin(pi/2+2.5*x)+1,label="function",ls=:dash)
plot(f, f1, layout = grid(1, 2), size=(800,300))
savefig("/home/.../assets/p4352.png")
```
![](/assets/p4352.png)

#### 4.4.1.1. Testowanie zgodności z rozkładem normalnym

```julia
using HypothesisTests, Random

x = rand(MersenneTwister(2305),Normal(10,1.4),100);
ad = OneSampleADTest(x,Normal(mean(x), std(x)))
One sample Anderson-Darling test
--------------------------------
Population details:
    parameter of interest:   not implemented yet
    value under h_0:         NaN
    point estimate:          NaN

Test summary:
    outcome with 95% confidence: fail to reject h_0
    one-sided p-value:           0.9318

Details:
    number of observations:   100
    sample mean:              9.96803843637786
    sample SD:                1.5048102075585787
    A² statistic:             0.308108433178711
```
```julia
pvalue(ad)
0.9317854744118788
```

#### 4.4.1.2. Testy zgodności z rozkładem jednostajnym

```julia
using CategoricalArrays, StatsBase, Distributions

przedzialy = 0:0.2:1
x = rand(MersenneTwister(2305),Uniform(),100);
table = countmap(cut(x,przedzialy))
Dict{CategoricalValue{String, UInt32}, Int64} with 5 entries:
  "[0.0, 0.2)" => 24
  "[0.8, 1.0)" => 22
  "[0.2, 0.4)" => 15
  "[0.6, 0.8)" => 24
  "[0.4, 0.6)" => 15
```

```julia
ChisqTest(collect(values(table)))
Pearson's Chi-square Test
-------------------------
Population details:
    parameter of interest:   Multinomial Probabilities
    value under h_0:         [0.2, 0.2, 0.2, 0.2, 0.2]
    point estimate:          [0.24, 0.22, 0.15, 0.24, 0.15]
    95% confidence interval: [(0.15, 0.346), (0.13, 0.326), (0.06, 0.256), (0.15, 0.346), (0.06, 0.256)]

Test summary:
    outcome with 95% confidence: fail to reject h_0
    one-sided p-value:           0.3669

Details:
    Sample size:        100
    statistic:          4.300000000000001
    degrees of freedom: 4
    residuals:          [0.894427, 0.447214, -1.11803, 0.894427, -1.11803]
    std. residuals:     [1.0, 0.5, -1.25, 1.0, -1.25]
```
```julia
przedzialy = 0:0.2:1
x = rand(MersenneTwister(2305),Beta(3,2),100);
table = countmap(cut(x,przedzialy))
Dict{CategoricalValue{String, UInt32}, Int64} with 5 entries:
  "[0.0, 0.2)" => 7
  "[0.8, 1.0)" => 18
  "[0.2, 0.4)" => 12
  "[0.6, 0.8)" => 31
  "[0.4, 0.6)" => 32
```
```julia
ChisqTest(collect(values(table)))
Pearson's Chi-square Test
-------------------------
Population details:
    parameter of interest:   Multinomial Probabilities
    value under h_0:         [0.2, 0.2, 0.2, 0.2, 0.2]
    point estimate:          [0.07, 0.18, 0.12, 0.31, 0.32]
    95% confidence interval: [(0.0, 0.1798), (0.09, 0.2898), (0.03, 0.2298), (0.22, 0.4198), (0.23, 0.4298)]

Test summary:
    outcome with 95% confidence: reject h_0
    one-sided p-value:           <1e-04

Details:
    Sample size:        100
    statistic:          25.1
    degrees of freedom: 4
    residuals:          [-2.90689, -0.447214, -1.78885, 2.45967, 2.68328]
    std. residuals:     [-3.25, -0.5, -2.0, 2.75, 3.0]
```

```julia
x = rand(MersenneTwister(2305),Uniform(),100);
ExactOneSampleKSTest(x,Uniform())
Exact one sample Kolmogorov-Smirnov test
----------------------------------------
Population details:
    parameter of interest:   Supremum of CDF differences
    value under h_0:         0.0
    point estimate:          0.083345

Test summary:
    outcome with 95% confidence: fail to reject h_0
    two-sided p-value:           0.4659

Details:
    number of observations:   100
```
```julia
ExactOneSampleKSTest(x,Normal(1,1))
Exact one sample Kolmogorov-Smirnov test
----------------------------------------
Population details:
    parameter of interest:   Supremum of CDF differences
    value under h_0:         0.0
    point estimate:          0.504398

Test summary:
    outcome with 95% confidence: reject h_0
    two-sided p-value:           <1e-23

Details:
    number of observations:   100
```

#### 4.4.1.3. Testy zgodności dla dwóch prób

```julia
x = rand(MersenneTwister(2305),Uniform(),100);
y = rand(MersenneTwister(2305),Normal(),100);
ApproximateTwoSampleKSTest(x,y)
Approximate two sample Kolmogorov-Smirnov test
----------------------------------------------
Population details:
    parameter of interest:   Supremum of CDF differences
    value under h_0:         0.0
    point estimate:          0.48

Test summary:
    outcome with 95% confidence: reject h_0
    two-sided p-value:           <1e-09

Details:
    number of observations:   [100,100]
    KS-statistic:              3.39411254969543
```

```julia
x = rand(MersenneTwister(2305),Exponential(),100);
y = rand(MersenneTwister(2305),Normal(),100);
qx = quantile(x,0.05:0.01:1);
qy = quantile(y,0.05:0.01:1);
scatter(qx,qy, size=(500,300))
savefig("/home/.../assets/p4413.png")
```
![](/assets/p4413.png)

### 4.4.2. Testowanie hipotezy o równości parametrów położenia

```julia
x = rand(MersenneTwister(4101),Normal(),50);
y = x .+ 0.3 .+ rand(MersenneTwister(2305),Normal(0,0.01),50);
OneSampleTTest(y,0)
One sample t-test
-----------------
Population details:
    parameter of interest:   Mean
    value under h_0:         0
    point estimate:          0.0919912
    95% confidence interval: (-0.1477, 0.3316)

Test summary:
    outcome with 95% confidence: fail to reject h_0
    two-sided p-value:           0.4442

Details:
    number of observations:   50
    t-statistic:              0.7714084088661292
    degrees of freedom:       49
    empirical standard error: 0.11925098833818996
```
```julia
pvalue(UnequalVarianceTTest(x,y))
0.07798218377393315
```
```julia
OneSampleTTest(x-y)
One sample t-test
-----------------
Population details:
    parameter of interest:   Mean
    value under h_0:         0
    point estimate:          -0.299833
    95% confidence interval: (-0.3028, -0.2969)

Test summary:
    outcome with 95% confidence: reject h_0
    two-sided p-value:           <1e-72

Details:
    number of observations:   50
    t-statistic:              -205.87586027158488
    degrees of freedom:       49
    empirical standard error: 0.0014563755298800714
```
```julia
SignedRankTest(y)
Exact Wilcoxon signed rank test
-------------------------------
Population details:
    parameter of interest:   Location parameter (pseudomedian)
    value under h_0:         0
    point estimate:          0.211529
    95% confidence interval: (-0.1523, 0.3561)

Test summary:
    outcome with 95% confidence: fail to reject h_0
    two-sided p-value:           0.5148

Details:
    number of observations:      50
    Wilcoxon rank-sum statistic: 706.0
    rank sums:                   [706.0, 569.0]
    adjustment for ties:         0.0
```
```julia
MannWhitneyUTest(x,y)
Approximate Mann-Whitney U test
-------------------------------
Population details:
    parameter of interest:   Location parameter (pseudomedian)
    value under h_0:         0
    point estimate:          -0.305415

Test summary:
    outcome with 95% confidence: fail to reject h_0
    two-sided p-value:           0.0693

Details:
    number of observations in each group: [50, 50]
    Mann-Whitney-U statistic:             986.0
    rank sums:                            [2261.0, 2789.0]
    adjustment for ties:                  0.0
    normal approximation (μ, σ):          (-264.0, 145.057)
```

### 4.4.3. Testowanie hipotezy o równości parametrów skali

```julia
x = rand(MersenneTwister(4101),Normal(2,1),20);
y = rand(MersenneTwister(4101),Normal(2,2),20);
z = rand(MersenneTwister(4101),Normal(2,3),20);
VarianceFTest(x,y)
Variance F-test
---------------
Population details:
    parameter of interest:   variance ratio
    value under h_0:         1.0
    point estimate:          0.25

Test summary:
    outcome with 95% confidence: reject h_0
    two-sided p-value:           0.0040

Details:
    number of observations: [20, 20]
    F statistic:            0.25
    degrees of freedom:     [19, 19]
```
```julia
zx = (x.-mean(x)).^2; SEx = sqrt(var(zx)/length(zx));
zy = (y.-mean(y)).^2; SEy = sqrt(var(zy)/length(zy));
Vd = var(x)-var(y); SEdv = sqrt.(SEx^2+SEy^2);
p = cdf(Normal(Vd,SEdv),0)
2*minimum([p,1-p])
0.0030936461440564944
quantile(Normal(Vd,SEdv),[0.025,0.975])
2-element Vector{Float64}:
 -2.527902137928901
 -0.5131176694326123
```
```julia
using Pingouin

Pingouin.homoscedasticity([x,y,z],method="bartlett")
1×3 DataFrame
 Row │ T        pval        equal_var 
     │ Float64  Float64     Bool      
─────┼────────────────────────────────
   1 │ 19.2678  6.54713e-5      false

Pingouin.homoscedasticity([x,y,z],method="levene")
1×3 DataFrame
 Row │ W        pval        equal_var 
     │ Float64  Float64     Bool      
─────┼────────────────────────────────
   1 │ 5.76081  0.00526435      false
```

### 4.4.4. Testowanie hipotez dotyczących wskaźnika struktury

```julia
z = [ones(594);zeros(1234-594)];
p = cdf(Normal(mean(z),sqrt(var(z)/length(z))),0.5);
2*minimum([p,1-p])
0.19024188987863844
quantile(Normal(mean(z),sqrt(var(z)/length(z))),[0.25,0.975])
2-element Vector{Float64}:
 0.4717638434936078
 0.5092505328698035
```
```julia
x = [11,120,1300,14000,150000];
n = [100,1000,10000,100000,1000000];
v = x./n
5-element Vector{Float64}:
 0.11
 0.12
 0.13
 0.14
 0.15
```
```julia
function prop_test(x1,n1,x2,n2;correct=:true)
                p1 = x1/n1; p2 = x2/n2
                SE = sqrt.(((x1+x2)/(n1+n2))*(1-((x1+x2)/(n1+n2)))*(1/n1+1/n2))
                Z= ((p1-p2))/SE
                Zc= ((p1-p2)+0.5*(1/n1+1/n2))/SE
                if correct == :true
                  p=cdf(Normal(),Zc)
                  return 2*minimum([p,1-p])
                elseif correct == :false
                  p=cdf(Normal(),Z)
                  return 2*minimum([p,1-p])
                end
end
```
```julia
using MultipleTesting

dat = DataFrame(v=x./n,g=1:5);
res = collect(combinations(1:5,2));
DF = [dat[dat.g .== res[i][1], :v] - dat[dat.g .== res[i][2], :v]
      for i in 1:length(res) ];
dat = DataFrame(diff_prop = abs.(collect(Iterators.flatten(DF))));
dat.a = [res[i][1] for i in 1:length(res)];
dat.b = [res[i][2] for i in 1:length(res)];
dat.pval = [prop_test(x[dat.a[i]], n[dat.a[i]],
                      x[dat.b[i]], n[dat.b[i]]) for i in 1:length(res)];
t = select(dat, [:a,:b] => ByRow((x,y) -> string(x,"-",y)) => :g,
        Not([:a,:b]));
t.pvalBonf = adjust(t.pval, Bonferroni());
t
10×4 DataFrame
 Row │ g       diff_prop  pval         pvalBonf    
     │ String  Float64    Float64      Float64     
─────┼─────────────────────────────────────────────
   1 │ 1-2          0.01  0.894614     1.0
   2 │ 1-3          0.02  0.658041     1.0
   3 │ 1-4          0.03  0.471495     1.0
   4 │ 1-5          0.04  0.327015     1.0
   5 │ 2-3          0.01  0.395453     1.0
   6 │ 2-4          0.02  0.0769083    0.769083
   7 │ 2-5          0.03  0.00901647   0.0901647
   8 │ 3-4          0.01  0.0061404    0.061404
   9 │ 3-5          0.02  2.66279e-8   2.66279e-7
  10 │ 4-5          0.01  2.66165e-17  2.66165e-16
```

#### 4.4.5.1. Dwie zmienne ilościowe i współczynnik korelacji

```julia
x = rand(MersenneTwister(227),LogNormal(),50);
y = x + rand(MersenneTwister(2305),LogNormal(),50);
CorrelationTest(x,y)
Test for nonzero correlation
----------------------------
Population details:
    parameter of interest:   Correlation
    value under h_0:         0.0
    point estimate:          0.845647
    95% confidence interval: (0.7419, 0.9098)

Test summary:
    outcome with 95% confidence: reject h_0
    two-sided p-value:           <1e-13

Details:
    number of observations:          50
    number of conditional variables: 0
    t-statistic:                     10.9769
    degrees of freedom:              48
```
```julia
CorrelationTest(tiedrank(x),tiedrank(y))
Test for nonzero correlation
----------------------------
Population details:
    parameter of interest:   Correlation
    value under h_0:         0.0
    point estimate:          0.799376
    95% confidence interval: (0.6701, 0.8816)

Test summary:
    outcome with 95% confidence: reject h_0
    two-sided p-value:           <1e-11

Details:
    number of observations:          50
    number of conditional variables: 0
    t-statistic:                     9.21762
    degrees of freedom:              48
```

#### 4.4.5.2. Inne testy dla współczynnika korelacji Pearsona

```julia
function cor_test1(rho,n,rho0;alt=:two)
             SE = sqrt.(1/(n-3))
             p = cdf(Normal(atanh(rho),SE),atanh(rho0))
              if alt == :two
               return 2*minimum([p,1-p])
              elseif alt == :left
               return p
              elseif alt == :right
               return 1-p
              end
end
```
```julia
function cor_test2(rho1,n1,rho2,n2;alt=:two)
             SE = sqrt.(1/(n1-3)+1/(n2-3))
             p = cdf(Normal(atanh(rho1)-atanh(rho2),SE),0)
              if alt == :two
               return 2*minimum([p,1-p])
              elseif alt == :left
               return p
              elseif alt == :right
               return 1-p
              end
end
```

#### 4.4.5.3. Dwie zmienne jakościowe i macierze kontyngencji

```julia
using FreqTables

D = DataFrame(plec=String.(daneSoc.plec),praca=String.(daneSoc.praca);
tabela = freqtable(D.plec,D.praca)
2×2 Named Matrix{Int64}
Dim1 ╲ Dim2 │       nie pracuje  uczen lub pracuje
────────────┼─────────────────────────────────────
kobieta     │                 7                 48
mezczyzna   │                45                104
```
```julia
chiwynik = ChisqTest(tabela)
Pearson's Chi-square Test
-------------------------
Population details:
    parameter of interest:   Multinomial Probabilities
    value under h_0:         [0.0687236, 0.186178, 0.200884, 0.544214]
    point estimate:          [0.0343137, 0.220588, 0.235294, 0.509804]
    95% confidence interval: [(0.0, 0.1082), (0.152, 0.2945), (0.1667, 0.3092), (0.4412, 0.5837)]

Test summary:
    outcome with 95% confidence: reject h_0
    one-sided p-value:           0.0110

Details:
    Sample size:        204
    statistic:          6.458331213117512
    degrees of freedom: 1
    residuals:          [-1.87476, 1.13902, 1.09654, -0.666213]
    std. residuals:     [-2.54132, 2.54132, 2.54132, -2.54132]
```

```julia
FisherExactTest(f[1],f[3],f[2],f[4])
Fisher's exact test
-------------------
Population details:
    parameter of interest:   Odds ratio
    value under h_0:         1.0
    point estimate:          0.338609
    95% confidence interval: (0.12, 0.8309)

Test summary:
    outcome with 95% confidence: reject h_0
    two-sided p-value:           0.0143

Details:
    contingency table:
         7   48
        45  104
```
##### 4.4.5.3.1. Test symetrii McNemara

```julia
chisq = (abs(tabela[3]-tabela[2])-1)^2/(tabela[3]+tabela[2])
0.043010752688172046
pv = 1-cdf(Chisq(1),chisq)
0.8357050269952793
```

##### 4.4.5.3.2. Test Cochrana-Mantela-Haenszela

```julia
kobieta =[100 10; 10 1]
2×2 Matrix{Int64}:
 100  10
  10   1
mezczyzna = [1 10; 10 100]
2×2 Matrix{Int64}:
  1   10
 10  100
laczna = kobieta + mezczyzna
2×2 Matrix{Int64}:
 101   20
  20  101
```
```julia
pvalue(ChisqTest(kobieta))
1.0
pvalue(ChisqTest(mezczyzna))
1.0
pvalue(ChisqTest(laczna))
2.145881323880324e-25
```

#### 4.4.5.4. Współczynnik zgodności k

```julia
ocena1 = 2 .+ Int64.(round.(rand(Uniform(),100)*4));
ocena2 = 2 .+ Int64.(round.(rand(Uniform(),100)*4));
tab = freqtable(ocena1,ocena2)
5×5 Named Matrix{Int64}
Dim1 ╲ Dim2 │  2   3   4   5   6
────────────┼───────────────────
2           │  1   4   3   3   1
3           │  1   7   9   5   2
4           │  2   5   3  10   3
5           │  4   6   6   6   2
6           │  4   3   5   5   0

R"vcd::Kappa($tab)"
RObject{VecSxp}
              value     ASE      z Pr(>|z|)
Unweighted -0.06057 0.04618 -1.312   0.1897
Weighted   -0.06244 0.05952 -1.049   0.2942
```

### 4.4.6. Testosowanie zbioru hipotez

```julia
using MultipleTesting

DataFrame(
  pwart = [ pvalue(UnequalVarianceTTest(rand(Normal(),20),rand(Normal(1,1),20))) for i in 1:6 ],
  Bonferroni = adjust(pwart, Bonferroni()),
  BH = adjust(pwart, BenjaminiHochberg())
         )
6×3 DataFrame
 Row │ pwart       Bonferroni  BH          
     │ Float64     Float64     Float64     
─────┼─────────────────────────────────────
   1 │ 0.0048388   0.00035974  0.00035974
   2 │ 0.0102359   0.0196016   0.00490041
   3 │ 0.341945    0.243988    0.0406646
   4 │ 9.4735e-5   0.00101412  0.000507059
   5 │ 6.81188e-7  0.00647018  0.00215673
   6 │ 0.001447    0.222047    0.0406646
```

### 4.4.7. Rozkład statystyki testowej

```julia
function getStat(;N=30, p=0.2)
     x = rand(Binomial(1, p),N)
     y = rand(Binomial(1, p),N)
     tab = freqtable(x,y)
     if size(tab)[1] == 2 & size(tab)[2] == 2
       c = sum(tab,dims=1)
       w = sum(tab,dims=2)
       s = sum(tab)
       r = [c[1]*w[1]/s c[2]*w[1]/s; c[1]*w[2]/s c[2]*w[2]/s]
       return sum(map(i-> (abs(tab[i]-r[i])-0.5)^2/r[i], 1:4))
      else 0 end
end
```
```julia
st_testowa = [getStat() for i in 1:10000];
yw = ecdf(st_testowa)(st_testowa);
scatter(st_testowa,yw,legend=false,xlim=(-0.25,5),titlefont=font(10),
        title="Rozkłady statystyki testowej\nz uwzględnieniem poprawki Yatesa",
        markershape = :circle, markercolor = :black,size=(500,300),
        markerstrokealpha= 0.25, markerstrokecolor = :white, markerstrokewidth= 1)
plot!(sort(st_testowa),map(x-> cdf(Chisq(1),x),sort(st_testowa)),color=:red)
vline!([quantile(st_testowa,0.95)],color=:black)
vline!([quantile(Chisq(1),0.95)],color=:red)
hline!([0.95],color=:orange)
annotate!(quantile(st_testowa,0.95),0.25, text(round(quantile(st_testowa,0.95),digits=4), :top, 8, :dark, rotation = 90 ))
annotate!(quantile(Chisq(1),0.95),0.25, text(round(quantile(Chisq(1),0.95),digits=4), :top, 8, :red, rotation = 90 ))
savefig("/home/.../assets/p447.png")
```
![](/assets/p447.png)


```julia
kidney = rcopy(R"PBImisc::kidney");
x = kidney[kidney.recipient_age .< 40,:MDRD30];
y = kidney[kidney.recipient_age .>= 40,:MDRD30];
n = length(x); m = length(y); B = 999;
```
```julia
function KS(x,y; op=:pe)
    n_x, n_y = length(x), length(y)
    sort_idx = sortperm([x; y])
    pdf_diffs = [ones(n_x)/n_x; -ones(n_y)/n_y][sort_idx]
    cdf_diffs = cumsum(pdf_diffs)
    mx = maximum(cdf_diffs)
    mm = -minimum(cdf_diffs)
    md = max(mx,mm)
    if op == :pe
      return md
    elseif op == :ks
      return md*sqrt((n_x*n_y)/(n_x+n_y))
    end
end

KS(x,y)
0.2199219921992202
```
```julia
function thetastar(x,y,B)
 U = vcat(x,y)
 N = length(U)
 n1 = length(x)
 z = ones(B)
 for i in 1:B
  s = sample(1:N,n1,replace=false)
  @inbounds z[i] = KS(U[s],U[Not(s)])
 end
 return z
end

res = thetastar(x,y,10000);

p = mean(res .< KS(x,y))
0.999
1-p
0.0010000000000000009
```


### 4.5.1. Rozkład i obciążenie estymatorów

```julia
wynikBoot = [mean(sample(daneO.VEGF,length(daneO.VEGF),replace=true)) for i in 1:10000]
quantile(wynikBoot,[0.025,0.975])
2-element Vector{Float64}:
 1974.5886597938145
 3357.6786082474223
mean(wynikBoot)
2624.936430927835
```
```julia
using Plots, StatsPlots, Distributions

p1 = histogram(wynikBoot,normalize=true,legend=false);
p2 = qqplot(Normal,wynikBoot);
plot(p1, p2, layout = grid(1, 2), size=(800,300));
savefig("/home/.../assets/p451.png")
```
![](/assets/p451.png)


```julia
using Bootstrap

bs = bootstrap(daneO.VEGF, mean, BasicSampling(10000))
Bootstrap Sampling
  Estimates:
     Var │ Estimate  Bias      StdError
         │ Float64   Float64   Float64
    ─────┼──────────────────────────────
       1 │  2626.61  -5.53833   368.931
  Sampling: BasicSampling
  Samples:  10000
  Data:     Vector{Int64}: { 97 }

bci = confint(bs, BasicConfInt(0.95))
((2626.6082474226805, 1847.1373711340211, 3266.8126288659796),)
```


### 4.5.2. Testy bootstrapowe

```julia
function bootstrapowyTestT(x,B; mu=0)
      res = [mean(sample(x,length(x),replace=true)) for i in 1:B]
      p = mean(res .< mu)
      return 2*minimum([p,1-p])
end

bootstrapowyTestT(daneO.VEGF,10000,mu=3000)
0.31499999999999995
bootstrapowyTestT(daneO.VEGF,10000,mu=4000)
0.0012000000000000899
```

## 4.6. Analiza przeżycia

```julia
using Survival, GLM

dO = dropmissing(daneO, [:Okres_bez_wznowy,:Niepowodzenia,:Nowotwor,:Wiek]);
dO.czasy = EventTime.(dO.Okres_bez_wznowy, dO.Niepowodzenia .== "wznowa")
86-element Vector{EventTime{Int64}}:
 22+
 53+
 38+
 26+
 36
 33+
 38+
 38
 ⋮
 35+
 43+
 50
 30+
 36+
 29+
 46+
```

### 4.6.1. Krzywa przeżycia

```julia
km = fit(KaplanMeier, dO.czasy);
conf = confint(km);
survfit = DataFrame(
 time = km.times,
 natrisk = km.natrisk,
 nevents = km.nevents,
 survival = km.survival,
 stderr = km.stderr,
 lower = first.(conf),
 upper = last.(conf),
 ncensor = km.ncensor)
first(survfit,6)
6×8 DataFrame
 Row │ time   natrisk  nevents  survival  ncensor  stderr     lower     upper    
     │ Int64  Int64    Int64    Float64   Int64    Float64    Float64   Float64  
─────┼───────────────────────────────────────────────────────────────────────────
   1 │    10       86        1  0.988372        0  0.0116961  0.920322  0.998354
   2 │    16       85        1  0.976744        0  0.016639   0.910202  0.994133
   3 │    18       84        0  0.976744        1  0.016639   0.910202  0.994133
   4 │    19       83        1  0.964976        1  0.020586   0.895336  0.988569
   5 │    21       81        1  0.953063        0  0.0240438  0.879744  0.982123
   6 │    22       80        0  0.953063        1  0.0240438  0.879744  0.982123
```
```julia
plot(km.survival,ylim=(0,1),legend=false,size=(500,300))
plot!(first.(conf),color=:red,ls=:dash)
plot!(last.(conf),color=:red,ls=:dash)
savefig("/home/.../assets/p461.png")
```
![](/assets/p461.png)


### 4.6.2. Model Coxa

```julia
fit = coxph(@formula(czasy ~ Nowotwor+Wiek), dO; tol=1e-8)
StatsModels.TableRegressionModel{CoxModel{Float64}, Matrix{Float64}}

czasy ~ Nowotwor + Wiek

Coefficients:
───────────────────────────────────────────────────
            Estimate  Std.Error   z value  Pr(>|z|)
───────────────────────────────────────────────────
Nowotwor   1.75372    0.595655    2.94419    0.0032
Wiek      -0.0563166  0.0434047  -1.29748    0.1945
───────────────────────────────────────────────────

loglikelihood(fit) > nullloglikelihood(fit)
true
```

### 4.7.1. Operacje na zbiorach

```julia
x = 1:10;
y = 5:15;
setdiff(union(x,y),intersect(x,y))
9-element Vector{Int64}:
  1
  2
  3
  4
 11
 12
 13
 14
 15
```
```julia
issetequal(x,setdiff(x,setdiff(y,x)))
true
```

### 4.7.2. Operacje arytmetyczne

```julia
1-exp(0.1^15)
-1.1102230246251565e-15
expm1(0.1^15)
1.0000000000000015e-15
log(1+0.1^20)
0.0
log1p(0.1^15)
1.0000000000000003e-15
```


### 4.7.3. Wielomiany

```julia
using Polynomials

p1 = Polynomial([2,0,1])
Polynomial(2 + x^2)
p2 = Polynomial([2,2,1,1])
Polynomial(2 + 2*x + x^2 + x^3)
```
```julia
p1+p2
Polynomial(4 + 2*x + 2*x^2 + x^3)
p1*p2
Polynomial(4 + 4*x + 4*x^2 + 4*x^3 + x^4 + x^5)
```
```julia
integrate(p1,0,1)
2.3333333333333335
derivative(p2)
Polynomial(2 + 2*x + 3*x^2)
```
```julia
fromroots([-1,1])
Polynomial(-1 + x^2)
```
```julia
roots(p2)
3-element Vector{ComplexF64}:
                  -1.0 + 0.0im
 -4.85722573273506e-16 - 1.4142135623730954im
 -4.85722573273506e-16 + 1.4142135623730954im
```
```julia
gcd(p1,p2)
Polynomial(2 + x^2)
```

### 4.7.4. Bazy wielomianów ortogonalnych

```julia
using Polynomials, SpecialPolynomials

p = [convert(Polynomial, Laguerre{0.2}(ones(i))) for i in 1:5]
5-element Vector{Polynomial{Float64}}:
 Polynomial(1.0)
 Polynomial(2.2 - 1.0*x)
 Polynomial(3.52 - 3.2*x + 0.5*x^2)
 Polynomial(4.928000000000001 - 6.72*x + 2.1*x^2 - 0.16666666666666666*x^3)
 Polynomial(6.406400000000001 - 11.648000000000001*x + 5.46*x^2 - 0.8666666666666667*x^3 + 0.041666666666666664*x^4)
```
```julia
c = -1:0.01:8;
w = [map(x->p[i](x),c) for i in 1:5];
fg = plot();
for i in 1:5
  plot!(fg,c,w[i])
end
plot!(fg, size=(500,300))
savefig("/home/.../assets/p474.png")
```
![](/assets/p474.png)

### 4.7.5. Szukanie maksimum/minimum/zer funkcji

```julia
function f(x;wyznacznik=2)
     (x-7)^wyznacznik-x
end
op = optimize(f, -1,10)
Results of Optimization Algorithm
 * Algorithm: Brent's Method
 * Search Interval: [-1.000000, 10.000000]
 * Minimizer: 7.500000e+00
 * Minimum: -7.250000e+00
 * Iterations: 5
 * Convergence: max(|x - x_upper|, |x - x_lower|) <= 2*(1.5e-08*|x|+2.2e-16): true
 * Objective Function Calls: 6
```
```julia
using Roots

xo = find_zeros(f, -1, 10)
1-element Vector{Float64}:
 4.807417596432748
f(xo[1])
1.7763568394002505e-15
```

### 4.7.6. Rachunek różniczkowo-całkowy

```julia
using Symbolics

@variables x
Symbolics.derivative(3*(x-2)^2-15, x)
6x - 12
```
```julia
using ForwardDiff

ForwardDiff.derivative.(x->3*(x-2)^2-15, [1,2,3])
3-element Vector{Int64}:
 -6
  0
  6
```

```julia
using QuadGK, Distributions

quadgk(x-> pdf(Normal(),x), 0, Inf)
(0.4999999999999999, 5.67681333382849e-9)
```

```julia
quadgk(x-> sin(x)^2, 0, 600)
(300.02206965161776, 4.411114566504892e-6)
```


