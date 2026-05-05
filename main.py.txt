import tkinter as tk
from tkinter import ttk, messagebox
import random
import json
import os

# Настройки
HISTORY_FILE = "tasks.json"
GENERATED_HISTORY_FILE = "generated_tasks.json"  # Отдельный файл для истории сгенерированных задач
DEFAULT_TASKS = [
    {"text": "Прочитать статью", "type": "учёба"},
    {"text": "Сделать зарядку", "type": "спорт"},
    {"text": "Написать отчёт", "type": "работа"},
    {"text": "Посмотреть обучающее видео", "type": "учёба"},
    {"text": "Разобрать почту", "type": "работа"},
    {"text": "Погулять на свежем воздухе", "type": "отдых"},
]

class TaskGeneratorApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Генератор случайных задач")
        self.root.geometry("500x550")

        # Загрузка данных
        self.tasks = self.load_tasks()
        self.generated_history = self.load_generated_history()

        # Текущая задача
        self.current_task_label = tk.Label(
            root, text="Нажмите «Сгенерировать задачу»",
            wraplength=450, justify="center",
            font=("Arial", 12), pady=10
        )
        self.current_task_label.pack()

        # Кнопка генерации
        tk.Button(root, text="Сгенерировать задачу",
                  command=self.generate_task,
                  bg="#4CAF50", fg="white",
                  font=("Arial", 10)).pack(pady=10)

        # Фильтр по типу
        filter_frame = tk.Frame(root)
        filter_frame.pack(pady=5)
        tk.Label(filter_frame, text="Фильтр:").pack(side=tk.LEFT)
        self.filter_var = tk.StringVar(value="все")
        filter_combo = ttk.Combobox(
            filter_frame,
            textvariable=self.filter_var,
            values=["все", "учёба", "работа", "спорт", "отдых"],
            state="readonly", width=10
        )
        filter_combo.pack(side=tk.LEFT, padx=5)
        filter_combo.bind("<<ComboboxSelected>>", self.update_history_list)

        # История задач
        history_frame = tk.Frame(root)
        history_frame.pack(fill=tk.BOTH, expand=True, padx=10, pady=10)
        tk.Label(history_frame, text="История сгенерированных задач:",
                 font=("Arial", 10, "bold")).pack()
        self.history_listbox = tk.Listbox(history_frame, height=10)
        scrollbar = tk.Scrollbar(history_frame)
        self.history_listbox.config(yscrollcommand=scrollbar.set)
        scrollbar.config(command=self.history_listbox.yview)
        self.history_listbox.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
        scrollbar.pack(side=tk.RIGHT, fill=tk.Y)

        # Добавление новых задач
        add_frame = tk.Frame(root)
        add_frame.pack(pady=10)
        tk.Label(add_frame, text="Новая задача:").pack(side=tk.LEFT)
        self.new_task_entry = tk.Entry(add_frame, width=25)
        self.new_task_entry.pack(side=tk.LEFT, padx=5)
        self.new_task_type = ttk.Combobox(
            add_frame, values=["учёба", "работа", "спорт", "отдых"],
            state="readonly", width=10
        )
        self.new_task_type.set("работа")
        self.new_task_type.pack(side=tk.LEFT, padx=5)
        tk.Button(add_frame, text="Добавить в список",
                 command=self.add_new_task).pack(side=tk.LEFT)


        self.update_history_list()

    def load_tasks(self):
        """Загрузка задач из JSON или создание файла с дефолтными задачами."""
        if os.path.exists(HISTORY_FILE):
            try:
                with open(HISTORY_FILE, 'r', encoding='utf-8') as f:
                    return json.load(f)
            except (json.JSONDecodeError, IOError) as e:
                messagebox.showerror("Ошибка", f"Не удалось загрузить задачи: {e}")
                return DEFAULT_TASKS.copy()
        # Если файла нет, создаём его с дефолтными задачами
        try:
            with open(HISTORY_FILE, 'w', encoding='utf-8') as f:
                json.dump(DEFAULT_TASKS.copy(), f, ensure_ascii=False, indent=2)
        except IOError as e:
            messagebox.showerror("Ошибка", f"Не удалось создать файл задач: {e}")
        return DEFAULT_TASKS.copy()

    def load_generated_history(self):
        """Загрузка истории сгенерированных задач."""
        if os.path.exists(GENERATED_HISTORY_FILE):
            try:
                with open(GENERATED_HISTORY_FILE, 'r', encoding='utf-8') as f:
                    return json.load(f)
            except (json.JSONDecodeError, IOError):
                return []
        return []

    def save_tasks(self):
        """Сохранение списка задач в JSON с обработкой ошибок."""
        try:
            with open(HISTORY_FILE, 'w', encoding='utf-8') as f:
                json.dump(self.tasks, f, ensure_ascii=False, indent=2)
        except IOError as e:
            messagebox.showerror("Ошибка", f"Не удалось сохранить задачи: {e}")

    def save_generated_history(self):
        """Сохранение истории сгенерированных задач с обработкой ошибок."""
        try:
            with open(GENERATED_HISTORY_FILE, 'w', encoding='utf-8') as f:
                json.dump(self.generated_history, f, ensure_ascii=False, indent=2)
        except IOError as e:
            messagebox.showerror("Ошибка", f"Не удалось сохранить историю: {e}")

    def update_history_list(self, *args):
        """Обновление виджета списка истории с учётом фильтра."""
        self.history_listbox.delete(0, tk.END)
        filter_type = self.filter_var.get()
        for task in reversed(self.generated_history):  # Показываем новые задачи сверху
            if filter_type == "все" or task["type"] == filter_type:
                self.history_listbox.insert(tk.END, f"{task['text']} ({task['type']}) — {task['timestamp']}")

    def generate_task(self):
        """Генерация случайной задачи."""
        if not self.tasks:
            messagebox.showwarning("Предупреждение", "Список задач пуст! Добавьте новые задачи.")
            return
        selected_task = random.choice(self.tasks)
        # Добавляем в историю сгенерированных задач с отметкой времени
        from datetime import datetime
        generated_task = selected_task.copy()
        generated_task["timestamp"] = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        self.generated_history.append(generated_task)
        self.save_generated_history()
        # Отображаем задачу в главном лейбле
        self.current_task_label.config(
            text=f"Задача: {selected_task['text']}\nТип: {selected_task['type'].capitalize()}",
            bg="#f0f0f0", relief="solid"
        )
        self.update_history_list()

    def add_new_task(self):
        """Добавление новой задачи с валидацией."""
        task_text = self.new_task_entry.get().strip()
        task_type = self.new_task_type.get()

        if not task_text:
            messagebox.showerror("Ошибка", "Задача не может быть пустой!")
            return

        new_task = {"text": task_text, "type": task_type}
