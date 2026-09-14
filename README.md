import random
from kivy.app import App
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.button import Button
from kivy.uix.scrollview import ScrollView
from kivy.uix.label import Label
from kivy.clock import Clock
from kivy.core.window import Window

# Эмуляция экрана телефона при тестах на ПК(на самом смартфоне растянется на весь экран)
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
            self.status_label.text = f"Твой хп {self.hp} твой урон {dm}" if self.hardcore else f"ХП: {self.hp} | Урон: {self.dm} | Репутация: {self.rp}"

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
            self.log("Поздравляю вы выбрали\tвойна! ХП 80 УРОН 25")
            
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
            self.add_option("пошутить", lambda x: self.spider_action("пошутить"))
            self.add_option("скрыться", lambda x: self.spider_action("скрыться"))
            self.add_option("издеваться", lambda x: self.spider_action("издеваться"))

    def spider_action(self, action):
        if action == "скрыться":
            self.log("Вы успешно сбежали с поля боя но вас всё таки цапнул паук!")
            self.hp -= self.spider_dm
            self.log(f"минус {self.spider_dm} hp ваш hp:{self.hp}")
            self.update_status()
            if self.hp <= 0:
                self.log("хп упало до нуля (Вы умерли)")
                self.log("К сожелению вас добил яд")
                self.game_over()
            else:
                self.post_spider_transition("скрыться")

        elif action == "атака":
            self.spider_hp -= self.dm
            self.rp -= 1
            self.update_status()
            if self.spider_hp <= 0:
                self.log("Ты убил паука зачем? У него же была семья")
            else:
                self.log("Вы ударили паука он испугался и убежал")
            self.post_spider_transition("атака")

        elif action == "пошутить":
            self.xa_count += 1
            self.clear_buttons()
            self.log("...") # Имитация ожидания
            Clock.schedule_once(lambda dt: self.spider_joke_done(), 3.5)

        elif action == "издеваться":
            self.joke_count += 1
            self.clear_buttons()
            self.log("выбери издевательство (оскорбить/игнорировать): ")
            self.add_option("оскорбить", lambda x: self.insult_spider())
            self.add_option("игнорировать", lambda x: self.ignore_spider())

    def spider_joke_done(self):
        self.log("Паук уполз, пока вы вместе смеялись!")
        self.rp += 1
        self.update_status()
        self.post_spider_transition("пошутить")

    def insult_spider(self):
        self.log("Паук возмущён вашим нахальством и стрельнул вам паутиной в рот")
        self.hp -= self.spider_dm
        self.rp -= 1
        self.log(f"Вы получили {self.spider_dm} урона")
        self.update_status()
        if self.hp <= 0:
            self.log("хп упало до нуля (Вы умерли)")
            self.log("К сожелению вас добил яд")
            self.game_over()
        else:
            self.spider_encounter_turn()

    def ignore_spider(self):
        self.log("Паук вас покусал и вы умерли от отравления (было глупо игнорировать врага)")
        self.spider_dm += 999
        self.hp -= self.spider_dm
        self.update_status()
        self.game_over()

    def post_spider_transition(self, last_action):
        self.clear_buttons()
        if last_action == "пошутить":
            Clock.schedule_once(lambda dt: self.log_before_map("Вы передумали спать и решили побродить"), 2.5)
        elif last_action in ["атака", "скрыться"]:
            Clock.schedule_once(lambda dt: self.log_before_map("Паук мог рассказать о вас лучше сменить местоположение"), 2.5)

    def log_before_map(self, text):
        self.log(text)
        Clock.schedule_once(lambda dt: self.show_map_selection(), 4.0)

    # --- ЭКРАН 4: ВЫБОР ЛОКАЦИИ (ГЛАВНАЯ КАРТА) ---
    def show_map_selection(self):
        self.clear_buttons()
        self.log("\n1. Кладбище мертвецов (рай некроманта)")
        self.log("2. Паучье логово (рекомендуется последним)")
        self.log("3. Пещера (возможно там будут артефакты)")
        
        self.add_option("1. Кладбище мертвецов", lambda x: self.go_to_location("1"))
        self.add_option("2. Паучье логово", lambda x: self.go_to_location("2"))
        self.add_option("3. Пещера", lambda x: self.go_to_location("3"))

    def go_to_location(self, action):
        if action == "кладбище мертвецов" or action == "1":
            if self.kl == 1: 
                self.log("Ты и так тут")
                self.show_map_selection()
            else:
                self.kl += 1
                if self.ph == 1: self.ph -= 1
                self.log("Вы проходили по местности")
                Clock.schedule_once(lambda dt: self.log("Осматривая могилы..."), 3.0)
                Clock.schedule_once(lambda dt: self.cemetery_event_trigger(), 8.0) # 3+5 секунд как в оригинале

        elif action == "пещера" or action == "3":
            if self.ph == 1:
                self.log("Ты и так тут")
                self.show_map_selection()
            else:
                self.ph += 1
                if self.kl == 1: self.kl -= 1
                self.log("Вы зашли в пещеру")
                Clock.schedule_once(lambda dt: self.log("Вы идёте по пещере"), 2.0)
                Clock.schedule_once(lambda dt: self.log("Вы идёте по пещере..."), 5.0)
                Clock.schedule_once(lambda dt: self.cave_event_trigger(), 8.0)

        elif action == "логово" or action == "2" or action == "паучье логово":
            self.log("Вы прошли за пауком")
просьбой убить его")Clock.schedule_once(lambda dt: self.kill_goblin_peacefully(), 2.0)def kill_goblin_peacefully(self):self.log("Вы с радостью убиваете гоблина")self.final_reward_check()# --- ЭКРАН 5: КОНЦОВКА И РЕЗУЛЬТАТЫ ---def final_reward_check(self):self.clear_buttons()if self.rp >= 1:Clock.schedule_once(lambda dt: self.log("Вы вернулись"), 5.0)Clock.schedule_once(lambda dt: self.log("Вам вручили награду"), 7.0)# Из-за ограничений мобильных платформ colorama заменена на Kivy-теги разметки (текст тот же!)Clock.schedule_once(lambda dt: self.log("[color=ffff00]Ваша награда: Ядовитый кинжал,Рюкзак в нём лежали перо с черниломи и бумагой.[/color]"), 8.5)Clock.schedule_once(lambda dt: self.give_final_rep(), 8.5)else:Clock.schedule_once(lambda dt: self.game_over(), 2.5)def give_final_rep(self):self.log("И одну репутацию")self.rp += 1self.update_status()Clock.schedule_once(lambda dt: self.game_over(), 2.5)def game_over(self):self.clear_buttons()self.update_status()self.log(f"""Это пока всё спасибо за игру вы:было хп {self.hp} было урона {self.dm} репутация среди пауков: {self.rp}""")if self.hp <= 0:self.log("вы не выжили")else:self.log("Вы выйграли")if self.hardcore:Clock.schedule_once(lambda dt: self.log("Вы играли в хардкор режим"), 2.0)else:Clock.schedule_once(lambda dt: self.log("Вы казуал"), 2.0)Clock.schedule_once(lambda dt: self.log("В разработке участвовал bluefire и 5 бессонных ночей"), 2.0)# Кнопка перезапуска квестаself.add_option("Сыграть заново", lambda x: self.restart_app())def restart_app(self):App.get_running_app().root.clear_widgets()App.get_running_app().root.init()class QuestApp(App):def build(self):self.title = "Квест Пауков"return TextQuestGame()if name == 'main':QuestApp().run()