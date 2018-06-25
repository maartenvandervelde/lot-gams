# GAMs summary

---

## Preparation

Load packages `mgcv` and `itsadug`.

Make sure to explicitly convert categorical predictors to factors (`as.factor()`).
If model fitting fails, this is the first thing to check.

---

## Fitting a model

### Selecting the right function

There are two functions in the `mgcv` package for fitting GAMs: `gam()` and `bam()`.

In general, `gam()` is considered the more 'precise', but this comes at the cost of speed.
When fitting a model to a dataset with > 10.000 observations, or when fitting a model with a complex (random) effects structure, `bam()` is usually preferred. In any case, it is good practice to verify final results from `bam()` with `gam()`.

### Model syntax

The most basic formula:

```
m1 <- gam(Y ~ ..., data = ...)
```

**Fixed effects:**

- Categorical intercept: **X**

- Continuous intercept (apply smooth): **s(X)**

- Split intercept by group G: **s(X, by = G)** (if grouping X by G, also include G as a main effect, so that this smooth does not have to fit the entire difference between levels of G)

- Interaction between A and B: **ti(A, B)** (equivalent to `A:B`)

- Interaction between A and B, including main effects: **te(A, B)** (equivalent to `A*B`, but not equivalent to `s(A) + s(B) + ti(A, B)`(!) because of difference in the default number of base functions available; a single `te()` will be slightly smoother)

- Interaction between isotropic variables A and B (measured in the same units on the same scale): **s(A, B)** or **ti(A, B, d = c(2))**

**Random effects:**

- Random intercept: **s(X, bs = "re")** (equivalent to `(1 | X)`)

- Random slope for Z over X: **s(X, Z, bs = "re")** (equivalent to `(0 + X | Z)`; note that this does not assume correlation between X slope and Z intercept!)

- Random factor smooth for Z over X: **s(X, Z, bs = "fs", m = 1)** (already accounts for random differences in intercept and slope, so it doesn't make sense to use this in combination with random intercept/slope)

### Additional options

- Speed up `bam()` by using `method = "fREML"` (fast REML), `discrete = TRUE` (discretise covariates), and `cluster = ...`/`nthreads` (run computations in parallel).

- Fit a different DV distribution (default is normal distribution): `family = ...` (see `?family.mgcv` and `?family` for options).


## Evaluating a model fit

### Checking assumptions

At a minimum, these four assumptions should be checked:

- **Normality of residuals**: are the residuals normally distributed? If not, consider transforming the DV or fitting to a different distribution.

```
qqnorm(resid(m1))
qqline(resid(m1))
```

- **No autocorrelation**: is there no autocorrelation in the residuals? If there is, make sure to account properly for random effects (e.g. subjects, items), and if necessary include an AR1 model (see Lab 3 for an example).


```
acf(resid(m1))
```

**No heteroskedacity**: is the variance in the residuals independent of the fitted mean? If not, there is structure left in the data that the model should account for.

```
plot(resid(m1) ~ fitted(m1))
```

### Inspecting the model fit

The model fit can be inspected in a view that is similar to the way linear models are described:
```
summary(m1)
```

### Visualising smooths

The effect of a single smooth predictor X can be plotted as follows:
```
plot_smooth(m1, view = "X")
```

To split the predictor by a grouping factor G:
```
plot_smooth(m1, view = "X", plot_all = "G")
```

If there are random effects in the model, this plotting function will just select one of the levels.
To make interpretation easier, it makes sense to exclude random effects:
```
plot_smooth(m1, view = "X", rm.ranef = TRUE)
```

If each trial has a unique identifier E, we can also plot the model fit on a random selection of individual trials (see Lab 3):
```
plot_modelfit(m1, view = "X", event = dat$E)
```

Plot the difference between conditions of grouping factor G:
```
plot_diff(m1, view = "X", comp = list(G = c("level1", "level2")))
```

### Visualising interactions of smooths

Nonlinear interactions can be visualised in several ways:

- **`pvisgam()`**: Make a two-dimensional plot of the partial effects of an interaction term.
For example, to plot the partial effect of the term with index 2 in the model summary:

```
pvisgam(m1, view = c("A", "B"), select = 2)
```

Tip: we can make a 3D plot by adding `plot.type = "persp"`.

- **fvisgam()**: Make a two dimensional plot of the summed fitted effects (i.e., the model prediction). If not specified with `cond`, the function just chooses a value for each predictor that is not on one of the axes. Random effects can be included or excluded with `rm.ranef`.

```
fvisgam(m1, view = c("A", "B"))
```

## Comparing model fits

Unlike nested linear models, GAMs models are not strictly nested, since the presence or absence of each predictor influences the fit of smooths to other predictors. This means that model comparison is not as straightforward, and we should not rely too much on things like AIC score.

Nevertheless, there are some functions that can give an indication of which model is better:
```
AIC(m1, m2)
compareML(m1, m2) # Requires models to be fit with method = "ML"
```

