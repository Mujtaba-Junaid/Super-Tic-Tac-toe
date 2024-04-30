

from tkinter import *
from tkinter.font import Font
from tkinter import messagebox
from copy import deepcopy
import os



class GameBoard:

    def __init__(self, other=None):
        self.player = 'X'
        self.opponent = 'O'
        self.empty = '.'
        self.size = 3
        self.spaces = {}
        for y in range(self.size):
            for x in range(self.size):
                self.spaces[x, y] = self.empty
        if other:
            self.__dict__ = deepcopy(other.__dict__)

    def move(self, x, y):
        board = GameBoard(self)
        board.spaces[x, y] = board.player
        (board.player, board.opponent) = (board.opponent, board.player)
        return board

    def __minmax(self, player):
        if self.won():
            if player:
                return (-1, None)
            else:
                return (+1, None)
        elif self.tied():
            return (0, None)
        elif player:
            best = (-2, None)
            for x, y in self.spaces:
                if self.spaces[x, y] == self.empty:
                    value = self.move(x, y).__minmax(not player)[0]
                    if value > best[0]:
                        best = (value, (x, y))
            return best
        else:
            best = (+2, None)
            for x, y in self.spaces:
                if self.spaces[x, y] == self.empty:
                    value = self.move(x, y).__minmax(not player)[0]
                    if value < best[0]:
                        best = (value, (x, y))
            return best

    def best_move(self):
        return self.__minmax(True)[1]

    def best_move_1(self, player):
        if self.won():
            if player:
                return (-1, None)
            else:
                return (+1, None)
        elif self.tied():
            return (0, None)
        elif player:
            for x, y in self.spaces:
                if self.spaces[x, y] == self.empty:
                    return (2, (x, y))

    def tied(self):
        for (x, y) in self.spaces:
            if self.spaces[x, y] == self.empty:
                return False
        return True

    def won(self):
        for y in range(self.size):
            winning = []
            for x in range(self.size):
                if self.spaces[x, y] == self.opponent:
                    winning.append((x, y))
            if len(winning) == self.size:
                return winning
        for x in range(self.size):
            winning = []
            for y in range(self.size):
                if self.spaces[x, y] == self.opponent:
                    winning.append((x, y))
            if len(winning) == self.size:
                return winning
        winning = []
        for y in range(self.size):
            x = y
            if self.spaces[x, y] == self.opponent:
                winning.append((x, y))
        if len(winning) == self.size:
            return winning
        winning = []
        for y in range(self.size):
            x = self.size - 1 - y
            if self.spaces[x, y] == self.opponent:
                winning.append((x, y))
        if len(winning) == self.size:
            return winning
        return None

    def __str__(self):
        string = ''
        for y in range(self.size):
            for x in range(self.size):
                string += self.spaces[x, y]
            string += "\n"
        return string


class StartGame:

    def __init__(self):
        self.window = Tk()
        self.window.title(' Super TicTacToe')
        self.window.geometry("400x400")
        self.window.resizable(width=False, height=False)
        self.button1 = Button(self.window, text='Beginner', command=self.beginner)
        self.button2 = Button(self.window, text='Advanced', command=self.advance)
        self.font = Font(family="Arial", size=32)
        self.label1 = Label(self.window, text="Welcome!!!", font=self.font)
        self.label1.place(x=50, y=20, height=50, width=300)
        self.button1.place(x=150, y=120, height=50, width=100)
        self.button2.place(x=150, y=190, height=50, width=100)

    def run_mainloop(self):
        self.window.mainloop()

    def advance(self):
        messagebox.showinfo("Game Mode", "You think to be pro . Let see")
        GameInterface().run_mainloop()

    def beginner(self):
        messagebox.showinfo("Game Mode", "OK Beginner")
        BeginnerMode().run_mainloop()


class BeginnerMode:

    def __init__(self):
        self.window = Tk()
        self.window.title('TicTacToe')
        self.window.resizable(width=False, height=False)
        self.font = Font(family="Arial", size=32)
        self.buttons = {}
        self.board = GameBoard()
        for x, y in self.board.spaces:
            handler = lambda x=x, y=y: self.move(x, y)
            button = Button(self.window, command=handler, font=self.font, width=12, height=6)
            button.grid(row=y, column=x)
            self.buttons[x, y] = button
        handler = lambda: self.reset()
        button = Button(self.window, text='reset', command=handler)
        button.grid(row=self.board.size + 1, column=0, columnspan=self.board.size, sticky="WE")
        self.update()

    def move(self, x, y):
        self.window.config(cursor="watch")
        self.window.update()
        self.board = self.board.move(x, y)
        self.update()
        move = self.board.best_move_1(True)[1]
        if move:
            self.board = self.board.move(*move)
            self.update()
        self.window.config(cursor="")

    def update(self):
        for (x, y) in self.board.spaces:
            text = self.board.spaces[x, y]
            self.buttons[x, y]['text'] = text
            self.buttons[x, y]['disabledforeground'] = 'black'
            if text == self.board.empty:
                self.buttons[x, y]['state'] = 'normal'
            else:
                self.buttons[x, y]['state'] = 'disabled'
        winning = self.board.won()
        if winning:
            for x, y in winning:
                self.buttons[x, y]['disabledforeground'] = 'red'
            if self.buttons[winning[0]]['text'] == 'O':
                messagebox.showinfo("Win status", "Beginner cannot even win at beginner, Try again")
            else:
                messagebox.showinfo("Win status", "Still thinking to be Beginner, Be a pro")
            for x, y in self.buttons:
                self.buttons[x, y]['state'] = 'disabled'
        for (x, y) in self.board.spaces:
            self.buttons[x, y].update()

    def run_mainloop(self):
        self.window.mainloop()

    def reset(self):
        self.board = GameBoard()
        self.update()


class GameInterface:

    def __init__(self):
        self.window = Tk()
        self.window.title('TicTacToe')
        self.window.resizable(width=False, height=False)
        self.board = GameBoard()
        self.font = Font(family="Arial", size=32)
        self.buttons = {}
        for x, y in self.board.spaces:
            handler = lambda x=x, y=y: self.move(x, y)
            button = Button(self.window, command=handler, font=self.font, width=12, height=6)
            button.grid(row=y, column=x)
            self.buttons[x, y] = button
        handler = lambda: self.reset()
        button = Button(self.window, text='reset', command=handler)
        button.grid(row=self.board.size + 1, column=0, columnspan=self.board.size, sticky="WE")
        self.update()

    def reset(self):
        self.board = GameBoard()
        self.update()

    def move(self, x, y):
        self.window.config(cursor="watch")
        self.window.update()
        self.board = self.board.move(x, y)
        self.update()
        move = self.board.best_move()
        if move:
            self.board = self.board.move(*move)
            self.update()
        self.window.config(cursor="")

    def update(self):
        for (x, y) in self.board.spaces:
            text = self.board.spaces[x, y]
            self.buttons[x, y]['text'] = text
            self.buttons[x, y]['disabledforeground'] = 'black'
            if text == self.board.empty:
                self.buttons[x, y]['state'] = 'normal'
            else:
                self.buttons[x, y]['state'] = 'disabled'
        winning = self.board.won()
        if winning:
            for x, y in winning:
                self.buttons[x, y]['disabledforeground'] = 'red'
                messagebox.showinfo("Win status", "Won")
        if winning and self.buttons[winning[0]]['text'] == 'O':
            messagebox.showinfo("Win status", "Still thinking to be pro, you are Beginner!!!")
        else:
            if winning:
                messagebox.showinfo("Win status", "You proved yourself")
        if winning:
            for x, y in self.buttons:
                self.buttons[x, y]['state'] = 'disabled'
        for (x, y) in self.board.spaces:
            self.buttons[x, y].update()

    def run_mainloop(self):
        self.window.mainloop()


if __name__ == "__main__":
    messagebox.showinfo("Start", "Members: Mujtaba Junaid (22k-8709), Arman Faisal (22k-8708), Rayyan Farhan (22k-4102) (MAR) ")
    #Members: Mujtaba Junaid (22k-8709), Arman Faisal (22k-8708), Rayyan Farhan (22k-4102) (MAR)
    #during pro matches when the function loads donot press in between as it might be keyboard interupt and give an error . Wait for ai to complete
    #its moves
    StartGame().run_mainloop()
