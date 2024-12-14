# Версия на php

как дополнение/другая реализация к jsVersion

создаём в папке lab5 файл index.php

```php
<?php

// Функция для генерации случайной матрицы
function generateMatrix($n) {
    $matrix = [];
    for ($i = 0; $i < $n; $i++) {
        for ($j = 0; $j < $n; $j++) {
            $matrix[$i][$j] = rand(-100, 100);
        }
    }
    return $matrix;
}

// Функция для нахождения минимального элемента под главной диагональю
function findMinElementBelowMainDiagonal($matrix) {
    $min = null;
foreach ($matrix as $rowIndex => $row) {
        foreach ($row as $colIndex => $value) {
            if ($colIndex > $rowIndex && ($min === null || $value < $min)) {
                $min = $value;
            }
        }
    }
    return $min;
}

if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    // Получаем размерность матрицы из формы
    $size = $_POST['size'];

    // Генерируем матрицу
    $matrix = generateMatrix($size);
// Находим минимальный элемент под главной диагональю
    $minElement = findMinElementBelowMainDiagonal($matrix);

    // Выводим матрицу и результат
?>
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Минимальный элемент под главной диагональю</title>
    <style>
        table {
            border-collapse: collapse;
        }
        td {
            padding: 10px;
text-align: center;
            border: 1px solid #000;
        }
    </style>
</head>
<body>
<h1>Матрица:</h1>
<table>
    <?php foreach ($matrix as $row): ?>
        <tr>
            <?php foreach ($row as $cell): ?>
                <td><?php echo $cell ?></td>
            <?php endforeach; ?>
        </tr>
    <?php endforeach; ?>
</table>
<h1>Минимальный элемент под главной диагональю: <?= $minElement ?></h1>

<a href="/">Вернуться к форме</a>

</body>
</html>
<?php
} else {
?>
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Введите размер матрицы</title>
</head>
<body>
<form method="post">
    <label for="size">Размер матрицы (N x N):</label><br>
    <input type="number" name="size" id="size" min="2" required><br><br>
    <button type="submit">Отправить</button>
</form>
</body>
</html>
<?php
}
?>
```

Dockerfile будет выглядеть так:
```Dockerfile
FROM php:7.4-apache

# Копируем наши файлы в контейнер
COPY . /var/www/html/

# Указываем рабочую директорию
WORKDIR /var/www/html

# Открываем порт 80
EXPOSE 80
```

docker-compose:
```
version: '3'
services:
  web:
    build: .
    container_name: php-site
    ports:
      - "8000:80"
    volumes:
      - ./:/var/www/html
```

к такому контейнеру надо будет стучаться на 8000 порт
> но тесты к такой реализации сами пишите