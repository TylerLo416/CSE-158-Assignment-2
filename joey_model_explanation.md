## Model Explanation

## Data Used **NEW**

We used 68000 entries to train our data and 1200 to test. This difference, while appearing large, is reduced by how we use the data: for each training entry, there are constraints for how we use it so that 50% of training data is spoilers and the other half non-spoilers. These constraints are not in place for testing data, where we just use all of the sentences. With this in mind, training data comes out to 51000 sentences and 15000 for testing (testing is ~20% of total data).

### Data Prep
For training data, the goal is to have 50% of sentences to be non-spoilers (0) and 50% to be spoilers (1). Spoilers only happen like 2% of the time. Therefore, we sample sentences collecting every spoiler sentence and then an even amount of non-spoilers. In addition, we want a variety of spoilers per book. Thus, I get a variety of spoiler/total sentence per book %, ranging from all spoilers to all non-spoilers (and everywhere between).

For testing data, we take every sentence.

Then, we collect the BERT CLS embedding for each sentence. This takes a while to construct. Not entirely sure what it collects beyond token sentiment, but its output size is 768 for each sentence (padding shorter sentences). **These are saved in a separate file so that you can call the embeddings at later times without needing to run this code again (as it takes very, very long.**

Then, we create common relationship-focused dictionaries: usersPerItem, itemsPerUser, reviewsPerItem (sentence spoiler label), reviewsPerUser (sentence spoiler label), ratingsPerUser, ratingsPerItem. On top of this, I calculate book and user popularity by how common they show up in the training data. userIDs and itemIDs are collected to create a sparse matrix of interactions in the dataset between books and users (and for other later reasons).
                                                                                                                                                                                                                                                                                                                                                                                                                                  
Lastly, numerous averaged are calcualted based on the training data. These include the following:
                                                                                                                                                                                                                                                                                                                                                                                                                                  
- avg_book_pop (NOT USED) - average frequency of a book based on training data
- avg_user_freq (NOT USED) - average frequency of a user based on training data
- avg_pop_by_freq (NOT USED) - avg_user_pop * avg_user_freq
- avg_book_rank (NOT USED) - average ranking of a book based on popularity in training data
- avg_user_rank (NOT USED)  - average ranking of a user based on popularity in training data
- avg_ct_book_spoilers - average number of spoilers per book in training data
- avg_ct_user_spoilers (NOT USED) - average number of spoilers per user in training data
- ovr_avg_rating_item - average rating given for a book in training data
- ovr_avg_rating_user - average rating given for a user in training data
                                                                                                                                                                                                                                                                                                                                                                                                                                  
 ### Vector Prep and Explanation   
                                                                                                                                                                                                                                                                                                                                                                                                                                  
users and books sets are created to check if books and users were in the training data. It determines how its vector is created.                                                                                                                                                                                                                                                                                                                                                                                                                                   

For each sentence in the training and testing data, a vector is created in the following way:

- 1: bias term                                                                                                                                                                                                                                                                                                                                                                                                                                
- pop_by_freq: user popularity, based off of books_read_by_user list, multiplied by book_pop
- book_rank: ranking of book based on how many times it has been read
- length: length of the sentence
- ct_book_spoilers: number of spoilers for the book
- ct_user_spoilers: number of spoilers user has made
- avg_rating_item: average rating given to book in training


Then, BERT CLS embedding information is added to the end of the vector, all 768 values.                                                                                                                                                                                           

### Model

Train LinearRegression model with c=1 and balanced class weight. Then get the decision function for the testing data. Since 3.5% of data is spoilers use a number in that range and assign the 90 percentile and higher values with 1 and the rest 0. We do this as spoilers, on average, are concentrated within the top 10% of results; because we want to be overly cautious of spoilers (meaning we care more about false negatives than false positive), we add the 6.5% of cushion. Then, evaluation metrics are used, like accuracy, F1, ROC AUC, information on the position of spoilers' decision function values, and tp, tn, fp, fn.

## Best Model

As of now, the best model uses the variables [1, pop_by_freq, book_rank, length, ct_book_spoilers, ct_user_spoilers, avg_rating_item] with a c of 1. This achieves the following:

- accuracy:  0.9118937048503611
- tp:  334 tn:  13804 fp:  1216 fn:  150
- F1:  0.32841691248770893
- AUC: 0.8961211497617502

## Goal Comparison

We came short of our goal due to limiting the data we used (as a result of space constraints caused by BERT model taking up so much space):

- accuracy:  0.9
- F1:  0.65
- AUC: 0.75

I ordered the scores assigned by the function to rows, highest to lowest. Below descirbes different stats on the positions of spoilers on the list (out of 15.5K entries in test data):

- First spoiler position:  75 (top 0.5%)
- Last spoiler position:  12104 (top 78%)
- Average spoiler position:  1805.1797520661157 (top 12%)

                                                                                                                                                                                                                                                                                                                                                                                                                                  
                                                                                                                                                                                                                                                                                                                                                                                                                                  
