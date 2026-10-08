# MIREA_Config_Upr_Pavlyuchuk_Egor
Репозиторий для выполнения заданий по конфигурационной практике 
# Задание 1 
Для выполнения задания 1 в командную строку Linux необходимо ввести следующую команду:
grep -v '^#' /etc/passwd | cut -d: -f1 | sort
# Задание 2
Для выполнения задания 2 в командную строку Linux необходимо ввести следующую команду:
grep -v '^#' /etc/protocols | awk '{print $2, $1}' | sort -rn | head -n 5
# Задание 3
Для выполнения задания 3 необходимо сначала создать текстовый файл в виртуальном линуксе с помощью nano banner создать, после чего записать следующий код:
#!/bin/bash
text="$1"
len=${#text}
printf "+-"
for ((i=0; i<len; i++))
do
printf "-"
done
printf "%s" "-+"
printf "\n"
printf "| %s |\n" "$text"
printf "+-"
for ((i=0; i<len; i++))
do
printf "-"
done
printf "%s" "-+"
printf "\n"
