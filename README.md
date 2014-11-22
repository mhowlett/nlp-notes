# Strategy for Extraction of Predictions from Arbitrary Bodies of Text

This is a specific case of information extraction. 

We want to extract relations of the form:

(person or entity)  is long/short  (security/topic name).

from arbitrary natural language text. Also, there my be meta-data available (such as prior 
knowledge that an article is by a particular person, or categorized in a certain way).

General procedure:

1. Perform any processing required to extract meta data.

2. Pre-process input data, for example with boilerpipe in order to remove content that is unlikely to 
   be useful.

3. Tokenize the text with a custom tokenizer appropriate to our problem (fairly similar to standard 
   tokenizers that are available).

4. Perform POS tagging on the tokenized text. Due to our custom tokenization, we need to train 
   a POS tagger for this task (we use a standard methodology).

5. Identify and tag tokens or groups of tokens that:
   1. correspond to stock symbols or other topics known to our system. 
   2. correspond to company names known to our system.
   3. correspond to people/entities known to our system.

   Identifying company names is the most difficult of these and described in more detail below:

   1. For each company known to us, we construct a list of alternatives by which it may be
      referred. For example Apple or Apple Inc. These lists are partly generated automatically
      and partly by hand. For example, in the above example, we know that Inc or Inc. 
      are common extensions that may be commonly appended to company names, and which also
      may be commonly omitted. IBM may also be referred to as International Business Machines.
      In this case, we explicitly store IBM as an alternate name for International Business 
      Machines in our system. 

   2. We do a dumb match of our tokenized text with our company name list. This is done greedily 
      (longest alternative first). In many instances this match will not correspond to a real 
      mention of the company. For example, the word Apple can be used in other contexts. We train a 
      binary classifier to make this decision. Features used by this classifier are POS tags of
      the tokens immediately prior and after the company, whether or not the company name ends
      with a standard token (eg inc.), whether or not the candidate company
      name appears in an english dictionary, and whether or not there is a stock symbol in nearby
      text that corresponds to the company name.

6. Identify and tag other miscellaneous token types specific to our domain including:
   1. Dollar amounts
   2. Day of week.
   3. Month of year.
   4. market direction token (long/short/bearish/bullish, etc.)

   In some cases, (eg month of year), the task requires a classifier methodology similar to company
   name. In some cases, text pattern matching is sufficient.

7. The tokenized text has now been tagged with quite a lot of high level information and we are
   ready for prediction extraction. First, we decide on some features on which to anchor our 
   search for predictions. Currently, we consider any occurance of a person name, and 
   a company name in text separated by not more than 10 additional tokens as a candidate prediction.
   We train a binary classifier to decide whether a candidate prediction is an actual prediction.
   For each positively identified prediction, we then train another classifier to decide whether
   the prediction is bearish, neutral or bullish.
   
   Features used by the first classifier include: 
    1. number of tokens between company name and person name.
    2. bag of words betwetween company name and person name.
    3. POS tags of tokens immediately before and after 
    4. existance of a stock symbol name 
    5. existance of a market direction token.
   
   Features used by the second classifier include:
    1. Existance of a market direction token and it's value.
    2. other features.




