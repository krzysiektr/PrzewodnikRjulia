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
t = scrape_tables(url);
d = t[69];
nam = map(chomp,Tables.getcolumn(d, 2));
col = [map(chomp,Tables.getcolumn(d, 1)[i]) for i in 1:32];
v = vcat(col[5:32]);
D = DataFrame(v,:auto);
rename!(D,nam[18:45]);
D.miesiac = convert.(String, nam[2:13]);
H = stack(D,1:29);
H.value = convert.(String, H.value);
rename!(H,[:variable => :rok]);
H.c = tryparse.(Float64, H.value) .!== nothing;
H.z = ifelse.(H.c .== true, H.value, "NaN");
H.value = map(x -> parse(Float64,x),H.z);
df = select(H, Not([:c,:z]));
df = filter(row -> row.rok != "miesiac", df);
df.miesiac = repeat(nam[2:13], inner = 1, outer = 28);
dfu = unstack(df, :miesiac, :value);
first(dfu,6)
6×13 DataFrame
 Row │ rok     I         II        III       IV        V         VI        VII       VIII   ⋯
     │ String  Float64?  Float64?  Float64?  Float64?  Float64?  Float64?  Float64?  Float6 ⋯
─────┼───────────────────────────────────────────────────────────────────────────────────────
   1 │ 1994       NaN        24.0      24.0      28.0      27.0      32.0      32.0     NaN ⋯
   2 │ 1995       NaN        32.0     NaN        34.0      36.0      29.0      32.0      28
   3 │ 1996        35.0      37.0     NaN        40.0      42.0     NaN        50.0      55
   4 │ 1997       NaN        56.0     NaN        52.0      57.0      57.0      51.0      53
   5 │ 1998       NaN        60.0      61.0      55.0      54.0     NaN        48.0      40 ⋯
   6 │ 1999        29.0      28.0      27.0      30.0      29.0      23.0      26.0      27

first(df,6)
6×3 DataFrame
 Row │ rok     value    miesiac   
     │ String  Float64  SubStrin… 
─────┼────────────────────────────
   1 │ 1994      NaN    I
   2 │ 1994       24.0  II
   3 │ 1994       24.0  III
   4 │ 1994       28.0  IV
   5 │ 1994       27.0  V
   6 │ 1994       32.0  VI

keep = map(!isnan, df.value)
first(df[keep,:],6)
6×3 DataFrame
 Row │ rok     value    miesiac   
     │ String  Float64  SubStrin… 
─────┼────────────────────────────
   1 │ 1994       24.0  II
   2 │ 1994       24.0  III
   3 │ 1994       28.0  IV
   4 │ 1994       27.0  V
   5 │ 1994       32.0  VI
   6 │ 1994       32.0  VII
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
