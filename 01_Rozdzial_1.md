---
layout: page
mathjax: true
title:  "Rozdział 1"
date:   2021-06-01 20:18:11 +0200
---

## 1.2. Jak wygląda praca z programem R

```julia
chop("Co jest supeR?", head = 12, tail = 1)
"R"
```

### 1.2.1. Przykład: Pozycja Polski w rankingu FIFA

```julia
using TableScraper, Tables, DataFrames

url = "https://pl.wikipedia.org/wiki/Reprezentacja_Polski_w_pi%C5%82ce_no%C5%BCnej_m%C4%99%C5%BCczyzn"
t = scrape_tables(url)
d = t[86];
nam = map(chomp,Tables.getcolumn(d, 2));
col = [map(chomp,Tables.getcolumn(d, 1)[i]) for i in 1:32];
a = [vcat(col[i],repeat(["-"],12-length(col[i]))) for i in 1:4];
v = vcat(a,col[5:32]);
D = DataFrame(v,:auto);
rename!(D,nam[14:45]);
D.miesiac = convert.(String, nam[2:13]);
H = stack(D,1:32);
H.value = convert.(String, H.value);
rename!(H,[:variable => :rok]);
H.c = tryparse.(Float64, H.value) .!== nothing;
H.z = ifelse.(H.c .== true, H.value, "NaN");
H.value = map(x -> parse(Float64,x),H.z);
df = select(H, Not([:c,:z]));
dfu = unstack(df, :miesiac, :value);
first(dfu,6)
6×13 DataFrame
 Row │ rok     I         II        III       IV        V         VI        VII     ⋯
     │ String  Float64?  Float64?  Float64?  Float64?  Float64?  Float64?  Float64 ⋯
─────┼──────────────────────────────────────────────────────────────────────────────
   1 │ 1990         NaN      22.0     NaN       NaN       NaN       NaN       NaN  ⋯
   2 │ 1991         NaN      22.0     NaN       NaN       NaN       NaN       NaN
   3 │ 1992         NaN      20.0     NaN       NaN       NaN       NaN       NaN
   4 │ 1993         NaN      20.0      22.0      23.0      26.0      28.0     NaN
   5 │ 1994         NaN      24.0      24.0      28.0      27.0      32.0      32. ⋯
   6 │ 1995         NaN      32.0     NaN        34.0      36.0      29.0      32.

keep = map(!isnan, df.value)
first(df[keep,:],6)
6×3 DataFrame
 Row │ miesiac  rok     value   
     │ String   String  Float64 
─────┼──────────────────────────
   1 │ II       1990       22.0
   2 │ II       1991       22.0
   3 │ II       1992       20.0
   4 │ II       1993       20.0
   5 │ III      1993       22.0
   6 │ IV       1993       23.0
```

```julia
using Gadfly, Cairo

p1 = Gadfly.plot(df[keep,:], x=:rok, y=:value, Geom.boxplot,
                 Guide.xlabel("Data publikacji rankingu"),
                 Guide.ylabel("Pozycja"),
                 Guide.title("Pozycja Polski w rankingu FIFA"));
draw(PNG("/home/krz/Przewodnik_Julia/assets/boxPlot01.png", 7inch, 5inch), p1)
```

<figure>
<center>
<img alt="png" src="/assets/boxPlot01.png">
</center>
<figcaption><b>Rysunek 1.</b> Wykres pudełkowy. </figcaption>
</figure>
