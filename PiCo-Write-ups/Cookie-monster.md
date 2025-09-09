# Write-up: picoCTF - Cookie Monster

Este documento detalha o processo de resolução do desafio de segurança
web "Cookie Monster" da plataforma picoCTF.

## 📝 Descrição do Desafio

O desafio apresenta uma página web simples que parece ter uma
funcionalidade baseada em cookies. O objetivo é inspecionar e manipular
os cookies do site para encontrar a flag escondida.

## 🕵️‍♂️ Análise e Exploração

O processo de resolução foi inteiramente realizado através do navegador
web, utilizando as ferramentas de desenvolvedor.

### Análise Inicial do Site:

-   Ao logar ao endereço
    `http://verbal-sleep.picoctf.net:58663/login.php`, a página exibe
    uma dica;
-   Esta mensagem sugere que a identidade do utilizador é controlada por
    algum mecanismo, provavelmente cookies.

### Inspeção de Cookies:

-   Utilizando as ferramentas de desenvolvedor do navegador (atalho
    `F12`), naveguei até à aba **Application** (ou **Storage** em alguns
    navegadores) e selecionei a secção **Cookies**.
-   Foi identificado um cookie com o nome `name` e o valor:

```{=html}
    cGljb0NURntjMDBrMWVfbTBuc3Rlcl9sMHZlc19jMDBraWVzX0IzQUQ5NEMyfQ
```
### Descoberta da Flag:

-   Continuei, copiando o valor do cookie e colocando num conversor de
    texto em base64. Foi utilizado o site:
    <https://www.base64decode.org/>

## 🚩 Flag

A flag obtida ao resolver o desafio é:

    picoCTF{c00k1e_m0nster_l0ves_c00kies_B3AD94C2}
