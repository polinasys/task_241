Task 1
Скриптуем по полной
1. Что такое шебанг?
Шебанг - последовательность из символов решётки и восклицательного знака ("#!") в начале файла скрипта. Указывает системе, каким интерпретатором выполнять файл.

2. Обязательно ли исполняемый файл дожен иметь соотвествующее расширение?
Нет, не обязательно.сВ Linux расширения файлов не определяют их исполняемость. Важны бит исполнения, шебанг, права доступа.

3. Напишите скрипт который выполнит автоматически действия из блока работы с файлами. ( не забудьте включить set -euo pipefail для того что бы ваш скрипт было удобнее отлаживать. Опишите что включают эти флаги)
#!/bin/bash

# Безопасные настройки скрипта
set -euo pipefail

# Описание флагов:
# set -e  - остановка скрипта при любой ошибке
# set -u  - ошибка при использовании необъявленных переменных
# set -o pipefail  - возвращает код ошибки пайплайна, если хоть одна команда пайпа упала

# Создание рабочей директории
WORK_DIR="/tmp/file_operations_$(date +%s)"
mkdir -p "$WORK_DIR"
cd "$WORK_DIR"

# 1. Папка с подпапками
mkdir -p project/{src,doc,bin,backup}

# 2. Внутри папки создаем файл и записываем в него данные
echo "Hello, World!" > project/src/main.txt
echo "Тестовые данные" > project/doc/readme.txt

# 3. Проверяем созданные файлы
ls -la project/src/
ls -la project/doc/

# 4. Перемещаем файл из одной директории в другую
mv project/src/main.txt project/backup/

# 5. Копируем файл из одной директории в другую
cp project/doc/readme.txt project/bin/readme_copy.txt

# 6. Переименовываем файл
mv project/bin/readme_copy.txt project/bin/renamed_file.txt

# 7. Создаем файлы для сравнения
echo "File A content" > file_a.txt
echo "File B content" > file_b.txt
echo "File A content" > file_c.txt

# 8. Сравниваем содержимое файлов
echo "Сравнение file_a.txt и file_b.txt:"
if diff file_a.txt file_b.txt; then
    echo "Файлы идентичны"
else
    echo "Файлы различаются"
fi

# 9. Создаем файл для сортировки
echo -e "banana\napple\ncherry\napple\norange" > fruits.txt

# 10. Сортируем содержимое файла по возрастанию и убыванию
echo "По возрастанию:"
sort fruits.txt > sorted_asc.txt
cat sorted_asc.txt

echo "По убыванию:"
sort -r fruits.txt > sorted_desc.txt
cat sorted_desc.txt

# 11. Создаем файл со всеми возможными правами
echo "Файл с полными правами" > all_access.txt
chmod 777 all_access.txt
ls -la all_access.txt

# 12. Показываем итоговую структуру
find . -type f -exec ls -la {} \; | head -20

# 13. Очистка 
rm -rf "$WORK_DIR"
