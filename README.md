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
# Этап 3
При выполнении 3-его этапа мы работаем с виртуальной файловой системой VFS, для того, чтобы её имплементировать в наш эмулятор нам необходимо добавить 2 новых файла Test_script.sh который будет содержать скрипт со всеми необходимыми нам проверками и файл vfs.csv содержащий в себе нашу виртуальную файловую систему.

В файле Test_script.sh напишем следующее:
```
# Тест вывода информации о VFS
vfs-info

# Тест заглушек ls и cd внутри VFS
ls
cd /home/user
ls -la

# Проверка обработки ошибок 
vfs-info extra_argument
```

В файле vfs.csv напишем (в текстовом формате, который pycharm сам переведёт в табличный вид) следующее:
```
path,type,content
/,dir,
/home,dir,
/home/user,dir,
/home/user/documents,dir,
/home/user/documents/secret.txt,file,VGhpcyBpcyBhIHNlY3JldA==
/var,dir,
/var/log,dir,
/var/log/syslog,file,QWxsIHN5c3RlbXMgb3BlcmF0aW9uYWw=
```

Основной код программы в файле main.py будет выглядеть теперь следующим образом: 
```
import os
import sys
import getpass
import socket
import shlex
import argparse
import csv
import hashlib
import base64

# Глобальные переменные для хранения состояния VFS в памяти
VFS_DATA = {}  # Словарь: {абсолютный_путь: {'type': 'dir'/'file', 'content': '...'}}
VFS_NAME = ""  # Имя файла VFS
VFS_SHA256 = ""  # Хеш SHA-256 файла VFS
CURRENT_DIR = "/"  # Текущая директория внутри VFS


def load_vfs(vfs_path):

    global VFS_DATA, VFS_NAME, VFS_SHA256

    if not os.path.exists(vfs_path):
        print(f"Критическая ошибка: Файл VFS '{vfs_path}' не найден.", file=sys.stderr)
        sys.exit(1)

    VFS_NAME = os.path.basename(vfs_path)

    # Пункт 4: Вычисление хеша SHA-256 данных файла
    sha256_hash = hashlib.sha256()
    try:
        with open(vfs_path, "rb") as f:
            for byte_block in iter(lambda: f.read(4096), b""):
                sha256_hash.update(byte_block)
        VFS_SHA256 = sha256_hash.hexdigest()
    except Exception as e:
        print(f"Критическая ошибка при чтении файла VFS: {e}", file=sys.stderr)
        sys.exit(1)

    # Загрузка структуры CSV в память
    try:
        with open(vfs_path, mode="r", encoding="utf-8") as f:
            reader = csv.DictReader(f)

            # Проверка обязательных колонок (Пункт 3: неверный формат)
            if not reader.fieldnames or not all(col in reader.fieldnames for col in ["path", "type", "content"]):
                print(f"Критическая ошибка: Неверный формат CSV в '{vfs_path}'. Требуются колонки path, type, content.",
                      file=sys.stderr)
                sys.exit(1)

            # Всегда инициализируем корневой каталог
            VFS_DATA["/"] = {"type": "dir", "content": ""}

            for row in reader:
                path = row["path"].strip()
                node_type = row["type"].strip()
                content = row["content"]

                # Нормализация путей (чтобы всегда начинались с / и не дублировали слэши)
                if not path.startswith("/"):
                    path = "/" + path
                path = os.path.normpath(path).replace("\\", "/")

                VFS_DATA[path] = {"type": node_type, "content": content}

    except Exception as e:
        print(f"Критическая ошибка парсинга VFS (неверный формат): {e}", file=sys.stderr)
        sys.exit(1)


def get_prompt():
    """Формирует приглашение к вводу на основе реальных данных ОС и виртуального пути."""
    username = getpass.getuser()
    hostname = socket.gethostname()
    return f"{username}@{hostname}:{CURRENT_DIR}$ "


def execute_command(user_input):
    """Обрабатывает и выполняет одну команду."""
    global CURRENT_DIR
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

    elif command == "vfs-info":
        # Вывод информации о загруженной VFS
        if args:
            print("vfs-info: команда не принимает аргументы", file=sys.stderr)
            return False
        print(f"Имя VFS: {VFS_NAME}")
        print(f"Хеш SHA-256: {VFS_SHA256}")
        return True

    elif command in ["ls", "cd"]:

        print(f"[Заглушка VFS] Вызвана команда: {command}")
        print(f"[Заглушка VFS] Аргументы: {args}")
        return True

    else:
        print(f"shell-emulator: command not found: {command}", file=sys.stderr)
        return False


def run_script(script_path):
    """Выполняет команды из стартового скрипта с имитацией ввода-вывода."""
    if not os.path.exists(script_path):
        print(f"Ошибка: Стартовый скрипт '{script_path}' не найден.", file=sys.stderr)
        return False

    print(f"--- Запуск стартового скрипта: {script_path} ---")
    try:
        with open(script_path, 'r', encoding='utf-8') as f:
            for line_num, line in enumerate(f, 1):
                clean_line = line.strip()
                if not clean_line or clean_line.startswith("#"):
                    continue

                prompt = get_prompt()
                print(f"{prompt}{clean_line}")

                status = execute_command(clean_line)
                if status == "exit":
                    return True
                elif not status:
                    print(f"shell-emulator: ошибка на строке {line_num} в скрипте '{script_path}'", file=sys.stderr)
                    print("Выполнение скрипта прервано из-за ошибки.", file=sys.stderr)
                    return False
    except Exception as e:
        print(f"Критическая ошибка при чтении скрипта: {e}", file=sys.stderr)
        return False

    print("--- Стартовый скрипт успешно выполнен ---\n")
    return True


def main():
    parser = argparse.ArgumentParser(description="Эмулятор UNIX-оболочки (Этап 3)")
    parser.add_argument("--vfs", required=True, help="Путь к CSV-файлу виртуальной файловой системы")
    parser.add_argument("--script", help="Путь к стартовому скрипту")

    args = parser.parse_args()

    # Отладочный вывод
    print("===== ОТЛАДОЧНЫЙ ВЫВОД ПАРАМЕТРОВ =====")
    print(f"Путь к VFS: {args.vfs}")
    print(f"Стартовый скрипт: {args.script if args.script else 'Не задан'}")
    print("=======================================\n")

    # Загружаем VFS в память перед стартом
    load_vfs(args.vfs)

    print("Эмулятор UNIX-оболочки запущен. Для выхода введите 'exit'.\n")

    if args.script:
        run_script(args.script)

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


