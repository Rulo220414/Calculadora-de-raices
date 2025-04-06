# Calculadora-de-raices
"Aplicación en Python para calcular raíces usando métodos numéricos".
import tkinter as tk
from tkinter import messagebox
import matplotlib.pyplot as plt
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg
from sympy import symbols, sympify, lambdify, diff
import numpy as np

x = symbols('x')

class RaicesApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Calculadora de Raíces")
        self.root.configure(bg="#222831")
        self.build_gui()

    def build_gui(self):
        # Estilo visual
        style = {"bg": "#393E46", "fg": "#EEEEEE", "font": ("Arial", 11), "relief": "solid", "bd": 1}

        # Título
        tk.Label(self.root, text="Métodos: Falsa Posición y Newton-Raphson", bg="#00ADB5", fg="white",
                 font=("Helvetica", 16, "bold"), pady=10).pack(fill="x")

        frame = tk.Frame(self.root, bg="#222831", pady=10)
        frame.pack()

        # Entradas
        tk.Label(frame, text="Función f(x):", **style).grid(row=0, column=0, sticky="e")
        self.entry_func = tk.Entry(frame, width=30)
        self.entry_func.grid(row=0, column=1)

        tk.Label(frame, text="Valor a (Falsa Posición):", **style).grid(row=1, column=0, sticky="e")
        self.entry_a = tk.Entry(frame)
        self.entry_a.grid(row=1, column=1)

        tk.Label(frame, text="Valor b (Falsa Posición):", **style).grid(row=2, column=0, sticky="e")
        self.entry_b = tk.Entry(frame)
        self.entry_b.grid(row=2, column=1)

        tk.Label(frame, text="x0 (Newton-Raphson):", **style).grid(row=3, column=0, sticky="e")
        self.entry_x0 = tk.Entry(frame)
        self.entry_x0.grid(row=3, column=1)

        tk.Label(frame, text="Tolerancia:", **style).grid(row=4, column=0, sticky="e")
        self.entry_tol = tk.Entry(frame)
        self.entry_tol.grid(row=4, column=1)

        # Botones
        tk.Button(frame, text="Falsa Posición", bg="#00ADB5", fg="white", command=self.run_falsa).grid(row=5, column=0, pady=10)
        tk.Button(frame, text="Newton-Raphson", bg="#00ADB5", fg="white", command=self.run_newton).grid(row=5, column=1)

        # Salida
        self.result_label = tk.Label(self.root, text="", bg="#222831", fg="#FFD369", font=("Consolas", 11))
        self.result_label.pack(pady=10)

    def validar_entrada(self):
        try:
            f = sympify(self.entry_func.get())
            fx = lambdify(x, f, modules=['numpy'])
            return f, fx
        except:
            messagebox.showerror("Error", "Función no válida.")
            return None, None

    def run_falsa(self):
        f, fx = self.validar_entrada()
        if f is None:
            return

        try:
            a = float(self.entry_a.get())
            b = float(self.entry_b.get())
            tol = float(self.entry_tol.get())
        except:
            messagebox.showerror("Error", "Datos numéricos inválidos.")
            return

        if fx(a) * fx(b) >= 0:
            messagebox.showwarning("Advertencia", "No hay cambio de signo entre a y b.")
            return

        iteraciones = 0
        errores = []
        aproximaciones = []
        while True:
            xr = b - (fx(b) * (a - b)) / (fx(a) - fx(b))
            iteraciones += 1
            errores.append(abs(fx(xr)))
            aproximaciones.append(xr)

            if fx(a) * fx(xr) < 0:
                b = xr
            else:
                a = xr

            if abs(fx(xr)) < tol or iteraciones > 100:
                break

        self.result_label.config(text=f"Raíz ≈ {xr:.6f} | Iteraciones: {iteraciones}")
        self.plot_graph(errores, aproximaciones, "Falsa Posición")

    def run_newton(self):
        f, fx = self.validar_entrada()
        if f is None:
            return

        try:
            x0 = float(self.entry_x0.get())
            tol = float(self.entry_tol.get())
        except:
            messagebox.showerror("Error", "Datos numéricos inválidos.")
            return

        dfx = diff(f, x)
        dfxn = lambdify(x, dfx, modules=['numpy'])

        iteraciones = 0
        errores = []
        aproximaciones = []

        while True:
            try:
                x1 = x0 - fx(x0)/dfxn(x0)
            except ZeroDivisionError:
                messagebox.showerror("Error", "Derivada igual a cero. Método falla.")
                return

            iteraciones += 1
            errores.append(abs(fx(x1)))
            aproximaciones.append(x1)

            if abs(x1 - x0) < tol or iteraciones > 100:
                break
            x0 = x1

        self.result_label.config(text=f"Raíz ≈ {x1:.6f} | Iteraciones: {iteraciones}")
        self.plot_graph(errores, aproximaciones, "Newton-Raphson")

    def plot_graph(self, errores, aproximaciones, metodo):
        fig, axs = plt.subplots(1, 2, figsize=(8, 3), tight_layout=True)
        axs[0].plot(errores, marker='o', color="#FF5722")
        axs[0].set_title("Error en cada iteración")
        axs[0].set_xlabel("Iteración")
        axs[0].set_ylabel("Error")

        axs[1].plot(aproximaciones, marker='o', color="#03A9F4")
        axs[1].set_title("Aproximaciones")
        axs[1].set_xlabel("Iteración")
        axs[1].set_ylabel("x")

        window = tk.Toplevel(self.root)
        window.title(f"Gráficas de {metodo}")
        canvas = FigureCanvasTkAgg(fig, master=window)
        canvas.draw()
        canvas.get_tk_widget().pack()

# Crear ventana
root = tk.Tk()
app = RaicesApp(root)
root.mainloop()
