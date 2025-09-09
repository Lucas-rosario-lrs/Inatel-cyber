# Write-up: interencdec

Este documento detalha o processo de resolução do desafio CTF **interencdec**, que envolveu múltiplas camadas de codificação, exigindo a identificação e reversão de cada etapa para obter a flag final.

## 📝 Descrição do Desafio

O desafio consiste em um único arquivo, chamado `enc_flag`, com a seguinte pergunta: "Can you get the real meaning from this file?" (Você consegue extrair o significado real deste arquivo?).

## 🕵️‍♂️ Análise e Resolução

O processo de resolução foi dividido em etapas sequenciais para decodificar cada camada de criptografia.

### 1. Análise Inicial e Primeira Camada (Base64)

Ao abrir o arquivo `enc_flag` utilizando um editor de código como o **Visual Studio Code**, o conteúdo é revelado:

```
YidkM0JxZGtwQlRYdHFhR3g2YUhsZmF6TnFlVGwzWVROclgyZzBOMm8yYXpZNWZRPT0nCg==
```

A presença de caracteres maiúsculos, minúsculos, números e o preenchimento `==` no final são um forte indicativo de codificação **Base64**.

### 2. Segunda Camada (Base64 aninhado)

Decodificando a string inicial, obtemos uma nova "string de bytes" do Python:

```python
b'dM0JqdkpBTXhqaGx6aHlfazNqelT3YTNRclX2g0N2o2azY5fQ=='
```

Isso revela que o conteúdo era, na verdade, outra string codificada em Base64. Extraindo e decodificando a string interna (`dM0Jqdkp...`), chegamos ao seguinte resultado:

```
s0IqdjITXhqhlzh_k3jzS7a3QCrW2g4N7j6kY9}
```

Esta é a string cifrada, a um passo da flag final.

### 3. Decifragem Final (Cifra de César)

A string final estava protegida por uma **Cifra de César**. Este tipo de cifra funciona deslocando cada caractere por um número fixo de posições no alfabeto (a "chave").

Aplicando a rotação correta para cada caractere, a mensagem é finalmente decifrada, revelando a flag.

## 🚩 Flag Final

Após reverter todas as camadas de codificação e decifrar a Cifra de César, a flag obtida foi:

```
picoCTF{caesar_d3cr9pt3d_a47c6d69}
```

## 💡 Conceitos Utilizados

* **Base64:** Um esquema de codificação para representar dados binários em formato de texto ASCII, facilmente identificável pela sua gama de caracteres e pelo uso do caractere `=` como preenchimento.
* **Cifra de César:** Uma das cifras de substituição mais simples, onde cada letra do texto é trocada por outra que se encontra um número fixo de posições à frente no alfabeto.
