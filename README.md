Устанавливка:
git clone GitHub - tech4242/docker-hadoop-hive-parquet: Hadoop, Hive, Parquet and Hue in docker-compose v3 (в заранее созданной папке)
cmd → docker-compose up

Отлично, как только образ собран, а контейнер поднят — заходим по http://localhost:8888/hue и попадаем в HUE. Придумываем произвольную пару логина-пароля для будущей авторизации и приступаем к работе. 

1. Скачиваем все доступные тома произведения «Война и мир» Л.Н. Толстого.

2. Далее подключаемся к контейнеру «datanode-1», создаем внутри папку и переносим в нее скачанные файлы.

   docker ps
   Смотрим CONTAINER ID (7a2f6a78e991)
   Проваливаемся в контейнер: docker exec -it 7a2f6a78e991 bash
   Создаем папку: mkdir War_and_piece
   Команда, чтобы проверить содержимое: ls
   
3. Файлы предварительно «схлопываем» в один.
   Заходим в созданную папку: cd War_and_piece
   Объединяем в один файл: cat voyna-i-mir-tom-1.txt voyna-i-mir-tom-2.txt voyna-i-mir-tom-3.txt
   voyna-i-mir-tom-4.txt>war_and_piece.txt

4. Загружаем полученный файл на hdfs в вашу личную папку.
   hdfs dfs -copyFromLocal war_and_piece.txt /user/javascript_rocker
   
Если все пройдет удачно, то по возвращению в hue вас ждет сюрприз: вы увидите полное произведение «Война и мир» на hdfs (не забудьте обновить страницу). На самом деле интерфейс HUE поддерживает возможность переноса небольших(!) файлов с помощью drag&drop — но для нас это слишком просто.

//image

5. Возвращаемся в терминал и продолжаем изучать hdfs — попробуйте выполнить команду, которая
   выводит содержимое  вашей личной папки. 

   hdfs dfs -ls /user/javascript_rocker

   (OUTPUT: Found 1 items
   -rw-r--r--   3 root javascript_rocker    3048008 2023-08-20 10:15 /user/javascript_rocker/war_and_piece.txt)

6. Обратите внимание на права доступа к вашим файлам — их явно недостаточно, если вы решите поделиться столь важной книгой с вашим коллегой — давайте изменим права доступа к нашему файлу. Установите режим доступа, который дает полный доступ для владельца файла, а для сторонних пользователей возможность читать и выполнять.

   hdfs dfs -chmod 755 /user/javascript_rocker/war_and_piece.txt

7. Попробуйте заново использовать команду для вывода содержимого папки и обратите
внимание как изменились права доступа к файлу.

   hdfs dfs -ls /user/javascript_rocker

   (OUTPUT:Found 1 items
   -rwxr-xr-x   3 root javascript_rocker    3048008 2023-08-20 10:15 /user/javascript_rocker/war_and_piece.txt )

8. Теперь попробуем вывести на экран информацию о том, сколько места на диске
занимает наш файл. Желательно, чтобы размер файла был удобочитаемым.

    hdfs dfs -du -h /user/javascript_rocker/war_and_piece.txt
    (OUTPUT: -rwxr-xr-x   3 root javascript_rocker      2.9 M 2023-08-20 10:15 /user/javascript_rocker/war_and_piece.txt)

9. На экране вы можете заметить 2 числа. Первое число — это фактический размер файла,
а второе — это занимаемое файлом место на диске с учетом репликации — измените фактор репликации на 2.

    hdfs dfs -setrep 2  /user/javascript_rocker/war_and_piece.txt

10. Повторите команду, которая выводит информацию о том, какое место на диске
занимает файл и убедитесь, что изменения произошли.
    hdfs dfs -ls -h /user/javascript_rocker/war_and_piece.txt

    (OUTPUT: -rwxr-xr-x   2 root javascript_rocker      2.9 M 2023-08-20 10:15 /user/javascript_rocker/war_and_piece.txt)
 
11. И финальное — напишите команду, которая подсчитывает количество строк в произведении «Война и мир».
    hdfs dfs -cat /user/javascript_rocker/war_and_piece.txt | wc -l
 
    (OUTPUT:10272)
