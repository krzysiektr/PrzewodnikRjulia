---
layout: page
mathjax: true
title:  "Rozdział 2"
date:   2021-06-02 20:18:11 +0200
---

## 2.1 Wczytywanie danych

```julia
using CSV, HTTP, DataFrames

url = "http://www.biecek.pl/R/auta.csv";
auta = CSV.File(HTTP.get(url).body, delim=';') |> DataFrame;
first(auta,6)
6×8 DataFrame
 Row │ Marka    Model   Cena     KM     Pojemnosc  Przebieg  Paliwo   Produkcja 
     │ String   String  Float64  Int64  Int64      Float64   String   Int64     
─────┼──────────────────────────────────────────────────────────────────────────
   1 │ Peugeot  206      8799.0     60       1100   85000.0  benzyna       2003
   2 │ Peugeot  206     15500.0     60       1124  114000.0  benzyna       2005
   3 │ Peugeot  206     11900.0     90       1997  215000.0  diesel        2003
   4 │ Peugeot  206     10999.0     70       1868  165000.0  diesel        2003
   5 │ Peugeot  206     11900.0     70       1398  146000.0  diesel        2005
   6 │ Peugeot  206     19900.0     70       1398   86400.0  diesel        2006
```

## 2.2.1. Wektory

```julia
[2, 3, 5, 7, 11, 13, 17]
7-element Vector{Int64}:
  2
  3
  5
  7
 11
 13
 17

[2, 3, 5, 7, 11, 13, 17]'
1×7 adjoint(::Vector{Int64}) with eltype Int64:
 2  3  5  7  11  13  17
``` 

```julia
collect(-3:3)
7-element Vector{Int64}:
 -3
 -2
 -1
  0
  1
  2
  3

collect(-1:0.5:1)
5-element Vector{Float64}:
 -1.0
 -0.5
  0.0
  0.5
  1.0
```

```julia
collect(range(0,stop=100,step=11))
10-element Vector{Int64}:
  0
 11
 22
 33
 44
 55
 66
 77
 88
 99

collect(range(0,stop=100,length=5))
5-element Vector{Float64}:
   0.0
  25.0
  50.0
  75.0
 100.0
```

```julia
using Dates

monthname.(1:12)
12-element Vector{String}:
 "January"
 "February"
 "March"
 "April"
 "May"
 "June"
 "July"
 "August"
 "September"
 "October"
 "November"
 "December"

dayname.(1:7)
7-element Vector{String}:
 "Monday"
 "Tuesday"
 "Wednesday"
 "Thursday"
 "Friday"
 "Saturday"
 "Sunday"
```

```julia
Letters = String('A':'Z')
"ABCDEFGHIJKLMNOPQRSTUVWXYZ"

LETTERS = split(Letters,"")
26-element Vector{SubString{String}}:
 "A"
 "B"
 "C"
 "D"
 "E"
 "F"
 "G"
 "H"
 ⋮
 "T"
 "U"
 "V"
 "W"
 "X"
 "Y"
 "Z"
```

#### 2.2.1.1. Indeksowanie wektorów

```julia
LETTERS[5:10]
6-element Vector{SubString{String}}:
 "E"
 "F"
 "G"
 "H"
 "I"
 "J"

LETTERS[union([1],[2],9:14)]
8-element Vector{SubString{String}}:
 "A"
 "B"
 "I"
 "J"
 "K"
 "L"
 "M"
 "N"
```

```julia
co_drugi = range(1,stop=length(LETTERS),step=2)
1:2:25

LETTERS[co_drugi]
13-element Vector{SubString{String}}:
 "A"
 "C"
 "E"
 "G"
 "I"
 "K"
 "M"
 "O"
 "Q"
 "S"
 "U"
 "W"
 "Y"

split(String('A':2:'Z'),"")
13-element Vector{SubString{String}}:
 "A"
 "C"
 "E"
 "G"
 "I"
 "K"
 "M"
 "O"
 "Q"
 "S"
 "U"
 "W"
 "Y"
```

```julia
monthname.(1:12)[Not(5:9)]
7-element Vector{String}:
 "January"
 "February"
 "March"
 "April"
 "October"
 "November"
 "December"
```

```julia
wartosc = Dict("pion"=>1, "skoczek"=>3, "goniec"=>3,
               "wieza"=>5, "hetman"=>9, "krol"=>Inf)
Dict{String, Real} with 6 entries:
  "goniec"  => 3
  "skoczek" => 3
  "pion"    => 1
  "hetman"  => 9
  "krol"    => Inf
  "wieza"   => 5
map(k -> wartosc[k], ["goniec","wieza"])
2-element Vector{Int64}:
 3
 5
```

```julia
filter(kv -> kv.second < 6, wartosc)
Dict{String, Real} with 4 entries:
  "goniec"  => 3
  "skoczek" => 3
  "pion"    => 1
  "wieza"   => 5
```

#### 2.2.1.2. Zmiana elementów wektora

```julia
wartosc["wieza"] = 6;
wartosc["hetman"] = 7;
wartosc
Dict{String, Real} with 7 entries:
  "goniec"  => 3
  "skoczek" => 3
  "pion"    => 1
  "hetman"  => 7
  "krol"    => Inf
  "wieza"   => 6
```

```julia
wartosc["bierka"] = 5;
wartosc
Dict{String, Real} with 7 entries:
  "goniec"  => 3
  "skoczek" => 3
  "pion"    => 1
  "hetman"  => 7
  "bierka"  => 5
  "krol"    => Inf
  "wieza"   => 6
```

### 2.2.2. Ramki danych

```julia
using RCall

@rlibrary PogromcyDanych
koty_ptaki = rcopy(R"PogromcyDanych::koty_ptaki");
first(koty_ptaki,6)
6×7 DataFrame
 Row │ gatunek  waga     dlugosc  predkosc  habitat  zywotnosc  druzyna 
     │ Cat…     Float64  Float64  Int64     Cat…     Int64      Cat…    
─────┼──────────────────────────────────────────────────────────────────
   1 │ Tygrys     300.0      2.5        60  Azja            25  Kot
   2 │ Lew        200.0      2.0        80  Afryka          29  Kot
   3 │ Jaguar     100.0      1.7        90  Ameryka         15  Kot
   4 │ Puma        80.0      1.7        70  Ameryka         13  Kot
   5 │ Leopard     70.0      1.4        85  Azja            21  Kot
   6 │ Gepard      60.0      1.4       115  Afryka          12  Kot
```

#### 2.2.2.1. Indeksowanie ramek danych

```julia
koty_ptaki[[1,3,5],:]
3×7 DataFrame
 Row │ gatunek  waga     dlugosc  predkosc  habitat  zywotnosc  druzyna 
     │ Cat…     Float64  Float64  Int64     Cat…     Int64      Cat…    
─────┼──────────────────────────────────────────────────────────────────
   1 │ Tygrys     300.0      2.5        60  Azja            25  Kot
   2 │ Jaguar     100.0      1.7        90  Ameryka         15  Kot
   3 │ Leopard     70.0      1.4        85  Azja            21  Kot

koty_ptaki[[1,3,5],2:5]
3×4 DataFrame
 Row │ waga     dlugosc  predkosc  habitat 
     │ Float64  Float64  Int64     Cat…    
─────┼─────────────────────────────────────
   1 │   300.0      2.5        60  Azja
   2 │   100.0      1.7        90  Ameryka
   3 │    70.0      1.4        85  Azja
```

```julia
koty_ptaki[koty_ptaki[:,:predkosc].>100, [:gatunek,:predkosc,:dlugosc]]
5×3 DataFrame
 Row │ gatunek         predkosc  dlugosc 
     │ Cat…            Int64     Float64 
─────┼───────────────────────────────────
   1 │ Gepard               115      1.4
   2 │ Jerzyk               170      0.2
   3 │ Orzel przedni        160      0.9
   4 │ Sokol wedrowny       110      0.5
   5 │ Albatros             120      0.8
```

```julia
koty_ptaki[:,:predkosc]'
1×13 adjoint(::Vector{Int64}) with eltype Int64:
 60  80  90  70  85  115  65  170  70  160  110  100  120
```

```julia
koty_ptaki.predkosc_mile = koty_ptaki[:,:predkosc]*1.6;
first(koty_ptaki,2)[:,2:end]
2×7 DataFrame
 Row │ waga     dlugosc  predkosc  habitat  zywotnosc  druzyna  predkosc_mile 
     │ Float64  Float64  Int64     Cat…     Int64      Cat…     Float64       
─────┼────────────────────────────────────────────────────────────────────────
   1 │   300.0      2.5        60  Azja            25  Kot               96.0
   2 │   200.0      2.0        80  Afryka          29  Kot              128.0
```

### 2.2.3. Listy

```julia
trojka = Dict(:liczby=>1:5,:litery=>'A':'Z',:logika=>[true,true,true,false])
Dict{Symbol, AbstractVector{T} where T} with 3 entries:
  :liczby => 1:5
  :logika => Bool[1, 1, 1, 0]
  :litery => 'A':1:'Z'
```
```julia
split(String(trojka[:litery]),"")
26-element Vector{SubString{String}}:
 "A"
 "B"
 "C"
 "D"
 "E"
 "F"
 "G"
 ⋮
 "T"
 "U"
 "V"
 "W"
 "X"
 "Y"
 "Z"

trojka[:logika]
4-element Vector{Bool}:
 1
 1
 1
 0

collect(trojka[:liczby])'
1×5 adjoint(::Vector{Int64}) with eltype Int64:
 1  2  3  4  5
```

## 2.3. Statystyki opisowe

```julia
using RCall

@rlibrary Przewodnik
daneSoc = rcopy(R"Przewodnik::daneSoc");
first(daneSoc,4)
4×7 DataFrame
 Row │ wiek   wyksztalcenie  st_cywilny    plec          praca              cisnien ⋯
     │ Int64  Categorical…   Categorical…  Categorical…  Categorical…       Int64   ⋯
─────┼───────────────────────────────────────────────────────────────────────────────
   1 │    70  zawodowe       w zwiazku     mezczyzna     uczen lub pracuje          ⋯
   2 │    66  zawodowe       w zwiazku     kobieta       uczen lub pracuje
   3 │    71  zawodowe       singiel       kobieta       uczen lub pracuje
   4 │    57  srednie        w zwiazku     mezczyzna     uczen lub pracuje
                                                                    2 columns omitted
```

### 2.3.1. Zmienne ilościowe

```julia
extrema(daneSoc[:,:wiek])
(22, 75)
```

```julia
mean(daneSoc[:,:wiek])
43.161764705882355
```

```julia
using StatsBase

mean(trim(daneSoc[:,:wiek], prop=0.2))
42.77777777777778
```

```julia
using Statistics

median(daneSoc[:,:wiek])
45.0
```

```julia
summarystats(daneSoc[:,:wiek])
Summary Stats:
Length:         204
Missing Count:  0
Mean:           43.161765
Minimum:        22.000000
1st Quartile:   30.000000
Median:         45.000000
3rd Quartile:   53.000000
Maximum:        75.000000
```

```julia
std(daneSoc[:,:wiek])
13.84709991950723
```

```julia
#kurtosis type 1:
kurtosis(daneSoc[:,:wiek])
-0.9356588758458164
````

```julia
#skewnes type 1:
skewness(daneSoc[:,:wiek])
0.23487589610214846
```

```julia
quantile(daneSoc[:,:wiek],[0.1,0.25,0.5,0.75,0.9])
5-element Vector{Float64}:
 26.0
 30.0
 45.0
 53.0
 62.39999999999998
```

```julia
cor(Matrix(daneSoc[:,[1,6,7]]))
3×3 Matrix{Float64}:
  1.0        -0.0276524  -0.0831366
 -0.0276524   1.0         0.678527
 -0.0831366   0.678527    1.0
```

### 2.3.2. Zmienne jakościowe

```julia
countmap(daneSoc[:,:wyksztalcenie])
Dict{CategoricalArrays.CategoricalValue{String, UInt32}, Int64} with 4 entries:
  "zawodowe"   => 22
  "srednie"    => 55
  "wyzsze"     => 34
  "podstawowe" => 93
```

```julia
gdf = groupby(daneSoc, [:wyksztalcenie,:praca])
combine(gdf, nrow)
8×3 DataFrame
 Row │ wyksztalcenie  praca              nrow  
     │ Categorical…   Categorical…       Int64 
─────┼─────────────────────────────────────────
   1 │ podstawowe     nie pracuje           22
   2 │ podstawowe     uczen lub pracuje     71
   3 │ srednie        nie pracuje           16
   4 │ srednie        uczen lub pracuje     39
   5 │ wyzsze         nie pracuje            6
   6 │ wyzsze         uczen lub pracuje     28
   7 │ zawodowe       nie pracuje            8
   8 │ zawodowe       uczen lub pracuje     14
```


```julia
describe(daneSoc[:,1:4])
4×7 DataFrame
 Row │ variable       mean     min         median  max        nmissing  eltype      ⋯
     │ Symbol         Union…   Any         Union…  Any        Int64     DataType    ⋯
─────┼───────────────────────────────────────────────────────────────────────────────
   1 │ wiek           43.1618  22          45.0    75                0  Int64       ⋯
   2 │ wyksztalcenie           podstawowe          zawodowe          0  Categorical
   3 │ st_cywilny              singiel             w zwiazku         0  Categorical
   4 │ plec                    kobieta             mezczyzna         0  Categorical
                                                                     1 column omitted

describe(daneSoc)[:,[:variable,:eltype]]
7×2 DataFrame
 Row │ variable               eltype                           
     │ Symbol                 DataType                         
─────┼─────────────────────────────────────────────────────────
   1 │ wiek                   Int64
   2 │ wyksztalcenie          CategoricalValue{String, UInt32}
   3 │ st_cywilny             CategoricalValue{String, UInt32}
   4 │ plec                   CategoricalValue{String, UInt32}
   5 │ praca                  CategoricalValue{String, UInt32}
   6 │ cisnienie_skurczowe    Int64
   7 │ cisnienie_rozkurczowe  Int64
```

## 2.4. Statystyki graficzne

### 2.4.1. Wykres słupkowy, funkcja: barplot()

```julia
using Plots, StatsBase, StatsPlots, DataFrames

daneSoc[!,:wyksztalcenie] = convert.(String,daneSoc[!,:wyksztalcenie]);
tab1 = sort(countmap(daneSoc[:,:wyksztalcenie]))
p241a = bar(collect(keys(tab1)),collect(values(tab1)),orientation=:h,legend=:false,
        fillalpha=0.4,lc="white");
daneSoc[!,:plec] = convert.(String,daneSoc[!,:plec]);
gdf = groupby(daneSoc, [:wyksztalcenie,:plec]);
tab2 = combine(gdf, nrow)
p241b = groupedbar(tab2[:,:wyksztalcenie], tab2[:,:nrow], group = tab2[:,:plec],
        fillalpha=0.4,lc="white");
plot(p241a,p241b, layout = grid(1, 2),size=(800,300))
savefig("/home/.../Przewodnik_Julia/assets/p241.png")
```
![](/assets/p241.png)


### 2.4.2. Histogram, funkcja: hist()

```julia
h1 = histogram(daneSoc[:,:wiek], legend=false,nbins=15,color="white",
               title="Histogram wieku", xlabel="wiek", ylabel="liczba");
h2 = histogram(daneSoc[:,:wiek], legend=false,nbins=15,
               color="blue",fillalpha = 0.3,lc="white",normalize=true,
               title="Histogram wieku", xlabel="wiek", ylabel="czestosc");
plot(h1, h2, layout = grid(1, 2), size=(800,300));
savefig("/home/.../Przewodnik_Julia/assets/p242.png")
```
![](/assets/p242.png)


### 2.4.3. Wykres pudełkowy: boxplot()

```julia
s1 = stack(daneSoc[:,:6:7],1:2);
b1 = boxplot(s1.variable, s1.value, legend=false,
             color="blue",fillalpha = 0.3,alpha=0.4,xticks=(unique(s1.variable),["rozskurczowe","skurczowe"]));
s2 = daneSoc[:,1:2];
b2 = boxplot(s2.wyksztalcenie, s2.wiek, legend=false,
             color="blue",fillalpha = 0.3,alpha=0.4,
             ylabel="wiek");
plot(b1, b2, layout = grid(1, 2), size=(800,300));
savefig("/home/.../Przewodnik_Julia/assets/p243.png")
```
![](/assets/p243.png)

### 2.4.4. Jądrowy estymator gęstości, funkcja: density()

```julia
using KernelDensity

z1 = kde(daneSoc.wiek);
d1 = plot(z1.x,z1.density,legend=false,
          title="Rozklad wieku");
z2 = kde(daneSoc.wiek,bandwidth =1.5);
d2 = plot(z2.x,z2.density,legend=false,
          color="blue",fillalpha = 0.3,fillrange=0,alpha=0.4,
          title="Rozklad wieku");
plot(d1, d2, layout = grid(1, 2), size=(800,300));
savefig("/home/.../Przewodnik_Julia/assets/p244.png")
```
![](/assets/p244.png)

```julia
yw = ecdf(daneSoc.wiek)(daneSoc.wiek);
c1 = plot(daneSoc.wiek,yw,title="Dystrybuanta wieku",linecolor=false,legend=false,
          markershape = :circle, markercolor = :black,
          markerstrokealpha= 0.25, markerstrokecolor = :white, markerstrokewidth= 1);
mezczyzna = filter(row -> row.plec == "mezczyzna", daneSoc);
kobieta = filter(row -> row.plec == "kobieta", daneSoc);
ym = ecdf(mezczyzna.wiek)(mezczyzna.wiek);
yk = ecdf(kobieta.wiek)(kobieta.wiek);
c2 = plot(mezczyzna.wiek,ym,linecolor=false,labels="mezczyzna",legend=:topleft,
          markershape = :circle, markercolor = :white,
          markerstrokealpha= 0.25, markerstrokecolor = :black, markerstrokewidth= 1);
c2 = plot!(kobieta.wiek,yk,title="Wiek / plec",linecolor=false,labels="kobieta",
           markershape = :circle, markercolor = :orangered4,markeralpha=0.75,
           markerstrokealpha= 0.25, markerstrokecolor = :white, markerstrokewidth= 1);
plot(c1, c2, layout = grid(1, 2), size=(800,300));
savefig("/home/.../Przewodnik_Julia/assets/p244a.png")
```
![](/assets/p244a.png)

### 2.4.5. Wykres kropkowy, funkcja: scatterplot()

```julia
using Gadfly, Cairo

p1 = Gadfly.plot(daneSoc, x=:cisnienie_skurczowe, y=:cisnienie_rozkurczowe,
     Theme(key_position=:inside),
     layer(Stat.smooth(method=:lm,levels=[0.9]),Geom.line,Geom.ribbon,color=["Smooth"]),
     layer(Geom.point, color=["Points"]));
p2 = Gadfly.plot(daneSoc, x=:cisnienie_skurczowe, y=:cisnienie_rozkurczowe, color=:plec,
     Theme(key_position=:inside),
     Geom.smooth(method=:lm), Geom.point, alpha=[0.6],
     layer(Geom.smooth(method=:loess), color=[colorant"grey"], order=2));
draw(PNG("/home/.../Przewodnik_Julia/assets/p245.png", 8inch, 3inch), hstack(p1, p2))
```
![](/assets/p245.png)


### 2.4.6. Wykres mozaikowy, funkcja: mosaicplot()

```julia
using OnlineStats

o = Mosaic(eltype(daneSoc.wyksztalcenie), eltype(daneSoc.praca));
fit!(o, zip(daneSoc.wyksztalcenie,daneSoc.praca));
plot(o, legendtitle="praca", xlabel="wyksztalcenie",fillalpha=0.4)
savefig("/home/.../Przewodnik_Julia/assets/p246.png")
```
![](/assets/p246.png)


## 2.5. Jak przetwarzać dane z pakietem `dplyr`

### 2.5.1. Jak filtrować wiersze

```julia
using DataFrames, DataFramesMeta

tylkoCorsa = @where(auta, :Model .== "Corsa");
first(tylkoCorsa,6)
6×8 DataFrame
 Row │ Marka   Model   Cena     KM     Pojemnosc  Przebieg  Paliwo   Produkcja 
     │ String  String  Float64  Int64  Int64      Float64   String   Int64     
─────┼─────────────────────────────────────────────────────────────────────────
   1 │ Opel    Corsa   13450.0     70       1248  190000.0  diesel        2004
   2 │ Opel    Corsa   25990.0     75       1300   84000.0  diesel        2008
   3 │ Opel    Corsa   17990.0     80       1229   48606.0  benzyna       2007
   4 │ Opel    Corsa   23999.0     60        998   63000.0  benzyna       2009
   5 │ Opel    Corsa   16500.0    101       1700  118000.0  diesel        2006
   6 │ Opel    Corsa   27900.0     80       1229   73500.0  benzyna       2007
```

```julia
tylkoCorsa = @where(auta, :Model .== "Corsa", :Produkcja .== 2010, :Paliwo .== "diesel");
first(tylkoCorsa,6)
6×8 DataFrame
 Row │ Marka   Model   Cena     KM     Pojemnosc  Przebieg  Paliwo  Produkcja 
     │ String  String  Float64  Int64  Int64      Float64   String  Int64     
─────┼────────────────────────────────────────────────────────────────────────
   1 │ Opel    Corsa   13450.0     70       1248  190000.0  diesel       2010
   2 │ Opel    Corsa   25990.0     75       1300   84000.0  diesel       2010
   3 │ Opel    Corsa   16500.0    101       1700  118000.0  diesel       2010
   4 │ Opel    Corsa   18800.0     69       1248   76000.0  diesel       2010
   5 │ Opel    Corsa   26900.0    125       1686  129100.0  diesel       2010
   6 │ Opel    Corsa   29900.0     75       1248   76800.0  diesel       2010
```

### 2.5.2. Jak wybierać kolumny

```julia
trzyKolumny = @select(auta, :Marka, :Model, :Cena);

first(trzyKolumny,6)
6×3 DataFrame
 Row │ Marka    Model   Cena    
     │ String   String  Float64 
─────┼──────────────────────────
   1 │ Peugeot  206      8799.0
   2 │ Peugeot  206     15500.0
   3 │ Peugeot  206     11900.0
   4 │ Peugeot  206     10999.0
   5 │ Peugeot  206     11900.0
   6 │ Peugeot  206     19900.0
```

```julia
first(@select(auta, cols(names(auta, r"^M"))),6)
6×2 DataFrame
 Row │ Marka    Model  
     │ String   String 
─────┼─────────────────
   1 │ Peugeot  206
   2 │ Peugeot  206
   3 │ Peugeot  206
   4 │ Peugeot  206
   5 │ Peugeot  206
   6 │ Peugeot  206
```

### 2.5.3. Jak tworzyć i transformować zmienne

```julia
autaZWiekiem = @linq auta |>
               transform(Wiek = 2013 .- :Produkcja) |>
               transform(PrzebiegNaRok = round.(:Przebieg ./ :Wiek));
first(@select(autaZWiekiem, :Marka, :Model, :Cena, :Paliwo, :Wiek, :PrzebiegNaRok),6)
6×6 DataFrame
 Row │ Marka    Model   Cena     Paliwo   Wiek   PrzebiegNaRok 
     │ String   String  Float64  String   Int64  Float64       
─────┼─────────────────────────────────────────────────────────
   1 │ Peugeot  206      8799.0  benzyna     10         8500.0
   2 │ Peugeot  206     15500.0  benzyna      8        14250.0
   3 │ Peugeot  206     11900.0  diesel      10        21500.0
   4 │ Peugeot  206     10999.0  diesel      10        16500.0
   5 │ Peugeot  206     11900.0  diesel       8        18250.0
   6 │ Peugeot  206     19900.0  diesel       7        12343.0
```

### 2.5.4. Jak sortować wiersze

```julia
posortowaneAuta = @orderby(auta, :Model, :Cena);
first(posortowaneAuta,6)
6×8 DataFrame
 Row │ Marka    Model   Cena     KM     Pojemnosc  Przebieg  Paliwo       Produkcja ⋯
     │ String   String  Float64  Int64  Int64      Float64   String       Int64     ⋯
─────┼───────────────────────────────────────────────────────────────────────────────
   1 │ Peugeot  206     6330.7      90       1997   90000.0  diesel            2004 ⋯
   2 │ Peugeot  206     6599.0      70       1398  277000.0  diesel            2004
   3 │ Peugeot  206     6900.0      90       1997  132000.0  diesel            2004
   4 │ Peugeot  206     7900.0      88       1360  114000.0  benzyna           2004
   5 │ Peugeot  206     8469.45     60       1124   77800.0  benzyna           2005 ⋯
   6 │ Peugeot  206     8500.0      60       1124  117000.0  benzyna+LPG       2004
```

```julia
posortowaneAuta = @orderby(auta, :Model, -:Cena);
first(posortowaneAuta,2)
2×8 DataFrame
 Row │ Marka    Model   Cena     KM     Pojemnosc  Przebieg  Paliwo   Produkcja 
     │ String   String  Float64  Int64  Int64      Float64   String   Int64     
─────┼──────────────────────────────────────────────────────────────────────────
   1 │ Peugeot  206     36115.0     60       1100     100.0  benzyna       2010
   2 │ Peugeot  206     33600.0     65       1100     380.0  benzyna       2011
```

### 2.5.5. Jak pracować z potokami

**Problem cebulki**

```julia
tylkoKia = @where(auta, :Marka .== "Kia");
posortowane = @orderby(tylkoKia, :Cena);
czteryKolumny = @select(posortowane, :Model, :Cena, :Przebieg, :Produkcja);
first(czteryKolumny, 4)
4×4 DataFrame
 Row │ Model   Cena     Przebieg  Produkcja 
     │ String  Float64  Float64   Int64     
─────┼──────────────────────────────────────
   1 │ Cee'd    1026.6   28000.0       2008
   2 │ Cee'd   13900.0  129000.0       2009
   3 │ Cee'd   14700.0   34000.0       2009
   4 │ Cee'd   14900.0  158500.0       2005
```

```julia
first(
  @select(
    @orderby(
       @where(auta,:Marka .== "Kia"),
    :Cena),
  :Model, :Cena, :Przebieg, :Produkcja), 4)
4×4 DataFrame
 Row │ Model   Cena     Przebieg  Produkcja 
     │ String  Float64  Float64   Int64     
─────┼──────────────────────────────────────
   1 │ Cee'd    1026.6   28000.0       2008
   2 │ Cee'd   13900.0  129000.0       2009
   3 │ Cee'd   14700.0   34000.0       2009
   4 │ Cee'd   14900.0  158500.0       2005
```

**Potoki w akcji**

```julia
posortowane = @linq auta |>
    where(:Marka .== "Kia") |>
    orderby(:Cena) |>
    select(:Model, :Cena, :Przebieg, :Produkcja);
first(posortowane,6)
6×4 DataFrame
 Row │ Model   Cena     Przebieg  Produkcja 
     │ String  Float64  Float64   Int64     
─────┼──────────────────────────────────────
   1 │ Cee'd    1026.6   28000.0       2008
   2 │ Cee'd   13900.0  129000.0       2009
   3 │ Cee'd   14700.0   34000.0       2009
   4 │ Cee'd   14900.0  158500.0       2005
   5 │ Cee'd   16900.0    9900.0       2010
   6 │ Cee'd   18803.9   15000.0       2010
```

### 2.5.6. Jak wyznaczać agregaty / statystyki w grupach

**Agregaty**

```julia
@linq auta |>
      combine(sredniaCena = mean(:Cena),
              medianaPrzebiegu = median(:Przebieg),
              sredniWiek = mean(2013 .- :Produkcja),
              liczba = length(:Marka))
1×4 DataFrame
 Row │ sredniaCena  medianaPrzebiegu  sredniWiek  liczba 
     │ Float64      Float64           Float64     Int64  
─────┼───────────────────────────────────────────────────
   1 │     33340.4          122000.0     6.63833    2400
```

**Grupowanie**

```julia
@linq auta |>
      groupby(:Marka) |>
      combine(sredniaCena = mean(:Cena),
              medianaPrzebiegu = median(:Przebieg),
              sredniWiek = mean(2013 .- :Produkcja),
              liczba = length(:Marka)) |> orderby(:Marka)
6×5 DataFrame
 Row │ Marka       sredniaCena  medianaPrzebiegu  sredniWiek  liczba 
     │ String      Float64      Float64           Float64     Int64  
─────┼───────────────────────────────────────────────────────────────
   1 │ Audi            59891.6         1.50452e5       6.55      200
   2 │ Fiat            15772.3     75000.0             6.33      200
   3 │ Kia             36982.6     42850.0             3.98      200
   4 │ Opel            33590.7    120550.0             6.615     800
   5 │ Peugeot         17388.5    127750.0             8.165     400
   6 │ Volkswagen      39432.7    146448.0             6.67      600
```

```julia
@linq auta |>
      groupby(:Marka) |>
      combine(sredniaCena = mean(:Cena),
              medianaPrzebiegu = median(:Przebieg),
              sredniWiek = mean(2013 .- :Produkcja),
              liczba = length(:Marka)) |> orderby(:sredniaCena)
6×5 DataFrame
 Row │ Marka       sredniaCena  medianaPrzebiegu  sredniWiek  liczba 
     │ String      Float64      Float64           Float64     Int64  
─────┼───────────────────────────────────────────────────────────────
   1 │ Fiat            15772.3     75000.0             6.33      200
   2 │ Peugeot         17388.5    127750.0             8.165     400
   3 │ Opel            33590.7    120550.0             6.615     800
   4 │ Kia             36982.6     42850.0             3.98      200
   5 │ Volkswagen      39432.7    146448.0             6.67      600
   6 │ Audi            59891.6         1.50452e5       6.55      200
```

```julia
@linq auta |>
      select(:Marka, :Cena, :Przebieg, :Model) |>
      groupby(:Marka) |>
      transform(Przebieg = :Przebieg ./ mean(:Przebieg))
2400×4 DataFrame
  Row │ Marka    Cena     Przebieg  Model  
      │ String   Float64  Float64   String 
──────┼────────────────────────────────────
    1 │ Peugeot   8799.0  0.671202  206
    2 │ Peugeot  15500.0  0.9002    206
    3 │ Peugeot  11900.0  1.69775   206
    4 │ Peugeot  10999.0  1.30292   206
    5 │ Peugeot  11900.0  1.15289   206
    6 │ Peugeot  19900.0  0.682257  206
  ⋮   │    ⋮        ⋮        ⋮        ⋮
 2396 │ Opel     24000.0  0.710207  Zafira
 2397 │ Opel     21900.0  1.68345   Zafira
 2398 │ Opel     29990.0  0.815423  Zafira
 2399 │ Opel     34048.9  0.988056  Zafira
 2400 │ Opel     36769.4  0.987135  Zafira
```

### 2.5.7. Postać szeroka / postać wąska

```julia
using DBnomics

# dostępne zbiory danych:
df = rdb_datasets("Eurostat", simplify = true);
o = occursin.("Modal split of passenger transport", df.name);
s = df[o,:]
2×2 DataFrame
 Row │ code           name                              
     │ String         String                            
─────┼──────────────────────────────────────────────────
   1 │ t2020_rk310    Modal split of passenger transpo…
   2 │ tran_hv_psmod  Modal split of passenger transpo…

d1 = rdb_dimensions("Eurostat", "t2020_rk310", simplify=true)
Dict{Symbol, DataFrame} with 4 entries:
  :unit    => 1×2 DataFrame…
  :vehicle => 3×2 DataFrame…
  :geo     => 37×2 DataFrame…
  :FREQ    => 1×2 DataFrame…

G = DataFrame(d1[:geo])
37×2 DataFrame
 Row │ Geopolitical entity (reporting)    geo    
     │ String                             String 
─────┼───────────────────────────────────────────
   1 │ Malta                              MT
   2 │ Slovenia                           SI
   3 │ Spain                              ES
   4 │ United Kingdom                     UK
   5 │ Luxembourg                         LU
   6 │ Hungary                            HU
  ⋮  │                 ⋮                    ⋮
  33 │ Czechia                            CZ
  34 │ Slovakia                           SK
  35 │ Denmark                            DK
  36 │ Estonia                            EE
  37 │ Greece                             EL

df2 = rdb("Eurostat", "t2020_rk310", dimensions = Dict(:geo => G.geo))
d = df2[:,[:unit, :vehicle, :geo, :original_period, :value]]
2798×5 DataFrame
  Row │ unit    vehicle  geo     original_period  value    
      │ String  String   String  String           Float64? 
──────┼────────────────────────────────────────────────────
    1 │ PC      BUS_TOT  AT      1990                  8.2
    2 │ PC      BUS_TOT  AT      1991                  8.1
    3 │ PC      BUS_TOT  AT      1992                  8.2
    4 │ PC      BUS_TOT  AT      1993                  8.4
    5 │ PC      BUS_TOT  AT      1994                  8.8
    6 │ PC      BUS_TOT  AT      1995                  9.0
  ⋮   │   ⋮        ⋮       ⋮            ⋮            ⋮
 2794 │ PC      TRN      UK      2014                  8.5
 2795 │ PC      TRN      UK      2015                  8.7
 2796 │ PC      TRN      UK      2016                  8.8
 2797 │ PC      TRN      UK      2017                  8.9
 2798 │ PC      TRN      UK      2018                  8.9
                                          2787 rows omitted
```

**Rozciągnij na kolumny**

```julia
szeroka = unstack(d, Not([:original_period, :value]), :original_period, :value);
szeroka[szeroka.geo .=="PL", :]
3×32 DataFrame
 Row │ unit    vehicle  geo     1990      1991      1992      1993      1994      1995  ⋯
     │ String  String   String  Float64?  Float64?  Float64?  Float64?  Float64?  Float ⋯
─────┼───────────────────────────────────────────────────────────────────────────────────
   1 │ PC      BUS_TOT  PL          28.1      25.6      24.4      23.2      20.9      1 ⋯
   2 │ PC      CAR      PL          41.3      49.8      55.3      57.9      62.3      6
   3 │ PC      TRN      PL          30.6      24.6      20.3      18.9      16.8      1
                                                                       24 columns omitted
```

```julia
W = d[d.original_period .=="2010", :];
szeroka2 = unstack(W, Not([:geo, :value]), :geo, :value)
3×40 DataFrame
 Row │ unit    vehicle  original_period  AT        BE        BG        CH        CY     ⋯
     │ String  String   String           Float64?  Float64?  Float64?  Float64?  Float6 ⋯
─────┼───────────────────────────────────────────────────────────────────────────────────
   1 │ PC      BUS_TOT  2010                  9.3      12.7      16.4       5.1       1 ⋯
   2 │ PC      CAR      2010                 79.6      79.7      80.0      77.3       8
   3 │ PC      TRN      2010                 11.1       7.7       3.6      17.6  missin
                                                                       33 columns omitted
```

**Zbierz kolumny**

```julia
stack(szeroka, Not([:geo,:vehicle,:unit]))
3219×5 DataFrame
  Row │ unit    vehicle  geo     variable  value     
      │ String  String   String  String    Float64?  
──────┼──────────────────────────────────────────────
    1 │ PC      BUS_TOT  AT      1990            8.2
    2 │ PC      BUS_TOT  BE      1990           10.6
    3 │ PC      BUS_TOT  BG      1990      missing   
    4 │ PC      BUS_TOT  CH      1990            3.7
    5 │ PC      BUS_TOT  CY      1990      missing   
    6 │ PC      BUS_TOT  CZ      1990      missing   
  ⋮   │   ⋮        ⋮       ⋮        ⋮          ⋮
 3215 │ PC      TRN      SE      2018            9.7
 3216 │ PC      TRN      SI      2018            1.8
 3217 │ PC      TRN      SK      2018            9.9
 3218 │ PC      TRN      TR      2018            1.7
 3219 │ PC      TRN      UK      2018            8.9
                                    3208 rows omitted
```

### 2.5.8. Sklejanie / rozcinanie kolumn

**Sklejanie kolumn**

```julia
select(d, [:geo,:original_period] => ByRow((x,y) -> string(x,":",y)) => :geo_time,
       Not([:geo,:original_period]))
2798×4 DataFrame
  Row │ geo_time  unit    vehicle  value    
      │ String    String  String   Float64? 
──────┼─────────────────────────────────────
    1 │ AT:1990   PC      BUS_TOT       8.2
    2 │ AT:1991   PC      BUS_TOT       8.1
    3 │ AT:1992   PC      BUS_TOT       8.2
    4 │ AT:1993   PC      BUS_TOT       8.4
    5 │ AT:1994   PC      BUS_TOT       8.8
    6 │ AT:1995   PC      BUS_TOT       9.0
  ⋮   │    ⋮        ⋮        ⋮        ⋮
 2794 │ UK:2014   PC      TRN           8.5
 2795 │ UK:2015   PC      TRN           8.7
 2796 │ UK:2016   PC      TRN           8.8
 2797 │ UK:2017   PC      TRN           8.9
 2798 │ UK:2018   PC      TRN           8.9
                           2787 rows omitted
```

**Rozcinanie kolumn**

```julia
df = DataFrame(daty=["2004-01-01","2012-04-15","2006-10-29","2010-03-03"],id=1:4)
4×2 DataFrame
 Row │ daty        id    
     │ String      Int64 
─────┼───────────────────
   1 │ 2004-01-01      1
   2 │ 2012-04-15      2
   3 │ 2006-10-29      3
   4 │ 2010-03-03      4
select(df, :daty => ByRow(x -> split(x,"-")) => [:rok, :miesiac, :dzien],:id)
4×4 DataFrame
 Row │ rok        miesiac    dzien      id    
     │ SubStrin…  SubStrin…  SubStrin…  Int64 
─────┼────────────────────────────────────────
   1 │ 2004       01         01             1
   2 │ 2012       04         15             2
   3 │ 2006       10         29             3
   4 │ 2010       03         03             4
```

## 2.6. Jak wczytać i zapisać dane w różnych formatach

### 2.6.1. Wczytywanie danych z pakietów

```julia
using RCall

@rlibrary PogromcyDanych
koty_ptaki = rcopy(R"PogromcyDanych::koty_ptaki");
first(koty_ptaki,6)
6×7 DataFrame
 Row │ gatunek  waga     dlugosc  predkosc  habitat  zywotnosc  druzyna 
     │ Cat…     Float64  Float64  Int64     Cat…     Int64      Cat…    
─────┼──────────────────────────────────────────────────────────────────
   1 │ Tygrys     300.0      2.5        60  Azja            25  Kot
   2 │ Lew        200.0      2.0        80  Afryka          29  Kot
   3 │ Jaguar     100.0      1.7        90  Ameryka         15  Kot
   4 │ Puma        80.0      1.7        70  Ameryka         13  Kot
   5 │ Leopard     70.0      1.4        85  Azja            21  Kot
   6 │ Gepard      60.0      1.4       115  Afryka          12  Kot
```

```julia
using RDatasets

RDatasets.packages()
34×2 DataFrame
 Row │ Package     Title                             
     │ String      String                            
─────┼───────────────────────────────────────────────
   1 │ COUNT       Functions, data and code for cou…
   2 │ Ecdat       Data sets for econometrics
   3 │ HSAUR       A Handbook of Statistical Analys…
   4 │ HistData    Data sets from the history of st…
   5 │ ISLR        Data for An Introduction to Stat…
   6 │ KMsurv      Data sets from Klein and Moeschb…
  ⋮  │     ⋮                       ⋮
  30 │ rpart       Recursive Partitioning and Regre…
  31 │ sandwich    Robust Covariance Matrix Estimat…
  32 │ sem         Structural Equation Models
  33 │ survival    Survival Analysis
  34 │ vcd         Visualizing Categorical Data
```
```julia
RDatasets.datasets("ggplot2")[:,2:end]
8×4 DataFrame
 Row │ Dataset       Title                              Rows   Columns 
     │ String        String                             Int64  Int64   
─────┼─────────────────────────────────────────────────────────────────
   1 │ diamonds      Prices of 50,000 round cut diamo…  53940       10
   2 │ economics     US economic time series.             478        6
   3 │ midwest       Midwest demographics.                437       28
   4 │ movies        Movie information and user ratin…  58788       24
   5 │ mpg           Fuel economy data from 1999 and …    234       11
   6 │ msleep        An updated and expanded version …     83       11
   7 │ presidential  Terms of 10 presidents from Eise…     10        4
   8 │ seals         Vector field of seal movements.     1155        4
```

```julia
diamonds = dataset("ggplot2", "diamonds");
first(diamonds,6)
6×10 DataFrame
 Row │ Carat    Cut        Color  Clarity  Depth    Table    Price  X        Y     ⋯
     │ Float64  Cat…       Cat…   Cat…     Float64  Float64  Int32  Float64  Float ⋯
─────┼──────────────────────────────────────────────────────────────────────────────
   1 │    0.23  Ideal      E      SI2         61.5     55.0    326     3.95     3. ⋯
   2 │    0.21  Premium    E      SI1         59.8     61.0    326     3.89     3.
   3 │    0.23  Good       E      VS1         56.9     65.0    327     4.05     4.
   4 │    0.29  Premium    I      VS2         62.4     58.0    334     4.2      4.
   5 │    0.31  Good       J      SI2         63.3     58.0    335     4.34     4. ⋯
   6 │    0.24  Very Good  J      VVS2        62.8     57.0    336     3.94     3.
```

### 2.6.2. Wczytywanie danych z plików tekstowych

```julia
using CSV, HTTP, DataFrames

url = "http://www.biecek.pl/R/koty_ptaki.csv";
koty_ptaki = CSV.File(HTTP.get(url).body, delim=';', decimal=',') |> DataFrame;
first(koty_ptaki,6)
6×7 DataFrame
 Row │ gatunek  waga     dlugosc  predkosc  habitat  zywotnosc  druzyna 
     │ String   Float64  Float64  Int64     String   Int64      String  
─────┼──────────────────────────────────────────────────────────────────
   1 │ Tygrys     300.0      2.5        60  Azja            25  Kot
   2 │ Lew        200.0      2.0        80  Afryka          29  Kot
   3 │ Jaguar     100.0      1.7        90  Ameryka         15  Kot
   4 │ Puma        80.0      1.7        70  Ameryka         13  Kot
   5 │ Leopard     70.0      1.4        85  Azja            21  Kot
   6 │ Gepard      60.0      1.4       115  Afryka          12  Kot
```

### 2.6.4. Wczytywanie i zapisywanie z zastosowaniem formatu JSON

```julia
using JSON

e = JSON.parse("""
         { "zrodlo": "Przewodnik",
           "wskazniki": [
               {"rok": "2010", "wartosc": "15.5"},
               {"rok": "2011", "wartosc": "1.01"},
               {"rok": "2012", "wartosc": "12.2"} ] }
           """)

Dict{String, Any} with 2 entries:
  "zrodlo"   => "Przewodnik"
  "wskazniki" => Any[Dict{String, Any}("wartosc"=>"15.5", "rok"=>"2010"), Dict{String, A…

dt = [DataFrame(e["wskazniki"][i]) for i in 1:3]
3-element Vector{DataFrame}:
 1×2 DataFrame
 Row │ rok     wartosc 
     │ String  String  
─────┼─────────────────
   1 │ 2010    15.5
 1×2 DataFrame
 Row │ rok     wartosc 
     │ String  String  
─────┼─────────────────
   1 │ 2011    1.01
 1×2 DataFrame
 Row │ rok     wartosc 
     │ String  String  
─────┼─────────────────
   1 │ 2012    12.2

vcat(dt[1],dt[2],dt[3])
3×2 DataFrame
 Row │ rok     wartosc 
     │ String  String  
─────┼─────────────────
   1 │ 2010    15.5
   2 │ 2011    1.01
   3 │ 2012    12.2
```

```julia
JSON.print(koty_ptaki)

{"gatunek":["Tygrys","Lew","Jaguar","Puma","Leopard","Gepard","Irbis","Jerzyk","Strus",
"Orzel przedni","Sokol wedrowny","Sokol norweski","Albatros"],"waga":[300.0,200.0,100.0,
80.0,70.0,60.0,50.0,0.05,150.0,5.0,0.7,2.0,4.0],"dlugosc":[2.5,2.0,1.7,1.7,1.4,1.4,1.3,
0.2,2.5,0.9,0.5,0.7,0.8],"predkosc":[60,80,90,70,85,115,65,170,70,160,110,100,120],
"habitat":["Azja","Afryka","Ameryka","Ameryka","Azja","Afryka","Azja","Euroazja","Afryka",
"Polnoc","Polnoc","Polnoc","Poludnie"],"zywotnosc":[25,29,15,13,21,12,18,20,45,20,15,20,50],
"druzyna":["Kot","Kot","Kot","Kot","Kot","Kot","Kot","Ptak","Ptak","Ptak","Ptak","Ptak","Ptak"]}
```

### 2.6.5. Wczytywanie danych w formatach HTML i XML

```julia
using HTTP, Cascadia, Gumbo

```

### 2.6.6. Inne formaty plików tekstowych

```julia
using CSV

url = download("http://biecek.pl/R/dane/daneFWF.txt")
dane = CSV.File(url,header=false)
2-element CSV.File{false}:
 CSV.Row: (Column1 = "110 ALA STOP13",)
 CSV.Row: (Column1 = "111 OLA STOP 5",)
```

### 2.6.7. Wczytywanie danych z plików Excela

```julia
using XLSX, DataFrames

url = download("http://biecek.pl/R/dane/GUS_LUDN.xlsx")
xf = XLSX.readxlsx(url)
XLSXFile("jl_iSC9n6") containing 3 Worksheets
            sheetname size          range        
-------------------------------------------------
          DESCRIPTION 9x2           A1:B9        
                 DATA 28981x8       A1:H28981    
                PIVOT 27x1262       A1:AVN27
```

**Postać wąska**

```julia
m = XLSX.readdata(url,"DATA!A1:H28981");
ludnosc = DataFrame(m[Not(1),:], Symbol.(m[1,:]))
28980×8 DataFrame
   Row │ TERYT_CODE  TERYT_NAME           AGE    SEX      YEAR  MEASURE UNIT  VALUE    ⋯
       │ Any         Any                  Any    Any      Any   Any           Any      ⋯
───────┼────────────────────────────────────────────────────────────────────────────────
     1 │ 0000000000  POLAND               total  total    1995  person        38609399 ⋯
     2 │ 0000000000  POLAND               total  total    1996  person        38639341
     3 │ 0000000000  POLAND               total  total    1997  person        38659979
     4 │ 0000000000  POLAND               total  total    1998  person        38666983
     5 │ 0000000000  POLAND               total  total    1999  person        38263303 ⋯
     6 │ 0000000000  POLAND               total  total    2000  person        38253955
   ⋮   │     ⋮                ⋮             ⋮       ⋮      ⋮         ⋮           ⋮     
```

**Postać szeroka**

```julia
m = XLSX.readdata(url,"PIVOT!A1:AVN27");
ludnoscSzeroka = DataFrame(m[Not(1:4),:], Symbol.(m[4,:]), makeunique=true)
23×1262 DataFrame
 Row │ Row Labels  TERYT_NAME            1995      1996      1997      1998      1999  ⋯
     │ Any         Any                   Any       Any       Any       Any       Any   ⋯
─────┼──────────────────────────────────────────────────────────────────────────────────
   1 │ 0000000000  POLAND                38609399  38639341  38659979  38666983  38263 ⋯
   2 │ 1000000000  Central region        7747852   7741399   7737773   7730206   77500
   3 │ 1100000000  ŁÓDZKIE               2687761   2680350   2672823   2663608   26374
   4 │ 1140000000  MAZOWIECKIE           5060091   5061049   5064950   5066598   51126
   5 │ 2000000000  Southern region       8098116   8099977   8100860   8098333   79947 ⋯
   6 │ 2120000000  MAŁOPOLSKIE           3190186   3197064   3206630   3215885   32178

ludnoscSzeroka[!, Not(1:2)] = Float64.(ludnoscSzeroka[:,Not(1:2)]);
ludnoscSzeroka[!, 1:2] = String.(ludnoscSzeroka[:,1:2]);
first(ludnoscSzeroka,6)
6×1262 DataFrame
 Row │ Row Labels  TERYT_NAME       1995       1996       1997       1998       1999   ⋯
     │ String      String           Float64    Float64    Float64    Float64    Float6 ⋯
─────┼──────────────────────────────────────────────────────────────────────────────────
   1 │ 0000000000  POLAND           3.86094e7  3.86393e7  3.866e7    3.8667e7   3.8263 ⋯
   2 │ 1000000000  Central region   7.74785e6  7.7414e6   7.73777e6  7.73021e6  7.7500
   3 │ 1100000000  ŁÓDZKIE          2.68776e6  2.68035e6  2.67282e6  2.66361e6  2.6374
   4 │ 1140000000  MAZOWIECKIE      5.06009e6  5.06105e6  5.06495e6  5.0666e6   5.1126
   5 │ 2000000000  Southern region  8.09812e6  8.09998e6  8.10086e6  8.09833e6  7.9947 ⋯
   6 │ 2120000000  MAŁOPOLSKIE      3.19019e6  3.19706e6  3.20663e6  3.21588e6  3.2178
```

### 2.6.8. Wczytywanie danych z SPSS'a

```julia
using ReadStat, DataFrames

url = "http://www.biecek.pl/R/dane/daneSPSS.sav";
dane = read_sav(download(url));
df = DataFrame(dane.data,dane.headers)
11×4 DataFrame
 Row │ wiek       waga       diagnoza   urodzony   
     │ DataValu…  DataValu…  DataValu…  DataValu…  
─────┼─────────────────────────────────────────────
   1 │ 18.0       62.0       "zdrowy"   1.28431e10
   2 │ 19.0       66.0       "zdrowy"   1.27874e10
   3 │ 21.0       98.0       "chory"    1.2751e10
   4 │ 28.0       74.0       "chory"    1.25322e10
   5 │ 16.0       55.0       "chory"    1.28966e10
   6 │ 25.0       71.0       "zdrowy"   1.26006e10
   7 │ 27.0       70.0       "zdrowy"   1.25446e10
   8 │ 21.0       59.0       "chory"    1.27361e10
   9 │ 24.0       65.0       "zdrowy"   1.26371e10
  10 │ 24.0       86.0       "chory"    1.2637e10
  11 │ 20.0       82.0       "zdrowy"   1.27784e10
```

### 2.6.9. Wczytywanie danych z programu Matlab

```julia
using MAT

url = "http://biecek.pl/R/dane/daneMatlab.MAT";
file = matopen(download(url));
daneMat = read(file)
Dict{String, Any} with 2 entries:
  "wykladniczy" => [0.539925 1.56509 … 2.11158 1.46147; 0.859211 0.968062 … 0.796834 1.…
  "normalny"    => [0.135175 1.84822 … 2.37265 0.388083; -0.13904 -0.275106 … 0.229288 …
daneMat["normalny"]
10×10 Matrix{Float64}:
  0.135175    1.84822     -0.722271    0.636345  …   1.93746    2.37265    0.388083
 -0.13904    -0.275106    -0.72149     1.31008       1.63507    0.229288   0.393059
 -1.1634      2.21255     -0.201181    0.327098     -1.25594   -0.266623  -1.70733
  1.18372     1.50853     -0.0204642  -0.672993     -0.213538   0.701672   0.227859
 -0.0154297  -1.94508      0.27889    -0.149327     -0.198932  -0.48759    0.685633
  0.536219   -1.68054      1.05829    -2.44902   …   0.307499   1.86248   -0.63679
 -0.716429   -0.573534     0.621673    0.473286     -0.572325   1.10685   -1.00261
 -0.655559   -0.185817    -1.75062     0.116946     -0.977648  -1.22757   -0.185621
  0.314363    0.00893412   0.697348   -0.591104     -0.446809  -0.669885  -1.05403
  0.106814    0.83695      0.811486   -0.654708      1.08209    1.34093   -0.0715395
```

### 2.6.10. Wczytywanie danych z SAS

```julia
using ReadStat, DataFrames

url = "http://biecek.pl/R/dane/daneSAS.sas7bdat";
d = read_sas7bdat(download(url));
df = DataFrame(d.data,d.headers)
20×4 DataFrame
 Row │ NRPACJENTA  GRUPA       PYTANIE2   PYTANIE1  
     │ DataValue…  DataValue…  DataValu…  DataValu… 
─────┼──────────────────────────────────────────────
   1 │ "1"         "A"         2.0        2.0
   2 │ "2"         "A"         1.0        1.0
   3 │ "3"         "A"         1.0        1.0
   4 │ "4"         "A"         1.0        1.0
   5 │ "5"         "A"         1.0        2.0
   6 │ "6"         "A"         2.0        1.0
  ⋮  │     ⋮           ⋮           ⋮          ⋮
  16 │ "16"        "C"         1.0        1.0
  17 │ "17"        "C"         2.0        2.0
  18 │ "18"        "C"         2.0        2.0
  19 │ "19"        "C"         2.0        1.0
  20 │ "20"        "C"         2.0        2.0
                                      9 rows omitted
```




