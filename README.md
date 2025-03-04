# Convers-o-de-Imagem-para-Desenho
Projeto em Python utilizando a biblioteca OpenCV, com o objetivo de aprimoramento e entendimento da linguagem.
from tkinter import *
from tkinter import Tk, ttk, filedialog
from PIL import ImageTk, Image, ImageEnhance
import cv2
import os
import numpy as np

# Cores
co1 = "#feffff"  # Branco
co2 = "#4fa882"  # Verde
co3 = "#38576b"  # Azul escuro
co4 = "#403d3d"  # Cinza escuro
co6 = "#038cfc"  # Azul claro

# Criando a janela
janela = Tk()
janela.title("Conversor de Imagem")
janela.geometry('650x750')
janela.configure(background=co1)
janela.resizable(width=False, height=False)

# Variáveis Globais
imagem_original = None
imagem_convertida = None
caminho_imagem = ""

# Função para escolher a imagem
def escolher_imagem():
    global imagem_original, caminho_imagem
    caminho_imagem = filedialog.askopenfilename(filetypes=[("Imagens", "*.png;*.jpg;*.jpeg;*.bmp;*.tiff")])

    if caminho_imagem:
        caminho_imagem = os.path.normpath(caminho_imagem)  # Corrige o caminho para Windows
        print(f"Caminho da imagem selecionada: {caminho_imagem}")

        # Abrindo com PIL para evitar problemas de compatibilidade
        imagem_original = Image.open(caminho_imagem).convert("RGB")  
        imagem_preview = imagem_original.resize((200, 200))
        imagem_preview = ImageTk.PhotoImage(imagem_preview)
        l_preview_original.configure(image=imagem_preview)
        l_preview_original.image = imagem_preview

# Função para converter a imagem
def converter_imagem(event=None):
    global imagem_original, imagem_convertida, caminho_imagem  

    if not caminho_imagem or not os.path.exists(caminho_imagem):
        print(f"Erro: Arquivo não encontrado! Caminho: {caminho_imagem}")
        return

    try:
        # Carregando imagem com PIL e convertendo para OpenCV
        pil_img = Image.open(caminho_imagem).convert("RGB")  # Abre a imagem
        imagem_cv = np.array(pil_img)  # Converte para numpy array
        imagem_cv = cv2.cvtColor(imagem_cv, cv2.COLOR_RGB2GRAY)  # Converte para escala de cinza

        # Aplicando efeito lápis
        blur = cv2.GaussianBlur(imagem_cv, (21, 21), 0)
        sketch = cv2.divide(imagem_cv, blur, scale=s_intensidade.get())

        # Ajuste de brilho e contraste
        pil_sketch = Image.fromarray(sketch)
        enhancer_brilho = ImageEnhance.Brightness(pil_sketch)
        pil_sketch = enhancer_brilho.enhance(s_brilho.get() / 100)

        enhancer_contraste = ImageEnhance.Contrast(pil_sketch)
        pil_sketch = enhancer_contraste.enhance(s_contraste.get() / 100)

        # Atualizando a imagem convertida
        imagem_convertida = pil_sketch
        imagem_preview = imagem_convertida.resize((200, 200))
        imagem_preview = ImageTk.PhotoImage(imagem_preview)
        l_preview_convertida.configure(image=imagem_preview)
        l_preview_convertida.image = imagem_preview

    except Exception as e:
        print(f"Erro ao processar a imagem: {e}")

#Função salvar imagem

def salvar_imagem():
    if imagem_convertida:
        caminho_imagem=filedialog.asksaveasfilename(defaultextension=".png", filetypes=[("PNG files", "*.png"),("All files", "*.*")])
    if caminho_imagem:
        imagem_convertida.save(caminho_imagem)


# Criando a interface gráfica
frame_top = Frame(janela, width=450, height=50, background=co1)
frame_top.grid(row=0, column=0, padx=10, pady=5)

frame_preview = Frame(janela, width=450, height=220, background=co1)
frame_preview.grid(row=1, column=0, padx=10, pady=5)

frame_controls = Frame(janela, width=450, height=226, background=co1)
frame_controls.grid(row=2, column=0, padx=10, pady=5)

# Logo e Título
logo = Label(frame_top, text="Conversor de Imagem para Desenho a Lápis", font=("Arial", 16, "bold"), background=co1, fg=co4)
logo.pack()

# Previews
l_preview_original = Label(frame_preview, text="Foto Original", font=("Arial", 12), background=co1, fg=co3)
l_preview_original.place(x=30, y=10)

l_preview_convertida = Label(frame_preview, text="Foto Convertida", font=("Arial", 12), fg=co3)
l_preview_convertida.place(x=240, y=10)

# Controles
ttk.Label(frame_controls, text="Intensidade", background=co1).place(x=10, y=10)
s_intensidade = Scale(frame_controls, command=converter_imagem, from_=50, to=300, orient=HORIZONTAL, length=200, background=co1, fg=co4)
s_intensidade.set(120)
s_intensidade.place(x=10, y=30)

ttk.Label(frame_controls, text="Brilho", background=co1).place(x=10, y=80)
s_brilho = Scale(frame_controls, command=converter_imagem, from_=50, to=200, orient=HORIZONTAL, length=200, background=co1, fg=co4)
s_brilho.set(100)
s_brilho.place(x=10, y=100)

ttk.Label(frame_controls, text="Contraste", background=co1).place(x=10, y=150)
s_contraste = Scale(frame_controls, command=converter_imagem, from_=50, to=200, orient=HORIZONTAL, length=200, background=co1, fg=co4)
s_contraste.set(100)
s_contraste.place(x=10, y=170)

# Botões
b_escolher = Button(janela, command=escolher_imagem, text="Escolher Imagem", background=co6, fg=co1, font=("Times New Roman", 10), width=15)
b_escolher.place(x=20, y=500)

b_converter = Button(janela, text="Converter", command=converter_imagem, background=co2, fg=co1, font=("Times New Roman", 10), width=15)
b_converter.place(x=160, y=500)

b_salvar = Button(janela, command=salvar_imagem, text="Salvar Imagem", background=co3, fg=co1, font=("Times New Roman", 10), width=15)
b_salvar.place(x=300, y=500)

janela.mainloop()

