# Projeto_Eletronica_Piano_Aproximacao

Projeto da disciplina de eletrônica para computação do grupo 11.

# Piano de aproximação:

O nosso projeto consiste de um piano por aproximação, o piano funciona da seguinte forma:
Um sensor de distância é posicionado e cada tom e semitom de uma oitava (dó, dó sustenido, ré, ré sustenido, ..., si)
são definidos pelo usuário de acordo com uma calibragem (a distância entre um tom não precisa ser a mesma, o usuário
pode definir de forma arbitrária onde cada nota se localiza). Após a calibragem o usuário começa a tocar uma música,
posicionando um anteparo (ou seu dedo) a uma determinada distânica do sensor, a medida da distância é então enviada para 
o arduíno e para o computador, que reproduz a nota mais próxima definida na calibragem 
(eg. distância atual = 4.7 cm, ré = 4.0 cm, ré sustenido = 5.0 cm, o som reproduzido pelo pc será ré sustenido).


# Código fonte em Python:
<details>
  <summary>Clique aqui para abrir o código fonte.</summary>
  
  ```
  """
Este script faz parte do Trabalho 2 da disciplina de Eletrônica para Computação (SSC0180), apresentado ao professor Eduardo do Valle Simoes.

O objetivo é construir um instrumento musical simples, que toca uma determinada frequência baseada na distância captada pelo sensor de distâncias presente no Arduino.

Todo o projeto foi desenvolvido e testado em ambiente Linux, especificamente em um computador com Linux Mint 20.3. Espera-se, entretanto que, para qualquer derivado do Ubuntu, apenas os comandos abaixos serão suficientes para instalar tudo que é necessário, mas não isso não foi testado.

Para instalar as dependências, execute os seguintes comandos:
    $ sudo apt install portaudio19-dev
    $ pip install pyserial pysine

Para rodar o programa, execute:
    $ python main.py

O restante deve ser autoexplicativo.
"""

import statistics
import json
import math
from numbers import Number
import time
from typing import List
import serial
import pysine

CALLIBRATION_FILENAME = "callibration.json"
MUSICAL_NOTES_NAMES = "C C# D D# E F F# G G# A A# B".split()

ser = serial.Serial("/dev/ttyACM0", 9600)


def get_distance_record():
    """
    Retorna a distância atualmente lida pelo sensor ultrassônico conectado ao Arduino.
    """

    ser.write(b"a")
    line_content = ser.readline()
    return float(line_content)


def get_index_with_closest_value(value: Number, elements: List[Number]):
    """
    Retorna o índice do elemento de [elements] cujo valor mais se aproxima ao de [value].
    """

    min_i = 0
    for i in range(1, len(elements)):
        if abs(elements[i]-value) < abs(elements[min_i]-value):
            min_i = i
    return min_i


def get_note_frequency_by_index(note_index: int):
    """
    Retorna a frequência de uma nota musical, baseada em seu índice ([note_index]).

    O índice 0 corresponde ao C4 do piano. Para cada índice acima, sobe-se um semitom.
    """

    return 2 * 261.63 * math.pow(2, note_index / 12)


def get_max_distance_with_margin(distances: List[Number]):
    """
    Retorna uma distância maior que máximo dentre os valores de [distances], com uma margem de erro.

    A marge é dada pela média entre as diferenças entre valores consecutivos em [distances].
    """

    deltas = [distances[i] - distances[i-1] for i in range(1, len(distances))]
    avg_delta = statistics.mean(deltas)
    return max(distances) + 2 * avg_delta


def play():
    """
    Toca notas musicais baseada na distância captada pelo Arduino.

    Baseia os valores das distâncias no arquivo JSON cujo nome está na constante [CALLIBRATION_FILENAME].
    """

    with open(CALLIBRATION_FILENAME, "r", encoding="utf-8") as file:
        notes_distances: List[Number] = json.load(file)

    max_distance = get_max_distance_with_margin(notes_distances)

    while True:
        current_distance = get_distance_record()
        if 1 < current_distance < max_distance:
            note_index = get_index_with_closest_value(
                current_distance, notes_distances)
            print(f"Nota atual é: {MUSICAL_NOTES_NAMES[note_index]}.")
            frequency = get_note_frequency_by_index(note_index)
            pysine.sine(frequency, .5)
        else:
            time.sleep(0.1)


def get_callibration_distance(current_note_index: int):
    """
    Pede que o usuário coloque um anteparo em frente ao sensor do Arduino para que seja coletada a distância.

    Retorna quando o usuário está satisfeito com o valor lido.
    """

    while True:
        print(f"\nColoque a palheta na distância esperada da nota \
            {MUSICAL_NOTES_NAMES[current_note_index]}.")
        input("Pressione ENTER para medir.")

        current_distance = get_distance_record()
        print(f"\nA distância calculada foi de {current_distance} cm.")

        user_option = input("Insira 1 para repetir ou 0 para continuar: ")
        if user_option == "0":
            return current_distance


def callibrate():
    """
    Pede que o usuário coloque o anteparo na posição esperada de cada uma das notas que ele deseja tocar, para que as distâncias sejam calibradas.

    Salva os valores das distâncias no arquivo JSON cujo nome está na constante [CALLIBRATION_FILENAME].
    """

    notes_distances: List[Number] = []
    while len(notes_distances) < len(MUSICAL_NOTES_NAMES):
        current_distance = get_callibration_distance(len(notes_distances))
        notes_distances.append(current_distance)
    with open(CALLIBRATION_FILENAME, "w", encoding="utf-8") as file:
        json.dump(notes_distances, file)


def main():
    """
    Função principal. Executa o programa.
    """

    print("Escolha uma opção:")
    print("[1] Calibrar")
    print("[2] Tocar")

    option = int(input())
    if (option == 1):
        callibrate()
    elif (option == 2):
        play()


if __name__ == "__main__":
    main()
  ```
</details>

# Código fonte do Arduíno (C++):

<details>
  <summary>Clique aqui para abrir o código fonte.</summary>
  ```
  int main(int argc, char **argv){
    
   return 0;
 }
 ```
</details>

# Imagem do projeto:

<img src="https://github.com/teosalves/Projeto_Eletronica_Piano_Aproximacao/blob/main/proj.jpg" width="400" height="400" />

# Vídeo demonstrativo:
https://drive.google.com/file/d/1m_6Y10k2mT53I9KyH7i8DNb5Wa1bqdPv/view (Vídeo com +10 MB).

# Schematic do projeto:
