# Write-up: picoCTF - Flag Hunters (Python)

Este documento detalha o processo de resolução do desafio de segurança "Flag Hunters", que envolve a exploração de uma vulnerabilidade de injeção de comando num script Python.

## 📝 Descrição do Desafio

É fornecido um script Python que simula um leitor de letras de música. O script lê a flag de um ficheiro `flag.txt` e insere-a no início da letra, mas começa a "cantar" a partir de uma secção posterior, escondendo a flag. O objetivo é manipular o fluxo de execução do script para que ele imprima o início da letra e revele a flag.

| Propriedade | Valor |
| :--- | :--- |
| **Nome** | Flag Hunters |
| **Categoria** | General Skills / Scripting |
| **Plataforma** | picoCTF |

## 🕵️‍♂️ Análise e Exploração

A solução envolve uma análise cuidadosa do código-fonte para encontrar uma forma de controlar o ponteiro de leitura da música (`lip`).

**Análise do Código-Fonte:**

* O script lê a `flag.txt` e armazena-a na variável `secret_intro`.
* A função `reader` inicia a leitura a partir da label `[VERSE1]`, ignorando a secção inicial que contém a flag.
* Existe uma secção interativa `CROWD (Singalong here!);` que pede um input ao utilizador.
* O input fornecido pelo utilizador é usado para sobrescrever a linha atual na lista `song_lines`. Esta é a vulnerabilidade chave.

```python
# Secção vulnerável do código
elif re.match(r"CROWD.*", line):
    crowd = input('Crowd: ')
    # O input do utilizador substitui a linha na letra da música
    song_lines[lip] = 'Crowd: ' + crowd 
    lip += 1
```

**Identificação do Vetor de Exploração:**

O script possui um comando interno `RETURN [número]` que força o ponteiro de leitura (`lip`) a saltar para a linha especificada.
Veja o comando interno:
```python
elif re.match(r"RETURN [0-9]+", line):
        lip = int(line.split()[1])
      elif line == 'END':
        finished = True
```
Portanto, pode-se escolher o verso que desejamos commeçar através de linha de comando, observando:
```python
for i in range(0, len(song_lines)):
    if song_lines[i] == startLabel:
      start = i + 1
    elif song_lines[i] == '[REFRAIN]':
      refrain = i + 1
    elif song_lines[i] == 'RETURN':
      refrain_return = i
```

A nossa estratégia será injetar o comando `RETURN 0` através do input do `CROWD` para forçar o script a saltar para o início da letra da música (linha 0).

**Execução do Exploit:**

1.  Primeiro, executamos o script no terminal:
    ```bash
    python lyric-reader.py
    ```
2.  O script começa a imprimir a letra. Aguardamos até que ele chegue ao refrão e peça o nosso input:
    ```
    Crowd: 
    ```
3.  Neste momento, inserimos o nosso payload. O script processa múltiplos comandos numa linha se estiverem separados por ponto e vírgula (`;`). O nosso payload será:
    ```
    texto_aaleatorio;RETURN 0
    ```
4.  Ao premir Enter, o script armazena `Crowd: texto_aleatorio;RETURN 0` na memória.

**Captura da Flag:**

* O script continua a sua execução normal. Quando a função `reader` volta a passar por essa linha modificada, ela irá processar o nosso comando injetado.
* O comando `RETURN 0` é executado, e o ponteiro de leitura (`lip`) salta para a linha 0.
* O script começa então a imprimir a secção `secret_intro`, que contém a flag.

## 🚩 Flag

A flag é revelada no início da saída do script após a injeção do payload, através do arquivo lido `flag.txt`.

**Flag:** `picoCTF{70637h3r_f0r3v3r_a5202532}`
