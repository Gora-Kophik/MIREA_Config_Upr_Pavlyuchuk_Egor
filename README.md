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
              # Вывод приглашения и чтение ввода
              prompt = get_prompt()
              user_input = input(prompt).strip()
              
              if not user_input:
                  continue
                  
              #  Парсинг строки с учетом кавычек
              parsed_input = shlex.split(user_input)
              command = parsed_input[0]
              args = parsed_input[1:]
              
              #  Обработка команд и заглушек
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
              # Обработка Ctrl+C 
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

# Этап 2
Для выполнения данного этапа нам понадобится создать дополнительную папку vfs_root в том же месте, что и наш проект эмулятора (она будет служить нам имитацией файловой системы), после чего создадим два дополнительных файла, которые можно будет вызывать с помощью этой системы настраивая параметры запуска эмулятора. Файлы назовём good.sh и bad.sh соответственно для демонстрации работы скрипта с полностью существующими командами и с несуществующими соответственно. 

В файле good.sh напишем следующее: 
```
ls
cd /home/user
```

В файле bad.sh напишем следующее: 
```
ls
unknown_command_test
cd /root
```

Код main.py для этого этапа будет выглядеть следующим образом: 
```
import os
import sys
import getpass
import socket
import shlex
import argparse


def get_prompt():
    """Формирует приглашение к вводу на основе реальных данных ОС."""
    username = getpass.getuser()
    hostname = socket.gethostname()

    current_dir = os.getcwd()
    home_dir = os.path.expanduser("~")
    if current_dir.startswith(home_dir):
        current_dir = current_dir.replace(home_dir, "~", 1)

    return f"{username}@{hostname}:{current_dir}$ "


def execute_command(user_input):
    
    parsed_input = shlex.split(user_input)
    if not parsed_input:
        return True

    command = parsed_input[0]
    args = parsed_input[1:]

    if command == "exit":
        if args:
            print("exit: команда не принимает аргументы", file=sys.stderr)
            return False
        print("Завершение работы эмулятора.")
        return "exit"

    elif command in ["ls", "cd"]:
        print(f"[Заглушка] Вызвана команда: {command}")
        print(f"[Заглушка] Аргументы: {args}")
        return True

    else:
        print(f"shell-emulator: command not found: {command}", file=sys.stderr)
        return False


def run_script(script_path):
    """ Выполняет команды из стартового скрипта с имитацией ввода-вывода. """
    if not os.path.exists(script_path):
        print(f"Ошибка: Стартовый скрипт '{script_path}' не найден.", file=sys.stderr)
        return False

    print(f"--- Запуск стартового скрипта: {script_path} ---")

    try:
        with open(script_path, 'r', encoding='utf-8') as f:
            for line_num, line in enumerate(f, 1):
                clean_line = line.strip()
                # Игнорируем пустые строки и комментарии
                if not clean_line or clean_line.startswith("#"):
                    continue

                # Имитируем ввод пользователя 
                prompt = get_prompt()
                print(f"{prompt}{clean_line}")

                # Выполняем команду
                status = execute_command(clean_line)

                if status == "exit":
                    return True
                elif not status:
                    # Остановка при первой ошибке и вывод сообщения
                    print(f"shell-emulator: ошибка на строке {line_num} в скрипте '{script_path}'", file=sys.stderr)
                    print("Выполнение скрипта прервано из-за ошибки.", file=sys.stderr)
                    return False
    except Exception as e:
        print(f"Критическая ошибка при чтении скрипта: {e}", file=sys.stderr)
        return False

    print("--- Стартовый скрипт успешно выполнен ---\n")
    return True


def main():
    #  Настройка параметров командной строки через argparse
    parser = argparse.ArgumentParser(description="Эмулятор UNIX-оболочки (Этап 2)")
    parser.add_argument("--vfs", required=True, help="Путь к физическому расположению VFS (архив или папка)")
    parser.add_argument("--script", help="Путь к стартовому скрипту для автоматического выполнения")

    args = parser.parse_args()

    # Отладочный вывод всех заданных параметров 
    print("===== ОТЛАДОЧНЫЙ ВЫВОД ПАРАМЕТРОВ =====")
    print(f"Путь к VFS: {args.vfs}")
    print(f"Стартовый скрипт: {args.script if args.script else 'Не задан'}")
    print("=======================================\n")

    print("Эмулятор UNIX-оболочки запущен. Для выхода введите 'exit'.\n")

    # Если передан стартовый скрипт, сначала запускаем его
    if args.script:
        script_success = run_script(args.script)
        # Если скрипт завершился с ошибкой, мы можем либо выйти, либо продолжить REPL.
        # Обычно в оболочках выполнение переходит в интерактивный режим, но мы зафиксировали ошибку.

    # Интерактивный цикл REPL
    while True:
        try:
            prompt = get_prompt()
            user_input = input(prompt).strip()

            status = execute_command(user_input)
            if status == "exit":
                break

        except KeyboardInterrupt:
            print("\nИспользуйте команду 'exit' для выхода.")
        except EOFError:
            print("\nЗавершение работы эмулятора (EOF).")
            break
        except Exception as e:
            print(f"Ошибка выполнения: {e}", file=sys.stderr)


if __name__ == "__main__":
    main()
```
# Задание 3
Для выполнения задания 3 необходимо сначала создать текстовый файл в виртуальном линуксе с помощью nano banner создать, после чего записать следующий код:

<img width="435" height="385" alt="image" src="https://github.com/user-attachments/assets/da13821f-ada9-4fcc-932e-0dbfcd784af1" />


