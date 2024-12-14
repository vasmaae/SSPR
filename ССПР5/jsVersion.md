# Гайд на лаб 5

## arch+fedora, var10:
[![Typing SVG](https://readme-typing-svg.demolab.com?font=Roboto&weight=900&size=40&duration=3000&pause=2000&color=13B4F7&width=450&height=60&lines=i+use+arch+btw)](https://git.io/typing-svg)
1. Ставим docker.  
[вот так:](https://wiki.archlinux.org/title/Docker_(%D0%A0%D1%83%D1%81%D1%81%D0%BA%D0%B8%D0%B9))
   ```shell
   pacman -S docker docker-compose
   ```
   *с docker-compose будет удобно делать и лаб6*  
   Запустим демона (службу):
   ```shell
   systemctl start docker.service
   ```
   Добавляем в автозагрузку
     > чтобы потом не ебаться с запуском каждый раз при вкл контейнера:
   ```shell
   systemctl enable docker.service
   ```
   Проверяем, что всё успешно:
   или так:
   ```shell
   docker run hello-world
   ```
   или просто проверяем что такое существует:
   ```shell
   docker --help
   ```
3. Создаём папку проекта:
   где-нибудь в руте:
   ```shell
   mkdir lab5 && cd lab5
   ```
   начинаем создавать файлы для лабы:
   ```shell
   nano <имя файла>
   ```
   потом просто CTRL+V или, если не вставляется CTRL+SHIFT+V
   копировать [отсюда](https://github.com/DjonniStorm/secondYearAtUniversity/tree/master/sspr/lab5), просто чтобы не захламлять тут (код писался на похуй)
   > Dockerfile лучше взять на [alpine](https://github.com/DjonniStorm/secondYearAtUniversity/blob/master/sspr/lab6/server/Dockerfile), они поменьше весят

   вот такая штука должна получиться, само задание менять в файле public/task.js, тесты менять в файле tests/matrix.test.js (тут всё интуитивно должно быть):
   > если хотите, можете поменять порты в compose, и вписать тот который нравится, если вы это сделали, то его дальше надо будет вписать по ходу 
   ```
   |── Dockerfile
   |── docker-compose.yml
   |── package.json
   |── public
   |   |── index.html
   |   └── task.js
   |── server.js
   └── tests
     └── matrix.test.js
   ```
5. Запускаем в первом контейнере:
   чтобы собрать через compose сделаем это в директории lab5:
   ```shell
    docker-compose up --build -d
   ```
   пока рисуется tui, пробросим порты для того, чтобы можно было открыть HTML

   заходим в virtual box, добавляем порт какой нравится, у меня это будет 12088
   > также можно типо номер варианта сделать 10188, 10288, ни на что не влияет
   
   ![image](https://github.com/vasmaae/SSPR/blob/main/img/1%20-%20virtualBoxPorts.png)
   > также надо прописать для jenkins 8080 и для второго контейнера какой нравится

   теперь надо добавить в nginx.conf инфу о нашем сайтике (и об остальном чтобы не заходить несколько раз)
   ![image](https://github.com/vasmaae/SSPR/blob/main/img/2%20-%20nginxCfg.png)  
   (3 прикол это jenkins)  
   не забываем перезапустить nginx
   ```shell
   systemctl restart nginx
   ```


   когда наш image собрался, делаем так:
   ```shell
   docker images
   ```
   *запомнили image name*

   теперь запускаем:
   ```shell
   docker run -d -p 8020:3000 <image-name>
   ```
   тут -d значит, что всё будет работать в фоне, -p что порт 8020 снаружи (если поменяли то тут тоже) === 3000 на контейнере

   если всё было правильно сделано, то открывайте, как и всё в этих лабах, по адресу localhost:<порт в virtual box>
7. Переносим во второй:
   делать будем через [Docker Hub](https://hub.docker.com/), где сначала надо зарегаться

   после того, как зарегались, то в контейнере с arch делаем:
   ```shell
   docker login
   ```

   потом надо тегнуть существующий image:
   ```shell
   docker tag <image-name> <ваш_докер_ник>/lab5
   ```
   > по умолчанию будет тег latest
   
   и пушим наш image
   ```shell
   docker push <ваш_докер_ник>/lab5
   ```

   смотрим на docker hub наш крутой image
8. Контейнер 2 (у меня Fedora)  
   просто можно идти по [гайду](https://docs.docker.com/engine/install/fedora/)

   теперь пуллим наш image с Docker Hub:
   ```shell
   docker pull <ваш_докер_ник>/lab5
   ```

   теперь также можно запустить, но тут уже также, как при pull писали:
   ```shell
   docker run -d -p 8021:3000 <ваш_докер_ник>/lab5
   ```

   Посмотрев, что всё работает, переходим к jenkins
9. Jenkins  
   Вообще без разницы на какой контейнер ставить

   просто копировать команды не буду, качаем как [тут](https://www.jenkins.io/doc/book/installing/linux/) сказано в общем (стоит контейнеру выдать побольше ресурсов, если будет тормозить)

   это в конце:   ![image](https://github.com/vasmaae/SSPR/blob/main/img/3%20-%20jenkinsStart.png)

   т.к. порты мы уже пробросили (не забыли уже?), открываем в браузере просто порт, там всё должно быть понятно: пароль выводим через cat <filename>, устанавливаем рекомендуемые плагины
---
   Тут добавляем Docker, чтобы работал:

   ![Без имени](https://github.com/vasmaae/SSPR/blob/main/img/4%20-%20jenkins-plugins.png)

   (често хз нужны ли все) ![image](https://github.com/vasmaae/SSPR/blob/main/img/5%20-%20jenkinsDocker.png)

   дальше тут выбираем создать item ![image](https://github.com/vasmaae/SSPR/blob/main/img/6%20-%20jenkinsDashboard.png)

   и тут Pipeline ![image](https://github.com/vasmaae/SSPR/blob/main/img/7%20-%20jenkinsPipeline.png)

   проматываем вниз и вставляем код (либо, там ещё в других версиях гайда накидали возможностей) ![image](https://github.com/vasmaae/SSPR/blob/main/img/8%20-%20jenkinsPipelineScript.png)

   ### также используем <ваш_докер_ник>/lab5

   ```
pipeline {
    agent any
    stages {
        stage('Pull Docker Image') {
            steps {
                script {
                    sh 'docker pull <ваш_докер_ник>/lab5'
                }
            }
        }
        stage('Run Tests') {
            steps {
                script {
                    sh '''
                    docker run --rm <ваш_докер_ник>/lab5 npm install
                    docker run --rm <ваш_докер_ник>/lab5 npm run test
                    '''
                }
            }
        }
    }
}
```
