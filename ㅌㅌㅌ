# tic_tac_toe_pokemon.py
# --------------------------------------------------
# 실행 방법:
#   pip install pillow requests오ㅑㅕㄹ여ㅗㅁ새ㅑㅕㄱ내;ㅓㄹㅎㅈ
#   python tic_tac_toe_pokemon.py
# --------------------------------------------------

import io
import random
import threading
from dataclasses import dataclass
from typing import Optional, List

import tkinter as tk
from tkinter import ttk, messagebox

import requests
from PIL import Image, ImageTk

# ===== 설정 =====
USE_ONLY_DITTO = False  # True로 바꾸면 캐릭터 이미지는 항상 Ditto로 고정
POKEMON_LIST = ["ditto", "pikachu", "bulbasaur", "charmander", "squirtle", "eevee", "jigglypuff", "snorlax"]

CELL_SIZE = 120  # 보드 셀 버튼 크기


@dataclass
class Character:
    name: str
    image: Optional[Image.Image] = None
    imgtk_small: Optional[ImageTk.PhotoImage] = None


def fetch_pokemon_image(name: str) -> Optional[Image.Image]:
    try:
        mon = "ditto" if USE_ONLY_DITTO else name
        meta = requests.get(f"https://pokeapi.co/api/v2/pokemon/{mon}", timeout=12)
        meta.raise_for_status()
        sprite_url = meta.json().get("sprites", {}).get("front_default")
        if not sprite_url:
            return None
        img_resp = requests.get(sprite_url, timeout=12)
        img_resp.raise_for_status()
        return Image.open(io.BytesIO(img_resp.content)).convert("RGBA")
    except Exception:
        return None


# ===== 화면 1: 시작 화면 =====
class StartScreen(ttk.Frame):
    def __init__(self, master, on_mode):
        super().__init__(master, padding=24)
        ttk.Label(self, text="틱택토", font=("Malgun Gothic", 26, "bold")).pack(pady=(0, 12))
        ttk.Label(self, text="모드를 선택하세요", font=("Malgun Gothic", 12)).pack(pady=(0, 20))

        btns = ttk.Frame(self)
        btns.pack()
        ttk.Button(btns, text="사람 대 사람", command=lambda: on_mode("pvp")).grid(row=0, column=0, padx=8, pady=8)
        ttk.Button(btns, text="인공지능 대 사람", command=lambda: on_mode("pvai")).grid(row=0, column=1, padx=8, pady=8)


# ===== 개별 캐릭터 선택 위젯 =====
class CharacterPicker(ttk.Frame):
    def __init__(self, master, title: str, names: List[str]):
        super().__init__(master, padding=10)
        self.names = names
        self.idx = 0
        self.char: Optional[Character] = None
        self._imgtk_large = None

        ttk.Label(self, text=title, font=("Malgun Gothic", 12, "bold")).grid(row=0, column=0, columnspan=3, pady=(0, 8))
        self.canvas = tk.Canvas(self, width=200, height=200, bg="#f4f4f4", highlightthickness=0)
        self.canvas.grid(row=1, column=0, columnspan=3, pady=(0, 8))

        ttk.Button(self, text="▲", width=6, command=self.prev).grid(row=2, column=0, padx=4)
        self.name_label = ttk.Label(self, text="", font=("Malgun Gothic", 11))
        self.name_label.grid(row=2, column=1, padx=4)
        ttk.Button(self, text="▼", width=6, command=self.next).grid(row=2, column=2, padx=4)

        self._refresh()

    def current_name(self) -> str:
        return self.names[self.idx]

    def prev(self):
        self.idx = (self.idx - 1) % len(self.names)
        self._refresh()

    def next(self):
        self.idx = (self.idx + 1) % len(self.names)
        self._refresh()

    def _refresh(self):
        name = self.current_name()
        self.name_label.config(text=name.capitalize())
        self.canvas.delete("all")
        def worker():
            img = fetch_pokemon_image(name)
            if not img:
                self.after(0, lambda: self.canvas.create_text(100, 100, text="(이미지 없음)", fill="#666"))
                self.char = Character(name=name, image=None)
                return
            im = img.copy()
            im.thumbnail((200, 200), Image.LANCZOS)
            imgtk = ImageTk.PhotoImage(im)
            def show():
                self._imgtk_large = imgtk
                self.canvas.create_image(100, 100, image=self._imgtk_large)
                self.char = Character(name=name, image=img)
            self.after(0, show)
        threading.Thread(target=worker, daemon=True).start()


# ===== 화면 2: 캐릭터 선택 =====
class SelectScreen(ttk.Frame):
    def __init__(self, master, mode: str, on_start, on_back):
        super().__init__(master, padding=20)
        self.mode = mode
        self.on_start = on_start
        self.on_back = on_back

        ttk.Label(self, text="캐릭터 선택", font=("Malgun Gothic", 20, "bold")).pack(pady=(0, 12))

        area = ttk.Frame(self)
        area.pack(pady=6)

        if mode == "pvp":
            self.p1 = CharacterPicker(area, "플레이어 1", POKEMON_LIST)
            self.p2 = CharacterPicker(area, "플레이어 2", POKEMON_LIST)
            self.p1.grid(row=0, column=0, padx=8)
            self.p2.grid(row=0, column=1, padx=8)
        else:
            self.p1 = CharacterPicker(area, "플레이어", POKEMON_LIST)
            self.p1.grid(row=0, column=0, padx=8)
            self.p2 = None  # AI는 나중에 자동 할당

        btns = ttk.Frame(self)
        btns.pack(pady=10)
        ttk.Button(btns, text="게임 시작하기", command=self._start).grid(row=0, column=0, padx=6)
        ttk.Button(btns, text="처음으로", command=on_back).grid(row=0, column=1, padx=6)

    def _start(self):
        p1 = self.p1.char
        if not (p1 and p1.name):
            messagebox.showinfo("잠시만!", "플레이어 캐릭터 이미지 로딩 중입니다.")
            return

        if self.mode == "pvp":
            p2 = self.p2.char if self.p2 else None
            if not (p2 and p2.name):
                messagebox.showinfo("잠시만!", "플레이어 2 캐릭터 이미지 로딩 중입니다.")
                return
        else:
            # AI 캐릭터 랜덤 선택 + 이미지 로드(동기) 한 번만
            name = random.choice(POKEMON_LIST)
            img = fetch_pokemon_image(name)
            p2 = Character(name=name, image=img)

        # 보드용 소형 이미지 준비
        for ch in (p1, p2):
            if ch and ch.image:
                small = ch.image.copy()
                small.thumbnail((CELL_SIZE - 16, CELL_SIZE - 16), Image.LANCZOS)
                ch.imgtk_small = ImageTk.PhotoImage(small)

        self.on_start(p1, p2)


# ===== 화면 3: 게임 =====
class GameScreen(ttk.Frame):
    def __init__(self, master, mode: str, p1: Character, p2: Character, on_home):
        super().__init__(master, padding=16)
        self.mode = mode
        self.p1, self.p2 = p1, p2
        self.on_home = on_home

        self.board = [" "] * 9
        self.turn = "X"
        self.over = False

        title = "사람 대 사람" if mode == "pvp" else "인공지능 대 사람"
        top = ttk.Frame(self)
        top.pack(fill="x")
        ttk.Label(top, text=title, font=("Malgun Gothic", 18, "bold")).pack(side="left")
        self.turn_lbl = ttk.Label(top, text="", font=("Malgun Gothic", 11))
        self.turn_lbl.pack(side="right")

        grid = ttk.Frame(self)
        grid.pack(pady=10)
        self.cells: List[tk.Button] = []
        for r in range(3):
            for c in range(3):
                idx = r*3 + c
                b = tk.Button(grid, width=6, height=3, command=lambda i=idx: self.play(i))
                b.grid(row=r, column=c, padx=4, pady=4, ipadx=10, ipady=10)
                self.cells.append(b)

        self.result_lbl = ttk.Label(self, text="", font=("Malgun Gothic", 12))
        self.result_lbl.pack(pady=(8, 12))
        ttk.Button(self, text="처음으로", command=self.on_home).pack()

        self._update_turn()

    def _update_turn(self):
        who = "플레이어 1" if self.turn == "X" else ("플레이어 2" if self.mode == "pvp" else "인공지능")
        name = self.p1.name.capitalize() if self.turn == "X" else self.p2.name.capitalize()
        self.turn_lbl.config(text=f"{who} ({name}) 차례")

    def play(self, i: int):
        if self.over or self.board[i] != " ":
            return
        self._place(i, self.turn)

        w = self._winner()
        if w or self._draw():
            self._finish(w)
            return

        if self.mode == "pvai" and self.turn == "O" and not self.over:
            # 간단한 AI: 승리/방어 우선, 그다음 중앙/코너/랜덤
            self.after(150, self._ai_move)

    def _ai_move(self):
        move = self._best_move_for("O")
        if move is None:
            empties = [i for i, v in enumerate(self.board) if v == " "]
            if not empties: 
                return
            move = random.choice(empties)
        self._place(move, "O")
        w = self._winner()
        if w or self._draw():
            self._finish(w)

    def _place(self, i: int, mark: str):
        self.board[i] = mark
        btn = self.cells[i]
        char = self.p1 if mark == "X" else self.p2
        if char.imgtk_small:
            btn.config(image=char.imgtk_small, text="")
        else:
            btn.config(text=mark, font=("Malgun Gothic", 18, "bold"))
        btn.config(state="disabled")
        self.turn = "O" if mark == "X" else "X"
        self._update_turn()

    def _lines(self):
        return [
            (0,1,2),(3,4,5),(6,7,8),
            (0,3,6),(1,4,7),(2,5,8),
            (0,4,8),(2,4,6)
        ]

    def _winner(self) -> Optional[str]:
        for a,b,c in self._lines():
            if self.board[a] != " " and self.board[a]==self.board[b]==self.board[c]:
                return self.board[a]
        return None

    def _draw(self) -> bool:
        return all(cell != " " for cell in self.board)

    # 간단한 선택형 AI (미니맥스보다 가볍게)
    def _best_move_for(self, mark: str) -> Optional[int]:
        opponent = "X" if mark == "O" else "O"
        empties = [i for i, v in enumerate(self.board) if v == " "]
        # 1) 이길 수 있으면 이기기
        for i in empties:
            self.board[i] = mark
            if self._winner() == mark:
                self.board[i] = " "
                return i
            self.board[i] = " "
        # 2) 막아야 하면 막기
        for i in empties:
            self.board[i] = opponent
            if self._winner() == opponent:
                self.board[i] = " "
                return i
            self.board[i] = " "
        # 3) 중앙 -> 코너 -> 랜덤
        for pref in [4,0,2,6,8]:
            if pref in empties:
                return pref
        return random.choice(empties) if empties else None

    def _finish(self, winner: Optional[str]):
        self.over = True
        for b in self.cells:
            b.config(state="disabled")
        if winner == "X":
            msg = f"승자: 플레이어 1 ({self.p1.name.capitalize()})"
        elif winner == "O":
            msg = f"승자: {'플레이어 2' if self.mode=='pvp' else '인공지능'} ({self.p2.name.capitalize()})"
        else:
            msg = "무승부"
        self.result_lbl.config(text=msg)


# ===== 앱 컨테이너 =====
class App(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title("틱택토 — Pokémon")
        self.geometry("560x640")
        self.minsize(520, 600)
        self._screen = None
        self.show_start()

    def _set(self, widget: tk.Widget):
        if self._screen is not None:
            self._screen.destroy()
        self._screen = widget
        self._screen.pack(fill="both", expand=True)

    def show_start(self):
        self._set(StartScreen(self, self._on_mode))

    def _on_mode(self, mode: str):
        self._set(SelectScreen(self, mode, self._start_game, self.show_start))

    def _start_game(self, p1: Character, p2: Character):
        self._set(GameScreen(self, mode=("pvp" if isinstance(self._screen, SelectScreen) and self._screen.mode=="pvp" else "pvai"), p1=p1, p2=p2, on_home=self.show_start))


if __name__ == "__main__":
    App().mainloop()
