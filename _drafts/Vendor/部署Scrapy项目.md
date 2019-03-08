# scrapyd操作笔记

#### 爬虫线程 
pip install scrapyd 
#### 安装依赖（自动生成egg文件） 
pip install scrapyd-client
pip install apscheduler 
pip install requests 

#### 查看所有爬虫
curl http://localhost:6800/listspiders.json?project=VehicleOrderScrapy 
#### 查看爬虫状态 
curl http://localhost:6800/listjobs.json?project=VehicleOrderScrapy 
#### 开启爬虫 
格式举例：
curl http://localhost:6800/schedule.json -d project=myproject -d spider=somespider -d setting=DOWNLOAD_DELAY=2 -d arg1=val1 

curl http://localhost:6800/schedule.json -d project=VehicleOrderScrapy -d spider=vehicle_order_86huoche -d latestUpdateDate=2018-01-23 

#### 每次更新后需要重新部署
scrapyd-deploy vehicle_order -p VehicleOrderScrapy

#### 我现在用corn来定期执行爬虫，贴一段半成品代码
```
#coding:utf-8
import os
import time
import requests
from project_config import *
from apscheduler.schedulers.background import BackgroundScheduler
import pymysql
from datetime import datetime

LATEST_UPDATE_DATE = None
CONNECT = None

def task():
    LATEST_UPDATE_DATE = getLatestUpdateDate()
    print('LATEST_UPDATE_DATE：'+LATEST_UPDATE_DATE)

    spider_list = ['vehicle_order_58','vehicle_order_ganji','vehicle_order_baixing','vehicle_order_86huoche']
    # http://localhost:6800/schedule.json?project=VehicleOrderScrapy&spider=vehicle_order_86huoche&latestUpdateDate=2018-01-23
    for name in spider_list:
        data = {'project':'VehicleOrderScrapy','spider':name,'latestUpdateDate':formatLatestUpdateDate(name,LATEST_UPDATE_DATE)}
        print('spider-->%s'%data)
        requests.post('http://localhost:6800/schedule.json', data = data)
    updateLatestUpdateDate(time.strftime('%Y-%m-%d %H:%M:%S', time.localtime()))

def formatLatestUpdateDate(spider_name,latestUpdateDate):
    return #格式化后的字符串
def connectDB():
    global CONNECT

    dbparams = dict(
                host = db_config['MYSQL_HOST'],
                db = db_config['MYSQL_DBNAME'],
                user = db_config['MYSQL_USER'],
                passwd = db_config['MYSQL_PASSWD'],
                charset = 'utf8',#编码要加上，否则可能出现中文乱码问题
                )
    try:
        CONNECT = pymysql.connect(**dbparams)
    except Exception as e:
        raise

def closeDB():
    global CONNECT
    CONNECT.close()

def getLatestUpdateDate():
    global CONNECT

    #检测数据库连接状态，如果失联，自动连接
    CONNECT.ping(True)

    sql = "SELECT value FROM %s WHERE name=\'%s\'"%(db_config['MYSQL_SETTINGS_TABLENAME'],'latestUpdateDate')
    cursor = CONNECT.cursor()
    try:
        cursor.execute(sql)
        result = cursor.fetchone()
        if result:
            return result[0]
        else:
            return result
    except Exception as e:
        raise

def updateLatestUpdateDate(latestUpdateDate):
    global CONNECT
    #检测数据库连接状态，如果失联，自动连接
    CONNECT.ping(True)

    cue = CONNECT.cursor()
    try:
        sql = "UPDATE %s SET value = \'%s\' WHERE name=\'%s\'"
        params = (db_config['MYSQL_SETTINGS_TABLENAME'],latestUpdateDate,'latestUpdateDate')
        print (sql%params)
        cue.execute(sql%params)
    except Exception as e:
        CONNECT.rollback()
        raise
    else:
        CONNECT.commit()

if __name__ == "__main__":
    scheduler = BackgroundScheduler()
    # 每20分钟执行一次
    connectDB()
    scheduler.add_job(task, 'cron', day_of_week='tue,thu,sun', hour='23')
    scheduler.start()
    print('Press Ctrl+{0} to exit'.format('Break' if os.name == 'nt' else 'C'))
    try:
        while True:
            time.sleep(2)
    except (KeyboardInterrupt, SystemExit):
        closeDB()
        scheduler.shutdown()
```

# 参考资料
[scrapyd和scrapyd-client使用教程](http://blog.wiseturtles.com/posts/scrapyd.html)

