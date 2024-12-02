## Model Explanation

### Data Prep
For training data, the goal is to have 50% of sentences to be non-spoilers (0) and 50% to be spoilers (1). Spoilers only happen like 2% of the time. Therefore, we sample sentences collecting every spoiler sentence and then an even amount of non-spoilers. In addition, we want a variety of spoilers per book. Thus, I get a variety of spoiler/total sentence per book %, ranging from all spoilers to all non-spoilers (and everywhere between).

For testing data, we take every sentence.

Then, we collect the BERT CLS embedding for each sentence. This takes a while to construct. Not entirely sure what it collects beyond token sentiment, but its output size is 768 for each sentence (padding shorter sentences).

Then, we create common relationship-focused dictionaries: usersPerItem, itemsPerUser, reviewsPerItem (sentence spoiler label), reviewsPerUser (sentence spoiler label), ratingsPerUser, ratingsPerItem. On top of this, I calculate book and user popularity by how common they show up in the training data. userIDs and itemIDs are collected to create a sparse matrix of interactions in the dataset between books and users (and for other later reasons).
                                                                                                                                                                                                                                                                                                                                                                                                                                  
Lastly, numerous averaged are calcualted based on the training data. These include the following:
                                                                                                                                                                                                                                                                                                                                                                                                                                  
- avg_book_pop (NOT USED) - average frequency of a book based on training data
- avg_user_freq (NOT USED) - average frequency of a user based on training data
- avg_pop_by_freq (NOT USED) - avg_user_pop * avg_user_freq
- avg_book_rank (NOT USED) - average ranking of a book based on popularity in training data
- avg_user_rank (NOT USED)  - average ranking of a user based on popularity in training data
- avg_ct_book_spoilers - average number of spoilers per book in training data
- avg_ct_user_spoilers - average number of spoilers per user in training data
- ovr_avg_rating_item - average rating given for a book in training data
- ovr_avg_rating_user - average rating given for a user in training data
                                                                                                                                                                                                                                                                                                                                                                                                                                  
 ### Vector Prep and Explanation   
                                                                                                                                                                                                                                                                                                                                                                                                                                  
users and books sets are created to check if books and users were in the training data. It determines how its vector is created.                                                                                                                                                                                                                                                                                                                                                                                                                                   

For sentence in the training and testing data, its vector is created in the following way:

- 1: bias term                                                                                                                                                                                                                                                                                                                                                                                                                                 
- book_pop: book popularity, based off of book_popularity list
- pop_by_freq: user popularity, based off of books_read_by_user list, multiplied by book_pop
- user_rank: ranking of user based on how many books they read
- user_rating: rating user gave the book
- temporal_feature: time of review 
- ct_book_spoilers: number of spoilers for the book
- ct_user_spoilers: number of spoilers user has made
- avg_rating_item: average rating given to book in training
- avg_rating_user: average rating user has given out                                                                                                                                                                                                                                                                                                                                                                                                                             
                                                                                                                                                                                                                                                                                                                                                                                                                                  
(considering adding another temporal of position of sentence compared to others, adding book_rank, and messing with using mean/other default answers for books/users not in training data lists -> since they not there sometimes, cannot calculcate some things for them)

Then, BERT embedding information is added to the end, all 768 values. This created our vectors.


### Model

Train LinearRegression model with c=100 and balanced class weight. Then get the decision function for the testing data. Since 2-3% of data is spoilers use a number in that range and assign the 97 percentile (or 98 percentile) and higher values with 1 and the rest 0. Then, evaluation metrics are used, like accuracy, F1, ROC AUC, information on the position of spoilers' decision function values, and tp, tn, fp, fn.

## Best Model

As of now, the best model uses the variables [1, book_pop, pop_by_freq, user_rank, user_rating, temporal_feature, ct_book_spoilers, ct_user_spoilers, avg_rating_item, avg_rating_user] with a c of 100. This achieves the following:

- accuracy:  0.9919566644780039
- tp:  24 tn:  6019 fp:  43 fn:  6
- F1:  0.4948453608247423
- AUC: 0.9931430770922687
- First spoiler position:  13
- Last spoiler position:  341
- Average spoiler position:  62.13333333333333

## Goal

The goal is for the model to achieve the following:

- accuracy:  0.9
- F1:  0.65
- AUC: 0.75

# To-Do (for the Model)

- Test data on Kaggle, LCS datasets, if possible
- Improve model in terms of evaluation metrics
- Figure out how much of data can be used as **USING ALL OF THE DATA, EVEN JUST THE FIRST 150K ENTRIES, CRASHES MY COMPUTER AFTER LIKE 5 HOURS OF BERT EMBEDDINGS**.

                                                                                                                                                                                                                                                                                                                                                                                                                                  
                                                                                                                                                                                                                                                                                                                                                                                                                                  
