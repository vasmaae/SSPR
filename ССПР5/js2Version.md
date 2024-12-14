# гайд js 2
## спасибо Денису и ПИбд-21

## 1 Container:
```shell
apt install docker.io
mkdir matrix-max-app
cd matrix-max-app/
nano index.js
```

```js
const readline = require('readline');

function findMaxAboveDiagonal(matrix) {
    let max = -Infinity;
    let n = matrix.length;

    for (let i = 0; i < n; i++) {
        for (let j = i + 1; j < n; j++) {
            if (matrix[i][j] > max) {
                max = matrix[i][j];
            }
        }
    }

    return max;
}

const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
});

function inputMatrix() {
    rl.question("Введите размер матрицы: ", (size) => {
        size = parseInt(size);
        let matrix = [];
        let rowCount = 0;

        function inputRow() {
            if (rowCount < size) {
                rl.question(Введите ${size} элементов для строки ${rowCount + 1} (через пробел): , (row) => {
                    let rowElements = row.split(" ").map(Number);
                    matrix.push(rowElements);
                    rowCount++;
                    inputRow();
                });
            } else {
                const max = findMaxAboveDiagonal(matrix);
                console.log("Максимальный элемент выше главной диагонали:", max);
                rl.question("Хотите ввести новую матрицу? (y/n): ", (answer) => {
                    if (answer.toLowerCase() === 'y') {
                        matrix = [];
                        rowCount = 0;
                        inputMatrix();
                    } else {
                        rl.close();
                    }
                });
            }
        }

        inputRow();
    });
}

inputMatrix();

module.exports = { findMaxAboveDiagonal };
```
```shell
nano Dockerfile
```
```Dockerfile
FROM node:16

WORKDIR /app

COPY . .

RUN npm install mocha chai --save-dev

ENTRYPOINT ["sh", "-c"]
CMD ["node index.js"]  # По умолчанию запускаем index.js
```
```shell
nano test.js
```
```js
const { strictEqual } = require('assert');
const { findMaxAboveDiagonal } = require('./index');

describe('Тестирование findMaxAboveDiagonal', () => {
    it('Должен вернуть максимальный элемент выше главной диагонали', () => {
        const matrix = [
            [1, 2, 3],
            [4, 5, 6],
            [7, 8, 9]
        ];
        const result = findMaxAboveDiagonal(matrix);
        strictEqual(result, 6);
    });

    it('Должен вернуть -Infinity для пустой матрицы', () => {
        const matrix = [];
        const result = findMaxAboveDiagonal(matrix);
        strictEqual(result, -Infinity);
    });
});
```
```shell
nano package.json
```
```json
{
  "name": "matrix-max-app",
  "version": "1.0.0",
  "scripts": {
    "test": "mocha test.js"
  }
}
```
```shell
docker build -t matrix-max-app .
docker run -it --rm matrix-max-app
docker run --rm matrix-max-app  "npm test"
docker save -o /root/max-element-app.tar matrix-max-app
scp /root/max-element-app.tar root@192.168.28.129:/tmp/
```
## 2 Container
```shell
dnf install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
dnf install -y docker-ce docker-ce-cli containerd.io
systemctl start docker
systemctl enable docker
usermod -aG docker $USER
newgrp docker
docker load -i /tmp/max-element-app.tar
dnf update -y
dnf install java-17-openjdk -y
dnf install wget -y
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
dnf install jenkins -y --nogpgcheck
systemctl enable --now Jenkins
systemctl start Jenkins
cat /var/lib/jenkins/secrets/initialAdminPassword
usermod -aG docker Jenkins
systemctl restart Jenkins
systemctl restart docker
```
### Вирт Машина
```shell
nano /etc/nginx/sites-enabled/example.conf
```
### меняем location /:
```
location / {
        proxy_pass http://192.168.28.129:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
}
```
### Браузер 
http://192.168.1.200/

## Создаем pipeline

```
pipeline {
    agent any

    stages {
        stage('Pull Docker Image') {
            steps {
                sh 'docker images'
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    sh 'docker run --rm matrix-max-app "npm test"'
                }
            }
        }
    }
}
```