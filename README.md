# python-
pandas模块
#爬取网页
1、已知网页：https://www.qidian.com/rank/yuepiao?chn=-1&page=1
是起点网的18年1月份的月票榜。
共计：  25页(page=25)
2、抓取名称、作者、类型、是否连载、最近一次更新时间、月票个数
   存储为csv格式
3、将更新时间拆分细化为：年、月、日、时 几列
4、有没有作者本月有多个作品上榜？有的话是谁？
5、哪一类作品上榜数量最多
6、票数最多的三类作品
7、作者更新时间更偏向于哪个时间段
8、抓取页面的种类作品的数量的数据，与该类作品月票上榜的月票总数，制作一个参考曲线图，分析之间的关系。
 
#coding:utf-8
import re
import urllib2
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
name = []
author = []
typa = []
serial = []
update = []
number = []
for i in range(1,26,1):
    url = 'https://www.qidian.com/rank/yuepiao?chn=-1&page='+str(i)
    req = urllib2.Request(url, headers={'User-Agent': 'Magic Broswer'})
    strw = urllib2.urlopen(req).read()
    start = strw.find('<div class="book-img-text">')
    end = strw.find('<div class="page-box cf" data-eid="qd_C44">')
    tmp = strw[start:end]
    a = tmp.split('<div class="book-img-box">')
    for j in a[1:]:
        b = re.findall('target="_blank" data-eid="qd_C40" data-bid="\d+">(.+)</a></h4>', j)
        name.append(b[0])
        c = re.findall(' target="_blank" data-eid="qd_C41">(\W+)</a><em>|</em><a ', j)
        author.append(c[0])
        d = re.findall('qd_C42">(\W+)</a><em>', j)
        typa.append(d[0])
        e = re.findall('</em><span>(\W+)</span>', j)
        serial.append(e[0])
        f = re.findall('</a><em>&#183;</em><span>(.{16})', j)
        update.append(f[0])
        g = re.findall('<p><span>(.+)</span>月票</p>', j)
        number.append(g[0])
df1 = pd.DataFrame({'name':name,'author':author,'typa':typa,'serial':serial,'update':update,'number':number})
df1.to_csv('qidian.csv')

url = 'https://www.qidian.com/'
req = urllib2.Request(url, headers={'User-Agent': 'Magic Broswer'})
strw = urllib2.urlopen(req).read()
start = strw.find('qd_A71"><cite><em class="iconfont">')
end = strw.find('<div class="focus-slider">')
tmp = strw[start:end]

month = []
year = []
day = []
hour = []
for i in update:
    time = re.findall('(\d{4})-(\d{2})-(\d{2}) (\d{2}):(\d{2})',i)
    year.append(time[0][0])
    month.append(time[0][1])
    day.append(time[0][2])
    hour.append(time[0][3])
df2 = pd.DataFrame({'year':year,'month':month,'day':day,'hour':hour})
df2.to_csv('update.csv')
newdf = pd.concat([df1,df2],axis=1)
#print newdf
#sort_df = newdf.sort_values(by='number',ascending=False)
#print sort_df['typa'].head(1)
print newdf['typa'].value_counts(ascending=False).head(1)
print newdf['number'].groupby(newdf['typa']).sum()
#print  newdf['number'].groupby(newdf['typa']).size()
