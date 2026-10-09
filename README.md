# MIREA_Config_Upr_Pavlyuchuk_Egor
Репозиторий для выполнения заданий по конфигурационной практике 
# Этап 1 (python)
```
  import os
  import sys
  import getpass
  import socket
  import shlex

  def get_prompt():
      """Формирует приглашение к вводу на основе реальных данных ОС."""
      username = getpass.getuser()
      hostname = socket.gethostname()
      
      # Получаем текущую директорию
      current_dir = os.getcwd()
      # Пытаемся заменить домашний каталог пользователя на '~' для схожести с UNIX
      home_dir = os.path.expanduser("~")
      if current_dir.startswith(home_dir):
          current_dir = current_dir.replace(home_dir, "~", 1)
          
      return f"{username}@{hostname}:{current_dir}$ "
  
  def main():
      print("Эмулятор UNIX-оболочки (Прототип REPL) запущен. Для выхода введите 'exit'.\n")
      
      while True:
          try:
              # Этап 1: Вывод приглашения и чтение ввода
              prompt = get_prompt()
              user_input = input(prompt).strip()
              
              # Если пользователь просто нажал Enter, продолжаем цикл
              if not user_input:
                  continue
                  
              # Этап 2: Парсинг строки с учетом кавычек
              parsed_input = shlex.split(user_input)
              command = parsed_input[0]
              args = parsed_input[1:]
              
              # Этап 3: Обработка команд и заглушек
              if command == "exit":
                  if args:
                      print("exit: команда не принимает аргументы", file=sys.stderr)
                      continue
                  print("Завершение работы эмулятора.")
                  break
                  
              elif command in ["ls", "cd"]:
                  # Команды-заглушки: выводят свое имя и аргументы
                  print(f"[Заглушка] Вызвана команда: {command}")
                  print(f"[Заглушка] Аргументы: {args}")
                  
              else:
                  # Обработка неизвестной команды
                  print(f"shell-emulator: command not found: {command}", file=sys.stderr)
                  
          except KeyboardInterrupt:
              # Красивая обработка Ctrl+C (как в реальном терминале)
              print("\nИспользуйте команду 'exit' для выхода.")
          except EOFError:
              # Обработка Ctrl+D
              print("\nЗавершение работы эмулятора (EOF).")
              break
          except Exception as e:
              print(f"Ошибка парсинга или выполнения: {e}", file=sys.stderr)
  
  if __name__ == "__main__":
      main()
```

# Задание 2
Для выполнения задания 2 в командную строку Linux необходимо ввести следующую команду:
grep -v '^#' /etc/protocols | awk '{print $2, $1}' | sort -rn | head -n 5
# Задание 3
Для выполнения задания 3 необходимо сначала создать текстовый файл в виртуальном линуксе с помощью nano banner создать, после чего записать следующий код:

<img width="435" height="385" alt="image" src="https://github.com/user-attachments/assets/da13821f-ada9-4fcc-932e-0dbfcd784af1" />


