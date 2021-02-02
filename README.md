# Ptt_WomenTalk_crawler
#python #crawler #ptt

```python
#以更新為使用api抓
from bs4 import BeautifulSoup
import re
import requests
from datetime import date
import xlwt

print(date.today())
workbook = xlwt.Workbook()
worksheet = workbook.add_sheet('Dcard生理用品看板文章')
style = xlwt.XFStyle()
font = xlwt.Font()
font.bold = True 
style.font = font 

worksheet.write(0,0,'標題', style)
worksheet.write(0,1,'連結', style)
worksheet.write(0,2,'內文', style)
worksheet.write(0,3,'Tags', style)

tampon = 0
napkin = 0
cup = 0
pants = 0

t_tampon = 0
t_napkin = 0
t_cup = 0
t_pants = 0

res = requests.get('https://www.dcard.tw/f/menstrual?latest=true')
if res.status_code == requests.codes.ok:
    soup = BeautifulSoup(res.text, 'html.parser', from_encoding = 'utf-8')
    #stories = soup.find_all('article', class_ = 'sc-1v1d5rx-0')  
    stories = soup.select('article.sc-1v1d5rx-0 h2 a')
    row = 1
    for s in stories:
        worksheet.write(row,0,s.text)
        worksheet.write(row,1,'https://www.dcard.tw'+s.get('href'))
        
        print(s.text)
        print('https://www.dcard.tw'+s.get('href'))
        
        if('棉條' in s.text or '衛生棉條' in s.text):
            t_tampon = t_tampon+1
        if('衛生棉' in s.text):
            t_napkin = t_napkin+1
        if('月亮杯' in s.text or '月釀杯' in s.text):
            t_cup = t_cup+1
        if('衛生棉褲' in s.text):
            t_pants = t_pants+1                   
        res1 = requests.get('https://www.dcard.tw'+s.get('href'))
        soup1 = BeautifulSoup(res1.text, 'html.parser', from_encoding = 'utf-8')
        article= soup1.select('article div.sc-4ihej7-0')
        if(len(article)>0):
            worksheet.write(row,2,article[0].text)
        lis= soup1.select('div.sc-405p19-6 li')
        tags = []
        if(len(article)>0):
            for l in lis:
                tags.append(l.text)
                if(l.text == '棉條' or l.text == '衛生棉條'):
                    tampon = tampon+1
                if(l.text == '衛生棉'):
                    napkin = napkin+1
                if(l.text == '月亮杯'):
                    cup = cup+1
                if(l.text == '衛生棉褲'):
                    pants = pants+1
            print(tags)
            worksheet.write(row,3,str(tags))        
        row = row+1
    worksheet2 = workbook.add_sheet('統計')
    worksheet2.write(0,1,'Tags', style)
    worksheet2.write(0,2,'Title關鍵字', style)
    worksheet2.write(1,0,'棉條', style)
    worksheet2.write(1,1,tampon)
    worksheet2.write(1,2,t_tampon)
    worksheet2.write(2,0,'衛生棉', style)
    worksheet2.write(2,1,napkin)
    worksheet2.write(2,2,t_napkin)
    worksheet2.write(3,0,'月亮杯', style)
    worksheet2.write(3,1,cup)
    worksheet2.write(3,2,t_cup)
    worksheet2.write(4,0,'衛生棉褲', style)
    worksheet2.write(4,1,pants)
    worksheet2.write(4,2,t_pants)
    workbook.save(str(date.today())+'_Dcard生理用品看板.xls')
```

    2020-06-05
    棉條小問題
    https://www.dcard.tw/f/menstrual/p/233777809
    ['棉條']
    #詢問 衛生棉條擺放
    https://www.dcard.tw/f/menstrual/p/233752752
    ['衛生棉條', '月經', '生理期']
    #分享 靠得住棉條試用心得
    https://www.dcard.tw/f/menstrual/p/233749997
    ['棉條', '月經']
    Tampax VS. Playtex 衛生棉條不專業實際使用紀錄
    https://www.dcard.tw/f/menstrual/p/233748798
    ['衛生棉條', '月經', '生理期', '棉條']
    第一次衛生棉條使用問題
    https://www.dcard.tw/f/menstrual/p/233741999
    ['衛生棉條', '月經', '生理期', '問題', '棉條']
    Kotex靠得住又有瑕疵品！
    https://www.dcard.tw/f/menstrual/p/233740695
    ['女孩', '生理期', '衛生棉', '月經']
    調理貼
    https://www.dcard.tw/f/menstrual/p/233740027
    ['問題', '私密處']
    (...省)
    


```python
#punctuation list:
punc_list=[]
file1 = open('punctuation.txt','r',encoding="utf-8")  #watch out the encoding!
Lines = file1.readlines() 
for line in Lines: 
    line = line.replace('\n', '')
    punc_list.append(line)

file1.flush()
file1.close()

#stopword list:
stopword_list=[]
file = open('stopword_chinese.txt','r',encoding="utf-8")  #watch out the encoding!
Lines = file.readlines() 
for line in Lines: 
    line = line.replace('\n', '')
    stopword_list.append(line)

file.flush()
file.close()
```


```python
from bs4 import BeautifulSoup
import re
import requests
import jieba.analyse
import jieba
from datetime import date

jieba.set_dictionary('dict.txt.big') 
jieba.load_userdict('userdict.txt')

allarticles = ''

res = requests.get('https://www.dcard.tw/f/menstrual?latest=true')
if res.status_code == requests.codes.ok:
    soup = BeautifulSoup(res.text, 'html.parser', from_encoding = 'utf-8')
    #stories = soup.find_all('article', class_ = 'sc-1v1d5rx-0')  
    stories = soup.select('article.sc-1v1d5rx-0 h2 a')
    row = 1
    for s in stories:               
        res1 = requests.get('https://www.dcard.tw'+s.get('href'))
        soup1 = BeautifulSoup(res1.text, 'html.parser', from_encoding = 'utf-8')
        article= soup1.select('article div.sc-4ihej7-0')
        if(len(article)>0):
            allarticles = allarticles + article[0].text + '\n'

fo = open(str(date.today())+"_生理用品看板最新貼文.txt", "w", encoding = 'utf-8')
fo.write(allarticles)
fo.close()

#print(allarticles)
# result = jieba.cut(allarticles, cut_all = False)  
# result_list = ','.join(result).split(',')
# term_list = [w for w in result_list if not w in stopword_list if not w in punc_list] 
# print(term_list)

jieba.analyse.set_stop_words('stopword_chinese.txt')
tags = jieba.analyse.extract_tags(allarticles, 50)
print(",".join(tags))
```

    Building prefix dict from C:\Users\User\Downloads\衛生棉爬蟲\dict.txt.big ...
    Loading model from cache C:\Users\User\AppData\Local\Temp\jieba.u2162e425c744c9e2c9f97c4d87558608.cache
    Loading model cost 0.958 seconds.
    Prefix dict has been built successfully.
    

    衛生棉,棉條,覺得,Playtex,公分,月亮杯,還是,Ultra,Tampax,時候,不會,比較,LadyCup,衛生,小時,使用,已經,Lunette,出來,感覺,Super,Pearl,外漏,經血,氣孔,拋棄,內褲,真的,不過,子宮,月經,一樣,蘇菲,Sport,滲漏,經期,大家,更換,應該,起來,雖然,Gentle,Glide,生理期,個人,還有,一個,一點,發現,試試看
    


```python
import matplotlib.pyplot as plt
import numpy as np
import jieba.analyse
import codecs
from wordcloud import WordCloud
from bs4 import BeautifulSoup
import re
import requests
import jieba
from datetime import date


jieba.set_dictionary("dict.txt.big")
jieba.load_userdict('userdict.txt')

#stopword list:
stopword_list=[]
file = open('stopword_chinese.txt','r',encoding="utf-8")  #watch out the encoding!
Lines = file.readlines() 
for line in Lines: 
    line = line.replace('\n', '')
    stopword_list.append(line)
file.flush()
file.close()
stopwords = {}.fromkeys(stopword_list)

allarticles = ''

res = requests.get('https://www.dcard.tw/f/menstrual?latest=true')
if res.status_code == requests.codes.ok:
    soup = BeautifulSoup(res.text, 'html.parser', from_encoding = 'utf-8')
    #stories = soup.find_all('article', class_ = 'sc-1v1d5rx-0')  
    stories = soup.select('article.sc-1v1d5rx-0 h2 a')
    row = 1
    for s in stories:               
        res1 = requests.get('https://www.dcard.tw'+s.get('href'))
        soup1 = BeautifulSoup(res1.text, 'html.parser', from_encoding = 'utf-8')
        article= soup1.select('article div.sc-4ihej7-0')
        if(len(article)>0):
            allarticles = allarticles + article[0].text + '\n'

def generate_wordcloud(keywords, stopwords, file_path):
    wc = WordCloud(font_path = 'msyh.ttf', background_color = "white",
                    max_words = 2000, stopwords = stopwords)
    wc.generate_from_frequencies(keywords)
    plt.imshow(wc)
    plt.axis("off")
    plt.figure(figsize = (10, 6), dpi = 100)
    plt.show()
    wc.to_file(file_path)
    
def get_keywords(article, topN):
    keywords = {}
    tags = jieba.analyse.extract_tags(article, topK = topN, withWeight = True)
    for tag, weight in tags:
        keywords[tag] = weight
    return keywords

keywords = get_keywords(allarticles, 30)
#print(keywords)
generate_wordcloud(keywords, stopwords, "pic.jpg")
```

    Building prefix dict from C:\Users\User\Downloads\衛生棉爬蟲\dict.txt.big ...
    Loading model from cache C:\Users\User\AppData\Local\Temp\jieba.u2162e425c744c9e2c9f97c4d87558608.cache
    Loading model cost 0.944 seconds.
    Prefix dict has been built successfully.
    


![png](https://i.imgur.com/vnXcdEZ.png)



    <Figure size 1000x600 with 0 Axes>



```python
#https://tlyu0419.github.io/2019/04/06/Crawl-Dcard/
#直接用api抓
import matplotlib.pyplot as plt
import numpy as np
import jieba.analyse
import codecs
from wordcloud import WordCloud
from bs4 import BeautifulSoup
import requests
import jieba
from datetime import date
import pandas as pd
import requests
from requests_html import HTML
import re

jieba.set_dictionary("dict.txt.big")
jieba.load_userdict('userdict.txt')

a_tampon = 0
a_napkin = 0
a_cup = 0
a_pants = 0

new_articles = ''

# 撰寫簡單的函數，透過輸入文章ID，就輸出文章的資料
def Crawl(ID):
    global a_tampon
    global a_napkin
    global a_cup
    global a_pants
    global new_articles
    link = 'https://www.dcard.tw/_api/posts/' + str(ID)
    requ = requests.get(link)
    rejs = requ.json()
    new_articles = new_articles + rejs['content']   #加所有的內容
    
    if('棉條' in rejs['content'] or '衛生棉條' in rejs['content']):
        a_tampon = a_tampon+1
    if('衛生棉' in rejs['content']):
        a_napkin = a_napkin+1
    if('月亮杯' in rejs['content'] or '月釀杯' in rejs['content']):
        a_cup = a_cup+1
    if('衛生棉褲' in rejs['content']):
        a_pants = a_pants+1
    return(pd.DataFrame(
        data=
        [{'ID':rejs['id'],
          'title':rejs['title'],
          'content':rejs['content'],
          'excerpt':rejs['excerpt'],
          'createdAt':rejs['createdAt'],
          'updatedAt':rejs['updatedAt'],
          'commentCount':rejs['commentCount'],
          'forumName':rejs['forumName'],
          'forumAlias':rejs['forumAlias'],
          'gender':rejs['gender'],
          'likeCount':rejs['likeCount'],
          'reactions':rejs['reactions'],
          'topics':rejs['topics']}],
        columns=['ID','title','content','excerpt','createdAt','updatedAt','commentCount','forumName','forumAlias','gender','likeCount','reactions','topics']))


# 嘗試使用撰寫出的函數，抓取編號231030181的文章
# Crawl(233752752)


# 一次讀取100篇最熱門的文章
url = 'https://www.dcard.tw/_api/forums/menstrual/posts'
resq = requests.get(url)
rejs = resq.json()
df = pd.DataFrame()

for i in range(len(rejs)):
    df = df.append(Crawl(rejs[i]['id']),ignore_index=True)
print(df.shape)
df

# 透過迴圈讀取10*100篇文章，若需讀取更多資料，可以將range(10)中的數值提升
for j in range(10):
    #last = str(int(df.tail(1).ID)) # 找出爬出資料的最後一筆ID
    url = 'https://www.dcard.tw/_api/forums/menstrual/posts'
    resq = requests.get(url)
    rejs = resq.json()
    for i in range(len(rejs)):
        df = df.append(Crawl(rejs[i]['id']), ignore_index=True)
print(df.shape)
df

df1 = pd.DataFrame([[a_tampon, a_napkin, a_cup, a_pants]],
                   index=['出現次數'],
                   columns=['棉條','衛生棉','月亮杯','生理褲'])

print(a_tampon)
print(a_napkin)
print(a_cup)
print(a_pants)

# 將資料存
# https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.to_excel.html
df.to_excel(str(date.today())+'Dcard生理用品看板.xlsx',sheet_name='生理用品看板爬蟲')
df1.to_excel(str(date.today())+'Dcard生理用品看板統計.xlsx',sheet_name='統計')


def generate_wordcloud(keywords, stopwords, file_path):
    wc = WordCloud(font_path = 'msyh.ttf', background_color = "white",
                    max_words = 2000, stopwords = stopwords)
    wc.generate_from_frequencies(keywords)
    plt.imshow(wc)
    plt.axis("off")
    plt.figure(figsize = (10, 6), dpi = 100)
    plt.show()
    wc.to_file(file_path)
    
def get_keywords(article, topN):
    keywords = {}
    jieba.analyse.set_stop_words('stopword_chinese.txt')
    tags = jieba.analyse.extract_tags(article, topK = topN, withWeight = True)
    for tag, weight in tags:
        keywords[tag] = weight
    return keywords

keywords = get_keywords(new_articles, 50)
# print(keywords)
generate_wordcloud(keywords, stopwords, "Dcard生理用品看板wordcloud.jpg")

#寫入txt
fo = open(str(date.today())+"_生理用品看板最新貼文.txt", "w", encoding = 'utf-8')
fo.write(new_articles)
fo.close()
```

    Building prefix dict from C:\Users\User\Downloads\衛生棉爬蟲\dict.txt.big ...
    Loading model from cache C:\Users\User\AppData\Local\Temp\jieba.u2162e425c744c9e2c9f97c4d87558608.cache
    Loading model cost 0.946 seconds.
    Prefix dict has been built successfully.
    

    (30, 13)
    (330, 13)
    99
    154
    44
    11
    


![png](https://i.imgur.com/GiplggJ.png)



    <Figure size 1000x600 with 0 Axes>



```python

```

