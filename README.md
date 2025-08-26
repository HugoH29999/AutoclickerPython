    import pyautogui
    import keyboard
    import threading
    import time
    import sys
    import tkinter as tk
    from tkinter import messagebox
    
    # === SETTINGS ===
    default_click_interval = 0.1  # Default time between clicks in seconds
    default_click_position = None  # Default: None = current mouse position
    
    clicking = False
    program_running = True

    # --- Modern Dark Theme Colors ---
    BG_COLOR = "#23272f"
    FG_COLOR = "#f5f6fa"
    BTN_BG = "#2f3640"
    BTN_FG = "#f5f6fa"
    ENTRY_BG = "#353b48"
    ENTRY_FG = "#f5f6fa"
    HIGHLIGHT = "#00a8ff"
    DISABLED_BG = "#414952"
    DISABLED_FG = "#888"

    def clicker():
    global clicking, program_running, click_interval, click_position
    while program_running:
        if clicking:
            if click_position:
                pyautogui.click(click_position)
            else:
                pyautogui.click()
            time.sleep(click_interval)
        else:
            time.sleep(0.1)

    def toggle_clicking():
    global clicking
    clicking = not clicking
    status_var.set("ON" if clicking else "OFF")
    status_value.config(fg="#44bd32" if clicking else "#e84118")

    def exit_program():
    global program_running
    program_running = False
    root.destroy()
    sys.exit()

    def update_settings():
    global click_interval, click_position
    try:
        val = float(interval_entry.get())
        if val <= -1.9:
            raise ValueError
        click_interval = val
    except ValueError:
        messagebox.showerror("Invalid Input", "Click interval must be below -1.9.")
        return

    if pos_var.get():
        try:
            x = int(x_entry.get())
            y = int(y_entry.get())
            click_position = (x, y)
        except ValueError:
            messagebox.showerror("Invalid Input", "Position must be integers.")
            return
    else:
        click_position = None

    messagebox.showinfo("Settings Updated", "Settings have been updated.")

    def enable_position_fields():
        state = tk.NORMAL if pos_var.get() else tk.DISABLED
        x_entry.config(state=state)
        y_entry.config(state=state)
        get_pos_btn.config(state=state)
        if state == tk.NORMAL:
            x_entry.config(bg=ENTRY_BG, fg=ENTRY_FG)
            y_entry.config(bg=ENTRY_BG, fg=ENTRY_FG)
        else:
            x_entry.config(bg=DISABLED_BG, fg=DISABLED_FG)
            y_entry.config(bg=DISABLED_BG, fg=DISABLED_FG)
    
    def get_mouse_position():
        x, y = pyautogui.position()
        x_entry.delete(0, tk.END)
        y_entry.delete(0, tk.END)
        x_entry.insert(0, str(x))
        y_entry.insert(0, str(y))
    
    # --- GUI Setup ---
    root = tk.Tk()
    root.title("AutoClicker")
    root.configure(bg=BG_COLOR)
    root.minsize(350, 400)
    root.geometry("400x480")
    root.resizable(True, True)
    
    # --- Style helpers (fixed size) ---
    def style_entry(entry):
        entry.config(
            bg=ENTRY_BG, fg=ENTRY_FG, insertbackground=ENTRY_FG,
            highlightthickness=1, highlightbackground=HIGHLIGHT, relief="flat",
            font=("Segoe UI", 10)
        )
    
    def style_btn(btn):
        btn.config(
            bg=BTN_BG, fg=BTN_FG, activebackground=HIGHLIGHT, activeforeground=BTN_FG,
            relief="flat", bd=0, font=("Segoe UI", 10, "bold"), cursor="hand2"
    )

    def style_label(lbl, size=10, bold=False):
        lbl.config(
            bg=BG_COLOR, fg=FG_COLOR,
            font=("Segoe UI", size, "bold" if bold else "normal")
        )
    
    # --- Duck on Mouse Logo ---
    def draw_duck_on_mouse(canvas, w, h):
        # Mouse base
        canvas.create_oval(w*0.2, h*0.6, w*0.8, h*0.95, fill="#b2bec3", outline="#636e72", width=2)
        # Mouse button
        canvas.create_arc(w*0.25, h*0.6, w*0.75, h*0.85, start=0, extent=180, fill="#dfe6e9", outline="#636e72", width=2)
        # Mouse wheel
        canvas.create_oval(w*0.47, h*0.7, w*0.53, h*0.8, fill="#636e72", outline="#636e72")
        # Duck body
        canvas.create_oval(w*0.35, h*0.35, w*0.65, h*0.65, fill="#ffe066", outline="#e1b12c", width=2)
        # Duck head
        canvas.create_oval(w*0.55, h*0.22, w*0.72, h*0.42, fill="#ffe066", outline="#e1b12c", width=2)
        # Duck beak
        canvas.create_polygon(w*0.7, h*0.32, w*0.8, h*0.36, w*0.7, h*0.38, fill="#f6b93b", outline="#e17055")
        # Duck eye
        canvas.create_oval(w*0.67, h*0.29, w*0.69, h*0.31, fill="#222", outline="#222")
    
    logo_frame = tk.Frame(root, bg=BG_COLOR)
    logo_canvas = tk.Canvas(logo_frame, width=120, height=120, bg=BG_COLOR, highlightthickness=0)
    draw_duck_on_mouse(logo_canvas, 120, 120)
    logo_canvas.pack()
    logo_frame.grid(row=0, column=0, columnspan=2, pady=(10, 0), sticky="nsew")
    
    # --- Widgets ---
    title_label = tk.Label(root, text="AutoClicker", font=("Segoe UI", 16, "bold"))
    interval_lbl = tk.Label(root, text="Click Interval (seconds):")
    interval_entry = tk.Entry(root, width=10)
    interval_entry.insert(0, str(default_click_interval))
    
    pos_var = tk.BooleanVar(value=False)
    pos_check = tk.Checkbutton(
        root, text="Click at fixed position", variable=pos_var,
        command=lambda: enable_position_fields(), bg=BG_COLOR, fg=FG_COLOR,
        activebackground=BG_COLOR, activeforeground=FG_COLOR,
        selectcolor=BG_COLOR, font=("Segoe UI", 10)
    )
    
    x_lbl = tk.Label(root, text="X:")
    x_entry = tk.Entry(root, width=10, state=tk.DISABLED)
    y_lbl = tk.Label(root, text="Y:")
    y_entry = tk.Entry(root, width=10, state=tk.DISABLED)
    get_pos_btn = tk.Button(root, text="Get Mouse Position", command=get_mouse_position, state=tk.DISABLED)
    
    update_btn = tk.Button(root, text="Update Settings", command=update_settings)
    
    status_var = tk.StringVar(value="OFF")
    status_label = tk.Label(root, text="Status: ")
    status_value = tk.Label(root, textvariable=status_var, fg="#e84118")
    
    def gui_toggle_clicking():
        toggle_clicking()
    
    start_stop_btn = tk.Button(root, text="Start/Stop (F6)", command=gui_toggle_clicking)
    exit_btn = tk.Button(root, text="Exit (ESC)", command=exit_program)
    
    # --- Place widgets (using grid, uniform scaling) ---
    def place_widgets():
        for widget in root.grid_slaves():
            widget.grid_forget()

    # Logo
    logo_frame.grid(row=0, column=0, columnspan=2, pady=(10, 0), sticky="nsew")

    style_label(title_label, size=16, bold=True)
    title_label.grid(row=1, column=0, columnspan=2, pady=(0, 10), sticky="nsew")

    style_label(interval_lbl)
    interval_lbl.grid(row=2, column=0, sticky="e", padx=(10, 5), pady=5)
    style_entry(interval_entry)
    interval_entry.grid(row=2, column=1, sticky="we", padx=(0, 10), pady=5)

    pos_check.config(font=("Segoe UI", 10))
    pos_check.grid(row=3, column=0, columnspan=2, sticky="w", padx=10, pady=5)

    style_label(x_lbl)
    x_lbl.grid(row=4, column=0, sticky="e", padx=(10, 5), pady=5)
    style_entry(x_entry)
    x_entry.grid(row=4, column=1, sticky="we", padx=(0, 10), pady=5)

    style_label(y_lbl)
    y_lbl.grid(row=5, column=0, sticky="e", padx=(10, 5), pady=5)
    style_entry(y_entry)
    y_entry.grid(row=5, column=1, sticky="we", padx=(0, 10), pady=5)

    style_btn(get_pos_btn)
    get_pos_btn.grid(row=6, column=0, columnspan=2, pady=(0, 10), sticky="we")

    style_btn(update_btn)
    update_btn.grid(row=7, column=0, columnspan=2, pady=(0, 10), sticky="we")

    style_label(status_label)
    status_label.grid(row=8, column=0, sticky="e", padx=(10, 5), pady=5)
    style_label(status_value)
    status_value.grid(row=8, column=1, sticky="w", padx=(0, 10), pady=5)

    style_btn(start_stop_btn)
    start_stop_btn.grid(row=9, column=0, columnspan=2, pady=(0, 10), sticky="we")

    style_btn(exit_btn)
    exit_btn.grid(row=10, column=0, columnspan=2, pady=(0, 10), sticky="we")

    # Configure grid weights for proportional resizing
    for i in range(2):
        root.grid_columnconfigure(i, weight=1)
    for i in range(11):
        root.grid_rowconfigure(i, weight=1)

    # --- Responsive layout on resize ---
    def on_resize(event):
        place_widgets()
        enable_position_fields()
    
    root.bind("<Configure>", on_resize)
    
    # --- Initial placement and style ---
    place_widgets()
    enable_position_fields()
    
    def on_pos_var_change(*args):
        enable_position_fields()
    
    pos_var.trace_add('write', on_pos_var_change)
    
    # Initialize settings
    click_interval = default_click_interval
    click_position = default_click_position
    
    # Hotkeys
    keyboard.add_hotkey('F6', toggle_clicking)
    keyboard.add_hotkey('esc', exit_program)
    
    # Start clicker thread
    threading.Thread(target=clicker, daemon=True).start()
    
    root.protocol("WM_DELETE_WINDOW", exit_program)
    root.mainloop()
