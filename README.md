
# Predicting Irregular Sleeping Patterns from Tweets

## Overview
Predict whether a user is suffering from ISP based on their social media interactions on Twitter. We identify users' ISP from their psycholinguistic characteristics and social media word usage patterns.

 - **Domain**:  Data Science, Deep Learning, Natural Language Processing, Mental Health, Psycholinguistic Patterns, Social Media.
 - **Tools used**: Twitter API - Tweepy, LIWC2015, IBM SPSS Statistical Software.

## Authors

- **Corresponding author:** [Dr. Md. Saddam Hossain Mukta](https://cse.uiu.ac.bd/profiles/saddam/), Associate Professor and Undergraduate Coordinator, United International University (UIU), Bangladesh
- **Other authors:** [Mohammed Jawwadul Islam](https://www.linkedin.com/in/jawwadfida/), [Mohammad Fahad Al Rafi](https://www.linkedin.com/in/md-fahad-al-al-rafi-14b968111/), Nafisa Akhter, [Moumy Kabir](https://www.linkedin.com/in/pranto-podder-b78b97162/), [Pranto Podder](https://www.linkedin.com/in/aysha-siddika-577ba5224/), [Aysha Siddika](https://www.linkedin.com/in/moumy-kabir-156a0a232/), and [Dr. Mohammed Eunus Ali](https://cse.buet.ac.bd/faculty_list/detail/eunus), Professor, Bangladesh University of Engineering and Technology (BUET), Bangladesh. 

## Data Availability

When collecting data from Twitter for our experiments, we followed Twitter's rules and regulations for data privacy. We collect tweets after getting permission from Twitter. According to [Twitter's policy](https://developer.twitter.com/en/developer-terms/policy), the tweets collected are their public property, which should only be exposed partially publicly with their consent, a consent that is granted with the Twitter API. Therefore, a modified version of the data containing psycholinguistic characteristics of the tweets is distributed.


## Overall steps for dataset creation. 

<img src="https://user-images.githubusercontent.com/64092765/199010521-7c2b2123-62db-4308-a6a6-bfb65931b9b6.png" width="75%">

## Data Collection
We used Twitter to collect 924 tweets from certain nations or continents, such as the United States, the United Kingdom, Canada, Australia, New Zealand, Bangladesh, and some countries in Africa. We looked at the times when their tweets were tweeted. We used random sampling on the user based on their tweeting time and word choices and labeled their file with "Irregular Sleeping Pattern Yes" and "Irregular Sleeping Pattern No" labels. For Irregular Sleeping Pattern Yes, a time interval of 1 am to 5 am, which indicated that these users suffered from irregular sleeping patterns. For Regular Sleeping Pattern users, we randomly selected users who did not tweet at night, in the time range from 1 am to 5 am. 

## Filtering tweets based on time

For time filtering, we first divide the users (Irregular and Regular Sleeping Pattern users) based on their geographical location. Then we convert the UTC to standard geographical UTC based on the user’s location. For example, for the USA, we have to subtract 4 hours from Bangladesh standard UTC to convert it into (UTC-4) standard geographical UTC. For this conversion, we needed to separate the user CSV files into six sections based on areas: USA, Europe, Asia (Bangladesh, India, Pakistan, Korea, Japan), New Zealand, Australia, and South Africa.

### Fixing the time range

For Irregular Sleep Users time range is from 1 am - 6 am (excluding). Regular sleep users are outside this time range.

```Python
def get_time_date(data):
  # time range: 1 am - 6 am(excluded)

  time_range_list = []
  time_freq = 0

  date_range_list = []

  date_list = []
  tweet_list = []

  import datetime
  def time_in_range(start, end , current):
    return start <= current <= end 

  time_freq =0
  for i, row in data.iterrows():
    date_row = row['date']
    tweet_row = row['tweet']

    temp2 = date_row
    start = datetime.time(1,0,0) # start time
    end = datetime.time(6, 0, 0 ) # end time
      
    current = temp2.time() # current time
    current_date = temp2.date() # current date
     
    tweet_time_in_range = time_in_range(start,end,current)
    
    if tweet_time_in_range != True:
      # == True --> Irregular  
      # != True --> Regular
      time_range_list.append(current)
      time_freq += 1
      date_range_list.append(current_date)

      # Append to new lists
      date_list.append(date_row)
      tweet_list.append(tweet_row)

  # initialize data of lists.
  time_data = {'date': date_list,'tweet': tweet_list}
 
  # Create DataFrame
  con_df = pd.DataFrame(time_data)

  return con_df
```

### Converting to current Geographical UTC time

```Python
def time_conversion(time_df):
  for i, row_value in time_df['date'].iteritems():
    current_utc_time = row_value
    temp_object = datetime.strptime(current_utc_time, "%Y-%m-%d %H:%M:%S")

    # UTC time conversion --> + or - hours 
    
    temp_hours_from_now = temp_object + timedelta(hours=10) # Australia
    # temp_hours_from_now = temp_object + timedelta(hours=6) # BD
    # temp_hours_from_now = temp_object + timedelta(hours=8) # BD time ++
    # temp_hours_from_now = temp_object + timedelta(hours=4) # BD time --
    # temp_hours_from_now = temp_object + timedelta(hours=12) # New Zealand
    # temp_hours_from_now = temp_object + timedelta(hours=1) # UK
    # temp_hours_from_now = temp_object + timedelta(hours=2) # South Africa
    # temp_hours_from_now = temp_object - timedelta(hours=4) # USA
    
    time_df['date'][i] = temp_hours_from_now

  return time_df
```

## Data Preprocessing

For raw data pre-processing, we discarded the username and mentions. We translated tweets that were in other languages to English using langdetect and googletrans. We kept the users' retweets because retweeting shows the various personality traits of the users. We removed hashtags and converted them into text. We also removed URLs and HTTP links. We also converted contractions, e.g., "they're" to "they are." We tokenize the sentences and removed punctuation using the ntlk library. Finally, we removed emojis by using the Python demoji package.

```Python
# Function to Preprocess Data

def data_preprocessing(text_sample):
  # Regex Patterns
  smileemoji = r"[8:=;]['`\-]?[)d]+"
  sademoji = r"[8:=;]['`\-]?\(+"
  neutralemoji = r"[8:=;]['`\-]?[\/|l*]"
  lolemoji = r"[8:=;]['`\-]?p+"
  userPattern = '@[^\s]+ '
  urlPattern = r"((http://)[^ ]*|(https://)[^ ]*|(www\.)[^ ]*)"

  # Use demoji
  text_sample = d.replace_with_desc(text_sample, "")

  text_sample = text_sample.lower() # Convert text to lowercase

  text_sample = re.sub(userPattern,'', text_sample) # Remove mentions

  text_sample = re.sub(urlPattern,'',text_sample) # Remove urls

  text_sample = re.sub(r'^rt[\s]+', '', text_sample) # Remove the Retweets Symbol RT

  text_sample = re.sub(r'#', '', text_sample) # Remove Hastags

  text_sample = re.sub(r'[0-9]', '', text_sample) # Remove Digits
  
  text_sample = re.sub(r'\n', '', text_sample) # Remove \n in text

  # Remove Emojis
  text_sample = re.sub(r'<3', 'heart', text_sample)
  text_sample = re.sub(smileemoji, 'smile', text_sample)
  text_sample = re.sub(sademoji, 'sad', text_sample)
  text_sample = re.sub(neutralemoji, 'neutral', text_sample)
  text_sample = re.sub(lolemoji, 'lol', text_sample) 

  text_sample = re.sub(" ’ ", "'", text_sample)
  text_sample = re.sub("’", "'", text_sample)

  # Replace Contractions

  for word in text_sample.split():
    if word.lower() in contractions:
      text_sample = text_sample.replace(word, contractions[word.lower()])

  # instantiate the tokenizer class
  tokenizer = TweetTokenizer(preserve_case=False, strip_handles=True,reduce_len=True)

  tweet_tokens = tokenizer.tokenize(text_sample)

  # Remove punctuations 
  tweets_clean = []

  for word in tweet_tokens:
    # Go through every word in your tokens list 
    if (word not in string.punctuation):  
      # remove punctuation
      tweets_clean.append(word)

  # Join the individual words into one complete string
  text_sample = ' '.join(tweets_clean)

  # Trim the sentence - remove empty spaces 
  text_sample = text_sample.strip()

  # Replace multiple dots with space
  text_sample = re.sub('\.\.+', '', text_sample) 
  
  # Remove single dots
  text_sample = re.sub('\.', '', text_sample) 

  # Remove large empty spaces in middle
  text_sample = re.sub("\s{2,}"," ",text_sample)
  text_sample = re.sub("\s{3,}"," ",text_sample)
  text_sample = re.sub("\s{4,}"," ",text_sample)

  rx = re.compile(r'_{2,}')
  text_sample = rx.sub('', text_sample)
  
  # Translate language for non-english speaking countries

  # Translate language
  try:
    if langdetect.detect(text_sample) != 'en':
      translator = Translator()
      translated_text = translator.translate(text_sample)
      text_sample = translated_text.text
  except:
      text_sample = "can not translate"
      
  text_sample = re.sub('…','',text_sample) # Remove
  text_sample = re.sub(",", "", text_sample)
  text_sample = re.sub("“", "", text_sample)
  text_sample = text_sample.strip()

  return text_sample

```

## Filtering Tweets using words

After filtering our dataset based on time, we applied another filtering technique to find tweets that are relevant to irregular sleeping and regular sleeping patterns for our final dataset. The filtering technique is based on users’ way of using certain words as we mentioned in the data collection section.

### Finding out LIWC correlated categories

Linguistic Inquiry and Word Count (LIWC) is a text analysis program that counts words in psychologically significant categories. LIWC enables users to examine the inner workings of literary works. When a user uploads a text to LIWC, he or she will receive an output with more than 70 columns of data. Here, we have analyzed users’ time-filtered tweets using LIWC, which sorts different features from texts into more than 70 different categories. To uncover the correlated LIWC word categories, we used Fisher’s linear discriminant analysis with IBM Corp’s SPSS statistical program. We got 34 word categories in total with high correlation coefficient scores. Categories with higher scores (mostly greater than 2.0) are better predictors.  The scores are in the range from 2.686 to 42.778.

```Python
# Correlated LIWC categories

liwc_category = {
    "posemo": ['good','love','happy','hope','heart','smile','laugh','kiss','wink','hug'],
    "anger":['angry','hate','mad','frustration','pouting'],
    "anx":['worry','fear','afraid','nervous','neutral','confuse','tired'],
    "sad": ['sad','disapoint', 'cry','tears'],
    "future": ['will', 'going to', 'have to', 'may'],
    "negate":['not', 'no', 'never', 'nothing'],
    "pronoun" : ['you', 'it', 'yourself', 'myself', 'we', 'our', 'us', 'me', 'my'],
    "adverb" : ['so', 'just', 'about', 'there'],
    "conj" : ['and', 'but', 'so', 'as'],
    "quant" : ['all', 'one', 'more', 'some'],
    "number" : ['one', 'two', 'first', 'once'],
    "swear" : ['shit', 'fuckin', 'fuck', 'damn'],
    "family" : ['parent', 'mother', 'father', 'baby'],
    "friend" : ['friend', 'boyfriend', 'girlfriend', 'dude'],
    "insight" : ['think', 'know', 'consider'],
    "cause" : ['because', 'effect', 'hence'],
    "discrep" : ['should', 'would', 'could'],
    "tentat" : ['maybe', 'perhaps', 'guess'],
    "certain" : ['always', 'never'],
    "inhib" : ['block', 'constrain', 'stop'],
    "incl" : ['with', 'include'],
    "excl" : ['But', 'without', 'exclude'],
    "hear" : ['listen', 'hearing'],
    "body" : ['cheek', 'hands', 'spit', 'face', 'arm', 'leg'],
    "health" : ['medic', 'patients', 'physician', 'health'],
    "sexual" : ['sex', 'gay', 'pregnant', 'dick'],
    "ingest" : ['dish', 'eat', 'pizza', 'dinner', 'lunch'],
    "relativ" : ['area', 'exit', 'stop'],
    "achieve" : ['earn', 'hero', 'win'],
    "leisure" : ['cook', 'chat', 'movie', 'apartment', 'kitchen'],
    "money" : ['business', 'pay', 'price', 'market'],
    "death" : ['death', 'dead', 'die', 'kill'],
    "assent" : ['yeah', 'yes', 'okay', 'ok'],
    "filler"  : ['wow', 'you know']
}
```

### Filtering using WordNet

WordNet provides a list of the alternatives from which different words can be taken. Words and word senses can have a far more comprehensive range of semantic links in WordNet. Nouns, verbs, adjectives, and adverbs are organized into synsets of cognitive synonyms, each expressing a separate notion. In our experiment, from the selected LIWC categories in the subsection, we created a word pool for each type. This wordpool contains example words from the LIWC dictionary. We will find synonyms of the words in the word pool by using WordNet. We then checked if a tweet had a word present in the wordpool. For a particular user, if a word in the wordpool is present in the tweet, we consider that tweet. Otherwise, that tweet was discarded. Finally, the user CSV files were updated with word-filtered tweets. 

#### WordPool Generation

```Python
def create_wordpool(word_list):
  wordlist_synonyms = []
  for k in range(len(word_list)):
    #print(posemo[k])
    for syn in wn.synsets(word_list[k]):
      for l in syn.lemmas():
          wordlist_synonyms.append(l.name())
    #print(f"{word_list[k]} --> {wordlist_synonyms}")
  wordlist_synonyms = list(dict.fromkeys(wordlist_synonyms)) # remove duplicates

  for i in range(len(wordlist_synonyms)):
    if "_" in wordlist_synonyms[i]:
      wordlist_synonyms[i] = re.sub(r'_', ' ', wordlist_synonyms[i])
    if "-" in wordlist_synonyms[i]:
      wordlist_synonyms[i] = re.sub(r'-', ' ', wordlist_synonyms[i])
  #print(wordlist_synonyms)
  
  return wordlist_synonyms
```

#### Create Wordpools

```Python
def start_wordpool(liwc_category):
  synonym_list = []
  for key in liwc_category.keys():
    word_list = liwc_category[key]
    # print(word_list)
    synonym_list = create_wordpool(word_list)
    #print(synonym_list)
    liwc_category[key] = synonym_list
  #print(liwc_category)
  return liwc_category
```

#### Filter tweets based on the words in the wordpool

```Python
def word_filtering(data,liwc_category):
  selected_tweets = []
  for key in liwc_category.keys():
    word_list = liwc_category[key] # liwc category - word list array
    for i in range(len(word_list)): # access each word in list array
      word = word_list[i]
      for  j,row in data.iterrows():
          tweet_row = row['processed_tweet']
          test = tweet_row.split()    
          for k in test:
            if k == word:
              selected_tweets.append(tweet_row)
    tweets_non_redundent = list(dict.fromkeys(selected_tweets))        

  return tweets_non_redundent
```

### Contextualized Word Embeddings using BERT

We used Bidirectional Encoder Representations from Transformers (BERT) to generate contextualized word embeddings. We generated BERT vectors using a pre-trained model named [Bert-based-uncased](https://huggingface.co/bert-base-uncased) from Hugging Face sentence-transformers library. The tweet sentences were converted into a 768-dimensional vector space. Shape of the data after word embedding is 924x768. BERT word embeddings is our final dataset which contains both psycholinguistic and word embedding–based analyses regarding the users' tweets. These analysis are the independent variables, while the dependent variable is Irregular Sleep Yes and Irregular Sleep No. BERT word embeddings is our final dataset which is used as input to our classification models (deep learning architectures). 


