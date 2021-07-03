---
layout: page
mathjax: true
title:  "Rozdział 5"
date:   2021-06-18 20:18:11 +0200
---

## 5.1. Znajdź siedem różnic

```julia
using Gadfly, Cairo, Plots, RCall

kidney = rcopy(R"PBImisc::kidney");
p1 = scatter(kidney.MDRD7, kidney.MDRD12)
savefig("/home/.../assets/p51a.png")
p2 = Gadfly.plot(kidney, x=:MDRD7, y=:MDRD12,  Geom.point)
draw(PNG("/home/.../assets/p51b.png", 4inch, 5inch), p2)
```
![](/assets/p51a.png)
![](/assets/p51b.png)

### 5.3.2. Szablony dla wykresów

```julia
p = Gadfly.plot(kidney, xgroup=:discrepancy_DR, x=:MDRD7, y=:MDRD12,
                Geom.subplot_grid(layer(Geom.point,alpha = [0.3])))
draw(PNG("/home/.../assets/p532a.png", 7inch, 3.5inch), p)
```
![](/assets/p532a.png)

```julia
p = Gadfly.plot(kidney, xgroup=:discrepancy_DR, x=:MDRD7, y=:MDRD12,
                Geom.subplot_grid(
                 layer(Geom.point,alpha = [0.3]),
                 layer(Geom.smooth(method=:loess,smoothing=0.9),
                  color=[colorant"red"],style(line_width=0.5mm), order=2),
                 layer(Stat.smooth(method=:lm,levels=[0.9]),
                  color=[colorant"blue"],Geom.line,Geom.ribbon,order=1)
                ))
draw(PNG("/home/.../assets/p532b.png", 7inch, 3.5inch), p)
```
![](/assets/p532b.png)

### 5.3.4. Panele i mechanizm warunkowania

```julia
using CategoricalArrays, DataFrames

kidneyO = transform(kidney, :donor_age => (x -> cut(x,4)) => :cut_age);
kidneyO.cut_age = string.(kidneyO.cut_age);
select!(kidneyO, :cut_age => ByRow(x -> split(x,":")) => [:Q, :przedzial],Not(:cut_age));
p = Gadfly.plot(kidneyO, xgroup=:Q, x=:MDRD12,
                       Geom.subplot_grid(
                        layer(Geom.histogram(bincount=5),alpha = [0.3])
                       ))
draw(PNG("/home/.../assets/p534.png", 7inch, 3.5inch), p)
```
![](/assets/p534.png)


### 5.3.5. Mechanizm grupowania

```julia
p = Gadfly.plot(kidney, color=:therapy, x=:MDRD12, Geom.density,
                Theme(key_position = :none))
draw(PNG("/home/.../assets/p535.png", 7inch, 3.5inch), p)
```
![](/assets/p535.png)

### 5.3.6. Legenda wykresu

```julia
using Compose

p = Gadfly.plot(kidney, color=:therapy, x=:MDRD12, Geom.density,
                Guide.colorkey(title="Therapy",labels=["TC","CM","CA"],pos=[0.05w,-0.28h]),
                layer(x=:MDRD12, y=zeros(nrow(kidney)), Geom.point, alpha=[0.4], color=[colorant"blue"]))
draw(PNG("/home/.../assets/p536.png", 7inch, 3.5inch), p)
```
![](/assets/p536.png)

#### 5.3.7.1. Wykres kropkowy, funkcja: xyplot(), splom()

```julia
kidney.discrepancy_DR01 = replace!(kidney.discrepancy_DR, 2 => 1)
p = Gadfly.plot(kidney, xgroup=:discrepancy_DR,Geom.subplot_grid(
           layer(x=:MDRD7, y=:MDRD12, Geom.point,color=[colorant"red"],alpha = [0.3],
                 Geom.smooth(method=:loess,smoothing=0.9),order=1),
           layer(x=:MDRD7, y=:MDRD36, Geom.point,color=[colorant"blue"],alpha = [0.3],
                 Geom.smooth(method=:loess,smoothing=0.9),order=2)))
draw(PNG("/home/.../assets/p5371a.png", 7inch, 3.5inch), p)
```
![](/assets/p5371a.png)


```julia
using StatsPlots

cornerplot(Matrix(kidney[:,[9,11,13,15]]),label = ["MDRD$i" for i=[7,3,12,36]],size=(500,300))
savefig("/home/.../assets/p5371b.png")
```
![](/assets/p5371b.png)

#### 5.3.7.2. Wykres pudełkowy i paskowy, funkcja: stripplot(), bwplot()

```julia
using CategoricalArrays, DataFrames

kidney1 = transform(kidney, :discrepancy_AB => (x -> cut(x,3)) => :cut_AB);
kidney1.cut_age = string.(kidney1.cut_AB);
select!(kidney1, :cut_AB => ByRow(x -> split(x,":")) => [:Q, :przedzial],Not(:cut_AB));

p = Gadfly.plot(kidney1, xgroup=:Q, y=:MDRD12, x=:therapy,
                Geom.subplot_grid(
                 layer(Geom.boxplot(suppress_outliers=false))
                ))
draw(PNG("/home/.../assets/p5372a.png", 7inch, 3.5inch), p)
```
![](/assets/p5372a.png)

#### 5.3.7.3. Wykres kropkowy i paskowy, funkcja: dotplot(), barchart()

```julia
using FreqTables

daneSoc = rcopy(R"Przewodnik::daneSoc");
D = DataFrame(wyksztalcenie=String.(daneSoc.wyksztalcenie),plec=String.(daneSoc.plec));
wPlec = freqtable(D.wyksztalcenie,D.plec)
4×2 Named Matrix{Int64}
Dim1 ╲ Dim2 │   kobieta  mezczyzna
────────────┼─────────────────────
podstawowe  │        22         71
srednie     │        16         39
wyzsze      │        10         24
zawodowe    │         7         15
```
```julia
gdf = groupby(daneSoc, [:wyksztalcenie,:plec]);
df1 = combine(gdf, nrow)
8×3 DataFrame
 Row │ wyksztalcenie  plec          nrow  
     │ Categorical…   Categorical…  Int64 
─────┼────────────────────────────────────
   1 │ podstawowe     kobieta          22
   2 │ podstawowe     mezczyzna        71
   3 │ srednie        kobieta          16
   4 │ srednie        mezczyzna        39
   5 │ wyzsze         kobieta          10
   6 │ wyzsze         mezczyzna        24
   7 │ zawodowe       kobieta           7
   8 │ zawodowe       mezczyzna        15
```

```julia
p = Gadfly.plot(df1,x=:nrow,y=:wyksztalcenie,xgroup=:plec,
           Geom.subplot_grid(
                layer(Geom.point,
                      Geom.bar(orientation = :horizontal))),Theme(bar_spacing=15mm))
draw(PNG("/home/.../assets/p5373a.png", 7inch, 3.5inch), p)
```
![](/assets/p5373a.png)

```julia
p = Gadfly.plot(df1,x=:nrow,y=:wyksztalcenie,color=:plec,Geom.point,Geom.line)
draw(PNG("/home/.../assets/p5373b.png", 7inch, 3.5inch), p)
```
![](/assets/p5373b.png)

#### 5.3.7.4. Wykres profili, funkcja: parallel()

```julia
df2 = kidney[:,9:16];
co(x) = ifelse(x < 30, :red, :blue);
c = co.(df2.MDRD7);
d = [(col .- mean(col))/std(col) for col = eachcol(df2)];
d = DataFrame(d,Symbol.(names(kidney)[9:16]));
d.c = c;

fg = Plots.plot();
for i in 1:nrow(d)
  Plots.plot!(fg,1:1:8,collect(d[i,Not(:c)]),color=d[i,:c],legend=false,alpha=0.4)
end
xticks!([1:1:8;],names(kidney)[9:16])
Plots.plot!(fg, size=(700,300))
savefig("/home/.../assets/p5374.png")
```
![](/assets/p5374.png)

#### 5.3.7.5. Wykresy dystrybuanty empirycznej i gęstości, funkcje: histogram(), density(), qq(), qqmath()

```julia
p = Gadfly.plot(kidney, x=:MDRD7, xgroup=:therapy,
                Geom.subplot_grid(
                     layer(Geom.histogram(bincount=5),alpha = [0.3])
                ))
draw(PNG("/home/.../assets/p5375a.png", 7inch, 3.5inch), p)
```
![](/assets/p5375a.png)


```julia
using StatsBase

k0 = select(kidney, [:MDRD7, :diabetes, :therapy]);
k1 = sort(k0, [:diabetes, :therapy], rev=(false, false));
k2 = groupby(k1, [:diabetes, :therapy]);
k3 = combine(k2, nrow, [:MDRD7 => (x -> [ecdf(x)(x)]) => :cf]);
m = [repeat([string(k3.therapy[i])], inner = k3.nrow[i]) for i in 1:6 ];

D = DataFrame(
 Diabetes = Int.(vcat(zeros(sum(k3.nrow[1:3])), ones(sum(k3.nrow[4:6])))),
 Therapy = collect(Iterators.flatten(m)),
 MDRD7_cf = collect(Iterators.flatten(k3.cf)),
 MDRD7 = k1.MDRD7);

p = Gadfly.plot(D, x=:MDRD7, y=:MDRD7_cf, color=:Therapy, xgroup=:Diabetes,
                Geom.subplot_grid(
                 layer(Geom.step())
                ))
draw(PNG("/home/.../assets/p5375b.png", 7inch, 3.5inch), p)
```
![](/assets/p5375b.png)

```julia
k1 = sort(kidney, [:diabetes, :therapy], rev=(false, false));
p = Gadfly.plot(k1, x=:MDRD7, color=:therapy, xgroup=:diabetes,
                Geom.subplot_grid(
                 layer(Geom.density())
                ))
draw(PNG("/home/.../assets/p5375c.png", 7inch, 3.5inch), p)
```
![](/assets/p5375c.png)

```julia
using Distributions

p = Gadfly.plot(kidney, y=:MDRD7, x=Normal(), xgroup=:diabetes, color=:therapy,
                Geom.subplot_grid(
                     layer(Stat.qq, Geom.point)
                ))
draw(PNG("/home/.../assets/p5375d.png", 7inch, 3.5inch), p)
```
![](/assets/p5375d.png)

```julia
k0 = select(kidney, [:MDRD7, :diabetes, :therapy]);
k1 = sort(k0, [:diabetes, :therapy], rev=(false, false));
k2 = groupby(k1, [:diabetes, :therapy]);
k3 = combine(k2, nrow, [:MDRD7 => (x -> [quantile(x,0.01:0.01:0.99)]) => :qf]);
m = [repeat([string(k3.therapy[i])], inner = length(0.01:0.01:0.99)) for i in 1:6 ];

D = DataFrame(
 Diabetes = Int.(vcat(zeros(99*3), ones(99*3))),
 Therapy = collect(Iterators.flatten(m)),
 MDRD7_qf = collect(Iterators.flatten(k3.qf)))

D0 = filter(x -> x.Diabetes == 0, D); rename!(D0,:MDRD7_qf => :MDRD7_qf_0);
D1 = filter(x -> x.Diabetes == 1, D); rename!(D1,:MDRD7_qf => :MDRD7_qf_1);
DF = select!(D0, Not(:Diabetes));
DF.MDRD7_qf_1 = D1.MDRD7_qf_1;

p = Gadfly.plot(DF, y=:MDRD7_qf_1, x=:MDRD7_qf_0, xgroup=:Therapy, intercept=[0], slope=[1],
                Geom.subplot_grid(
                 layer(Geom.point),
                 layer(Geom.abline(color="red"),order=2)
               ))
draw(PNG("/home/.../assets/p5375e.png", 7inch, 3.5inch), p)
```
![](/assets/p5375e.png)

#### 5.3.7.6. Wykresy trójwymiarowe, funkcje: cloud(), levelplot(), contourplot()

```julia
kid0 = filter(:diabetes => x -> x == 0, kidney);
kid1 = filter(:diabetes => x -> x == 1, kidney);
p1 = scatter(kid0.MDRD30,kid0.MDRD12,kid0.MDRD7,legend=false,
     title="diabetes 0",xlabel="MDRD30",ylabel="MDRD12",zlabel="MDRD7");
p2 = scatter(kid1.MDRD30,kid1.MDRD12,kid1.MDRD7,legend=false,
     title="diabetes 1",xlabel="MDRD30",ylabel="MDRD12",zlabel="MDRD7");
plot(p1, p2, layout = grid(1, 2), size=(700,300));
savefig("/home/.../assets/p5376a.png")
```
![](/assets/p5376a.png)

```julia
using GRUtils, KernelDensity

k7 = kidney.MDRD7;
k30 = kidney.MDRD30;
z = kde((k30, k7));

fig = Figure((600,250), "px")
subplot(1, 2, 1)
GRUtils.heatmap(z.density)
subplot(1, 2, 2)
GRUtils.contour(z.x, z.y, z.density)
GRUtils.savefig("/home/.../assets/p5376b.png",fig)
```
![](/assets/p5376b.png)

```julia
using GRUtils

z = kde((k7, k30));

fig = Figure((600,250), "px")
subplot(1, 2, 1)
GRUtils.surface(z.x, z.y, z.density, colorbar=false)
subplot(1, 2, 2)
GRUtils.wireframe(z.x, z.y, z.density)
GRUtils.savefig("/home/.../assets/p5376c.png",fig)
```
![](/assets/p5376c.png)

#### 5.3.8.1. Uwolnione osie

```julia
using RDatasets
iris = dataset("datasets", "iris")

p = Gadfly.plot(iris, xgroup=:Species, x=:SepalLength, y=:PetalLength, 
                 Geom.subplot_grid(Geom.point, free_y_axis=true,free_x_axis=true))
draw(PNG("/home/.../assets/p5381.png", 7inch, 3.5inch), p)
```
![](/assets/p5381.png)

#### 5.3.8.2. Modyfikacja funkcji rysującej panele

```julia
gdf = groupby(daneSoc, [:wyksztalcenie,:plec,:praca]);
df1 = combine(gdf, nrow);

p = Gadfly.plot(df1, xgroup=:plec, y=:wyksztalcenie,x=:nrow,color=:praca, 
                 Geom.subplot_grid(
                  Geom.bar(position=:dodge, orientation=:horizontal)),
                 style(bar_spacing=10pt))
draw(PNG("/home/.../assets/p5382.png", 7inch, 3.5inch), p)
```
![](/assets/p5382.png)

#### 5.3.10.1. Pozycjonowanie wykresu, wiele wykresów na rysunku

```julia
l = @layout [a{0.5w} [grid(1,1)
                      grid(1,2)]
            ]

Plots.plot(scatter(kidney.MDRD12, kidney.MDRD7,leg=false), scatter(kidney.MDRD12, kidney.MDRD7,leg=false),
           scatter(kidney.MDRD12, kidney.MDRD7,leg=false), scatter(kidney.MDRD12, kidney.MDRD7,leg=false),
           layout = l,size=(700,600))
savefig("/home/.../assets/p53101a.png")
```
![](/assets/p53101a.png)

```julia
p = Plots.scatter(kidney.MDRD12,kidney.MDRD7,
                  inset = (1, bbox(0.0, 0.0, 0.5, 0.5, :right)))
p1 = Plots.scatter!(p[2],kidney.MDRD12,kidney.MDRD7,color=:red,leg=false)

Plots.scatter(kidney.MDRD12,kidney.MDRD7,leg=false)
p2 = lens!([30,60],[0,20],framestyle=:box,
           inset=(1, bbox(0.0, 0.0, 0.5, 0.5, :right)))

Plots.plot(p1, p2, layout = grid(1, 2), size=(700,300));
savefig("/home/.../assets/p53101b.png")
```
![](/assets/p53101b.png)

#### 5.3.10.2. Proporcje jednostek na osiach i reguła 45 stopni

```julia
sunspot_year = rcopy(R"datasets::sunspot.year");
p1 = Plots.plot(1:140,sunspot_year[1:140],aspectratio=0.1,leg=false)
p2 = Plots.plot(141:280,sunspot_year[141:280],aspectratio=0.1,leg=false)
Plots.plot(p2, p1, layout = grid(2, 1), size=(700,300));
savefig("/home/.../assets/p53102a.png")
```
![](/assets/p53102a.png)

```julia
p = Gadfly.plot(kidney, x=:MDRD12, y=:MDRD7, Geom.point,
                Coord.cartesian(yflip=true, aspect_ratio=2.0))
draw(PNG("/home/.../assets/p53102b.png", 7inch, 3.5inch), p)
```
![](/assets/p53102b.png)

### 5.4.1. Wprowadzenie

```julia
countries = rcopy(R"SmarterPoland::countries");
first(countries,6)
×5 DataFrame
 Row │ country              birth_rate  death_rate  population  continent 
     │ String               Float64?    Float64     Float64     String    
─────┼────────────────────────────────────────────────────────────────────
   1 │ Afghanistan                34.1         7.7     30552.0  Asia
   2 │ Albania                    12.9         9.4      3173.0  Europe
   3 │ Algeria                    24.3         5.7     39208.0  Africa
   4 │ Andorra                     8.9         8.4        79.0  Europe
   5 │ Angola                     44.1        13.9     21472.0  Africa
   6 │ Antigua and Barbuda        16.5         6.8        90.0  Americas
```

### 5.4.2. Warstwy wykresu

```julia
p1 = Gadfly.plot(dropmissing(countries), x= :birth_rate, y= :death_rate, Geom.blank);
p2 = Gadfly.plot(dropmissing(countries), x= :birth_rate, y= :death_rate, Geom.point);
p3 = Gadfly.plot(dropmissing(countries), x= :birth_rate, y= :death_rate, Geom.point,
                       layer(Geom.smooth(method=:loess,smoothing=0.9),
                             color=[colorant"red"],order=1,style(line_width=1mm)));
p = gridstack([p1 p2 p3])
draw(PNG("/home/.../assets/p542a.png", 7inch, 3inch), p)
```
![](/assets/p542a.png)

```julia
using Colors

p = Gadfly.plot(dropmissing(countries), x= :continent, y= :birth_rate,
                layer(Geom.boxplot,style(default_color=RGBA(.8, .5, .8, .3),boxplot_spacing=30pt)),
                layer(Geom.violin,style(default_color=RGBA(255, .0, .0, .3)),order=1),
                layer(Geom.beeswarm,style(default_color=RGBA(.1, .1, .1, .5)),order=2),
                layer(Geom.label(position=:right),label=1),
                Coord.cartesian(ymin=0, ymax=55),
                Scale.x_discrete(levels=levels(countries.continent)))
draw(PNG("/home/.../assets/p542b.png", 7inch, 7inch), p)
```
![](/assets/p542b.png)

### 5.4.3. Mapowanie zmiennych na atrybuty wykresu

```julia
sizemap(p::Float64; min=0.75mm, max=1.5mm) = min + p*(max-min)
p = Gadfly.plot(layer(countries, x= :birth_rate, y= :death_rate, color= :continent,
                      size= :population,alpha= [0.4], Geom.point),
                Scale.size_area(sizemap,minvalue=1,maxvalue=1e+05))
draw(PNG("/home/.../assets/p543a.png", 7inch, 4inch), p)
```
![](/assets/p543a.png)

```julia
p = Gadfly.plot(dropmissing(countries),x=:birth_rate, y=:death_rate, color=:birth_rate, alpha=[0.4],
                Scale.color_continuous(colormap=p->RGB(0,p,0)))
draw(PNG("/home/.../assets/p543b.png", 7inch, 4inch), p)
```
![](/assets/p543b.png)

### 5.4.4. Geometria warstwy

```julia
p1 = Gadfly.plot(dropmissing(countries), x= :continent, y= :birth_rate,color=:continent,
                 Geom.beeswarm);
p2 = Gadfly.plot(dropmissing(countries), x= :continent, y= :birth_rate,
                 Geom.boxplot);
p3 = Gadfly.plot(dropmissing(countries), x= :continent, y= :birth_rate,
                 Geom.violin);
p4 = Gadfly.plot(dropmissing(countries), x= :birth_rate, y= :death_rate,
                 Geom.point);
p = gridstack([p1 p2; p3 p4])
draw(PNG("/home/.../assets/p544.png", 7inch, 7inch), p)
```
![](/assets/p544.png)

### 5.4.5. Statystyki i agregacje

```julia
p1 = Gadfly.plot(countries, x=:continent,
                 Geom.histogram,
                 style(bar_spacing=2mm),Scale.x_discrete(levels=levels(countries.continent)));
p2 = Gadfly.plot(dropmissing(countries), x=:birth_rate, y=:death_rate,
                 Geom.point,
                 layer(Geom.smooth(method=:loess,smoothing=0.9),
                       color=[colorant"red"],style(line_width=0.5mm),order=2),
                 layer(Stat.smooth(method=:lm, levels=[0.95]), Geom.line, Geom.ribbon,
                       color=[colorant"green"],style(line_width=1mm),order=1));
p = gridstack([p1 p2])
draw(PNG("/home/.../assets/p545.png", 7inch, 3inch), p)
```
![](/assets/p545.png)

### 5.4.6. Mechanizm warunkowania

```julia
p = Gadfly.plot(dropmissing(countries), x=:birth_rate, y=:death_rate,xgroup=:continent,
                Geom.subplot_grid(
                layer(Geom.point,alpha = [0.5]), layer(Geom.ellipse)
                ));
draw(PNG("/home/.../assets/p546a.png", 7.5inch, 4inch), p)
```
![](/assets/p546a.png)

```julia
c = rcopy(R"SmarterPoland::countries");
colr = [occursin.(u[i],c.continent) for i in 1:5];
df_asia = DataFrame(birth_rate=c.birth_rate, death_rate=c.death_rate, colr=colr[1]);
df_asia.continent = repeat([u[1]], inner = nrow(df_asia));
df_europe = DataFrame(birth_rate=c.birth_rate, death_rate=c.death_rate, colr=colr[2]);
df_europe.continent = repeat([u[2]], inner = nrow(df_europe));
df_africa = DataFrame(birth_rate=c.birth_rate, death_rate=c.death_rate, colr=colr[3]);
df_africa.continent = repeat([u[3]], inner = nrow(df_africa));
df_america = DataFrame(birth_rate=c.birth_rate, death_rate=c.death_rate, colr=colr[4]);
df_america.continent = repeat([u[4]], inner = nrow(df_america));
df_oceania = DataFrame(birth_rate=c.birth_rate, death_rate=c.death_rate, colr=colr[5]);
df_oceania.continent = repeat([u[5]], inner = nrow(df_oceania));
df = vcat(df_asia,df_europe,df_africa,df_america,df_oceania);
```

```julia
p = Gadfly.plot(dropmissing(df), x=:birth_rate, y=:death_rate, xgroup=:continent,
                group=:continent, color=:colr,
                Geom.subplot_grid(
                layer(Geom.point,alpha = [0.5]), layer(Geom.ellipse)));
draw(PNG("/home/.../assets/p546b.png", 7.5inch, 4inch), p)
```
![](/assets/p546b.png)

### 5.4.7. Kontrola skal

```julia
p1 = Gadfly.plot(countries, x=:birth_rate, y=:death_rate, shape=:continent,
                 Theme(discrete_highlight_color=x->"red", default_color="white"));
LETTERS = replace(countries.continent, "Africa"=>"A", "Americas"=>"B", "Asia"=>"C",
                                       "Europe"=>"D", "Oceania"=>"E");
p2 = Gadfly.plot(countries, x=:birth_rate, y=:death_rate,
                 Geom.label(position=:centered), label= LETTERS,
                 Guide.manual_color_key("Legend", ["A: Africa", "B: America",
                                        "C: Asia","D: Europe","E: Oceania"]));
p = gridstack([p1 p2]);
draw(PNG("/home/.../assets/p547a.png", 7.5inch, 3inch), p)
```
![](/assets/p547a.png)


```julia
p1 = Gadfly.plot(countries, x=:birth_rate, y=:death_rate, Geom.point,
                 Guide.xticks(ticks=[1,2,5,10,50]));
p2 = Gadfly.plot(countries, x=:birth_rate, y=:death_rate, Geom.point,
                 Coord.cartesian(yflip=true, xflip=true));
p = gridstack([p1 p2]);
draw(PNG("/home/.../assets/p547b.png", 7inch, 3inch), p)
```
![](/assets/p547b.png)

### 5.4.8. Układ współrzędnych i osie wykresu

```julia
p1 = Gadfly.plot(dropmissing(countries), x=:birth_rate, y=:death_rate, Geom.point,
                 layer(Geom.smooth(method=:loess,smoothing=0.9),
                       color=[colorant"red"],order=1,style(line_width=1mm)),
                 Coord.cartesian(fixed=true));
p2 = Gadfly.plot(dropmissing(countries), y=:birth_rate, x=:death_rate, Geom.point,
                 layer(Geom.smooth(method=:loess,smoothing=0.9),
                       color=[colorant"red"],order=1,style(line_width=1mm)));
p = gridstack([p1 p2]);
draw(PNG("/home/.../assets/p548.png", 7inch, 3inch), p)
```
![](/assets/p548.png)


### 5.4.9. Motywy i kompozycje graficzne

```julia
p1 = Gadfly.plot(dropmissing(countries), x=:birth_rate, y=:death_rate, Geom.point,
                 layer(Geom.smooth(method=:loess,smoothing=0.9),
                       color=[colorant"red"],order=1,style(line_width=1mm)),
                 Theme(panel_stroke=colorant"white", grid_line_width=0mm));
p2 = Gadfly.plot(dropmissing(countries), x=:birth_rate, y=:death_rate, Geom.point,
                 layer(Geom.smooth(method=:loess,smoothing=0.9),
                       color=[colorant"red"],order=1,style(line_width=1mm)),
                 Theme(panel_fill="gray92",grid_color="white",grid_line_style=:solid));
p = gridstack([p1 p2]);
draw(PNG("/home/.../assets/p549.png", 7.5inch, 3inch), p)
```
![](/assets/p549.png)


### 5.4.10. Pozycjonowanie wykresów na rysunku

```julia
using Compose, DataFrames, Gadfly

p1 = Gadfly.plot(dropmissing(countries),x=:birth_rate, y=:death_rate, Geom.point,
    Theme(default_color="red", panel_fill="CornSilk"));
p2 = Gadfly.plot(dropmissing(countries),x=:birth_rate, y=:death_rate, Geom.point,
    Guide.annotation(compose(context(0w,0h,0.5w,1.0h), render(p1))));
draw(PNG("/home/.../assets/p5410.png", 7.5inch, 3inch), p2)
```
![](/assets/p5410.png)

### 5.5.1. Wprowadzenie

```julia
x = -2*pi:0.3:2*pi;
Plots.plot(x,sin.(x),leg=false,title="Wykres sin(x) i cos(x)",color=:red,size=(500,300));
Plots.scatter!(-2*pi:0.5:2*pi,sin.(-2*pi:0.5:2*pi),leg=false,color=:red);
Plots.plot!(x,cos.(x),leg=false,color=:blue);
savefig("/home/.../assets/p551a.png")
```
![](/assets/p551a.png)

```julia
x = collect(-2.25:0.3:2.25);
fg = Plots.plot();
for i in 1:10
  Plots.plot!(fg, x, x.*i, xlim=(-2,2),ylim=(-2,2),leg=false)
end
Plots.plot!(fg, size=(500,300));
Plots.abline!(0,-1,lw=3);
Plots.abline!(0,0);
Plots.plot!([-1,-1], [-2, 2], seriestype = :straightline, ls=:dash,lw=3);
annotate!(1.7,0.2,"a=0, b=0",10);
annotate!(1.7,1.1,"a=1, b=0",10);
annotate!(1.3,1.7,"a=2, b=0",10);
annotate!(1.7,-0.8,"h = -1",10);
annotate!(-0.6,1.1,"v = -1",10);
savefig("/home/.../assets/p551b.png")
```
![](/assets/p551b.png)



### 5.5.3. Funkcja `matplot()`

```julia
using Distributions, DataFrames

ruchBrowna = rand(Normal(),200,4);
ruchBrowna = [cumsum(i) for i in eachcol(ruchBrowna)];
Plots.plot(ruchBrowna, leg=false, size=(500,300));
savefig("/home/.../assets/p553.png")
```
![](/assets/p553.png)

### 5.5.4. Osie wykresu, funkcja: axis()

```julia
Plots.plot([x-> x, x-> x^2, x-> x^3, x-> x^4],
           bottom_margin= 5mm, top_margin= 10mm, right_margin= 10mm,
           xlim=(0,6), ylim=(0,9), legend_columns=2,
           xlabel="independent values", ylabel="dependent values")
annotate!(0, 10,"cos",10);
annotate!(5, -2.5,"cos",10);
Plots.xticks!(1:5,["przed","chwile przed","w trakcie","chwile po","po"],xrotation = 45)
Plots.plot!(twinx(),xticks = (1:1:6,["","","","","",""]),ylim=(4.5,8.5),ylabel="value axis",
            xmirror=:true,grid=:false)
savefig("/home/.../assets/p554.png")
```
![](/assets/p554.png)

### 5.5.6. Funkcja `image()`

```julia
using GRUtils

function plusk(x,y,s1,s2)
    return sin(sqrt((x-s1)^2+(y-s2)^2)/4)/(abs(x-s1)+abs(y-s2)+25)
end
x = reverse(collect(1:1:200));
y = collect(1:1:200);
mat1 = plusk.(x,y',100,50);
mat2 = plusk.(x,y',50,100);
mat3 = plusk.(x,y',20,20);
mat = mat1 + mat2 + mat3;
fig = Figure((800,600), "px")
GRUtils.imshow(mat1+mat2+mat3,[minimum(mat),maximum(mat)])
GRUtils.savefig("/home/.../assets/p556.png",fig)
```
![](/assets/p556.png)

### 5.5.7. Wrażenia matematyczne

```julia
using Plots, LaTeXStrings

Plots.plot([2,5,8,9,6,7], ratio=1, labels=L"f(x)=\frac{y^7}{x+6}");
annotate!([3],[4], Plots.text(L"f(x)=\frac{y^7}{x+6}", 11, :black, :center));
Plots.savefig("/home/.../assets/p557.png")
```
![](/assets/p557.png)

#### 5.5.8.1. Nazwy kolorów

```julia
Plots.plot(x->x,color=:red,size=(500,300))
savefig("/home/.../assets/p5581.png")
```
![](/assets/p5581.png)

#### 5.5.8.2. Palety kolorów

[https://docs.juliaplots.org/latest/generated/colorschemes/](https://docs.juliaplots.org/latest/generated/colorschemes/)

#### 5.5.8.3. Definiowanie własnych kolorów

[https://juliagraphics.github.io/Colors.jl/stable/](https://juliagraphics.github.io/Colors.jl/stable/)

### 5.5.9. Właściwości linii

```julia
fg = Plots.plot();
for i in 1:5
   e = [:solid, :dash, :dot, :dashdot, :dashdotdot]
   Plots.plot!(fg, x->x-i, lw=2, ls=e[i],label=string(e[i]),title="linestyle")
end
Plots.plot!(fg, size=(500,300))
savefig("/home/.../assets/p559a.png")
```
![](/assets/p559a.png)

```julia
Plots.scatter(collect(1:1:20),lt=:steppost, title="linetype", size=(500,300))
savefig("/home/.../assets/p559b.png")
```
![](/assets/p559b.png)

### 5.5.10. Właściwości punktów/symboli

```julia
fg = Plots.plot();
for i in 1:10
   e1 = [:circle, :rect, :star5, :diamond, :hexagon, :cross, :xcross, :utriangle, :dtriangle, :rtriangle]
   e2 = [:ltriangle, :pentagon, :heptagon, :octagon, :star4, :star6, :star7, :star8, :+, :x]
   Plots.plot!(fg, [i],[i+1], linecolor=false,markersize=10, markershape=e1[i],title="markershape",leg=false)
   Plots.plot!(fg, [i],[i+2], linecolor=false,markersize=10, markershape=e2[i],leg=false)
end
Plots.plot!(fg, size=(500,300))
savefig("/home/.../assets/p5510.png")
```
![](/assets/p5510.png)

### 5.5.11. Atomowe, niskopoziomowe funkcje graficzne

```julia
p1 = Plots.plot([0,1,3,1,5,3], fillrange = 0, fillalpha = 0.3,
                lw=3,ticks=nothing,framestyle=:box,leg=false);
p1 = Plots.plot!([0,1,2,3,2,1], fillrange = 0,fillalpha= 0.3,
                lw=3,ticks=nothing,framestyle=:box,leg=false);
p2 = Plots.plot([0,2], [0,1], arrow = 2, aspectratio=1,
                lw=3,ticks=nothing,framestyle=:box,leg=false);
p2 = Plots.plot!(sin, cos, 0, 2*pi, ls=:dash);
p2 = Plots.plot!(x-> 2*sin(x), x->3*cos(x), 0, 2*pi);
Plots.plot(p1, p2, layout = grid(1, 2), size=(700,300));
savefig("/home/.../assets/p5511.png")
```
![](/assets/p5511.png)

### 5.5.12. Tytuł, podtytuł i opisy osi wykresu

```julia
using Statistics

Plots.scatter(daneSoc.wiek,daneSoc.cisnienie_skurczowe,leg=:outertopright,
              xlabel="wiek", ylabel="ciśnienie skurczowe",
              title="Tytuł główny\nwykresu punktowego", top_margin= 12mm, bottom_margin= 2mm,
              size=(700,300));
TXT = ["Średnia ciśnienia skurczowego wynosi:", round(mean(daneSoc.cisnienie_skurczowe),digits=4)];
annotate!(50,190, Plots.text(join(TXT,' '), :red, :center, 8));
savefig("/home/.../assets/p5512.png")
```
![](/assets/p5512.png)

#### 5.5.13.1. Równomierna siatka wykresów

```julia
p = Plots.plot();
Plots.plot(p, p, p, layout = @layout [a _; b c])
savefig("/home/.../assets/p55131.png")
```
![](/assets/p55131.png)

#### 5.5.13.2. Nierównomierna siatka wykresów

```julia
p = Plots.plot();
Plots.plot(p, p, p, p, p, layout = @layout [grid(2,2) a{0.25w}]);
savefig("/home/.../assets/p55132.png")
```
![](/assets/p55132.png)

### 5.5.14. Wykres słonecznikowy

```julia
using Distributions, FreqTables

zm1= rand(Binomial(5,0.5),200); zm2 = rand(Binomial(5,0.5),200);
tab = freqtable(zm1, zm2);
x = string.(collect(0:1:5));
y = string.(collect(0:1:5));
z = Matrix(tab);
Plots.heatmap(x, y, z, levels=maximum(tab), c= :PuRd_9, size=(400,400));
annotate!( vec(tuple.((1:length(x))' .-0.5, (1:length(y)).-0.5, string.(z))) )
savefig("/home/.../assets/p5514.png")
```
![](/assets/p5514.png)

### 5.5.15. Wykresy kropkowe, dwu- i wielowymiarowe, funkcje: pairs, scatterplot.matrix(), gpairs(), coplot() i scatterplot3d()

```julia
corrplot(Matrix(daneSoc[:,[:wiek, :cisnienie_skurczowe, :cisnienie_rozkurczowe]]),
         bins=10,label=[names(daneSoc)[i] for i in [1,6,7]],size=(700,600))
savefig("/home/.../assets/p5515a.png")
```
![](/assets/p5515a.png)

### 5.5.16. Wykres macirzy korelacji, funkcja: plotcorr()

```julia
using Statistics

c = cor(Matrix(daneSoc[:,[:wiek,:cisnienie_skurczowe,:cisnienie_rozkurczowe]]));
cs = c[2,1];
cr = c[3,1];
cscr = c[3,2];
p1 = Plots.plot(x-> cs*sin(x),cos,0,2*pi,xlim=(-1,1),ylim=(-1,1),color="red",
                aspectratio=1,fillrange = 0,fillalpha= 0.3,label="wiek_cs");
p1 = annotate!(0,1.2, Plots.text(round(cs,digits=4), :red, :center, 8));
p2 = Plots.plot(x-> cr*sin(x),cos,0,2*pi,xlim=(-1,1),ylim=(-1,1),color="red",
                aspectratio=1,fillrange = 0,fillalpha= 0.3,label="wiek_cr");
p2 = annotate!(0,1.2, Plots.text(round(cr,digits=4), :red, :center, 8));
p3 = Plots.plot(x-> cscr*sin(x),cos,0,2*pi,xlim=(-1,1),ylim=(-1,1),
                aspectratio=1,fillrange = 0,fillalpha= 0.3,label="cs_cr");
p3 = annotate!(0,1.2, Plots.text(round(cscr,digits=4), :blue, :center, 8));
Plots.plot(p1, p2, p3, layout = (1, 3), size=(700,300));
savefig("/home/.../assets/p5516.png")
```
![](/assets/p5516.png)

### 5.5.17. Wykres konturowy dwuwymiarowego rozkładu normalnego, funkcja: data.ellipse()

```julia
using StatsPlots, Distributions

q = quantile(Normal(),[0.90,0.95,0.99]);
Plots.scatter(daneSoc.cisnienie_skurczowe,daneSoc.cisnienie_rozkurczowe,leg=:outertopright,
              xlabel="cisnienie skurczowe",ylabel="cisnienie rozkurczowe");
covellipse!([mean(daneSoc[:,6]),mean(daneSoc[:,7])], cov(Matrix(daneSoc[:,6:7])),
            n_std=q[1], aspect_ratio=1, showaxes =true,label="0.90");
covellipse!([mean(daneSoc[:,6]),mean(daneSoc[:,7])], cov(Matrix(daneSoc[:,6:7])),
            n_std=q[2], aspect_ratio=1, showaxes =true,label="0.95");
covellipse!([mean(daneSoc[:,6]),mean(daneSoc[:,7])], cov(Matrix(daneSoc[:,6:7])),
            n_std=q[3], aspect_ratio=1, showaxes =true,label="0.99");
savefig("/home/.../assets/p5517.png")
```
![](/assets/p5517.png)

### 5.5.19. Wielowymiarowy, jądrowy estymator gęstości, funkcje: kde() i kde2d

```julia
using GRUtils, KernelDensity

cs = daneSoc[:,6];
cr = daneSoc[:,7];
z = kde((cs, cr));
fig = Figure((600,250), "px")
subplot(1, 2, 1)
GRUtils.wireframe(z.x, z.y, z.density)
subplot(1, 2, 2)
GRUtils.surface(z.x, z.y, z.density)
GRUtils.savefig("/home/.../assets/p5519a.png",fig)
```
![](/assets/p5519a.png)

### 5.5.20. Wykresy konturowe, funkcje: contour(), filled.contour()

```julia
fig = Figure((600,250), "px")
subplot(1, 2, 1)
GRUtils.contour(z.x, z.y, z.density, colorbar = false)
subplot(1, 2, 2)
GRUtils.contourf(z.x, z.y, z.density, levels = 25, majorlevels = 1)
GRUtils.savefig("/home/.../assets/p5519b.png",fig)
```
![](/assets/p5519b.png)

### 5.5.21. Wykres mapa ciepła, funkcja: heatmap()

```julia
p1 = Plots.heatmap(z.x,z.y,z.density);
p2 = Plots.histogram2d(daneSoc[:,6],daneSoc[:,7],
                       nbins= (20, 40), show_empty_bins= true, normed= true);
Plots.plot(p1, p2, layout = grid(1, 2), size=(700,300));
savefig("/home/.../assets/p5521.png")
```
![](/assets/p5521.png)

### 5.5.22. Wykres profili obserwacji, funkcja: parcoord()

```julia
d1 = extrema(daneSoc.cisnienie_skurczowe);
d2 = extrema(daneSoc.cisnienie_rozkurczowe);
d3 = extrema(daneSoc.wiek);
e = daneSoc.cisnienie_skurczowe.*daneSoc.cisnienie_rozkurczowe .* daneSoc.wiek;
mytrace = parcoords(;line = attr(color=e),
               dimensions = [
                   attr(range = [d1[1],d1[2]],
                        label = "cisnienie_skurczowe", values = daneSoc.cisnienie_skurczowe),
                   attr(range = [d2[1],d2[2]],
                        label = "cisnienie_rozkurczowe", values = daneSoc.cisnienie_rozkurczowe),
                   attr(range = [d3[1],d3[2]],
                        label = "wiek", values = daneSoc.wiek)
               ]);
myplot = PlotlyJS.plot(mytrace)
PlotlyJS.savefig(myplot, "/home/.../assets/p5522.png")
```
![](/assets/p5522.png)

#### 5.6.2.1. Wykres liniowy

```julia
using RCall, DataFrames

przezycia = rcopy(R"Przewodnik::przezycia");
przezycia2009 = filter([:Year, :Age] => (x, y) -> x == 2009 && y != "110+", przezycia);
first(pozycja2009[:,[1,2,3,7,11]],6)
6×5 DataFrame
 Row │ Year   Age   mx       dx     Gender 
     │ Int64  Cat…  Float64  Int64  Cat…   
─────┼─────────────────────────────────────
   1 │  2009  0     0.0051     507  Female
   2 │  2009  1     0.00032     32  Female
   3 │  2009  2     0.00018     18  Female
   4 │  2009  3     0.00018     18  Female
   5 │  2009  4     0.00016     16  Female
   6 │  2009  5     9.0e-5       9  Female
```

```julia
df_F = filter([:Gender] => x -> x =="Female",przezycia2009);
df_M = filter([:Gender] => x -> x =="Male",przezycia2009);

using PlotlyJS

function linescatter1()
  Female = PlotlyJS.scatter(;x=df_F.Age, y=log.(df_F.mx), mode="lines+markers",name="Famale")
  Male = PlotlyJS.scatter(;x=df_M.Age, y=log.(df_M.mx), mode="lines+markers",name="Male")
  PlotlyJS.plot([Female,Male])
end
linescatter1()
```
![](/assets/p5621.png)


