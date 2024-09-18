# AO3-Data-Scraping

## Table of Contents

- [Project Overview](#project-overview)
- [Methodology](#methodolody)
- [Result](#result)


  
## Project Overview
---

[Archive of our Own (Ao3)](https://archiveofourown.org/) is a noncommercial and nonprofit central hosting site that is designed and built by and for fans to post and showcase their transformative fanworks such as fanfiction, fanart, fan videos, and podfic. The goal for this project is to scrape the data in this webiste in which it will gather Gathering the fanworks title, author, date updated, fandoms, relationship tag, word numbers, chapters, and its kudos.


## Tool

- Jupyter Notebook


## Methodolody

The first thing that I did on before starting the project is go around and explore the Ao3. Which I found out the site is quite a good thing start on scraping to cause it doesn't have any complicated UI and quite basic compared the others. In addition it's HTML structure like the class and id is well organized and easily understood, therefore it won't be too much overwhelming and confusing for others play around to.

</n>
Now let's start scraping the site.



### Scraping the fanworks information

to begin the scraping I first install and import the libraries that we will be needed.

```python
from bs4 import BeautifulSoup
import requests
import pandas as pd
import time
```

After importing the necessary library, I then go forward on checking if the site is good to access to without any trouble, where I ask if the HTTP status code is == 200

```python
base_url = 'https://archiveofourown.org/tags/Haruno%20Sakura*s*Uchiha%20Obito/works'

response = requests.get(base_url)

if response.status_code == 200:
    soup = BeautifulSoup(response.text, 'html')
    print('success')
else:
    print(f'Failed: {response.status_code}')
```

Since the result is a success, we can now proceed on scraping the website. Where I start off creating 2 important functions that is required to scrape all the data in site, the pagination and extract_data. Let's first checkout the extract_data funtion.

</n>

The purpose on the extrac_data funtion, just like the variable name itself, we used this to extract the data on a page on the site. In which I mined the fanfictions title, authors, fandoms, relationships, dates, word numbers, chapters, and kudos, and I then append it to its designated variable list. After doing so, using the library pandas, I created a DataFrame where I placed all the data lists it there, and zip it all together to combine all of it by row. I then added columns to represent as a header of the DataFrame. After all of that, I then return the df, where it will direct to the pagination function that I will discuss next.

```python
def extract_data(soup):    
    titles = []
    authors = []
    fandoms = []
    relationships = []
    dates = []
    word_nums = []
    chapters = []
    kudos = []

    bookmarks = []

    articles = soup.find_all('li',role="article")

    for article in articles:
        # finding the title
        heading = article.find('h4', class_='heading').find('a') if article.find('h4', class_='heading') else None
        if heading:
            title = heading.text
            if title:
                titles.append(title)
        
        # finding author
        heading1 = article.find('h4', class_='heading')
        if heading1:
            author = heading1.find('a', rel = 'author')
            if author:
                authors.append(author.text)
            else:
                authors.append('Anonymous')

        # find the fandom
        heading2 = article.find('h5', class_ = 'fandoms heading')
        if heading2:
            span = heading2.find(class_ = 'tag')
            if span:
                fandoms.append(span.text)
        
        # finding relatioships
        heading3 = article.find('ul', class_='tags commas')
        if heading3:
            head = heading3.find('li', class_ = 'relationships')
            if head:
                rela = head.find('a', class_ = 'tag')
                if rela:
                    relationships.append(rela.text)
            else:
                relationships.append('None')
            

    
        # finding dates and filtering it
        heading4 = article.find('ul', class_ = 'recent module group')
        if heading4:
            bookmark = heading4.find(class_ = 'datetime')
            if bookmark:
                bookmarks.append(bookmark.text)

        date = article.find(class_ = 'datetime')
        if date:
            dates.append(date.text)    

        #filtering
        dates =[item for item in dates if item not in bookmarks] 

        # word numbers
        word_num = article.find('dd', class_ = 'words')
        if word_num:
            word_nums.append(word_num.text)

        # chapters
        heading5 = article.find('dl', class_= 'stats')
        if heading5:
            chapter = heading5.find('dd', class_ ='chapters')
            if chapter:
                chapters.append(chapter.text)
        
        # kudos
        kudo = article.find('dd', class_='kudos')
        if kudo:
            kudos.append(kudo.text)
    
    # creating dataframe
    df = pd.DataFrame(list(zip(titles, authors, fandoms,relationships, dates,
                               word_nums, chapters,kudos)), columns=['Title', 'Author', 'Fandom', 'Relationship', 'Date',
                            'Word Number', 'Chapters','Kudos'])
    
    return df
```

Now let's talk about the pagination function. Since Ao3 is paginated, meaning the sites content is divided my pages. I have to use this function to access all the pages and scrape it by each. What I did is I edit the base url where I added ?page={page_num} on it's end. This change of page parameter can help us successfully go to the page. After that It will then called the function we discuss earlier, the extract_data to collect the page data, where the return df will then be placed inside the all_data list. Afterward, the page_num will increment into 1 to move on the next page of the site. Another thing that it's important to also include, is everytime we move into the next page we must used the time.sleep(1) in between, so that the site will not get triggered by a sudden data scrapping. Just like what my instructor said, this small pause is like "softly knocking on the door" and waiting for a response, ensuring we don't overwhelm the site with too many requests in a short period.

When page_data is empty, it willl break the loop and it will return thr DataFrame of our gathered data

```python
def pagination(base_url):
    all_data = []
    page_num = 1

    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.3'
    }

    while True:
        url = f'{base_url}?page={page_num}'

        print(f'scrapping page num = {page_num} in url: {url}')
        
        response = requests.get(url, headers=headers)
        soup = BeautifulSoup(response.text, 'html')

        page_data = extract_data(soup)

        if page_data.empty:
            break

        all_data.append(page_data)
        page_num += 1
        time.sleep(1)

    if all_data:
        return pd.concat(all_data, ignore_index=True)
```

### Result
Here is the csv result of the DataFrame
![alt text](https://github.com/llaight/AO3-Data-Scraping/blob/main/ao3data.PNG)
