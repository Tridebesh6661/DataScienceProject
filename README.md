# DataScienceProject
# This is a show of a data science project I created using python. The main tasks performed in this project are web scraping and text analysis
# !/usr/bin/env python 
# coding: utf-8

# In[2]:


pip install pandas beautifulsoup4 requests


# In[3]:


pip install tqdm


# In[1]:


import re
import os
import docx2txt


# In[2]:


import nltk
nltk.download('punkt')


# In[3]:


from nltk.tokenize import word_tokenize


# In[4]:


stop_words_files = {
    'auditors': os.path.join('C:\\Users\\KIIT\\Downloads\\StopWords', 'StopWords_Auditor.txt'),
    'currencies': os.path.join('C:\\Users\\KIIT\\Downloads\\StopWords', 'StopWords_Currencies.txt'),
    'datesandnumbers': os.path.join('C:\\Users\\KIIT\\Downloads\\StopWords', 'StopWords_DatesAndNumbers.txt'),
    'generic': os.path.join('C:\\Users\\KIIT\\Downloads\\StopWords', 'StopWords_Generic.txt'),
    'genericlong': os.path.join('C:\\Users\\KIIT\\Downloads\\StopWords', 'StopWords_Genericlong.txt'),
    'names': os.path.join('C:\\Users\\KIIT\\Downloads\\StopWords', 'StopWords_Names.txt'),
    'geographics': os.path.join('C:\\Users\\KIIT\\Downloads\\StopWords', 'StopWords_Geographic.txt')
}


# In[5]:


stop_words = {}
for category, file_path in stop_words_files.items():
    with open(file_path, 'r', encoding='latin-1') as file:
        words = file.read().split()
        stop_words[category] = set(words)


# In[6]:


def clean_text(text):
    # Convert the text to lowercase
    text = text.lower()

    # Remove stop words from each category
    for category, words in stop_words.items():
        text = ' '.join(word for word in text.split() if word not in words)

    # Remove extra whitespaces
    text = re.sub(' +', ' ', text)

    return text


# In[7]:


# Define the file paths for the positive and negative word lists
positive_words_file = r'C:\Users\KIIT\Downloads\MasterDictionary\positive-words.txt'
negative_words_file = r'C:\Users\KIIT\Downloads\MasterDictionary\negative-words.txt'

# Load positive words from file into a set
with open(positive_words_file, 'r', encoding='latin-1') as file:
    positive_words = set(file.read().split())

# Load negative words from file into a set
with open(negative_words_file, 'r', encoding='latin-1') as file:
    negative_words = set(file.read().split())


# In[25]:


pos =0
neg =0
word_count =0
line_count =0
word = 0
line = 0
avg =0.0
word_count1 = 0
word1 = 0


# In[33]:


import pandas as pd
import requests as rq
import bs4
from tqdm import tqdm
import os

# Read the Excel file
df = pd.read_excel('C:/Users/KIIT/Downloads/Input.xlsx')
output_df = pd.DataFrame(columns=['URL', 'Head'])
# Iterate over each row in the DataFrame
for index, row in df.iterrows():
    url = row['URL']
    res = rq.get(url)
    soup = bs4.BeautifulSoup(res.text,'html.parser')
    head = soup.find('head')
    s = soup.find_all("p")
    for i in s:
        cont = i.get_text()
        cleaned_text = clean_text(cont)
        tokens = word_tokenize(cleaned_text)
        tokens1 = word_tokenize(cont)
        word_count += len(tokens1)
        word_count1 += len(tokens)
        line_count += 1
        positive_score = sum(1 for token in tokens if token in positive_words)
        negative_score = sum(1 for token in tokens if token in negative_words) * -1
        pos = pos+positive_score
        neg = neg+negative_score
        word =  word+word_count
        line = line+line_count
        word1 = word1+word_count1
    polarity_score = (pos + (neg)) / ((pos - (neg)) + 0.000001) 
    subjectivity_score = (pos + neg)/(word1+0.000001)
    avg = word/line
    print("Positive Score:", pos)
    print("Negative Score:", neg)
    print("Polarity Score:", polarity_score)
    print(word,line,int(avg),subjectivity_score)
    head_text = head.text.strip() if head else ''
    # Append the extracted values to the output DataFrame
    output_df = output_df.append({'URL': url,
                                  'Head': head_text,
                                  'Positive Score':pos,
                                  'Negative Score':neg,
                                  'Polarity':polarity_score,
                                  'No of words':word,
                                  'No of Lines':line,
                                  'Average No Of Words Per Line':int(avg),
                                  'Subjectivity Score(x10^-2)': subjectivity_score*100}, ignore_index=True)
    pos = 0
    neg = 0
    positive_score=0
    negative_score=0
    word= 0
    line = 0
    word_count = 0
    line_count = 0
    avg = 0.0
    word_count1 = 0
    word1 = 0
output_path = 'C:/Users/KIIT/Downloads/Outputss.xlsx'
output_df.to_excel(output_path, index=False)
