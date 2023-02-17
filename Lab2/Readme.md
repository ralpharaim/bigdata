# Lab 2 - Reports with Apache Spark
## Установка и запуск (все команды в *start.txt*)
1. cd (путь к папке с проектом)
2. Туда положить файл *docker-compose.yml*
3. В консоли запустить docker-compose up --build -d
4. Ждать.... После установки и запуска можно входить c помощью
*  ssh root@localhost -p 2222 
*  mapr
5. Переходим в папку, которую указали в docker-compose.yml
6. cd /home/mapr/lab_2
7. Проверить, что мы в том местоможно с помощью команды *dir*
8. Используем следующие команды для скачивания всего необходимого:  
echo 'export PATH=$PATH:/opt/mapr/spark/spark-3.2.0/bin' > /root/.bash_profile  
source /root/.bash_profile  
*apt-get update && apt-get install -y python3-distutils python3-apt*  
*wget https://bootstrap.pypa.io/pip/3.6/get-pip.py*  
*python3 get-pip.py*  
*pip install jupyter* 
*pip install pyspark* 
9. Запускаем ноубук с помощью:  
*jupyter-notebook --ip=0.0.0.0 --port=50001 --allow-root --no-browser*
10. Открываем ссылку в браузере
11. Работаем

## Топ 10 языков программирования с 2010 по 2020:
### 2010:
![image](https://user-images.githubusercontent.com/91950488/203827276-4e679dd0-70d5-455c-a513-fac6f1873224.png)
### 2011:
![image](https://user-images.githubusercontent.com/91950488/203827325-c5a06043-e016-4fa3-98fa-94c55749bbc3.png)
### 2012:
![image](https://user-images.githubusercontent.com/91950488/203827411-68af364c-cda3-4156-ac9f-9edd4644972d.png)
### 2013:
![image](https://user-images.githubusercontent.com/91950488/203827448-abbeabf2-6278-4bb0-9ac2-6f7c17c53aba.png)
### 2014:
![image](https://user-images.githubusercontent.com/91950488/203827479-4b157b44-6d46-4a8e-9360-fbd5783a4cfa.png)
### 2015:
![image](https://user-images.githubusercontent.com/91950488/203827525-78e20e0c-38dc-45db-abbc-89069a946fc1.png)
### 2016:
![image](https://user-images.githubusercontent.com/91950488/203827569-bd974b90-4e05-42b3-b202-7f7f1f3a13ea.png)
### 2017:
![image](https://user-images.githubusercontent.com/91950488/203827595-73bfa513-d483-4db3-8cc3-87355918d519.png)
### 2018:
![image](https://user-images.githubusercontent.com/91950488/203827810-7d9b4687-6e1d-4922-8d3e-b61583ac8574.png)
### 2019:
![image](https://user-images.githubusercontent.com/91950488/203827692-f5ae6726-86ef-4707-85ee-570a897e5394.png)
