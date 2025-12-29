import tkinter as tk
from tkinter import ttk, messagebox
from datetime import datetime
import math
import random
import json
import os

class EnhancedBirthdayCountdown:
    def __init__(self, root):
        """Initialize the birthday countdown application"""
        self.root = root
        self.root.title("🎉 Ultimate Birthday Countdown 🎉")
        self.root.geometry("700x900")
        self.root.configure(bg="#1a1a2e")

        # Birthday data
        self.birthdate = None
        
        # Animation variables
        self.animation_angle = 0  # Genel animasyon açısı
        self.flame_offset = 0  # Mum alevi animasyonu için
        self.is_birthday_today = False  # Bugün doğum günü mü?
        
        # Effect lists
        self.fireworks = []  # Havai fişek parçacıkları
        self.balloon_y_offset = 0  # Balon sallanma efekti
        self.stars = []  # Yüzen yıldızlar
        
        # Settings
        self.show_age = tk.BooleanVar(value=True)  # Yaş göster/gizle
        self.fullscreen_mode = False  # Tam ekran modu
        
        # Load data 
        self.saved_birthdays = self.load_birthdays()
        
        # Create UI and start background elements
        self.setup_ui()
        # Yüzen elementleri oluştur
        self.create_floating_elements()

    def load_birthdays(self):
        """Kaydedilen doğum günlerini JSON dosyasından yükle"""
        try:
            if os.path.exists("birthdays.json"):
                with open("birthdays.json", "r") as f:
                    return json.load(f)
        except:
            pass
        return {}

    def save_birthdays(self):
        """Doğum günlerini JSON dosyasına kaydet"""
        try:
            with open("birthdays.json", "w") as f:
                json.dump(self.saved_birthdays, f)
        except:
            pass

    def setup_ui(self):
        """Kullanıcı arayüzünü oluştur"""
        
        # Top Control Bar (Checkboxes and Fullscreen)
        control_frame = tk.Frame(self.root, bg="#1a1a2e")
        control_frame.pack(pady=10)
        
        # Yaş gösterme checkbox'ı
        age_check = tk.Checkbutton(
            control_frame, 
            text="Show Age", 
            variable=self.show_age, 
            bg="#1a1a2e", 
            fg="white", 
            selectcolor="#1a1a2e",
            font=("Arial", 11), 
            activebackground="#1a1a2e"
        )
        age_check.pack(side=tk.LEFT, padx=15)
        
        # Tam ekran butonu
        fullscreen_btn = tk.Button(
            control_frame, 
            text="⛶ Fullscreen", 
            command=self.toggle_fullscreen,
            bg="#1a1a2e", 
            fg="GREY", 
            font=("Arial", 11),
            relief=tk.FLAT, 
            cursor="hand2"
        )
        fullscreen_btn.pack(side=tk.LEFT, padx=15)

        # TITLE
        self.title_label = tk.Label(
            self.root,
            text="🎂 Birthday Countdown 🎂",
            font=("Arial", 32, "bold"),
            bg="#1a1a2e",
            fg="#f39c12"
        )
        self.title_label.pack(pady=20)

        # DATE ENTER
        tk.Label(
            self.root,
            text="🎈 Enter Birth Date 🎈",
            font=("Arial", 16, "bold"),
            bg="#1a1a2e",
            fg="#ffffff"
        ).pack(pady=10)

        # FRAME FOR ENTER DATE
        date_frame = tk.Frame(self.root, bg="#1a1a2e")
        date_frame.pack(pady=10)

        # DAY
        tk.Label(date_frame, text="Day:", font=("Arial", 12), bg="#1a1a2e", fg="#ffffff").grid(row=0, column=0, padx=5)
        self.day_entry = ttk.Combobox(date_frame, values=list(range(1, 32)), width=5, font=("Arial", 12), state="readonly")
        self.day_entry.grid(row=0, column=1, padx=5)

        # MONTH
        tk.Label(date_frame, text="Month:", font=("Arial", 12), bg="#1a1a2e", fg="#ffffff").grid(row=0, column=2, padx=5)
        months = ["Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"]
        self.month_entry = ttk.Combobox(date_frame, values=months, width=5, font=("Arial", 12), state="readonly")
        self.month_entry.grid(row=0, column=3, padx=5)

        # YEAR
        tk.Label(date_frame, text="Year:", font=("Arial", 12), bg="#1a1a2e", fg="#ffffff").grid(row=0, column=4, padx=5)
        years = list(range(2024, 1899, -1))
        self.year_entry = ttk.Combobox(date_frame, values=years, width=8, font=("Arial", 12), state="readonly")
        self.year_entry.grid(row=0, column=5, padx=5)
        
        # NAME
        name_frame = tk.Frame(self.root, bg="#1a1a2e")
        name_frame.pack(pady=5)
        tk.Label(name_frame, text="Name (optional):", font=("Arial", 11), bg="#1a1a2e", fg="#ffffff").pack(side=tk.LEFT, padx=5)
        self.name_entry = tk.Entry(name_frame, font=("Arial", 11), width=20)
        self.name_entry.pack(side=tk.LEFT, padx=5)

        # BUTTONS
        button_frame = tk.Frame(self.root, bg="#1a1a2e")
        button_frame.pack(pady=15)

        # START BUTTON
        start_button = tk.Button(
            button_frame,
            text="🚀 Start Countdown",
            command=self.start_countdown,
            font=("Arial", 13, "bold"),
            bg="#3498db",
            fg="GREY",
            padx=20,
            pady=10,
            cursor="hand2",
            relief=tk.FLAT,
            bd=0
        )
        start_button.pack(side=tk.LEFT, padx=5)
        
        # SAVE BUTTON
        save_button = tk.Button(
            button_frame,
            text="💾 Save",
            command=self.save_current_birthday,
            font=("Arial", 13, "bold"),
            bg="#2ecc71",
            fg="GREY",
            padx=15,
            pady=10,
            cursor="hand2",
            relief=tk.FLAT,
            bd=0
        )
        save_button.pack(side=tk.LEFT, padx=5)
        
        # LOAD BUTTON
        load_button = tk.Button(
            button_frame,
            text="📂 Load",
            command=self.load_saved_birthday,
            font=("Arial", 13, "bold"),
            bg="#9b59b6",
            fg="GREY",
            padx=15,
            pady=10,
            cursor="hand2",
            relief=tk.FLAT,
            bd=0
        )
        load_button.pack(side=tk.LEFT, padx=5)

        # COUNTDOWN SCREEN
        self.countdown_frame = tk.Frame(self.root, bg="#1a1a2e")
        self.countdown_frame.pack(pady=20)

        # TIMES 
        time_units = [
            ("days", "Days", "#3498db"),
            ("hours", "Hours", "#2ecc71"),
            ("minutes", "Minutes", "#9b59b6"),
            ("seconds", "Seconds", "#e67e22")
        ]

        # LABELS FOR TIME
        for i, (var_name, label_text, color) in enumerate(time_units):
            
            label = tk.Label(
                self.countdown_frame,
                text="0",
                font=("Arial", 48, "bold"),
                bg="#1a1a2e",
                fg="GREY",
                width=3
            )
            label.grid(row=0, column=i, padx=15)
            setattr(self, f"{var_name}_label", label)  # DYNAMIC VAR
            
            
            tk.Label(
                self.countdown_frame,
                text=label_text,
                font=("Arial", 12, "bold"),
                bg="#1a1a2e",
                fg="#ffffff"
            ).grid(row=1, column=i)

        # AGE LABEL
        self.age_label = tk.Label(
            self.root,
            text="",
            font=("Arial", 18, "bold"),
            bg="#1a1a2e",
            fg="#f39c12"
        )
        self.age_label.pack(pady=10)

        # animation canvas
        self.canvas = tk.Canvas(
            self.root,
            width=600,
            height=350,
            bg="#1a1a2e",
            highlightthickness=0
        )
        self.canvas.pack(pady=20)

        # MESSAGE
        self.special_label = tk.Label(
            self.root,
            text="",
            font=("Arial", 28, "bold"),
            bg="#1a1a2e",
            fg="#ff6b6b"
        )
        self.special_label.pack(pady=10)

        # PROGRESS 
        self.progress_label = tk.Label(
            self.root,
            text="",
            font=("Arial", 11),
            bg="#1a1a2e",
            fg="#95a5a6"
        )
        self.progress_label.pack()

        # DRAW FIRST SCENE
        self.draw_birthday_scene()

    def toggle_fullscreen(self):
        """Tam ekran modunu aç/kapat"""
        self.fullscreen_mode = not self.fullscreen_mode
        self.root.attributes("-fullscreen", self.fullscreen_mode)
        if not self.fullscreen_mode:
            self.root.geometry("700x900")

    def create_floating_elements(self):
        """Arka planda yüzen yıldızları oluştur"""
        for _ in range(30):
            x = random.randint(0, 600)
            y = random.randint(0, 350)
            size = random.randint(2, 5)
            speed = random.uniform(0.3, 0.8)
            self.stars.append({
                'x': x, 
                'y': y, 
                'size': size, 
                'speed': speed,
                'alpha': random.random()  
            })

    def draw_birthday_scene(self):
        """Doğum günü sahnesini çiz (pasta, mumlar, balonlar, konfetiler)"""
        # DELETE CANVAS
        self.canvas.delete("all")
        
        # STARS
        for star in self.stars:
           
            alpha = (math.sin(self.animation_angle + star['alpha'] * 10) + 1) / 2
            intensity = int(255 * alpha)
            color = f"#{intensity:02x}{intensity:02x}00"
            
            # DRAW STARS
            self.canvas.create_oval(
                star['x'], star['y'],
                star['x'] + star['size'], star['y'] + star['size'],
                fill=color, outline=""
            )
            
            # MOVE THE STARS
            star['y'] += star['speed']
            if star['y'] > 350:  
                star['y'] = 0
                star['x'] = random.randint(0, 600)

        # BIRTHDAY CAKE TRIPLE LAYER
        cake_colors = ["#8b4513", "#d2691e", "#cd853f"]  # cake colors
        layer_height = 45  # layer height
        
        # DRAW TRIPLE LAYER FROM BOTOM TO UP
        for i in range(3):
            y_pos = 250 - (i * layer_height)  
            width = 220 - (i * 30) 
            x1 = 300 - width // 2  
            x2 = 300 + width // 2  
            
            # Pasta katı
            self.canvas.create_rectangle(
                x1, y_pos, x2, y_pos + layer_height,
                fill=cake_colors[i], 
                outline="#654321", 
                width=2
            )
            
            # CREAMS
            for k in range(5):
                dec_x = x1 + (width // 6) * (k + 1)
                self.canvas.create_oval(
                    dec_x - 8, y_pos + layer_height - 5,
                    dec_x + 8, y_pos + layer_height + 5,
                    fill="#ffe4e1", 
                    outline="#ffb6c1"
                )

        # CNADLES
        # Doğum günüyse daha fazla mum göster
        candle_count = 10 if self.is_birthday_today else 7
        spacing = 200 // (candle_count + 1)  # Mumlar arası boşluk
        
        for i in range(candle_count):
            x = 200 + spacing * (i + 1)
            
            # CANDLE BODY
            self.canvas.create_rectangle(
                x - 6, 95, x + 6, 125,
                fill="#ff6b6b", 
                outline="#c92a2a", 
                width=1
            )
            
            
            self.canvas.create_line(x, 95, x, 85, fill="#2c2c2c", width=2)
            
            # FLAMES
            flame_y = 75 + math.sin(self.flame_offset + i * 0.5) * 4
            
            # 
            self.canvas.create_oval(
                x - 8, flame_y - 10,
                x + 8, flame_y + 5,
                fill="#ffcc00", 
                outline=""
            )
            # 
            self.canvas.create_oval(
                x - 5, flame_y - 7,
                x + 5, flame_y + 2,
                fill="#ffff00", 
                outline=""
            )

        # BALOONS
        balloon_colors = ["#ff6b6b", "#4ecdc4", "#45b7d1", "#f9ca24", "#6c5ce7", "#a29bfe"]
        
        for i, color in enumerate(balloon_colors):
            x = 80 + i * 90
            # Sinüs wave moves
            y = 50 + math.sin(self.balloon_y_offset + i) * 15
            
            # Balon
            self.canvas.create_oval(
                x - 15, y - 20, x + 15, y + 10,
                fill=color, 
                outline="#333", 
                width=2
            )
          
            self.canvas.create_oval(
                x - 8, y - 12, x - 3, y - 7,
                fill="white", 
                outline=""
            )
           
            self.canvas.create_line(x, y + 10, x + 5, y + 50, fill="#333", width=2)

        # ======= KONFETİLER =======
        confetti_colors = ["#ff6b6b", "#4ecdc4", "#45b7d1", "#f9ca24", "#6c5ce7", "#ff69b4"]
        # Doğum günüyse daha fazla konfeti
        confetti_count = 80 if self.is_birthday_today else 30
        
        for i in range(confetti_count):
            x = 50 + (i * 15) % 500
            # Sinüs dalgası ile hareket
            y = 30 + math.sin(self.animation_angle * 2 + i * 0.3) * 25
            
            color = confetti_colors[i % len(confetti_colors)]
            size = 3
            
            # CONFETTIS
            self.canvas.create_oval(
                x - size, y - size, x + size, y + size,
                fill=color, 
                outline=""
            )

        # SPARKLES
        if self.is_birthday_today:
          # DRAW SPARKLES
            for fw in self.fireworks[:]:
                for particle in fw['particles']:
                    # SPARKLES POSITION
                    x = fw['x'] + particle['vx'] * fw['age']
                    y = fw['y'] + particle['vy'] * fw['age'] + 0.5 * fw['age']
                    
                    # IF IT IS IN THE SCREEN DRAW
                    if 0 <= x <= 600 and 0 <= y <= 350:
                        size = max(1, 3 - (fw['age'] / 15))
                        self.canvas.create_oval(
                            x - size, y - size, x + size, y + size,
                            fill=fw['color'], 
                            outline=""
                        )
                
                fw['age'] += 1
                # REMOVE THE OLD SPARKLES
                if fw['age'] > 30:
                    self.fireworks.remove(fw)
            
            # IMPORT RANDOM SPARKLES
            if random.random() < 0.08 and len(self.fireworks) < 3:
                self.create_firework()

        # UPDATE ANIMATIONS ANGLES
        self.animation_angle += 0.08
        self.flame_offset += 0.15
        self.balloon_y_offset += 0.05

    def create_firework(self):
        """Havai fişek patlaması oluştur"""
        colors = ["#ff6b6b", "#4ecdc4", "#ffcc00", "#ff69b4", "#a29bfe"]
        firework = {
            'x': random.randint(100, 500),  #  X p
            'y': random.randint(50, 150),  #  Y 
            'color': random.choice(colors),  # color
            'age': 0,  # age
            'particles': []  # sparkles
        }
        
        # IMPORT 20 PIEES
        for _ in range(20):
            angle = random.uniform(0, 2 * math.pi)
            speed = random.uniform(2, 5)
            firework['particles'].append({
                'vx': math.cos(angle) * speed,  # X speed
                'vy': math.sin(angle) * speed   # Y speed
            })
        
        self.fireworks.append(firework)

    def start_countdown(self):
        """Geri sayımı başlat"""
        try:
            # area check
            if not self.day_entry.get() or not self.month_entry.get() or not self.year_entry.get():
                messagebox.showerror("Invalid Date", "Please select all date fields!")
                return

            # TAKE THE DATE
            day = int(self.day_entry.get())
            months = ["Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"]
            month = months.index(self.month_entry.get()) + 1
            year = int(self.year_entry.get())

            # save the birthdate
            self.birthdate = datetime(year, month, day)
            
            # START COUNTDOWN
            self.update_countdown()

        except (ValueError, AttributeError) as e:
            messagebox.showerror("Invalid Date", "Please enter a valid birth date!")

    def save_current_birthday(self):
        """Mevcut doğum gününü dosyaya kaydet"""
        if not self.birthdate:
            messagebox.showwarning("No Birthday", "Please set a birthday first!")
            return
        
        # TAKE THE NAME OR IMPORT RANDOM NAME
        name = self.name_entry.get().strip()
        if not name:
            name = f"Birthday_{len(self.saved_birthdays) + 1}"
        
        # SAve TO THE LIB
        self.saved_birthdays[name] = {
            'day': self.birthdate.day,
            'month': self.birthdate.month,
            'year': self.birthdate.year
        }
        
        # save
        self.save_birthdays()
        messagebox.showinfo("Saved!", f"Birthday saved as '{name}'")

    def load_saved_birthday(self):
        """Kaydedilmiş doğum gününü yükle"""
        if not self.saved_birthdays:
            messagebox.showinfo("No Saved Birthdays", "No saved birthdays found!")
            return
        
        
        select_window = tk.Toplevel(self.root)
        select_window.title("Load Birthday")
        select_window.geometry("300x400")
        select_window.configure(bg="#1a1a2e")
        
        tk.Label(
            select_window, 
            text="Select a Birthday", 
            font=("Arial", 14, "bold"), 
            bg="#1a1a2e", 
            fg="white"
        ).pack(pady=10)
        
        # LIST BOX
        listbox = tk.Listbox(select_window, font=("Arial", 12), height=15)
        listbox.pack(pady=10, padx=20, fill=tk.BOTH, expand=True)
        
        # SAVE TO THE LISTS
        for name in self.saved_birthdays.keys():
            listbox.insert(tk.END, name)
        
        def load_selected():
            """Seçili doğum gününü yükle"""
            selection = listbox.curselection()
            if selection:
                name = listbox.get(selection[0])
                bd = self.saved_birthdays[name]
                
                # Form alanlarını doldur
                self.day_entry.set(bd['day'])
                months = ["Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"]
                self.month_entry.set(months[bd['month'] - 1])
                self.year_entry.set(bd['year'])
                self.name_entry.delete(0, tk.END)
                self.name_entry.insert(0, name)
                
                select_window.destroy()
                self.start_countdown()
        
        # LOAD BUTTON
        tk.Button(
            select_window, 
            text="Load", 
            command=load_selected,
            bg="#3498db", 
            fg="white", 
            font=("Arial", 12),
            padx=20, 
            pady=5
        ).pack(pady=10)

    def update_countdown(self):
        """Geri sayımı güncelle (her 100ms'de bir)"""
        if self.birthdate is None:
            return

        # TAKE TODAYS  TIME
        now = datetime.now()
        
        # TODAYS DATE
        today_month = now.month
        today_day = now.day
        
        # BIRTHDAY TODAY ?
        birthday_today = (today_month == self.birthdate.month and today_day == self.birthdate.day)
        
        # DEBUG: Konsola yazdır
        print(f"Today: {today_day}/{today_month}, Birthday: {self.birthdate.day}/{self.birthdate.month}, Is Birthday: {birthday_today}")
        
        # CALCULATE NEXT BIRTHDAY
        next_birthday = datetime(now.year, self.birthdate.month, self.birthdate.day)
        
       
        if next_birthday < now:
            next_birthday = datetime(now.year + 1, self.birthdate.month, self.birthdate.day)

        # CALCULATE THE REMAINING TIME
        time_left = next_birthday - now
        days = time_left.days
        hours = time_left.seconds // 3600
        minutes = (time_left.seconds % 3600) // 60
        seconds = time_left.seconds % 60

        # CALCULATE THE AGE
        age = now.year - self.birthdate.year
        # Eğer doğum günü henüz gelmemişse 1 eksilt
        if now.month < self.birthdate.month or (now.month == self.birthdate.month and now.day < self.birthdate.day):
            age -= 1
        next_age = age + 1

        # CALCULATE THE YEARLY PROGRESS
        year_start = datetime(now.year, self.birthdate.month, self.birthdate.day)
        if year_start > now:
            year_start = datetime(now.year - 1, self.birthdate.month, self.birthdate.day)
        year_end = datetime(year_start.year + 1, self.birthdate.month, self.birthdate.day)
        
        total_seconds = (year_end - year_start).total_seconds()
        elapsed_seconds = (now - year_start).total_seconds()
        progress = (elapsed_seconds / total_seconds) * 100

        # BIRTHDAY MODE 
        if birthday_today:
            print("🎉 BIRTHDAY MODE ACTIVATED!")  # DEBUG
            self.is_birthday_today = True
            
            # SHOW EMOJIS
            self.days_label.config(text="🎉", fg="#ff6b6b")
            self.hours_label.config(text="🎂", fg="#ff6b6b")
            self.minutes_label.config(text="🎈", fg="#ff6b6b")
            self.seconds_label.config(text="🎁", fg="#ff6b6b")
            
            # MESSAGES
            messages = [
                "HAPPY BIRTHDAY! 🎉",
                "🎊 CELEBRATE! 🎊",
                "✨ SPECIAL DAY! ✨",
                "🎈 PARTY TIME! 🎈"
            ]
            message_index = int(self.animation_angle * 2) % len(messages)
            self.special_label.config(text=messages[message_index])
            
            # SHOW AGE
            if self.show_age.get():
                self.age_label.config(text=f"🎂 Turning {next_age} Today! 🎂")
            else:
                self.age_label.config(text="")
        else:
            # Normal mod
            self.is_birthday_today = False
            
            # SHOW THE NUMBERS
            self.days_label.config(text=str(days), fg="#3498db")
            self.hours_label.config(text=str(hours), fg="#2ecc71")
            self.minutes_label.config(text=str(minutes), fg="#9b59b6")
            self.seconds_label.config(text=str(seconds), fg="#e67e22")
            self.special_label.config(text="")
            
            # AGE INFO
            if self.show_age.get():
                self.age_label.config(text=f"Will be {next_age} years old")
            else:
                self.age_label.config(text="")
        
        #
        self.progress_label.config(text=f"Year Progress: {progress:.1f}%")

        # 
        self.draw_birthday_scene()
        
        # Call the update again after 100ms
        self.root.after(100, self.update_countdown)


# ======= PROGRAMI BAŞLAT =======
if __name__ == "__main__":
    root = tk.Tk()
    app = EnhancedBirthdayCountdown(root)
    root.mainloop()
