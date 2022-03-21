> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 http://bayesjumping.net/log-sum-exp-trick/

[The Log Sum Exp Trick](http://bayesjumping.net/log-sum-exp-trick/ "The Log Sum Exp Trick")
===========================================================================================

January 17, 2016
----------------

In machine learning, arithmetic underflow can become a problem when multiplying together many small probabilities. In many models it can be useful to calculate the log sum of exponentials.

log∑i=1nexp(xi)log⁡∑i=1nexp⁡(xi)\log \sum_{i = 1}^n \exp (x_{i})

If xixix_{i} is sufficiently large or small, this will result in an arithmetic overflow/underflow. To avoid this we can use a common trick called the Log Sum Exponential trick.

log∑i=1nexp(xi)=logexp(b)∑i=1nexp(xi−b)=b+log∑i=1nexp(xi−b)log⁡∑i=1nexp⁡(xi)=log⁡exp⁡(b)∑i=1nexp⁡(xi−b)=b+log⁡∑i=1nexp⁡(xi−b)% <![CDATA[ \begin{align} \log \sum_{i = 1}^n \exp (x_{i}) & = \log \exp(b) \sum_{i = 1}^n \exp (x_{i} - b) \\ & = b + \log \sum_{i = 1}^n \exp (x_{i} - b) \end{align} %]]>

Where bbb is max(x)max(x)\max(x).

We can calculate this in Python with:

or using Sci Py

```
from scipy.misc import logsumexp
logsumexp(ns)

```

Jupyter notebook [here](https://github.com/bayesjumping/JupyterServer/blob/master/notebooks/Log%20Sum%20Exp.ipynb).

[Numerical Optimisation](http://bayesjumping.net/tags/#Numerical Optimisation "Pages tagged Numerical Optimisation")[Python](http://bayesjumping.net/tags/#Python "Pages tagged Python")[Machine Learning](http://bayesjumping.net/tags/#Machine Learning "Pages tagged Machine Learning")

*   [Like](https://www.facebook.com/sharer/sharer.php?u=http://bayesjumping.net/log-sum-exp-trick/ "Share on Facebook")
*   [Tweet](https://twitter.com/intent/tweet?text=http://bayesjumping.net/log-sum-exp-trick/ "Share on Twitter")
*   [+1](https://plus.google.com/share?url=http://bayesjumping.net/log-sum-exp-trick/ "Share on Google Plus")

[Read More](http://bayesjumping.net/multiple-python-environments-package-install-conda/)

### [Generate Correlated Data](http://bayesjumping.net/generate-correlated-data/ "Generate Correlated Data")

Generate Correlated Data

[Read on →](http://bayesjumping.net/generate-correlated-data/)

#### [Gaussian mixture models in PyMc](http://bayesjumping.net/mixture-gaussians-pymc/ "Gaussian mixture models in PyMc")

Published on May 08, 2016

#### [Brier Score](http://bayesjumping.net/brier-score/ "Brier Score")

Published on May 08, 2016