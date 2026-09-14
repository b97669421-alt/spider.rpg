import random
from kivy.app import App
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.button import Button
from kivy.uix.scrollview import ScrollView
from kivy.uix.label import Label
from kivy.clock import Clock
from kivy.core.window import Window

# Эмуляция экрана телефона при тестах на ПК (на самом смартфоне растянется на весь экран)
Window.size = (400, 650)

class TextQuestGame(BoxLayout):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.orientation = 'vertical'
        self.spacing = 10
        self.padding = 10

        # Все оригинальные переменные из твоего кода
        self.rp = 0
        self.lg = 0 
        self.st = 0
        self.ph = 0
        self.kl = 0
        self.xa_count = 0 
        self.joke_count = 0
        self.hp = 0
        self.dm = 0
        self.role = ""
        self.hardcore = False
        
        self.spider_hp = 40
        self.spider_dm = 15

        # --- ИНТЕРФЕЙС ИГРЫ ---
        # 1. Верхняя статус-панель игрока
        self.status_label = Label(
            text="Выбери класс", 
            size_hint_y=0.1,
            color=(0.9, 0.9, 0.2, 1),
            font_size='14sp'
        )
        self.add_widget(self.status_label)

        # 2. Окно вывода игрового текста (Лог игры)
        self.scroll_view = ScrollView(size_hint_y=0.6)
        self.game_log = Label(
            text="", 
            size_hint_y=None, 
            halign='left', 
            valign='top',
            font_size='16sp',
            markup=True
        )
        self.game_log.bind(texture_size=self.game_log.setter('size'))
        self.scroll_view.add_widget(self.game_log)
        self.add_widget(self.scroll_view)

        # 3. Контейнер для кнопок управления
        self.buttons_layout = BoxLayout(orientation='vertical', size_hint_y=0.3, spacing=5)
        self.add_widget(self.buttons_layout)

        # Старт игры
        self.log("Выбери класс")
        self.show_class_selection()

    def log(self, text):
        """Добавление текста в игровое окно с автопрокруткой вниз"""
        self.game_log.text += text + "\n"
        Clock.schedule_once(lambda dt: setattr(self.scroll_view, 'scroll_y', 0), 0.1)

    def update_status(self):
        """Обновление верхней строчки статуса персонажа"""
        if self.role:
            self.status_label.text = f"Твой хп {self.hp} твой урон {self.dm}" if self.hardcore else f"ХП: {self.hp} | Урон: {self.dm} | Репутация: {self.rp}"

    def clear_buttons(self):
        self.buttons_layout.clear_widgets()

    def add_option(self, text, callback_func):
        """Создание кнопки действия"""
        btn = Button(text=text, font_size='16sp', background_color=(0.1, 0.5, 0.7, 1))
        btn.bind(on_press=callback_func)
        self.buttons_layout.add_widget(btn)

    # --- ЭКРАН 1: ВЫБОР КЛАССА ---
    def show_class_selection(self):
        self.clear_buttons()
        self.add_option("Стрелок", lambda x: self.select_class("стрелок", 65, 30))
        self.add_option("Маг", lambda x: self.select_class("маг", 50, 55))
        self.add_option("Воин", lambda x: self.select_class("воин", 80, 25))

    def select_class(self, name, hp, dm):
        self.role = name
        self.hp = hp
        self.dm = dm
        
        if name == "стрелок":
            self.log("вы выбрали стрелка,стреляй туды,стреляй сюда.")
            self.log("Поздравляю вы выбрали стрелка! ХП 65 УРОН 30")
        elif name == "маг":
            self.log("Вы выбрали мага, кастуй туды, кастуй сюда.")
            self.log("Поздравляю вы выбрали мага! ХП 50 УРОН 55")
        elif name == "воин":
            self.log("Вы выбрали воина, руби туды, руби сюда.")
            self.log("Поздравляю вы выбрали воина! ХП 80 УРОН 25")
            
        self.update_status()
        self.show_hardcore_prompt()

    # --- ЭКРАН 2: ХАРДКОР ---
    def show_hardcore_prompt(self):
        self.clear_buttons()
        self.log("\nЛюбишь хардкор (да/нет): ")
        self.add_option("Да", lambda x: self.set_hardcore(True))
        self.add_option("Нет", lambda x: self.set_hardcore(False))

    def set_hardcore(self, choice):
        if choice:
            self.hardcore = True
            self.hp -= 20
            self.dm -= 20
            self.log("Режим хардкора активирован")
            self.log(f"Твой хп {self.hp} твой урон {self.dm}")
            self.spider_hp += 10
            self.spider_dm += 10
        
        self.update_status()
        self.log("\nВы блуждаете по равнине и решив растелить ночлег на вас нападает гигантский паук")
        self.spider_encounter_turn()

    # --- ЭКРАН 3: ГИГАНТСКИЙ ПАУК ---
    def spider_encounter_turn(self):
        self.clear_buttons()
        if self.joke_count >= 1 or self.xa_count >= 1:
            self.add_option("атака", lambda x: self.spider_action("атака"))
            self.add_option("скрыться", lambda x: self.spider_action("скрыться"))
        else:
            self.add_option("атака", lambda x: self.spider_action("атака"))
            self.add_option("п