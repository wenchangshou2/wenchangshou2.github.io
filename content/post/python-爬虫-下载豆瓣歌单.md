---
title: "Python 爬虫 下载豆瓣歌单"
date: 2020-02-11T14:15:06+08:00
---


多线程下载douban歌单，BeautifulSoup解析标签
<!--more-->
运行的时候输入歌单的网址：https://music.douban.com/programme/9574867?sid=#play ，爬虫会自动抓取页面中所有音乐的下载链接

``` python
import requests
import simplejson as json
from bs4 import BeautifulSoup
import queue
import threading
import os

myQueue=queue.Queue()
MusicList=[]
class Music:
    def __init__(self,title,DownUri):
        self.title=title
        self.DownUri=DownUri

class MyThread(threading.Thread):
    def __init__(self,records, page_num):
        threading.Thread.__init__(self, name=page_num)
        self._records =records

    def run(self):
        while(self._records.qsize()):
            #page=read_page(url,self._records.get())
            #read_tag(page,tag)
            ll=self._records.get()
            print(ll.title,"正在下载")
            DownMusicFile(ll.DownUri,ll.title)
            print(ll.title,"下载完成")


def Resulturi(songid,ssid):
    uri='https://music.douban.com/j/songlist/get_song_url?sid=%s&ssid=%s'%(songid,ssid)
    content=json.loads(requests.session().get(uri).content)['r']
    return content

def GetDown(MusicUri):
    page=requests.session().get(MusicUri)
    content=page.content
    soup=BeautifulSoup(content,"html5lib")
    link = soup.find_all("div","song-item")
    for s in link:
        #print(i['data-index'],i['data-title'],i['data-performer'],i['data-songid'],i['data-ssid'])
        DownUri=Resulturi(s['data-songid'],s['data-ssid'])
        tmp=Music(s['data-title'],DownUri)
        myQueue.put(tmp)

def DownMusicFile(uri,title):
    r=requests.get(uri,stream=True)
    with open(os.getcwd()+'/'+title+".mp4",'wb') as fd:
        for chunk in r.iter_content():
            fd.write(chunk)


def reptile(records,threadNum):
    tasks=[]

    for page_num in range(1,10):
        Thread=MyThread(records,page_num)
        Thread.setDaemon(False)
        Thread.start()
        tasks.append(Thread)
    for task in tasks:
        if task.isAlive():
            tasks.append(task)
            continue

if __name__ == '__main__':
    MusicUri=input("请输入需要下载的专辑网址")
    GetDown(MusicUri)
    reptile(myQueue,10)

```
