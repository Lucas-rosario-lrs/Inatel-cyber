# Write-up: RED (picoCTF)

Este documento detalha o processo de resolução do desafio de forense "RED" da plataforma picoCTF. A solução envolve a análise de dicas, metadados e a extração de dados ocultos em uma imagem usando esteganografia LSB.

| Propriedade | Valor |
| :--- | :--- |
| **Nome** | RED |
| **Categoria** | Forense |
| **Plataforma** | picoCTF |

## 📝 Descrição do Desafio

É fornecido um arquivo de imagem chamado `red.png`, que é, como o nome sugere, uma imagem completamente vermelha. O objetivo é encontrar a flag escondida dentro do arquivo.

As dicas fornecidas para o desafio são:

`The picture seems pure, but is it though?`

`Red?Ged?Bed?Aed?`

`Check whatever Facebook is called now.`

## 🕵️‍♂️ Análise e Exploração

O processo para resolver este desafio começa com a interpretação das dicas para guiar nossa investigação.

### 1. Análise das Dicas

* As dicas nos sugerem que devemos nos atentar aos metadados da imagem, que a primeira vista se parecem apenas uma imagem vermelha, sem grandes significados;

* Os canais de cores padrão são Vermelho (Red), Verde (Green) e Azul (Blue), ou **RGB**. A dica siginifica para usar "todos" os canais implica a inclusão do canal **Alfa (Alpha)**, que controla a transparência. Isso nos leva ao formato **RGBA**. Essa é uma pista técnica crucial, apontando para um método de esteganografia que utiliza todos os quatro canais de cores. O método mais comum para isso é o **LSB (Least Significant Bit)**.

### 2. Escolha da Ferramenta: CyberChef

Para extrair e manipular os dados dos canais de cores, o **CyberChef** é a ferramenta escolhida. Ele funciona no navegador e permite resolver o desafio de maneira rápida e fácil.

## ⚙️ Execução com CyberChef

A solução foi implementada no CyberChef seguindo dois passos principais: extrair os dados LSB e decodificar o resultado.

1.  **Carregar a Imagem**: Primeiro, o arquivo `red.png` é carregado no campo de "Input" do CyberChef.

2.  **Operação 1: Extrair LSB (Extract LSB)**
    * Na lista de operações, procuramos e adicionamos **"Extract LSB"**.
    * Com base na dica do RGBA, configuramos a operação para extrair o bit menos significativo de cada canal: **Red**, **Green**, **Blue** e **Alpha**.
    * A ordem de extração é importante, então mantemos o padrão RGBA.
    * Imediatamente, a saída (Output) mostra uma longa string de texto que se parece muito com uma codificação **Base64**.

3.  **Operação 2: Decodificar de Base64 (From Base64)**
    * Adicionamos a operação **"From Base64"** à receita, logo após a extração LSB.
    * O CyberChef automaticamente pega a saída da primeira operação e a usa como entrada para a segunda.
    * O resultado final aparece instantaneamente no campo de "Output": a flag legível.

## 🚩 Flag Final

Após a execução dos passos no CyberChef, a flag revelada é:

```
picoCTF{r3d_1s_th3_ult1m4t3_cur3_f0r_54dn355_}
```
