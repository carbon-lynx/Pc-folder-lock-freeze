import os
import ctypes
import tkinter as tk
import threading
from datetime import datetime
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler

# === CONFIG ===
watched_folder = "C:\\Users\\Student\\Documents"  # Parent folder
target_folder_name = "LucaFolder"

# === LOCK FUNCTION ===
def freeze_computer():
    lock_time = datetime.now().strftime("%H:%M:%S")
    ctypes.windll.user32.BlockInput(True)

    def lock_screen():
        root = tk.Tk()
        root.attributes('-fullscreen', True)
        root.configure(bg='black')
        root.title("System Locked")

        message = (
            "THIS PC HAS BEEN LOCKED\n"
            f"DUE TO UNAUTHORIZED FOLDER DELETION\n\n"
            f"Time of deletion: {lock_time}\n"
            "Please contact the system administrator."
        )

        label = tk.Label(
            root,
            text=message,
            fg="red",
            bg="black",
            font=("Courier New", 30),
            justify="center"
        )
        label.pack(expand=True)

        root.protocol("WM_DELETE_WINDOW", lambda: None)
        root.bind("<Alt-F4>", lambda e: "break")
        root.bind("<Escape>", lambda e: "break")
        root.mainloop()

    threading.Thread(target=lock_screen).start()


# === WATCHDOG HANDLER ===
class FolderDeleteHandler(FileSystemEventHandler):
    def on_deleted(self, event):
        if event.is_directory and target_folder_name in event.src_path:
            freeze_computer()


# === START MONITORING ===
observer = Observer()
event_handler = FolderDeleteHandler()
observer.schedule(event_handler, watched_folder, recursive=False)
observer.start()

try:
    print("Monitoring folder deletion...")
    while True:
        pass
except KeyboardInterrupt:
    observer.stop()
observer.join()
