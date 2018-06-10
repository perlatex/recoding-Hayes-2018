Chapter 07
================
A Solomon Kurz
2018-06-09

7.1 Conditional and unconditional effects
-----------------------------------------

Let's make a tibble with the data Hayes showed in Table 7.1.

``` r
library(tidyverse)

d <-
  tibble(x = rep(-1:2, each = 3),
         w = rep(0:2, times = 4)) %>% 
  mutate(A = 4 + 1*x + 2*w,
         B = 4 + 1*x + 2*w + 1.5*x*w)
```

In previous chapters, we've been altering our plot themes using either built-in settings from ggplot2 or extra themes from the ggplot package. And when we wanted to change them further, we did so with extra `theme()` arguments. One, it's nice to be know how to make one's own custom theme and, two, it'd be handy to condense our code a bit. Here we'll take a few cues from the [Building a New Theme](https://bookdown.org/rdpeng/RProgDA/building-a-new-theme.html) of Peng, Kross, and Anderson's [*Mastering Software Development in R*](https://bookdown.org/rdpeng/RProgDA/). We'll make our own theme, `theme_07` by saving a handful of augmentations from the default `theme_gray()` theme.

``` r
library(dutchmasters)

theme_07 <- 
  theme_gray() + 
  theme(plot.background = element_rect(fill = dutchmasters$little_street[7]),
        panel.background = element_rect(fill = dutchmasters$little_street[2]),
        strip.background = element_rect(fill = alpha(dutchmasters$little_street[5], 2/3),
                                        color = "transparent"),
        legend.background = element_rect(fill = "transparent"),
        legend.key = element_rect(fill = "transparent", color = "transparent"),
        panel.grid = element_blank(),
        text = element_text(family = "Courier",
                            color = dutchmasters$little_street[6],
                            size = 14),
        axis.text = element_text(color = dutchmasters$little_street[6]))
```

This chapter's color palette comes from the [dutchmasters package](https://github.com/EdwinTh/dutchmasters), which was itself based of [Vermeer's *The Little Street*](https://en.wikipedia.org/wiki/The_Little_Street). To get a quick glance at the full palette, we'll also use `viz_palette()`, a convenience function from the [ochRe package](https://github.com/ropenscilabs/ochRe).

``` r
library(ochRe)

viz_palette(dutchmasters$little_street)
```

![](Chapter_07_files/figure-markdown_github/unnamed-chunk-3-1.png)

With our new `theme_07` in hand, we're ready for our version of Figure 7.2.

``` r
library(directlabels)

d %>% 
  gather(key, value, -x, -w) %>% 
  rename(y = value) %>% 
  mutate(label = str_c("W = ", w)) %>% 
  
  ggplot(aes(x = x, y = y, group = w, size = w %>% as.factor(), label = label)) +
  geom_line(color = dutchmasters$little_street[9]) +
  geom_dl(method = list(dl.trans(x = x + 0.2),
                        "last.points", cex = 1,
                        color = dutchmasters$little_street[9],
                        fontfamily = "Courier")) +
  scale_size_manual(values = c(.5, .75, 1)) +
  coord_cartesian(xlim = c(-1, 2.6),
                  ylim = 0:16) +
  labs(x = expression(italic("X")),
       y = expression(italic("Y"))) +
  facet_wrap(~key) +
  theme_07 +
  # because we didn't alter the `legend.position` argument in our `theme_07`, we need to use `theme()` to remove it
  theme(legend.position = "none")
```

![](Chapter_07_files/figure-markdown_github/unnamed-chunk-4-1.png)

I played around with the annotation in Figure 7.4 for a while and it was just way more trouble than it was worth. If you're ever inspired to work it out, please share your code.

7.2 An example: Climate change disasters and humanitarianism
------------------------------------------------------------

Here we load a couple necessary packages, load the data, and take a `glimpse()`.

``` r
library(readr)

disaster <- read_csv("data/disaster/disaster.csv")

glimpse(disaster)
```

    ## Observations: 211
    ## Variables: 5
    ## $ id      <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25...
    ## $ frame   <int> 1, 1, 1, 1, 1, 0, 0, 1, 0, 0, 1, 1, 0, 0, 1, 1, 1, 1, 0, 0, 1, 0, 1, 0, 1, 1, 0, 0, 0, 1,...
    ## $ donate  <dbl> 5.6, 4.2, 4.2, 4.6, 3.0, 5.0, 4.8, 6.0, 4.2, 4.4, 5.8, 6.2, 6.0, 4.2, 4.4, 5.8, 5.4, 3.4,...
    ## $ justify <dbl> 2.95, 2.85, 3.00, 3.30, 5.00, 3.20, 2.90, 1.40, 3.25, 3.55, 1.55, 1.60, 1.65, 2.65, 3.15,...
    ## $ skeptic <dbl> 1.8, 5.2, 3.2, 1.0, 7.6, 4.2, 4.2, 1.2, 1.8, 8.8, 1.0, 5.4, 2.2, 3.6, 7.8, 1.6, 1.0, 6.4,...

Here is how to get the ungrouped *μ* and *SD* values for `justify` and `skeptic`, as presented in Table 7.3.

``` r
disaster %>% 
  select(justify, skeptic) %>% 
  gather() %>% 
  group_by(key) %>% 
  summarise(mean = mean(value),
            sd = sd(value)) %>% 
  mutate_if(is.double, round, digits = 3)
```

    ## # A tibble: 2 x 3
    ##   key      mean    sd
    ##   <chr>   <dbl> <dbl>
    ## 1 justify  2.87 0.930
    ## 2 skeptic  3.38 2.03

And here we get the same summary values, this time grouped by `frame`.

``` r
disaster %>% 
  select(frame, justify, skeptic) %>% 
  gather(key, value, -frame) %>% 
  group_by(frame, key) %>% 
  summarise(mean = mean(value),
            sd = sd(value)) %>% 
  mutate_if(is.double, round, digits = 3)
```

    ## # A tibble: 4 x 4
    ## # Groups: frame [2]
    ##   frame key      mean    sd
    ##   <int> <chr>   <dbl> <dbl>
    ## 1     0 justify  2.80 0.849
    ## 2     0 skeptic  3.34 2.04 
    ## 3     1 justify  2.94 1.01 
    ## 4     1 skeptic  3.42 2.03

Anticipating Table 7.4 on page 234, we're going to name this first model `model1`.

``` r
library(brms)

model1 <-
  brm(data = disaster, family = gaussian,
      justify ~ 1 + frame,
      chains = 4, cores = 4)
```

``` r
print(model1)
```

    ##  Family: gaussian 
    ##   Links: mu = identity; sigma = identity 
    ## Formula: justify ~ 1 + frame 
    ##    Data: disaster (Number of observations: 211) 
    ## Samples: 4 chains, each with iter = 2000; warmup = 1000; thin = 1;
    ##          total post-warmup samples = 4000
    ## 
    ## Population-Level Effects: 
    ##           Estimate Est.Error l-95% CI u-95% CI Eff.Sample Rhat
    ## Intercept     2.80      0.09     2.63     2.98       3537 1.00
    ## frame         0.13      0.13    -0.11     0.39       3529 1.00
    ## 
    ## Family Specific Parameters: 
    ##       Estimate Est.Error l-95% CI u-95% CI Eff.Sample Rhat
    ## sigma     0.94      0.04     0.85     1.03       4000 1.00
    ## 
    ## Samples were drawn using sampling(NUTS). For each parameter, Eff.Sample 
    ## is a crude measure of effective sample size, and Rhat is the potential 
    ## scale reduction factor on split chains (at convergence, Rhat = 1).

The 'Estimate' (i.e., posterior mean) of the 'Intercept' is the expected `justify` value for when `frame` is 0. The 'Estimate' for `frame` is the expected difference when `frame` is a 1. If all you care about is the posterior mean, you could do

``` r
fixef(model1)["Intercept", 1] + fixef(model1)["frame", 1]
```

    ## [1] 2.938304

which matches up nicely with the equation on page 233. But this wouldn't be very Bayesian of us. It'd be more satisfying if we had an expression of the uncertainty in the value. For that, we'll follow our usual practice of extracting the posterior samples, making nicely-named vectors, and a little summarizing.

``` r
post <-
  posterior_samples(model1) %>% 
  mutate(when_x_is_0 = b_Intercept,
         when_x_is_1 = b_Intercept + b_frame)

post %>% 
  select(when_x_is_0, when_x_is_1) %>% 
  gather() %>% 
  group_by(key) %>% 
  summarize(mean = mean(value),
            sd = sd(value)) %>% 
  mutate_if(is.double, round, digits = 3)  
```

    ## # A tibble: 2 x 3
    ##   key          mean     sd
    ##   <chr>       <dbl>  <dbl>
    ## 1 when_x_is_0  2.80 0.0890
    ## 2 when_x_is_1  2.94 0.0950

Hayes referenced a *t*-test and accompanying *p*-value in the lower part of page 233. We, of course, aren't going to do that. But we do have the 95% intervals in our `print()` output, above, which we can also look at like so.

``` r
posterior_interval(model1)["b_frame", ]
```

    ##       2.5%      97.5% 
    ## -0.1144081  0.3859816

And we can always plot.

``` r
post %>% 
  ggplot(aes(x = b_frame)) +
  geom_density(size = 0, fill = dutchmasters$little_street[1]) +
  geom_vline(xintercept = 0, color = dutchmasters$little_street[11]) +
  scale_x_continuous(breaks = c(-.3, 0, .6)) +
  scale_y_continuous(NULL, breaks = NULL) +
  theme_07 +
  theme(legend.position = "none")
```

![](Chapter_07_files/figure-markdown_github/unnamed-chunk-13-1.png)

We'll use the `update()` function to hastily fit `model2` and `model3`.

``` r
model2 <-
  update(model1, newdata = disaster,
         formula = justify ~ 1 + frame + skeptic,
         chains = 4, cores = 4)

model3 <-
  update(model1, newdata = disaster,
         formula = justify ~ 1 + frame + skeptic + frame:skeptic,
         chains = 4, cores = 4)
```

Note our use of the `frame:skeptic` syntax in `model3`. With that syntax we didn't need to make an interaction variable in the data by hand. The brms package just handled it for us. An alternative syntax would have been `frame*skeptic`. But if you really wanted to make the interaction variable by hand, you'd do this.

``` r
disaster <-
  disaster %>% 
  mutate(interaction_variable = frame*skeptic)
```

Once you have `interaction_variable` in the data, you'd specify a model formula within the `brm()` function like `formula = justify ~ 1 + frame + skeptic + interaction_variable`. I'm not going to do that, here, but you can play around yourself if so inclined.

Here are the quick and dirty coefficient summaries for our two new models.

``` r
posterior_summary(model2)
```

    ##                 Estimate  Est.Error         Q2.5        Q97.5
    ## b_Intercept    2.1339427 0.12544619    1.8844194    2.3701587
    ## b_frame        0.1182908 0.11538348   -0.1099828    0.3406844
    ## b_skeptic      0.2004139 0.02887388    0.1433050    0.2569628
    ## sigma          0.8416236 0.04156953    0.7650352    0.9279472
    ## lp__        -268.3667671 1.41760183 -272.0178294 -266.5991465

``` r
posterior_summary(model3)
```

    ##                     Estimate  Est.Error          Q2.5        Q97.5
    ## b_Intercept        2.4542883 0.14674556    2.16308063    2.7322484
    ## b_frame           -0.5654322 0.21130727   -0.97078116   -0.1508213
    ## b_skeptic          0.1040891 0.03769022    0.03003897    0.1777466
    ## b_frame:skeptic    0.2021718 0.05443582    0.09747472    0.3068062
    ## sigma              0.8169193 0.03963189    0.74305641    0.8972033
    ## lp__            -262.2391852 1.52026151 -265.89063788 -260.2020148

Just focusing on our primary model, `model3`, here's another way to look at the coefficients.

``` r
stanplot(model3) +
  theme_07
```

![](Chapter_07_files/figure-markdown_github/unnamed-chunk-17-1.png)

By default, `brms::stanplot()` makes coefficient plots which depict the parameters of a model by their posterior means (i.e., dots), 50% intervals (i.e., the thick lines), and 95% intervals (i.e., the thin lines). As `stanplot()` returns a ggplot2 object, one can customize the theme and so forth.

To get the *R*<sup>2</sup> values

``` r
# for each of the three models, we create a separare R2 tibble
R2_model1 <- 
  bayes_R2(model1,
           summary = F) %>% 
  as_tibble()

R2_model2 <- 
  bayes_R2(model2,
           summary = F) %>% 
  as_tibble()

R2_model3 <- 
  bayes_R2(model3,
           summary = F) %>% 
  as_tibble()

# here we combine them into one tibble, indexed by `model`
R2s <-
  R2_model1 %>% 
  bind_rows(R2_model2) %>% 
  bind_rows(R2_model3) %>% 
  mutate(model = rep(c("model1", "model2", "model3"), each = 4000))

# now we plot
R2s %>% 
  ggplot(aes(x = R2)) +
  geom_density(aes(fill = model), size = 0, alpha = 2/3) +
  scale_fill_manual(NULL, 
                    values = dutchmasters$little_street[c(3, 4, 8)] %>% as.character()) +
  scale_y_continuous(NULL, breaks = NULL) +
  xlab(expression(paste(italic(R)^2, " distribution"))) +
  coord_cartesian(xlim = 0:1) +
  theme_07
```

![](Chapter_07_files/figure-markdown_github/unnamed-chunk-18-1.png)

Here's the *R*<sup>2</sup> difference score for `model3` minus `model2`.

``` r
R2_model2 %>% 
  rename(model2 = R2) %>% 
  bind_cols(R2_model3) %>% 
  rename(model3 = R2) %>% 
  mutate(dif = model3 - model2) %>% 
  
  ggplot(aes(x = dif)) +
  geom_density(color = "transparent",
               fill = dutchmasters$little_street[9]) +
  scale_y_continuous(NULL, breaks = NULL) +
  labs(title = expression(paste("The ", Delta, italic(R)^2)),
       subtitle = "Like in the text, the posterior mean\nis about 0.05.",
       x = NULL) +
  theme_07
```

![](Chapter_07_files/figure-markdown_github/unnamed-chunk-19-1.png)

In addition to the *R*<sup>2</sup>, one can use information criteria to compare the models. Here we'll use the LOO to compare all three.

``` r
loo(model1, model2, model3)
```

    ##                  LOOIC    SE
    ## model1          572.26 23.94
    ## model2          529.47 22.39
    ## model3          517.76 21.69
    ## model1 - model2  42.80 16.03
    ## model1 - model3  54.51 18.87
    ## model2 - model3  11.71  8.32

The point estimate for both multivariable models were clearly lower than that for `model1`. The point estimate for the moderation model, `model3`, was within the double-digit range lower than that for `model2`, which typically suggests better fit. But notice how wide the standard error was. There's a lot of uncertainty, there. Hopefully this isn't surprising. Our *R*<sup>2</sup> difference was small and uncertain, too. We can also compare them with AIC-type model weighting, which you can learn more about [starting at this point in this lecture](https://www.youtube.com/watch?v=t0pRuy1_190) or [this related vignette for the loo package](https://cran.r-project.org/web/packages/loo/vignettes/loo2-weights.html). Here we'll keep things simple and weight with the LOO.

``` r
model_weights(model1, model2, model3,
              weights = "loo")
```

    ##       model1       model2       model3 
    ## 1.453390e-12 2.854477e-03 9.971455e-01

The model LOO weights put almost all the relative weight on `model3`. This doesn't mean `model3` is the "true model" or anything like that. It just suggests that it's the better of the three with respect to the data.

Here are the results of the equations in the second half of page 237.

``` r
post <- posterior_samples(model3)

post %>% 
  transmute(if_2   = b_frame + `b_frame:skeptic`*2,
            if_3.5 = b_frame + `b_frame:skeptic`*3.5,
            if_5   = b_frame + `b_frame:skeptic`*5) %>% 
  gather() %>% 
  group_by(key) %>% 
  summarise(mean = mean(value),
            sd = sd(value)) %>% 
  mutate_if(is.double, round, digits = 3)
```

    ## # A tibble: 3 x 3
    ##   key      mean    sd
    ##   <chr>   <dbl> <dbl>
    ## 1 if_2   -0.161 0.131
    ## 2 if_3.5  0.142 0.110
    ## 3 if_5    0.445 0.143

### Estimation using ~~PROCESS~~ brms \[of course\].

Similar to what Hayes advertised with PROCESS, with our `formula = justify ~ 1 + frame + skeptic + frame:skeptic` code in `model3`, we didn't need to hard code an interaction variable into the data. brms handled that for us.

### Variable scaling and the interpretation of *b*<sub>1</sub> and *b*<sub>2</sub>.

Making the mean-centered version of our *W* variable, `skeptic`, is a simple `mutate()` operation. We'll just call it `skeptic_c`.

``` r
disaster <-
  disaster %>% 
  mutate(skeptic_c = skeptic - mean(skeptic))
```

And here's how we might fit the model.

``` r
model4 <-
  update(model3, newdata = disaster,
         formula = justify ~ 1 + frame + skeptic_c + frame:skeptic_c,
         chains = 4, cores = 4)
```

Here are the summaries of our fixed effects.

``` r
fixef(model4)
```

    ##                  Estimate  Est.Error        Q2.5     Q97.5
    ## Intercept       2.8069265 0.07944121  2.65301962 2.9591367
    ## frame           0.1174013 0.11418119 -0.10368033 0.3423860
    ## skeptic_c       0.1044873 0.03852573  0.02869671 0.1801432
    ## frame:skeptic_c 0.2015344 0.05511640  0.09510732 0.3094105

Here are the *R*<sup>2</sup> distributions for `model3` and `model4`. They're the same within simulaiton error.

``` r
bayes_R2(model3) %>% round(digits = 3)
```

    ##    Estimate Est.Error  Q2.5 Q97.5
    ## R2    0.249     0.043 0.166 0.328

``` r
bayes_R2(model4) %>% round(digits = 3)
```

    ##    Estimate Est.Error  Q2.5 Q97.5
    ## R2    0.249     0.044 0.157 0.335

If you're bothered by the differences resulting from sampling variation, you might increase the number of HMC iterations from the 2000-per-chain default. Doing so might look something like this:

``` r
model3 <-
  update(model3,
         chains = 4, cores = 4, warmup = 1000, iter = 10000)

model4 <-
  update(model4,
         chains = 4, cores = 4, warmup = 1000, iter = 10000)
```

Before we fit `model5`, we'll recode `frame` to a -.5/.5 metric and name it `frame_.5`.

``` r
disaster <-
  disaster %>% 
  mutate(frame_.5 = ifelse(frame == 0, -.5, .5))
```

Time to fit `model5`.

``` r
model5 <-
  update(model4, newdata = disaster,
         formula = justify ~ 1 + frame_.5 + skeptic_c + frame_.5:skeptic_c,
         chains = 4, cores = 4)
```

Our posterior summaries match up nicely with the output in Hayes's Table 7.4.

``` r
fixef(model5)
```

    ##                     Estimate  Est.Error        Q2.5     Q97.5
    ## Intercept          2.8636222 0.05684572  2.75242518 2.9732676
    ## frame_.5           0.1193686 0.11418597 -0.10541694 0.3434482
    ## skeptic_c          0.2044579 0.02818482  0.14875028 0.2604025
    ## frame_.5:skeptic_c 0.2011999 0.05586154  0.08852791 0.3077850

7.3 Visualizing moderation
--------------------------

To get quick plots for the interaction effect in brms, you might use the `marginal_effects()` function.

``` r
marginal_effects(model3)
```

![](Chapter_07_files/figure-markdown_github/unnamed-chunk-31-1.png)![](Chapter_07_files/figure-markdown_github/unnamed-chunk-31-2.png)![](Chapter_07_files/figure-markdown_github/unnamed-chunk-31-3.png)

By default, `margional_effects()` will show three levels of the variable on the right side of the interaction term. The formula in `model3` was `justify ~ frame + skeptic + frame:skeptic`, with `frame:skeptic` as the interaction term and `skeptic` making up the right hand side of the term. The three levels of `skeptic` in the plot, above, are the mean +/− 1 *SD*. See the brms [reference manual](https://cran.r-project.org/web/packages/brms/brms.pdf) for details on `marginal_effects()`.

On page 244, Hayes discussed using the 16th, 50th, and 84th percentiles for the moderator variable. We can compute those with `quantile()`.

``` r
quantile(disaster$skeptic, probs = c(.16, .5, .84))
```

    ## 16% 50% 84% 
    ## 1.6 2.8 5.2

The first two columns in Hayes’s Table 7.5 contain the values he combined with the point estimates of his model to get the *Y*-hat column. The way we’ll push those values through `model3`’s posterior is with `brms::fitted()`. As a preparatory step, we'll put the predictor values in a data object, `nd`.

``` r
(
  nd <-
  tibble(frame = rep(0:1, times = 3),
         skeptic = rep(quantile(disaster$skeptic, 
                                probs = c(.16, .5, .84)),
                       each = 2))
  )
```

    ## # A tibble: 6 x 2
    ##   frame skeptic
    ##   <int>   <dbl>
    ## 1     0    1.60
    ## 2     1    1.60
    ## 3     0    2.80
    ## 4     1    2.80
    ## 5     0    5.20
    ## 6     1    5.20

Now we've go our `nd`, we'll get our posterior estimates for *Y* with `fitted()`.

``` r
fitted(model3, newdata = nd)
```

    ##      Estimate  Est.Error     Q2.5    Q97.5
    ## [1,] 2.620831 0.10084849 2.425703 2.816001
    ## [2,] 2.378874 0.10465945 2.178460 2.589464
    ## [3,] 2.745738 0.08012862 2.589389 2.906914
    ## [4,] 2.746387 0.08196656 2.590480 2.911158
    ## [5,] 2.995552 0.10580420 2.781416 3.205238
    ## [6,] 3.481413 0.10620407 3.277658 3.694771

When using the default `summary = TRUE` settings in `fitted()`, the function returns posterior means, *SD*s and 95% intervals for *Y* based on each row in the `nd` data we specified in the `newdata = nd` argument. You don't have to name your newdata `nd` or anything like that; it's just my convention.

Here' a quick plot of what those values imply.

``` r
fitted(model3, newdata = nd) %>% 
  as_tibble() %>% 
  bind_cols(nd) %>% 
  
  ggplot(aes(x = skeptic, y = Estimate)) +
  geom_ribbon(aes(ymin = Q2.5, ymax = Q97.5, fill = frame %>% as.character()),
              alpha = 1/3) +
  geom_line(aes(color = frame %>% as.character())) +
  scale_fill_manual("frame",
                    values = dutchmasters$little_street[c(10, 5)] %>% as.character()) +
  scale_color_manual("frame",
                     values = dutchmasters$little_street[c(10, 5)] %>% as.character()) +
  theme_07
```

![](Chapter_07_files/figure-markdown_github/unnamed-chunk-35-1.png)

That plot is okay, but we can do better.

In order to plot the model-implied effects across the full range of `skeptic` values presented in Figure 7.7, you need to change the range of those values in the `nd` data. Also, although the effect is subtle in the above example, 95% intervals often follow a bowtie shape. In order to insure the contours of that shape are smooth, it's often helpful to specify 30 or so evenly-spaced values in the variable on the x-axis, `skeptic` in this case. We'll employ the `seq()` function for that. And in order to use those 30 values for both levels of `frame`, we'll nest the `seq()` function within `rep()`. In addition, we add a few other flourishes to make our plot more closely resemble the one in the text.

Here's our Figure 7.7.

``` r
nd <-
  tibble(frame = rep(0:1, times = 30),
         skeptic = rep(seq(from = 0, to = 7, length.out = 30),
                       each = 2))

fitted(model3, newdata = nd) %>% 
  as_tibble() %>% 
  bind_cols(nd) %>% 
  
  ggplot(aes(x = skeptic, y = Estimate)) +
  geom_ribbon(aes(ymin = Q2.5, ymax = Q97.5, fill = frame %>% as.character()),
              alpha = 1/3) +
  geom_line(aes(color = frame %>% as.character())) +
  scale_fill_manual("frame",
                    values = dutchmasters$little_street[c(10, 5)] %>% as.character()) +
  scale_color_manual("frame",
                     values = dutchmasters$little_street[c(10, 5)] %>% as.character()) +
  scale_x_continuous(breaks = 1:6) +
  coord_cartesian(xlim = 1:6,
                  ylim = 2:4) +
  labs(x = expression(paste("Climate Change Skepticism (", italic("W"), ")")),
       y = "Strength of Justification\nfor Withholding Aid") +
  theme_07 +
  theme(legend.position = "top")
```

![](Chapter_07_files/figure-markdown_github/unnamed-chunk-36-1.png)

7.4 Probing an interaction
--------------------------

### The pick-a-point approach.

#### The pick-a-point approach ~~implimented by regression centering~~ working directly with the posterior.

Yes, if you wanted to use the regression centering approach, you could do that in brms. Just center the necessary variables in the way Hayes described in the text, refit the model, and `summarize()`. I suspect this would be particularly approachable for someone new to R and the ins and outs of data wrangling. But I'm going leave that as an exercise for the interested reader.

Now that we've already got a posterior for our model, we can just either algebraically manipulate the vectors yielded by `posterior_samples()` or push predictions through `fitted()`. To give a sense, we'll start off with the 16th percentile for `skeptic`. Recall we can get that with the `quantile()` function.

``` r
 quantile(disaster$skeptic, probs = .16)
```

    ## 16% 
    ## 1.6

Now we just need to feed that value and different values of `frame` into the posterior samples of the model coefficients. We then create a `difference` score for the model-implied estimates given `frame` is either 0 or 1 and then plot that `difference`.

``` r
post %>% 
  mutate(Y_given_frame_0_skeptic_1.6 = b_Intercept + b_frame*0 + b_skeptic*1.6 + `b_frame:skeptic`*0*1.6,
         Y_given_frame_1_skeptic_1.6 = b_Intercept + b_frame*1 + b_skeptic*1.6 + `b_frame:skeptic`*1*1.6) %>% 
  mutate(difference = Y_given_frame_1_skeptic_1.6 - Y_given_frame_0_skeptic_1.6) %>% 
  
  ggplot(aes(x = difference)) +
  geom_density(color = "transparent",
               fill = dutchmasters$little_street[9]) +
  geom_vline(xintercept = 0, color = dutchmasters$little_street[7], linetype = 2) +
  scale_y_continuous(NULL, breaks = NULL) +
  labs(subtitle = "The effect of frame on justify given\nskeptic = 1.6",
       x = NULL) +
  theme_07
```

![](Chapter_07_files/figure-markdown_github/unnamed-chunk-38-1.png)

Note how nicely that distribution corresponds to the output in the lower left corner of Hayes's Figure 7.8. If we wanted the values for other values of `skeptic` (e.g., 2.8 and 5.2 as in the text), we'd just rinse, wash, and repeat. A nice quality of this method is it requires you to work explicitly with the model formula. But it's also clunky if you want to do this over many values. The `fitted()` function offers an alternative.

Recall how the default `fitted()` settings are to return summaries of a model's *Y*-variable given values of the predictor variables. In the previous section, we put our prefered `frame` and `skeptic` values into a data object named `nd` and used the `newdata` argument to push those values through `fitted()`. Buy default, this yielded the typical posterior mean, *SD*s, and 95% intervals. However, if one sets `summary = F`, the output will differ. First. Let's revisit what `nd` looks like.

``` r
(
  nd <-
  tibble(frame = rep(0:1, times = 3),
         skeptic = rep(quantile(disaster$skeptic, 
                                probs = c(.16, .5, .84)),
                       each = 2)) %>% 
    # This will make our lives easier in just a bit
    arrange(frame)
  )
```

    ## # A tibble: 6 x 2
    ##   frame skeptic
    ##   <int>   <dbl>
    ## 1     0    1.60
    ## 2     0    2.80
    ## 3     0    5.20
    ## 4     1    1.60
    ## 5     1    2.80
    ## 6     1    5.20

Here's what happens when we use `summary = F`.

``` r
f_model3 <-
  fitted(model3, newdata = nd, summary = F)

f_model3 %>% str()
```

    ##  num [1:4000, 1:6] 2.57 2.54 2.62 2.63 2.39 ...

``` r
f_model3 %>% head()
```

    ##          [,1]     [,2]     [,3]     [,4]     [,5]     [,6]
    ## [1,] 2.566699 2.656277 2.835434 2.449028 2.847341 3.643967
    ## [2,] 2.543361 2.629886 2.802935 2.482540 2.879052 3.672075
    ## [3,] 2.618046 2.777318 3.095862 2.341696 2.704754 3.430868
    ## [4,] 2.626090 2.736231 2.956514 2.331483 2.732592 3.534810
    ## [5,] 2.390788 2.559394 2.896608 2.409737 2.808812 3.606963
    ## [6,] 2.669859 2.854335 3.223288 2.300155 2.691699 3.474786

With `summary = F`, `fitted()` returned a matrix of 4000 rows (i.e., one for each posterior iteration) and 6 vectors (i.e., one for each row in our `nd` data). So now instead of summary information, we have a full expression of the uncertainty in terms of 4000 draws. If you prefer working within the tidyverse and plotting with ggplot2, matrices aren't the most useful data type. Let's wrangle a bit.

``` r
f_model3 <-
  f_model3 %>% 
  as_tibble() %>% 
  gather() %>% 
  select(-key) %>% 
  # We multiply 4000 (i.e., the # of iterations) by 3 because there are 3 distinct `skeptic` values
  mutate(frame = rep(0:1, each = 4000*3),
         # Note how we have `rep()` nested within `rep()`.
         skeptic = rep(rep(quantile(disaster$skeptic, probs = c(.16, .5, .84)),
                           each = 4000),
                       # We repeate the first `rep()` output 2 times because ther are 2 values of `frame` we'd like them for
                       times = 2),
         # We need an iteration index, `iter`, in order to help with `spread()`, below.
         iter = rep(1:4000, times = 6)) %>% 
  spread(key = frame, value = value) %>% 
  mutate(difference = `1` - `0`,
         # This isnt' necessary, but will help with the facet labels
         skeptic = str_c("skeptic = ", skeptic)) 

f_model3 %>% head()
```

    ## # A tibble: 6 x 5
    ##   skeptic        iter   `0`   `1` difference
    ##   <chr>         <int> <dbl> <dbl>      <dbl>
    ## 1 skeptic = 1.6     1  2.57  2.45    -0.118 
    ## 2 skeptic = 1.6     2  2.54  2.48    -0.0608
    ## 3 skeptic = 1.6     3  2.62  2.34    -0.276 
    ## 4 skeptic = 1.6     4  2.63  2.33    -0.295 
    ## 5 skeptic = 1.6     5  2.39  2.41     0.0189
    ## 6 skeptic = 1.6     6  2.67  2.30    -0.370

And here's a plot of what we've done.

``` r
f_model3 %>% 
  ggplot(aes(x = difference)) +
  geom_density(color = "transparent",
               fill = dutchmasters$little_street[9]) +
  geom_vline(xintercept = 0, color = dutchmasters$little_street[7], linetype = 2) +
  scale_y_continuous(NULL, breaks = NULL) +
  labs(subtitle = "The effect of frame on justify given three different values of skeptic",
       x = NULL) +
  theme_07 +
  facet_wrap(~skeptic)
```

![](Chapter_07_files/figure-markdown_github/unnamed-chunk-42-1.png)

And if you prefered summary information instead of plots, you'd use `summarize()` as usual.

``` r
f_model3  %>% 
  group_by(skeptic) %>% 
  summarize(median = median(difference),
            ll = quantile(difference, probs = .025),
            ul = quantile(difference, probs = .975)) %>% 
  mutate_if(is.double, round, digits = 3)
```

    ## # A tibble: 3 x 4
    ##   skeptic         median     ll     ul
    ##   <chr>            <dbl>  <dbl>  <dbl>
    ## 1 skeptic = 1.6 -0.249   -0.519 0.0380
    ## 2 skeptic = 2.8  0.00100 -0.221 0.223 
    ## 3 skeptic = 5.2  0.485    0.194 0.784

### The Johnson-Neyman technique.

The JN technique generalizes this approach over many values of *W* (i.e., `skeptic` in this example) in order to get a sense of the trend and summarize regions of the trend in terms of *p*-value thresholds. Since we're emphasizing modeling and deemphasizing null-hypothesis testing in this project, I'll show a Bayesian version of the approach without the *p*-values.

#### Implementation in ~~PROCESS~~ brms.

Since Figure 7.9 had `skeptic` values ranging from 1 to 6 with ticks on the 0.5s, we’ll use a similar approach for our version. We’ll estimate posterior samples with `fitted()` for `skeptic` values ranging from .5 to 6.5, one for each 0.5—13 in total. But since we have to levels of `frame` (i.e., 0 and 1), that really gives us 26. And we don’t just want 26 summaries; we want full posterior distributions for each of those 26.

We've got a lot of moving parts in the code, below. To help make sure everything adds up, we'll save several important values as R objects.

``` r
iter             <- 4000  # this number comes from the total number of post-warmup posterior iterations from the `brm()` function
n_frame_values   <- 2     # there are 2 levels of `frame`, 0 and 1
n_skeptic_values <- 13    # we're choosing 13 in this example to follow some of the sensibilities in Figure 7.9. You'll see.

# as before, we'll make `nd` to feed in to `fitted()`
nd <-
  tibble(frame = rep(0:1, each = n_skeptic_values),
         skeptic = rep(seq(from = .5, to = 6.5, length.out = n_skeptic_values), 
                       times = n_frame_values))

# after the initial `fitted()` action, we need a few steps to wrangle the data into a useful format
f_model3 <-
  fitted(model3, newdata = nd, summary = F) %>% 
  as_tibble() %>% 
  gather() %>% 
  mutate(frame = rep(0:1, each = iter*n_skeptic_values),
         skeptic = rep(rep(seq(from = .5, to = 6.5, length.out = n_skeptic_values), 
                           each = iter),
                       times = n_frame_values)) %>% 
  select(-key) %>% 
  rename(estimate = value) %>% 
  mutate(iter = rep(1:iter, times = n_frame_values*n_skeptic_values)) %>% 
  spread(key = frame, value = estimate) %>% 
  mutate(difference = `1` - `0`)

# finally, here's the plot
f_model3 %>% 
  ggplot(aes(x = skeptic %>% as.character, y = difference)) +
  geom_hline(yintercept = 0, color = dutchmasters$little_street[7]) +
  geom_violin(size = 0, fill = dutchmasters$little_street[6]) +
  stat_summary(fun.y = median,
               fun.ymin = function(x){quantile(x, probs = .025)},
               fun.ymax = function(x){quantile(x, probs = .975)},
               color = dutchmasters$little_street[5]) +
  labs(x = expression(paste("Climate Change Skepticism (", italic(W), ")")),
       y = expression(atop(theta[paste(italic(X), " on ", italic(Y))], paste("Conditional Effect of Disaster Frame")))) +
  theme_07
```

![](Chapter_07_files/figure-markdown_github/unnamed-chunk-44-1.png)

\[Note. I got the `atop()` trick for the label for the y-axis from [Drew Steen's answer to this stackoverflow question](https://stackoverflow.com/questions/13223846/ggplot2-two-line-label-with-expression?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa).\]

This isn’t quite our version of Figure 7.9, but I’m hoping it’ll add some pedagogical value for what we’re doing. Since we used `summary = F` in `fitted()`, we got full posterior distributions for each of our 26 conditions. Because Figure 7.9 is all about differences between each `frame` pair across the various `skeptic` values, we needed to make a `difference` score for each pair; this is what we did with the last `mutate()` line before the plot code. This initial version of the plot shows the full posterior distribution for each `difference` score. The posteriors are depicted with violin plots, which are density plots set on their side and symmetrically reflected as if by a mirror to give a pleasing leaf- or violin-like shape (though [beware](https://twitter.com/naomicaselli/status/973601256609599488?lang=en)). The light dots and vertical lines are the posterior medians and 95% intervals for each.

Going from left to right, it appears we have a clearly emerging trend. We can more simply express the trend by summarizing each posterior with medians and 95% intervals.

``` r
f_model3 %>% 
  group_by(skeptic) %>% 
  summarize(median = median(difference),
            ll = quantile(difference, probs = .025),
            ul = quantile(difference, probs = .975)) %>% 
  
  ggplot(aes(x = skeptic)) +
  geom_hline(yintercept = 0, color = dutchmasters$little_street[7]) +
  geom_vline(xintercept = c(1.171, 3.934), color = dutchmasters$little_street[7]) +
  geom_ribbon(aes(ymin = ll, ymax = ul),
              fill = dutchmasters$little_street[5],
              alpha = 1/2) +
  geom_line(aes(y = median),
          color = dutchmasters$little_street[5], size = 1) +
  scale_x_continuous(breaks = 1:6) +
  coord_cartesian(xlim = c(1, 6),
                  ylim = c(-1, 1.5)) +
  labs(x = expression(paste("Climate Change Skepticism (", italic(W), ")")),
       y = expression(atop(theta[paste(italic(X), " on ", italic(Y))], paste("Conditional Effect of Disaster Frame")))) +
  theme_07
```

![](Chapter_07_files/figure-markdown_github/unnamed-chunk-45-1.png)

Notice how the contour boundaries of the 95% intervals are a little clunky. That's because our bowtie-shape is based on only 13 x-axis values. If you wanted a smoother shape, you'd specify more `skeptic` values in the data object you feed into `fitted()`'s `newdata` argument. For linear effects, 30 or so usually does it.

Anyway, I got the values for the two vertical lines directly out of the text. It's not clear to me how one might elegantly determine those values within the paradigm we've been using. But that leads to an important digression. The two vertical lines are quite *p*-value centric. They are an attempt to separate the x-axis into areas where the `difference` trend either is or is not statistically-significantly different from zero. That is, we’re dichotomizing—or "trichotomizing", depending on how you look at it—a continuous phenomenon. This is somewhat at odds with the sensibilities of the Bayesians associated with Stan and brms (e.g., [here](https://github.com/paul-buerkner/brms/issues/309)).

On page 259, Hayes wrote:

> Although the JN technique eliminates the need to select arbitrary values of *W* when probing an interaction, it does not eliminate your need to keep your brain turned into the task and thinking critically about the answer the method gives you.

I think this is valuable advice, particularly when working within the Bayesian paradigm. Our version of Figure 7.9 gives some interesting insights into the moderation model, `model3`. I’m just not so sure I’d want to encourage people to interpret a continuous phenomenon by heuristically dividing it into discrete regions.

7.5 The difference between testing for moderation and probing it
----------------------------------------------------------------

This is another section where the NHST-type paradigm contrasts with many within the contemporary Bayesian paradigm. E.g., Hayes opened the section with: "We test for evidence of moderation when we want to know whether the relationship between *X* and *Y* varies systematically as a function of a proposed moderator *W*". His use of "whether" suggests we are talking about a binary answer--either there is an effect or there isn't. But, as [Gelman argued](http://www.stat.columbia.edu/~gelman/research/published/bayes_management.pdf), the default presumption in social science \[and warning, I'm a psychologist and thus biased towards thinking in terms of social science\] is that treatment effects--and more generally, causal effects--vary across contexts. As such, asking "whether" there's a difference or an interaction effect isn't really the right question. Rather, we should presume variation at the outset and ask instead what the magnitude of that variation is and how much accounting for it matters for our given purposes. If the variation--read *interaction effect*--is tiny and of little theoretical interest, perhaps we might just ignore it and not include it in the model. Alternatively, if the variation is large or of theoretical interest, we might should include it in the model regardless of statistical significance.

Another way into this topic is posterior predictive checking. We've already done a bit of this in previous chapters. The basic idea, recall, is that better models should give us a better sense of the patterns in the data. In the plot below, we continue to show the interaction effect with two regression lines, but this time we separate them into their own panels by `frame`. In addition, we add the original data which we also separate and color code by `frame`.

``` r
nd <-
  tibble(frame = rep(0:1, times = 30),
         skeptic = rep(seq(from = 0, to = 10, length.out = 30),
                       each = 2))

fitted(model3, newdata = nd) %>% 
  as_tibble() %>% 
  bind_cols(nd) %>% 
  
  ggplot(aes(x = skeptic, y = Estimate)) +
  geom_ribbon(aes(ymin = Q2.5, ymax = Q97.5, fill = frame %>% as.character()),
              alpha = 1/3) +
  geom_line(aes(color = frame %>% as.character())) +
  geom_point(data = disaster,
             aes(x = skeptic, y = justify, color = frame %>% as.character()),
             alpha = 3/4) +
  scale_fill_manual("frame",
                    values = dutchmasters$little_street[c(10, 5)] %>% as.character()) +
  scale_color_manual("frame",
                     values = dutchmasters$little_street[c(10, 5)] %>% as.character()) +
  scale_x_continuous(breaks = 1:9) +
  coord_cartesian(xlim = 1:9) +
  labs(title = "model3, the interaction model",
       x = expression(paste("Climate Change Skepticism (", italic("W"), ")")),
       y = "Strength of Justification\nfor Withholding Aid") +
  theme_07 +
  theme(legend.position = "top") +
  facet_wrap(~frame)
```

![](Chapter_07_files/figure-markdown_github/unnamed-chunk-46-1.png)

When we separate out the data this way, it really does appear that when `frame == 1`, the `justify` values do increase as the `skeptic` values increase, but not so much when `frame == 0`. We can use the same plotting approach, but this time with the results from the non-interaction multivariable model, `model2`.

``` r
fitted(model2, newdata = nd) %>% 
  as_tibble() %>% 
  bind_cols(nd) %>% 
  
  ggplot(aes(x = skeptic, y = Estimate)) +
  geom_ribbon(aes(ymin = Q2.5, ymax = Q97.5, fill = frame %>% as.character()),
              alpha = 1/3) +
  geom_line(aes(color = frame %>% as.character())) +
  geom_point(data = disaster,
             aes(x = skeptic, y = justify, color = frame %>% as.character()),
             alpha = 3/4) +
  scale_fill_manual("frame",
                    values = dutchmasters$little_street[c(10, 5)] %>% as.character()) +
  scale_color_manual("frame",
                     values = dutchmasters$little_street[c(10, 5)] %>% as.character()) +
  scale_x_continuous(breaks = 1:9) +
  coord_cartesian(xlim = 1:9) +
  labs(title = "model2, the multivariable model",
       x = expression(paste("Climate Change Skepticism (", italic("W"), ")")),
       y = "Strength of Justification\nfor Withholding Aid") +
  theme_07 +
  theme(legend.position = "top") +
  facet_wrap(~frame)
```

![](Chapter_07_files/figure-markdown_github/unnamed-chunk-47-1.png)

This time when we allow the intercept but not the slope to vary by `frame`, it appears the regression lines are missing part of the story. They look okay, but it appears that the red line on the left is sloping up to quickly and that the cream line on the right isn’t sloping steeply enough. We have missed an insight.

Now imagine scenarios in which the differences by `frame` are more or less pronounced. Imagine those scenarios fall along a continuum. It’s not so much that you can say with certainty where on such a continuous an interaction effect would exist or not, but rather, such a continuum suggests it would appear more or less important, of greater or smaller magnitude. It’s not that the effect exists or is non-zero. It’s that it’s orderly enough and of a large enough magnitude, and perhaps of theoretical interest, that it appears to matter in terms of explaining the data.

**And none of this is to serve as a harsh criticism of Andrew Hayes**. His text is a fine effort to teach mediation and moderation from a frequentist OLS perspective. I’ve benefited tremendously from his work. Yet I’d also like to connect his work to some other sensibilities.

From this perspective, consider this sentence from the text (pp. 259--260):

> Rather, probing moderation involves ascertaining whether the conditional effect of *X* on *Y* is different from zero at certain specified values of *W* (if using the pick-a-point approach) or exploring where in the distribution of *W* the conditional effect of *X* on *Y* transitions between statistically significant and non-significant (if using the Johnson-Neyman technique).

From an NHST/frequentist perspective, this makes clear sense. But we're dealing with an entire posterior distribution. Consider again a figure from above.

``` r
  nd <-
  tibble(frame = rep(0:1, times = 3),
         skeptic = rep(quantile(disaster$skeptic, 
                                probs = c(.16, .5, .84)),
                       each = 2)) %>% 
  arrange(frame)

fitted(model3, newdata = nd, summary = F) %>% 
  as_tibble() %>% 
  gather() %>% 
  select(-key) %>% 
  mutate(frame = rep(0:1, each = 4000*3),
         skeptic = rep(rep(quantile(disaster$skeptic, probs = c(.16, .5, .84)),
                           each = 4000),
                       times = 2),
         iter = rep(1:4000, times = 6)) %>% 
  spread(key = frame, value = value) %>% 
  mutate(difference = `1` - `0`,
         skeptic = str_c("skeptic = ", skeptic)) %>% 

  ggplot(aes(x = difference)) +
  geom_density(color = "transparent",
               fill = dutchmasters$little_street[9]) +
  scale_y_continuous(NULL, breaks = NULL) +
  labs(subtitle = "The effect of frame on justify given three different values of skeptic",
       x = NULL) +
  theme_07 +
  facet_wrap(~skeptic)
```

![](Chapter_07_files/figure-markdown_github/unnamed-chunk-48-1.png)

With the pick pick-a-point approach one could fixate on whether zero was a credible value within the posterior, given a particular `skeptic` value. And yet zero is just one point in the parameter space. One might also focus on the whole shapes of the posteriors of these three `skeptic` values. You could focus on where the most credible values (i.e., those at and around their peaks) are on the number line (i.e., the effect sizes) and you could also focus on the relative widths of the distributions (i.e., the precision with which the effect sizes are estimated). And you can apply these sensibilities to the JN technique. Yes, we might be interested in how credible zero is. But there's a lot more to notice, too.

Now consider a modified version of our JN technique plot, from above.

``` r
f_model3 %>% 
  group_by(skeptic) %>% 
  summarize(median = median(difference),
            ll_10 = quantile(difference, probs = .45),
            ul_10 = quantile(difference, probs = .55),
            ll_20 = quantile(difference, probs = .40),
            ul_20 = quantile(difference, probs = .60),
            ll_30 = quantile(difference, probs = .35),
            ul_30 = quantile(difference, probs = .65),
            ll_40 = quantile(difference, probs = .30),
            ul_40 = quantile(difference, probs = .70),
            ll_50 = quantile(difference, probs = .25),
            ul_50 = quantile(difference, probs = .75),
            ll_60 = quantile(difference, probs = .20),
            ul_60 = quantile(difference, probs = .80),
            ll_70 = quantile(difference, probs = .15),
            ul_70 = quantile(difference, probs = .85),
            ll_80 = quantile(difference, probs = .10),
            ul_80 = quantile(difference, probs = .90),
            ll_90 = quantile(difference, probs = .05),
            ul_90 = quantile(difference, probs = .95),
            ll_99 = quantile(difference, probs = .005),
            ul_99 = quantile(difference, probs = .995)) %>% 
  
  ggplot(aes(x = skeptic)) +
  geom_ribbon(aes(ymin = ll_10, ymax = ul_10),
              fill = dutchmasters$little_street[5],
              alpha = 1/4) +
  geom_ribbon(aes(ymin = ll_20, ymax = ul_20),
              fill = dutchmasters$little_street[5],
              alpha = 1/4) +
  geom_ribbon(aes(ymin = ll_30, ymax = ul_30),
              fill = dutchmasters$little_street[5],
              alpha = 1/4) +
  geom_ribbon(aes(ymin = ll_40, ymax = ul_40),
              fill = dutchmasters$little_street[5],
              alpha = 1/4) +
  geom_ribbon(aes(ymin = ll_50, ymax = ul_50),
              fill = dutchmasters$little_street[5],
              alpha = 1/4) +
  geom_ribbon(aes(ymin = ll_60, ymax = ul_60),
              fill = dutchmasters$little_street[5],
              alpha = 1/4) +
  geom_ribbon(aes(ymin = ll_70, ymax = ul_70),
              fill = dutchmasters$little_street[5],
              alpha = 1/4) +
  geom_ribbon(aes(ymin = ll_80, ymax = ul_80),
              fill = dutchmasters$little_street[5],
              alpha = 1/4) +
  geom_ribbon(aes(ymin = ll_90, ymax = ul_90),
              fill = dutchmasters$little_street[5],
              alpha = 1/4) +
  geom_ribbon(aes(ymin = ll_99, ymax = ul_99),
              fill = dutchmasters$little_street[5],
              alpha = 1/4) +
  scale_x_continuous(breaks = 1:6) +
  coord_cartesian(xlim = c(1, 6),
                  ylim = c(-1, 1.5)) +
  labs(x = expression(paste("Climate Change Skepticism (", italic(W), ")")),
       y = expression(atop(theta[paste(italic(X), " on ", italic(Y))], paste("Conditional Effect of Disaster Frame")))) +
  theme_07
```

![](Chapter_07_files/figure-markdown_github/unnamed-chunk-49-1.png)

This time we emphasized the shape of the posterior with stacked semitransparent 10, 20, 30, 40, 50, 60, 70, 80, 90, and 99% intervals. We also deemphasized the central tendency--our analogue to the OLS point estimate--by removing the median line. Yes, one could focus on where the 95% intervals cross zero. And yes one could request we emphasize central tendency. But such focuses miss a lot of information about the shape--the entire smooth, seamless distribution of credible values.

I suppose you could consider this our version of Figure 7.10.

7.6 Artificial categorization and subgroups
-------------------------------------------

There are multiple ways to dichotomize the data by `skeptic`. A quick simple way is to use `ifelse()` to make a `skeptic_hi` dummy.

``` r
disaster <-
  disaster %>% 
  mutate(skeptic_hi = ifelse(skeptic >= mean(skeptic), 1, 0))
```

With our dummy in hand, we're ready to fit the two models.

``` r
model6_low <-
  brm(data = disaster %>% filter(skeptic_hi == 0), 
      family = gaussian,
      justify ~ 1 + frame,
      chains = 4, cores = 4)

model6_high <-
  update(model6_low, newdata = disaster %>% filter(skeptic_hi == 1),
         chains = 4, cores = 4)
```

The coefficient summaries:

``` r
fixef(model6_low) %>% round(digits = 3)
```

    ##           Estimate Est.Error   Q2.5 Q97.5
    ## Intercept    2.622     0.098  2.421 2.821
    ## frame       -0.103     0.144 -0.390 0.179

``` r
fixef(model6_high) %>% round(digits = 3)
```

    ##           Estimate Est.Error  Q2.5 Q97.5
    ## Intercept    3.071     0.146 2.790 3.357
    ## frame        0.476     0.208 0.075 0.874

You can use `fitted()` to get the posterior means and other summaries for the two `frame` groups, by model.

``` r
fitted(model6_low, 
       newdata = tibble(frame = 0:1)) %>% 
  round(digits = 3)
```

    ##      Estimate Est.Error  Q2.5 Q97.5
    ## [1,]    2.622     0.098 2.421 2.821
    ## [2,]    2.519     0.102 2.319 2.718

``` r
fitted(model6_high, 
       newdata = tibble(frame = 0:1)) %>% 
  round(digits = 3)
```

    ##      Estimate Est.Error  Q2.5 Q97.5
    ## [1,]    3.071     0.146 2.790 3.357
    ## [2,]    3.546     0.148 3.249 3.847

Do note that though brms 'Est.Error' is the posterior *SD* for the coefficient, it is not the same thing as descriptive statistic *SD* of a subset of the data. Thus, although our 'Estimates' correspond nicely to the mean values Hayes reported in the middle of page 264, his *SD*s will not match up with our \`Est.Error' values, and nor should they.

Anyway, our results don't yield *t*-tests. But you don't need those anyway. We're working within the regression paradigm! But if you're really interested in the sub-model-implied differences between the two levels of `frame` by `skeptic_hi` subgroup, all you need is the `frame` coefficient of `model6_low` and `model6_high`. Here we'll use `bind_rows()` to combine their posterior samples and then plot.

``` r
posterior_samples(model6_low) %>% 
  select(b_frame) %>% 
  bind_rows(
    posterior_samples(model6_high) %>%
      select(b_frame)
  ) %>% 
  mutate(model = rep(c("model6_low", "model6_high"), each = 4000)) %>% 
    
  ggplot(aes(x = b_frame, fill = model)) +
  geom_density(size = 0, alpha = .8) +
  scale_fill_manual(NULL, 
                    values = dutchmasters$little_street[c(1, 9)] %>% as.character()) +
  scale_y_continuous(NULL, breaks = NULL) +
  labs(subtitle = "The difference score distributions between frame levels,\ncolor coded by mean-split skeptic",
       x = NULL) +
  theme_07
```

![](Chapter_07_files/figure-markdown_github/unnamed-chunk-54-1.png)

As within the frequentist paradigm, please don't mean split as a Bayesian. When possible, use all available data and use the regression formula to model theoretically-meaningful variables in your analyses.

Note. The analyses in this document were done with:

-   R 3.4.4
-   RStudio 1.1.442
-   rmarkdown 1.9
-   tidyverse 1.2.1
-   dutchmasters 0.1.0
-   ochRe 1.0.0
-   directlabels 2017.03.31
-   readr 1.1.1
-   rstan 2.17.3
-   brms 2.3.1

Reference
---------

Hayes, A. F. (2018). *Introduction to mediation, moderation, and conditional process analysis: A regression-based approach.* (2nd ed.). New York, NY, US: The Guilford Press.