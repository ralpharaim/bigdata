# Lab 1 - Introduction to Apache Spark
## Установка и запуск (все команды в *start.txt*)
1. cd (путь к папке с проектом)
2. Туда положить файл *docker-compose.yml*
3. В консоли запустить docker-compose up --build -d
4. Ждать.... После установки и запуска можно входить c помощью
*  ssh root@localhost -p 2222 
*  mapr
5. Переходим в папку, которую указали в docker-compose.yml
6. cd /home/mapr/lab_1
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
## Lab1.ipynb
* В ноубуке сначала идёт проверка работоспособности spark и hadoop, после идёт ознакомление с функциями pyspark, подготовка данных для заданий и сами задания
## Ответы:
1. Найти велосипед с максимальным временем пробега: 535
2. Найти наибольшее геодезическое расстояние между станциями: 69.92 км
3. Найти путь велосипеда с максимальным временем пробега через станции: Там 1328 строк
4. Найти количество велосипедов в системе: 700
5. Найти пользователей потративших на поездки более 3 часов: 8322 уникальных пользователей
