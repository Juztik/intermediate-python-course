pip install numpy matplotlib scipy pygame

import numpy as np
import matplotlib.pyplot as plt
import matplotlib.animation as animation
import wave
import scipy.fftpack
import pygame

# === Настройки ===
AUDIO_FILE = "music.wav"
WINDOW_MS = 50  # окно анализа в миллисекундах

# === Загружаем WAV-файл ===
wav = wave.open(AUDIO_FILE, 'rb')
framerate = wav.getframerate()
nframes = wav.getnframes()
nchannels = wav.getnchannels()
sampwidth = wav.getsampwidth()
audio = wav.readframes(nframes)
wav.close()

# Конвертируем в numpy-массив
data = np.frombuffer(audio, dtype=np.int16)
if nchannels == 2:
    data = data[::2]  # берём только левый канал, если стерео

# === Воспроизведение музыки (через pygame) ===
pygame.mixer.init(frequency=framerate)
pygame.mixer.music.load(AUDIO_FILE)
pygame.mixer.music.play()

# === График сердца ===
t = np.linspace(0, 2 * np.pi, 1000)
base_x = 16 * np.sin(t)**3
base_y = 13 * np.cos(t) - 5 * np.cos(2*t) - 2 * np.cos(3*t) - np.cos(4*t)

# === Анимация ===
fig, ax = plt.subplots()
line, = ax.plot([], [], lw=2)
fill = None

ax.set_aspect('equal')
ax.axis('off')

window_size = int(framerate * WINDOW_MS / 1000)
frame_count = len(data) // window_size

# === Функция для выбора цвета по громкости ===
def get_color(amplitude):
    if amplitude < 0.1:
        return 'pink'
    elif amplitude < 0.3:
        return 'red'
    else:
        return 'darkred'

# === Анимационная функция ===
def animate(i):
    global fill
    segment = data[i * window_size:(i+1) * window_size]
    if len(segment) == 0:
        return line,

    amplitude = np.abs(segment).mean() / 2000  # нормализация
    scale = 1 + 0.7 * amplitude
    x = base_x * scale
    y = base_y * scale

    color = get_color(amplitude)

    line.set_data(x, y)
    line.set_color(color)

    if fill:
        fill.remove()
    fill = ax.fill(x, y, color=color, alpha=0.5)[0]

    return line, fill

ani = animation.FuncAnimation(fig, animate, frames=frame_count, interval=WINDOW_MS, blit=False)
plt.title("Сердце танцует под музыку 💓", fontsize=16)
plt.show()

# Останавливаем музыку, когда график закрыт
pygame.mixer.music.stop()