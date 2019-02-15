# python爬虫小计

##笔趣阁小说目录爬虫

    import requests
    from bs4 import BeautifulSoup
    def main():
        target='https://www.biquge5200.com/38_38857/'
        req=requests.get(url=target)
        bf=BeautifulSoup(req.text,'html.parser')
        content=bf.find_all('div',id='list')
        bf_a=BeautifulSoup(str(content[0]),'html.parser')
        a=bf_a.find_all('a')
        for each in a:
            print(each.string,each.get('href'))
    if __name__=='__main__':
        main()

##杭电oj爬虫

    import requests
    import time
    import random
    import re
    from bs4 import BeautifulSoup
    oj_id=[]
    oj_name=[]
    oj_ac=[]
    oj_su=[]
    def get_html(url):
        try:
            
            request=requests.get(url,timeout=3)
            random_time=random.randint(2,5)
            time.sleep(random_time)
            return request.text
        except :  
            return ""
    def get_oj():
        t=0
        for i in range(0,56):
            
            url='http://acm.hdu.edu.cn/listproblem.php?vol='+str(i)
            html=get_html(url)
            soup=BeautifulSoup(html,"html.parser")
            c=1
            for a in soup.find_all('script'):
                if c==5:
                    strr=a.string
                    listt=re.split("p\(|\);", strr)
                    for b in listt:
                        if b!='':
                            temp = re.split(',', b)
                            lenn=len(temp)
                            oj_id.append(temp[1])
                            oj_name.append(temp[3])
                            oj_ac.append(temp[lenn-2])
                            oj_su.append(temp[lenn-1])
                c=c+1
            t=t+1
            print('\r当前进度：{:.2f}%'.format(t * 100 / 55, end=''))
    def main():
        get_oj()
        root='D://a//1.txt'
        lenn=len(oj_id)
        for i in range(0,lenn):
            with open(root,'a',encoding='utf-8') as k:
                k.write('hdu'+oj_id[i]+','+oj_name[i]+"," + oj_ac[i] + "," + oj_su[i] + '\n')
    
    if __name__=='__main__':
        main()

