This implementation is a continuation of my learning experience from a weather model I've made:

https://github.com/tobiasocula/Weather-HMM

Because this model involves higher dimensional data, normalizing all data was necessary.

I therefor decided to "normalize" some of the calculations, namely during the computation of $\alpha_t(i)$ and $\beta_t(i)$. This wasn't needed in the weather model, but here, because of the higher dimensionality of the data, it was.

Reference: https://gregorygundersen.com/blog/2022/08/13/hmm-scaling-factors/

It roughly works like this: instead of computing

$\alpha_t(i)=b_j(O_t)\sum_i\alpha_{t-1}(i)a_{i,j}$

We divide by the sum of $\alpha_t$ over all states:

$\alpha_t(i)=(b_j(O_t)\sum_i\alpha_{t-1}(i)a_{i,j})/(\sum_j\alpha_t(j))$

But of course here we compute the log value of $\alpha_t(i)$ for numerical stability. We also store the value of $s_t=\sum_j\alpha_t(j)$ in a sequence called the scaling-sequence.
We now use this scaling sequence to also normalize beta:

$\beta_t(i)=(\sum_j a_{i,j}b_j(O_{t+1})\beta_{t+1}(j))/s_t$

In the calculation of $\gamma_t(i)$ and $\xi_t(i,j)$, these factors conveniently cancel out.  
The computation of the log-likelyhood is now, instead of

$\text{log-likelyhood}=\text{log}\sum_i\alpha_T(i)$

It now becomes the sum of $\text{log}\ s_t$ over all $t$, because

\begin{equation}
\begin{aligned}
\text{log-likelyhood}=\text{log}\sum_i\alpha_{T}^{\text{unscaled}}(i)\\
=\text{log}\sum_i\alpha_{T}^{\text{scaled}}(\prod_t s_t)=\text{log}((\prod_t s_t)\sum_i\alpha_{T}^{\text{scaled}}(i))=\text{log}\prod_t s_t=\sum_t\text{log}\ s_t
\end{aligned}
\end{equation}

Because $\sum_i\alpha_{t}^{\text{scaled}}(i)=1$ for every $t$.

### Different implementations

I have tried a few different versions regarding the data-clustering step.

-Standard k-means

-PCA on volatility data, followed by k-means

-Spectral clustering

-Gaussian Mixture Model (GMM)

Because I have noticed that, using k-means, it struggles with regime estimation using volatility data, I have tried using these other implementations.  
PCA reduces the dimensionality of the volatility matrix (to fewer columns). I was hoping the model would make clearer distinctions between volatility regimes. Spectral clustering learns more complex non-linear relationships between rows in the volatility matrix, and the Gaussian Mixture Model computes a probability distribution over the regimes for every volatility datapoint, instead of hard-assigning regimes to every row.

However, I noticed that even when using these different techniques, the model still struggles with predicting the volatility sequence on the testing data. The predicted values are generally way higher in magnitude than the realised volatility sequence.