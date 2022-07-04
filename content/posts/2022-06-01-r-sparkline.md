---
title: R 中使用 sparkline（迷你图）的笔记
author: yuanfan
date: '2022-06-01T21:05:06+0800'
slug: r-sparkline
categories:
  - R
tags:
  - R
draft: no
---

<font face="微软雅黑">这篇笔记一共有两章，第一章记录 sparkline 包中可改动的参数细节，第二章记录将 sparkline 与 formattable 包、DT 包、reactable包结合使用的案例，其中一部分数据操作分 tidy 版本和 data.table 版本。

<!--more-->

据笔者所知，R 中现在有两个包可以画 sparkline（迷你图）：

+ 一个是[基于dataui包](https://timelyportfolio.github.io/dataui/articles/dataui_replicate_examples.html)的 reactablefmtr 包，其[提供的案例](https://kcuilla.github.io/reactablefmtr/articles/sparklines.html)有相当多的细节，若要应用直接看官网案例即可，不需要单独鼓捣一篇笔记出来；

+ 二个是 Rstudio 他们家出的 sparkline 包，此包的[文档](https://cran.r-project.org/web/packages/sparkline/sparkline.pdf)仅有7页，有[简洁的网站案例](https://cran.r-project.org/web/packages/sparkline/vignettes/intro_sparkline.html)，最近一次更新是2016年11月12日，不过由于这个包就是[jQuery Sparklines](https://omnipotent.net/jquery.sparkline/#s-about)的 R 版本，有些细节需要对照两种文档来看，所以笔者决定还是鼓捣一篇笔记出来。

# 一、sparkline 包的参数细节

这一章记录了sparkline包可以画的图形基本类型，如'line'（面积图/折线图）、'bar'（柱形图/堆叠柱形图）、'tristate'、'discrete'、'bullet'、'box'（箱线图）、'pie'（饼图），还记录了一些可以改动的图形细节，如图形区域的高度和宽度、颜色、标注点和区域等。

## 1.1. 迷你图的基本类型

首先，编造一些用于展示迷你图的基础数据。

```{r}
library(htmlwidgets)
library(sparkline)
library(dplyr) # tidy版操作数据
library(tibble) #使用tibble类型的数据
library(purrr) #使用 map_chr 函数
library(data.table) # data.table版操作数据
library(formattable)
library(DT)
library(reactable)

set.seed(1234)
x = rnorm(10)
y = c(rnorm(10), rep(0, 3))
z = c(rep(1, 3))
u = data.frame(serie1 = c(1:4), serie2 = c(4:1)) #堆叠柱形图的数据
u = as.matrix(u) # 转换成矩阵
```

为了方便显示代码，下表中“代码”一列中均增加了`eval = FALSE`这一参数。

![](https://yuanfan.vercel.app/images/2022/2022-06-01-1.png)

运用 sparkline 包中的`spk_composite()`函数可以将两个不同的迷你图一起展示。

```{r}
# 第一种写法
s1 <- sparkline(abs(x), type = 'bar')
s2 <- sparkline(y, type = "line", fillColor = FALSE)
spk_composite(s1, s2)

# 第二种写法
spk_composite(sparkline(abs(x), type = 'bar'),
              sparkline(y, type = "line", fillColor = FALSE))
```

![](https://yuanfan.vercel.app/images/2022/2022-06-01-2.png)

## 1.2. 迷你图的图形细节

原 jQuery Sparklines 的文档中详细介绍了迷你图的[通用参数](https://omnipotent.net/jquery.sparkline/#common)和[基于不同图形类型的个性参数](https://omnipotent.net/jquery.sparkline/#line)。这一小节仅记录部分细节。

### 1.2.1. 图形区域的高度和宽度

`sparkline()`函数中默认宽度和高度是`width = 60, height = 20`。

![](https://yuanfan.vercel.app/images/2022/2022-06-01-3.png)

宽度`width`对柱形图和 'tristate' 不起作用，需要通过修改`barWidth`（单个柱子的宽度）和`barSpacing`（柱子之间的空隙）来达到改变整个迷你图宽度的效果。

![](https://yuanfan.vercel.app/images/2022/2022-06-01-4.png)

### 1.2.2. 颜色

+ 一般折线图的颜色设置
    - 'lineColor'：指定线的颜色
    - 'fillColor'：指定折线下面积的填充颜色，设定为`fillColor = FALSE`时表示不展示折线下的面积。

![](https://yuanfan.vercel.app/images/2022/2022-06-01-5.png)

+ 柱形图的颜色设置
    - 'barColor'：指定普通柱子的颜色，通常就是指数值为正数的柱子
    - 'negBarColor'：指定数值为负数的柱子的颜色
    - 'zeroColor'：指定数值为零的柱子的颜色

![](https://yuanfan.vercel.app/images/2022/2022-06-01-6.png)

+ 堆叠柱形图的颜色设置
    - 'stackedBarColor'：指定堆叠的柱形图中柱子的颜色，如果是只堆叠了两个不同的数据系列，那么只需要指定两个不同的颜色。

![](https://yuanfan.vercel.app/images/2022/2022-06-01-7.png)

+ 饼图颜色设置
    - 'sliceColors'：指定饼图中各个扇形的颜色。

![](https://yuanfan.vercel.app/images/2022/2022-06-01-8.png)

### 1.2.3. 坐标轴

+ Y 轴最大、最小值
    - 'chartRangeMin'：指定直角坐标系下 Y 轴最大值
    - 'chartRangeMax'：指定直角坐标系下 Y 轴最小值

本文案例中'x'最大值为1.23，最小值为-2.42，因此若指定Y轴最小值为-5时，横轴之下的柱子都会变矮，同理若指定 Y 轴最大值为2时，横轴之上的柱子也会变矮。

![](https://yuanfan.vercel.app/images/2022/2022-06-01-9.png)

+ 旋转角度
    - 'offset'：指定饼图的旋转角度，取值范围是[-90,90]

![](https://yuanfan.vercel.app/images/2022/2022-06-01-10.png)

### 1.2.4. 标注点和区域

+ 标注点
    - 'spotColor'：折线图默认标记最大值和最小值点，设置`spotColor=FALSE`时表示不标注
    - 'spotRadius'：标记点的半径,默认值为1.5
    - 'minSpotColor'：指定标记的最小值的点的颜色
    - 'maxSpotColor'：指定标记的最大值的点的颜色

![](https://yuanfan.vercel.app/images/2022/2022-06-01-11.png)

+ 标注区域
    - `normalRangeMin` 和`normalRangeMax`这两个参数需要同时设置，只设置一个会不起作用。

![](https://yuanfan.vercel.app/images/2022/2022-06-01-12.png)

# 二、在表格中使用 sparkline

这一节记录了在 formattable 包和 DT 包中使用 sparkline 的案例，需使用 sparkline 包中的`spk_add_deps()`和`spk_chr()`函数。

## 2.1. 直接展示

一般来讲一个表格中的一个单元格里仅展示一个数值，如果一个单元格里要显示一个迷你图，那么迷你图中需要写入一串数据。若直接在表格中增加 展示迷你图，那么就需要在原来用于展示的数据框中写入列表，作为迷你图的那一列中的每一行数据都需要单独写入。这一小节的案例中使用的数据如下：

```{r}
set.seed(1234)
x = rnorm(10)

table1 <-
  tibble(
    column1 = c('坂田银时', '神乐', '志村新八', '定春'), # 第一列
    column2 = c(100, 10000, 10, 100),                 # 第二列
    column3 = c(1:4), # 第三列
    sparkline = list( # 第四列
      v1 = x,      # 第四列第一行
      v2 = abs(x), # 第四列第二行
      v3 = x,      # 第四列第三行
      v4 = abs(x)  # 第四列第四行
    )
  )
```

### 2.1.1. 与 formattable 包结合

```{r}
table1.formattable <- as.htmlwidget(formattable(
  data.frame(
    column1 = c('坂田银时', '神乐', '志村新八', '定春'), # 第一列
    column2 = c(100, 10000, 10, 100),                    # 第二列
    column3 = c(1:4), # 第三列
    sparkline = c(    # 第四列
      spk_chr(x, type = 'line'),       # 第四列第一行
      spk_chr(abs(x), type = 'line'),  # 第四列第二行
      spk_chr(x, type = 'bar'),        # 第四列第三行
      spk_chr(abs(x), type = 'bar')    # 第四列第四行
    ),
    stringsAsFactors = FALSE)))

spk_add_deps(table1.formattable)
```

![](https://yuanfan.vercel.app/images/2022/2022-06-01-13.png)

### 2.1.2. 与 DT 包结合

```{r}
table1.dt <- table1

table1.dt$sparkline <- table1.dt$sparkline %>%
  map(~ sparkline(.x, type = "box")) %>%
  map(htmltools::as.tags) %>%
  map_chr(as.character)

DT::datatable(table1.dt, escape = FALSE) %>% spk_add_deps()
```

![](https://yuanfan.vercel.app/images/2022/2022-06-01-14.png)

### 2.1.3. 与 reactable 结合

在写入需要展示迷你图的那列数据时，有两种写法。

+ 写法一：

写法是`columns = list( 列名 = colDef(cell = function(values){sparkline(values)}) )`。

```{r}
reactable(table1,
          columns = list(sparkline = colDef(
            cell = function(values) {
              sparkline(values,
                        type = "line")
            }
          )))
```

+ 写法二

写法是`columns = list( 列名 = colDef( cell = function(value, index){sparkline( 表名$列名[[index] )}))`。

```{r}
reactable(table1,
          columns = list(sparkline = colDef(
            cell = function(value, index) {
              sparkline(table1$sparkline[[index]], type = "line")
            }
          )))
```

![](https://yuanfan.vercel.app/images/2022/2022-06-01-15.png)

+ 写法3

reactable 包的案例中，还有一种[特别的写法](https://glin.github.io/reactable/articles/examples.html#footers)。

```{r}
reactable(
  iris[1:20, ],
  defaultPageSize = 5,
  bordered = TRUE,
  defaultColDef = colDef(footer = function(values) {
    if (!is.numeric(values)) return()
    sparkline(values, type = "box", width = 100, height = 30)
  })
)
```

![](https://yuanfan.vercel.app/images/2022/2022-06-01-16.png)

## 2.2. 分组聚合后展示

若是用于表格展示的数据需要先进行一些分组聚合的操作，而后又需要增加展示迷你图，那么……这一小节的案例中使用的数据如下：

```{r}
table2 <- data.frame(
  column1 = c(rep(c('坂田银时', '神乐', '志村新八', '定春'), 10)),
  column2 = sample(1:4, 40, replace = T),
  column3 = sample(10:20, 40, replace = T))
```

### 2.2.1. 与 formattable 结合

+ tidy 版本

```{r}
table2 %>%
  group_by(column1) %>%
  summarise(
    column2_mean = mean(column2),
    sparkline1 = spk_chr(column2, type = 'line'),
    sparkline2 = spk_chr(column3, type = 'bar')
  ) %>%
  formattable() %>%
  formattable::as.htmlwidget() %>%
  spk_add_deps()
```

![](https://yuanfan.vercel.app/images/2022/2022-06-01-17.png)

+ data.table 版本

```{r}
table2.dt <- as.data.table(table2)

table2.dt[, by = .(column1), .(
  column2_mean = mean(column2),
  sparkline1 = spk_chr(column2, type = 'line'),
  sparkline2 = spk_chr(column3, type = 'bar')
)] |>
  formattable() |>
  formattable::as.htmlwidget() |>
  spk_add_deps()
```

### 2.2.2. 与 DT 结合

+ tidy 版本

```{r}
table2.tidy <- table2 %>%
  group_by(column1) %>%
  summarise(column2_mean = mean(column2))

table2.tidy$sparkline1 <- table2$column2 %>%
  split(table2$column1) %>%
  map( ~ sparkline(.x, type = "line")) %>%
  map(htmltools::as.tags) %>%
  map_chr(as.character)

table2.tidy$sparkline2 <- table2$column3 %>%
  split(table2$column1) %>%
  map( ~ sparkline(.x, type = "bar")) %>%
  map(htmltools::as.tags) %>%
  map_chr(as.character)

DT::datatable(table2.tidy, escape = FALSE) %>% spk_add_deps()
```

![](https://yuanfan.vercel.app/images/2022/2022-06-01-18.png)

+ data.table 版本

下面的写法来源于统计之都论坛上的[一个帖子](<https://d.cosx.org/d/423198-tidy-datatable>)。不熟 data.table 包的时候，可以使用`dtplyr::lazy_dt()`把tidy版本转换成data.table版本，然后用`dtplyr:::dt_call()`提取出具体的代码。

```{r}
table2.dt.DT <- table2.dt[, .(
  column2_mean = mean(column2),
  sparkline1 = as.character(htmltools::as.tags(sparkline(column2, type = "line"))),
  sparkline2 = as.character(htmltools::as.tags(sparkline(column2, type = "bar")))
), keyby = .(column1)]

DT::datatable(table2.dt.DT, escape = FALSE) |> spk_add_deps()
```

### 2.2.3. 与reactable 结合

+ tidy 版本

```{r}
table2.tidy.react <- table2 %>%
  group_by(column1) %>%
  summarise(
    column2_mean = mean(column2),
    sparkline1 = list(column2),
    sparkline2 = list(column3)
  )

reactable(table2.tidy.react,
          columns = list(
            sparkline1 = colDef(
              cell = function(values) {
                sparkline(values,
                          type = "line")
              }
            ),
            sparkline2 = colDef(
              cell = function(value, index) {
                sparkline(table2.tidy.react$sparkline2[[index]], type = "bar")
              }
            )
          ))
```

![](https://yuanfan.vercel.app/images/2022/2022-06-01-19.png)

+ data.table 版本

```{r}
table2.dt.react <-
  table2.dt[, .(
    column2_mean = mean(column2),
    sparkline1 = list(column2),
    sparkline2 = list(column3)), keyby = .(column1)]

reactable(table2.dt.react,
          columns = list(
            sparkline1 = colDef(
              cell = function(values) {
                sparkline(values,
                          type = "line")
              }
            ),
            sparkline2 = colDef(
              cell = function(value, index) {
                sparkline(table2.dt.react$sparkline2[[index]], type = "bar")
              }
            )
          ))
```

## 2.3. 把 tidy 代码转换成 data.table 代码

举个栗子：

```{r}
library(dtplyr)
a <- lazy_dt(table2) %>%
  group_by(column1) %>%
  summarise(
    column2_mean = mean(column2),
    sparkline1 = list(column2),
    sparkline2 = list(column3)
  )

dtplyr:::dt_call(a)
```

得到：

```
`_DT2`[, .(column2_mean = mean(column2), sparkline1 = list(column2), 
    sparkline2 = list(column3)), keyby = .(column1)]
```

环境信息：

```
> sessionInfo()
R version 4.2.0 (2022-04-22 ucrt)
Platform: x86_64-w64-mingw32/x64 (64-bit)
Running under: Windows 10 x64 (build 19043)

Matrix products: default

locale:
[1] LC_COLLATE=Chinese (Simplified)_China.utf8  LC_CTYPE=Chinese (Simplified)_China.utf8    LC_MONETARY=Chinese (Simplified)_China.utf8
[4] LC_NUMERIC=C                                LC_TIME=Chinese (Simplified)_China.utf8    

attached base packages:
[1] stats     graphics  grDevices utils     datasets  methods   base     

other attached packages:
 [1] dtplyr_1.2.1      reactable_0.3.0   DT_0.23           formattable_0.2.1 data.table_1.14.2 purrr_0.3.4       tibble_3.1.6     
 [8] dplyr_1.0.8       sparkline_2.0     htmlwidgets_1.5.4

loaded via a namespace (and not attached):
 [1] bslib_0.3.1      jquerylib_0.1.4  pillar_1.7.0     compiler_4.2.0   tools_4.2.0      digest_0.6.28    jsonlite_1.8.0   evaluate_0.14   
 [9] lifecycle_1.0.1  pkgconfig_2.0.3  rlang_1.0.2      cli_3.2.0        DBI_1.1.2        rstudioapi_0.13  crosstalk_1.2.0  yaml_2.2.1      
[17] blogdown_1.5     xfun_0.26        fastmap_1.1.0    reactR_0.4.4     knitr_1.36       sass_0.4.0       generics_0.1.2   vctrs_0.3.8     
[25] tidyselect_1.1.2 glue_1.6.2       R6_2.5.1         fansi_1.0.2      rmarkdown_2.11   bookdown_0.24    magrittr_2.0.2   ellipsis_0.3.2  
[33] htmltools_0.5.2  assertthat_0.2.1 utf8_1.2.2       crayon_1.5.0    
```
