# 轻小说文库 wenku8

环境：

- python3.7
- BeautifulSoup4

代码 **Crawler.py**

```python
# -*- coding: utf8 -*-

from bs4 import BeautifulSoup
import re
import os
import requests

class Crawler:
    """
    www.wenku8.net 爬虫脚本
    -------

    使用方法

    >>> crawler = Crawler('https://www.wenku8.net/novel/1/1269/index.htm',basic_url='https://www.wenku8.net/novel/1/1269/',basic_dir='temp_dir')
    >>> crawler.handle()
    """
    def __init__(self,entry_url,basic_url,basic_dir='temp_dir',features='html.parser',encoding='gbk'):
        """
        init
        """
        self.entry_url = entry_url
        self.basic_url = basic_url
        self.basic_dir = basic_dir
        self.features = features
        self.encoding = encoding

    def handle(self):
        """
        开始爬虫
        <hr/>
        包括
        """
        r = requests.get(self.entry_url)
        r.encoding = self.encoding
        soup = BeautifulSoup(r.text,self.features)
        title,f,mf = None,None,None
        if os.path.isdir(self.basic_dir + '/pictures')  == False:
            os.makedirs(self.basic_dir + '/pictures')
        else:
            print(self.basic_dir+'目录已经存在，请先删除，再重新运行')
            return False
        for val in soup.table.findAll('td'):
            # print(val)
            if re.match('^\s*\S[\S\s]*$',val.text):
                if 'vcss' in val['class']: # 第几卷
                    if title:
                        f.close() # 关闭上一个章节的文件
                        mf.close()
                    title =  val.text
                    re_title = re.sub('\s+','-',title) # 过滤空格的标题
                    f = open(self.basic_dir + '/'+re_title+'.md', mode='a',encoding='utf-8')
                    f.write('# '+title+'\n\n')
                    mf = open(self.basic_dir + '/_sidebar.md', mode='a',encoding='utf-8')
                    mf.write('* ['+title+'](/'+self.basic_dir+'/'+re_title+')\n')
                else:
                    self.getPageContent(self.basic_url + val.a.attrs['href'],f = f,title = title)
        f.close()
        mf.close()
    def getPageContent(self,url,f,title):
        r = requests.get(url)
        r.encoding = self.encoding
        soup = BeautifulSoup(r.text,self.features)
        sub_title = soup.find('div',attrs = {'id':'title'}).text
        sub_match = sub_title.replace(title,'')
        print(sub_match)
        f.write('## '+sub_match+'\n'+soup.find('div',attrs = {'id':'content'}).text+'\n---\n')
        pngs_soup = soup.findAll('div',class_='divimage')
        if pngs_soup:
            self.getPagePicture(pngs_soup,f=f,title=title,sub_title=sub_title)


    def getPagePicture(self,pngs_soup,f,title,sub_title):
        pattern_t = re.compile("\S+")
        m = pattern_t.search(title)
        title_prefix = m.group(0)
        i = 1
        pattern_p = re.compile("(jpg|png|jpeg)$")
        for val in pngs_soup:
            pic_src = val.find('img')['src']
            print(pic_src)
            r = requests.get(pic_src,stream=True)
            m = pattern_p.search(pic_src)
            pic_name = str(i).zfill(2) +'.' + m.group(0)
            pic_path = 'pictures/' + title_prefix+'_'+ pic_name
            with open(self.basic_dir + '/' + pic_path, 'wb') as ff:
                ff.write(r.content)
                f.write('!['+pic_name+'](/'+pic_path+')\n')
                ff.close()
                i = i + 1

    def test(self,url):
        r = requests.get(url)
        r.encoding = self.encoding
        soup = BeautifulSoup(r.text,self.features)
        pngs_soup = soup.findAll('div',class_='divimage')
        pattern = re.compile("(jpg|png|jpeg)$")
        for val in pngs_soup:
            print(val.find('img')['src'])
            m = pattern.search(val.find('img')['src'])
            print(m.group(0))


crawler = Crawler('https://www.wenku8.net/novel/1/1269/index.htm',basic_url='https://www.wenku8.net/novel/1/1269/',basic_dir='NoGameNoLife',features='lxml')
crawler.handle()
```