# user2016-notes
Notes for tutorials and sessions at useR! 2016 Conference

The R User Conference 2016    
Monday, June 27 - Thursday, June 30 2016      
Stanford University, Stanford, California     

## Links     
* Twitter hashtag: [#useR2016](https://twitter.com/hashtag/useR2016?src=hash)
* [Full Schedule](http://schedule.user2016.org/)

____

## Monday

### Machine Learning Algorithmic Deep Dive | [repo](https://github.com/ledell/useR-machine-learning-tutorial)
* Leg 1: Decision trees, Random Forest, GBM
  * Install R kernal from Terminal not RStudio
  * Practical topics related to each of the algorithms:
  * class imbalance: ML algorithms use 'accuracy' for optimization which can be problematic since it is not performing as intended. i.e. Fraud data
  * Sparsity, dimensionality, normalization, overfit...

* Decision Tree: Classification and Regression Tree (CART)
  * good for: large data, mixed predictors, surrogate splits
  * sometimes overly used since they are easy to interpret
  * tree algorithms: ID3, **C4.5, C5.0**
  * splitting criterion: Gini, information-based used by C4.5
  * missing values through surrogate splits - software concerns
  * R package: *rpart*   

* Random Forest:
  * An application of decision trees, this takes models and **ensemble's** them together.
  * Easy to parallelize since the trees don't know about each other.
  * randomness comes from subset features/variables; bagging is short for bootstrap aggregating. Randomness helps de-correlate the trees
  * automatically get a an estimate of the generalization error (out-of-bag sample). Also get variable importance through permutations (how often does the variable get chosen)
  * can't overfit by adding trees, just adds computation time.
  * can overfit by increasing tree depth.
  * extremely randomized trees randomly selects how splits are computed
  * R package(s): *extraTrees*, for extremely randomized trees, another available in *h2o::randomForest* package, *randomForest*, *DistributedR*, *party::cForest*, *ranger* good with high dimensionality datasets, *Rborist*. Just find the optimal package based on the data benchmarking!

* Gradient Boosting Machines (GBM)
  * good for: many types of data -> potentially better than Random Forest.
  * An ensemble of weak prediction models
  * Iterative algorithm by allowing optimization of an arbitrary differential loss function.
  * AdaBoost algorithm: up-weight poor training results so that more attention can be paid for the next round
  * GBM focuses on the gradient of the loss function which is a shift in mindset.
  * Stochastic GBM: add randomness!
  * xgboost and H20 implementations GBM of have per-tree column and row sampling parameters which is the cause winning kaggle competitions usage.
  * GBM trees: should be "shrubs" or "stumps" aka shorter trees than Random Forest.
  * R package(s): *gbm*, *xgboost*

* Leg 2:

* Generalized Linear Model (GLM)
  * consider the distribution and response
  * training a GLM find coefficients that minimize the residual sum of squares (RSS)
  * "matrix inverse is not a good place to be at" - Erin LeDell
  * Gloss?  - link R to optimize basic core math to speed up glm's
  * Different solvers: Iteratively Re-weight Least squares with ADMM, Cyclical coordinate descent, Limited memory-BFGS optimization algorithm = butter knife approach
  * GLM `predict()` error for finding new factors when applying fit to test data. It gets encoded as not there
  * Ridge Regression: constraints square sum of squares of beta coefficients
    * L2 regularization: useful for preventing overfitting
    * good for: little bit of noise
  * LASSO (L1 Regularization)
    * good for: feature selection in a sparse matrix -> zero out coefficients
  * Elastic Net = Ridge + LASSO
  * R package(s): *speedglm*, *glmnet* uses cyclical coordinate descent for LASSO and Ridge, *caret::train* has a nice interface, *h20.glm* set lambda = 0 to remove default regularization.

* Deep Neural Networks
  * neural network with (multiple hidden layers): a bunch of linear models stitched together - can abstract higher dimensional features out of the data (spend less time on feature engineering and more time on parameter tuning)
  * iterations = epics (early stopping point in *h20* library)
  * sparse data should be run with neural nets and glm's not trees
  * use gridsearch to find parameters.
  * HOGWILD! method
  * difficult to know architecture structure up front -> parameter tune which is difficult (activation, regularization, drop-out)
  * multilayer perception or feed-forward artificial neural networks (ANN)
  * recurrent neural networks (RNN) has internal memory. good for: sequence learners (i.e. bots on twitter)
  * convolution neural networks (CNN): good for images. (i.e. trippy images from google!)
  * R package(s): *mxnet* has GPU support and can have specified output, *h20* has support for exponential families.

* DO EVERYTHING: Stacking
  * ensemble learning: combine models together with a machine learning algorithm ("METAlearner").
  * good for: not knowing what will work in advance or you don't care what algorithm to use
  * bad: takes longer and not as much software and ecosystem to do stacking.
  * Super Learner == Stacking
    * set-up ensemble
    * train ensemble: k-cross fold validation for base learners
  * R package(s): *SuperLearner*, *subsemble* faster alternative, *h20Ensemble* supports regression and binary classification

  _____

### Extracting data from the web APIs and beyond | [repo](https://github.com/ropensci/user2016-tutorial)
  * Part 1: Collecting data from an API by Scott Chamberlain
    * An API is programming instructions to interact with a piece of software
    * Could be: software package, public web API, database
    * REST (Representational State Transfer)
    * HTTP (Hyper Text Transfer Protocol): verbs for different actions. HTTP is behind the scene. (i.e. install.packages() uses the download)
    * Server: nginx, sql database; Client: httr in R
    * httr example in R:

    ```r
    httr_exp <- httr::GET("https://www.google.com/")
    str(httr_exp)
    httr_exp$headers
    ```

    * HTTP verbs:
      * GET => retrieve from Server. Identical to HEAD.
        * base url, path, query parameter (everything after the ? symbol),
      * POST => create
        * body (JSON blob)
      * PUT => update partial record
        * base url, path
      * DELETE => delete a record
        * base url, path, empty body (don't get anything back)
      * Components: URL, Method (what HTTP verb), Headers (any meta data to modify request), body (contains data)
    * Exercise with http://httpbin.org/ - mismatch with verb method and url yields a 405 status code! You don't want arbitrary things to be changed on certain url's
    * HTTP responses: the client sends back: status, headers, body/content. 3 digit numeric codes for status (1xx is information, 2xx is success, 3xx is redirection, 4xx is client error, 5xx server error). Servers do not always give correct codes!
    * Exercise with status code numbers with https://httpstatusdogs.com/ or https://http.cat/
    * headers: some are standardized, most headers are *key:value* pairs
    * content/body: sometimes you get back raw bytes

    ```r
    library(httr)
    x <- httr::GET(url="http://httpbin.org/get/405")
    str(x)
    x$status_code
    y = httr::GET("http://httpbin.org/get/405", accept_json())

    ```

    * Data Formats: JSON, XML (uglier!) *xml2*

    ```r
    x = httr::GET("http://www.omdbapi.com/?t=veronica+mars&y=&plot=short&r=json")
    content(x, as="text") # parse each format to text

    ```

  * Part 2: Wrapping an API with R by Karthik Ram
    * Beer tap analogy: write a set of functions that can talk to each of the taps
    * SWAPI: star wars data
    * result from get method is just a print with response, date, staus code (200 is code), content type.

    ```r
    x <- httr::GET("api.randomuser.me")
    x  # contains todays date, successful 200 status, json type and size

    y <- httr::GET("http://api.openweathermap.org/data/2.5/forecast?id=524901")
    y # todays date, bad 401 (user side) status, json type, size

    ### pro trip
    y$status_code # make sure you get a successful status code before contining
    httr::stop_for_status(y)
    ```

    * extract content from a GET request `r httr::content(y)`. R can figure out what do do with it by associating a function with the content-type (read_html, read_xml, read_csv, ...). application/*json* (will is the operating system to had the type of data )
    * Exercise with http://api.randomuser.me/ from https://randomuser.me/: When writing a R package look at the [documentation](https://randomuser.me/documentation)

    ```r
    request <- httr::GET("http://api.randomuser.me/")
    person <- httr::content(request, as="parsed")
    person # just a list needs to be put into a data.frame

    ### modified with a query content
    args = list(gender = "female", results = 5)
    request <- httr::GET("http://api.randomuser.me/", query=args)
    person <- httr::content(request, as="parsed")
    length(person$results)
    View(data.frame(t(unlist((person$results)))))
    ```

  * Part 3: Scraping data without an API by Garrett Grolemond
    * How to get data from the web without and API and breaking the Terms of service?
    * content is stored in HTML. HTML can be parsed in sub items of elements in a tree.
    * Knowing which tags surround your data is the key to extracting. <a href="url">title</a> stands for *anchor*
    * Finding source code for web pages!
    * CSS selectors: understand the class (.class) and id (#id), tag (no prefix). "Stealing the language that the CSS uses"

    ```css
    span {
      color: #fffff;
    }
    ```
    * **Strategy**: extract info from HTML, identify info to extract from CSS selectors
    * R Package: *rvest*
      * pull in HTML file and turn it into XML file with `r read_html()`
      * extract pieces of info out of nodes with `r html_nodes()` by interpreting the CSS selectors.
      * extract content from nodes with `r html_text`, `r html_table`, `r html_name`, `r html_children`, `r html_attrs`.

      ```r
      library(rvest)
      url <- "http://www.imdb.com/title/tt2294629/"
      frozen <- read_html(url)
      str(frozen)
      cast <- html_nodes(frozen, ".itemprop .itemprop")
      html_text(cast)
      ```

      * selectorGadget: vignette("selectorgadget") - help find the correct CSS selector.





## Tuesday

## Wednesday

## Thursday
