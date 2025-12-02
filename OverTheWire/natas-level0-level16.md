# Natas Level 0

### Objetivo

Acessar a senha para o nível 1. O nível começa com uma página simples contendo apenas texto.

### Solução

Ao acessar o site, é solicitado o login inicial (natas0/natas0).
![Login Inicial](images/natas0-login.png)

A página informa que a senha está escondida nela mesma, mas não há nada visível na interface do usuário. A primeira ação em qualquer teste de segurança web é verificar como a página foi construída.

Para isso, inspecionei o código-fonte da página (botão direito -\> Ver código-fonte ou `Ctrl+U`).

![Flag no Comentário](images/natas0-flag.png)
A senha foi encontrada dentro de um comentário HTML.

### Vulnerabilidade

A falha aqui é a exposição de informações sensíveis no lado do cliente (Client-Side). Comentários no código HTML (\`\`) não são renderizados pelo navegador visualmente, mas são baixados para a máquina do usuário. Desenvolvedores frequentemente usam comentários para notas internas e esquecem de removê-los em produção.

### Pesquisa

Pesquisei sobre **"HTML Comments sensitive data"** e como inspecionar elementos no navegador (DevTools).

-----

# Natas Level 1

### Objetivo

Acessar a senha para o nível 2. A página é idêntica à anterior, mas com uma restrição de interface.

![Mensagem na Página](images/natas1-problema1.png)
![Pop-up Bloqueio JS](images/natas1-problema2.png)
### Solução

Ao tentar clicar com o botão direito para ver o código-fonte, um alerta JavaScript bloqueia a ação.

No entanto, o bloqueio do botão direito é apenas um evento de interface (UI). O navegador ainda possui o código HTML carregado. Para contornar isso, utilizei o atalho de teclado `Ctrl + U` (ou digitei `view-source:` antes da URL).

Ao acessar o código-fonte, a senha estava novamente em um comentário.

![Flag no View-Source](images/natas1-flag.png)
### Vulnerabilidade

A "segurança" implementada aqui depende exclusivamente de restrições no navegador (JavaScript). Como o código roda no computador do cliente, o usuário tem controle total sobre ele. Bloquear o botão direito ou desabilitar teclas não impede que o usuário veja o código que já foi enviado pelo servidor.

### Pesquisa

Pesquisei sobre **"Bypass right click disable JavaScript"** e **"View source shortcut chrome"**.

-----

# Natas Level 2

### Objetivo

Acessar a senha para o nível 3. A página informa que não há nada nela.

![Página Inicial](images/natas2-problema.png)
### Solução

Acessando a página, não há links visíveis ou texto contendo a senha.

Ao inspecionar o código-fonte, notei que havia uma tag de imagem `<img src="files/pixel.png">`. Mesmo que a imagem seja um pixel quase invisível, ela indica a existência de um diretório chamado `/files/`.

![Tag de Imagem no HTML](images/natas2-dica1.png)

A URL da imagem confirma o caminho relativo:

![Segunda dica](images/natas2-dica2.png)

Ao tentar acessar o diretório raiz onde a imagem está hospedada (`/files/`), o servidor retornou uma lista de todos os arquivos presentes naquela pasta.

![Directory Listing](images/natas2-dica3.png)

Encontrei um arquivo chamado `users.txt`. Ao abri-lo, a lista de usuários e senhas foi revelada.

![Conteúdo users.txt](images/natas2-flag.png)
### Raciocínio

A vulnerabilidade explorada é conhecida como **Directory Listing** (Listagem de Diretório). Quando um servidor web não encontra um arquivo de índice padrão (como `index.html` ou `index.php`) em uma pasta e a listagem de diretórios não está desativada na configuração, ele mostra todos os arquivos contidos ali. Isso permite que atacantes descubram arquivos que não deveriam ser públicos.

### Pesquisa

Pesquisei sobre **"Apache Directory Listing vulnerability"** e **"Security by obscurity hidden files"**.

-----

# Natas Level 3

### Objetivo

Acessar a senha para o nível 4. A página inicial apresenta apenas a mensagem "There is nothing on this page" (Não há nada nesta página).

### Solução

Ao acessar o site, o conteúdo visível é inexistente.

![Página Vazia](images/natas3-problema.png)

Seguindo a metodologia padrão, inspecionei o código-fonte da página. Encontrei um comentário intrigante: "No more information leaks\!\! Not even Google will find it this time..." (Sem mais vazamentos de informação\!\! Nem mesmo o Google vai encontrar desta vez...).

![Dica no HTML](images/natas3-dica.png)

A menção ao Google refere-se a mecanismos de busca (crawlers). A forma padrão de instruir robôs de busca a não indexarem ou não acessarem certas partes de um site é através do arquivo `robots.txt`.

![Robots.txt](images/natas3-dica2.png)

Acessei `http://natas3.natas.labs.overthewire.org/robots.txt` e encontrei uma regra de "Disallow" (Não permitir) para o diretório `/s3cr3t/`.

Ao navegar para esse diretório proibido (`/s3cr3t/`), o servidor listou os arquivos contidos nele, revelando o arquivo `users.txt`.

![Pasta Secreta](images/natas3-dica3.png)

Abrindo o arquivo, obtive a credencial para o próximo nível.

![Flag no users.txt](images/natas3-flag.png)
### Vulnerabilidade

A vulnerabilidade aqui é a **Security by Obscurity** (Segurança por Obscuridade) e o vazamento de informações via **robots.txt**. O arquivo `robots.txt` é público. Atacantes frequentemente verificam esse arquivo para descobrir quais diretórios os administradores consideram sensíveis ou privados.

### Pesquisa

Pesquisei sobre **"robots.txt security risk"** e **"web crawler directives"**.

-----

# Natas Level 4

### Objetivo

Acessar a senha para o nível 5. O acesso é negado com base na origem da visita.

![Erro de Referer](images/natas4-dica1.png)
### Solução

Ao acessar a página, recebi uma mensagem de erro informando que o acesso é proibido porque estou visitando a partir do endereço atual (`natas4...`), enquanto usuários autorizados devem vir exclusivamente de `http://natas5.natas.labs.overthewire.org/`.

Isso indica uma verificação baseada no cabeçalho HTTP `Referer`. Para manipular essa requisição, utilizei a ferramenta **Burp Suite**. Configurei o proxy para interceptar o tráfego.

![Configuração Burp](images/natas4-dica2.png)

Enviei a requisição para o módulo "Repeater" do Burp Suite e alterei manualmente o cabeçalho `Referer` para o valor exigido pelo desafio: `http://natas5.natas.labs.overthewire.org/`.

Ao reenviar a requisição modificada, o servidor validou a origem falsa e retornou a página contendo a senha.

![Flag no Repeater](images/natas4-flag.png)

### Raciocínio (Vulnerabilidade)

A vulnerabilidade é a **Insecure Referer Checking** (Verificação Insegura de Referer). O cabeçalho `Referer` é enviado pelo navegador do cliente e pode ser facilmente modificado ou falsificado. Servidores nunca devem confiar nesse cabeçalho para autenticação ou controle de acesso.

### Pesquisa

Pesquisei sobre **"HTTP Referer header spoofing"**, **"Burp Suite Repeater tutorial"** e **"Modifying HTTP headers"**.

-----

# Natas Level 5

### Objetivo

Acessar a senha para o nível 6. O site informa que o usuário não está logado.

![Cookie Original](images/natas5-dica1.png)
### Solução

A página exibe a mensagem "Access disallowed. You are not logged in" (Acesso negado. Você não está logado).

Abri as ferramentas de desenvolvedor (F12) e naveguei até a aba **Application** (ou Storage) para inspecionar os cookies armazenados pelo site.

![Cookie Alterado](images/natas5-flag.png)

Encontrei um cookie chamado `loggedin` com o valor `0`. Editei o valor do cookie `loggedin` de `0` para `1` diretamente no navegador e recarreguei a página. O servidor leu o novo valor do cookie, assumiu que eu estava autenticado e exibiu a senha.

### Raciocínio

A falha é a **Insecure Client-Side Storage** e **Broken Authentication** (Autenticação Quebrada). O estado de autenticação foi confiado inteiramente a um dado armazenado no cliente (o cookie), sem validação criptográfica ou verificação no lado do servidor.

### Pesquisa

Pesquisei sobre **"Cookie manipulation attacks"**, **"Client-side authentication vulnerabilities"** e **"Editing cookies in DevTools"**.

-----

# Natas Level 6

### Objetivo

Acessar a senha para o nível 7. O site apresenta um formulário solicitando um segredo ("Input secret").

![Erro de Segredo](images/natas6-dica1.png)

### Solução

Ao tentar submeter qualquer valor aleatório no formulário, o sistema retorna "Wrong secret".

Cliquei no link "View sourcecode" para analisar a lógica de validação no back-end.

![Include no Código](images/natas6-dica2.png)

O código PHP revela que o segredo esperado é comparado com uma variável `$secret`, definida em um arquivo externo incluído: `include "includes/secret.inc";`. Tentei acessar o arquivo diretamente alterando a URL para `/includes/secret.inc`. Visualizei o código-fonte dessa página (`Ctrl+U`) para ler a variável.

![Conteúdo secret.inc](images/natas6-flag.png)

Copiei o segredo revelado (`FOEIUWGHFEEUHOFUOIU`), voltei à página inicial do nível e o submeti no formulário.

![Flag Revelada](images/natas6-flag2.png)
### Raciocínio

A falha aqui é a **Information Disclosure** (Divulgação de Informação) causada por má configuração de permissões de arquivo. Arquivos de configuração ou inclusão (como `.inc`) não devem estar acessíveis diretamente via URL.

### Pesquisa

Pesquisei sobre **"Preventing direct access to include files"** e **"Sensitive data exposure in source code"**.

-----

# Natas Level 7

### Objetivo

Acessar a senha para o nível 8. O site contém links para "Home" e "About" que mudam a URL via parâmetro `page`.

![Dica do Caminho](images/natas7-dica1.png)
### Solução

Ao inspecionar o código-fonte, encontrei um comentário contendo o caminho absoluto da senha.

A dica informa: `password for webuser natas8 is in /etc/natas_webpass/natas8`.

![Dica do Caminho](images/natas7-dica2.png)

Testei a vulnerabilidade de Inclusão de Arquivo Local (LFI) inserindo o caminho absoluto fornecido na URL: `index.php?page=/etc/natas_webpass/natas8`.

![LFI Executado](images/natas7-flag.png)

O servidor processou o caminho e exibiu o conteúdo do arquivo de senha diretamente na página.

### Vulnerabilidade

A vulnerabilidade explorada é **LFI (Local File Inclusion)**. Ela ocorre quando uma aplicação aceita entrada do usuário e a utiliza em funções de sistema de arquivos sem validação, permitindo a leitura de arquivos sensíveis do servidor.

### Pesquisa

Pesquisei sobre **"PHP Local File Inclusion LFI payloads"** e **"LFI /etc/passwd path traversal"**.

-----

# Natas Level 8

### Objetivo

Acessar a senha para o nível 9. O site solicita novamente um "secret".

![Input do Segredo](images/natas8-dica1.png)
### Solução

A página inicial contém um formulário de entrada.

Ao analisar o código-fonte ("View sourcecode"), identifiquei a função `encodeSecret` que ofusca o segredo.

![Função de Encode](images/natas8-dica2.png)

A lógica de encriptação é: `bin2hex(strrev(base64_encode($secret)))`.
Criei um script PHP simples para reverter a ordem das operações (`hex2bin` -\> `strrev` -\> `base64_decode`) usando a string hexadecimal fornecida no código.

![Script PHP Decoder](images/natas8-dica3.png)

O script retornou o valor `oubWYf2kBq`. Submeti este valor no formulário do desafio e obtive acesso.

![Flag Revelada](images/natas8-flag.png)
### Vulnerabilidade

A falha é o uso de **Encoding como Criptografia**. Codificação (Base64, Hex) apenas muda o formato dos dados e é facilmente reversível, não oferecendo proteção real aos dados.

### Pesquisa

Pesquisei sobre **"Difference between encoding and encryption"** e **"PHP reverse hex2bin base64"**.

-----

# Natas Level 9

### Objetivo

Acessar a senha para o nível 10. O site oferece uma ferramenta de busca em um dicionário.

![Interface de Busca](images/natas9-dica.png)
### Solução

A interface apresenta um campo de entrada "Find words containing".

O código-fonte mostra que a entrada é passada para `passthru("grep -i $key dictionary.txt");`.

![Código Passthru](images/natas9dica-1.png)

Utilizei o caractere `;` para separar comandos no Linux e injetei o comando `cat`.
Payload: `; cat /etc/natas_webpass/natas10`

Ao submeter a busca, o servidor executou o comando injetado e retornou a senha.

![Command Injection](images/natas9-flag.png)

### Raciocínio (Vulnerabilidade)

A vulnerabilidade é **Command Injection** (Injeção de Comando). Ocorre quando o input do usuário é concatenado diretamente em comandos do sistema operacional sem sanitização.

### Pesquisa

Pesquisei sobre **"PHP passthru command injection"** e **"Linux shell command separators"**.

-----

# Natas Level 10

### Objetivo

Acessar a senha para o nível 11. O site filtra certos caracteres de comando.

### Solução

A interface avisa que agora filtra certos caracteres por segurança.

![Aviso de Filtro](images/natas10.dica1.png)

O código-fonte mostra um filtro regex `/[;|&]/` que bloqueia `;`, `|` e `&`.

Como não podemos começar um novo comando, manipulamos os argumentos do `grep`. Forçamos o grep a ler o arquivo de senha passando-o como argumento extra.
Payload: `.* /etc/natas_webpass/natas11`

![Regex no Código](images/natas10-dica2.png)

Isso fez com que o `grep` imprimisse todo o conteúdo do arquivo de senha.

![Argument Injection](images/natas10-flag.png)
### Raciocínio

A vulnerabilidade é **Argument Injection** (Injeção de Argumento). Mesmo bloqueando separadores de comando, o controle sobre os argumentos de um programa pode permitir comportamentos não intencionais, como leitura de arquivos arbitrários.

### Pesquisa

Pesquisei sobre **"Grep argument injection"** e **"Bypassing command injection filters"**.

---

# Natas Level 11

### Objetivo

Acessar a senha para o nível 12. O site permite alterar a cor de fundo da página e informa que "Cookies are protected with XOR encryption" (Cookies são protegidos com encriptação XOR).

### Solução

Ao definir uma cor de fundo, o site salva um cookie chamado `data` contendo um valor codificado. Analisando o código-fonte ("View sourcecode"), identifiquei que o sistema utiliza uma criptografia XOR simples com uma chave fixa para proteger os dados do cookie.

A função `loadData` descriptografa o cookie e verifica o conteúdo de um array JSON:
`$defaultdata = array( "showpassword"=>"no", "bgcolor"=>"#ffffff");`

Para obter a senha, é necessário alterar o valor `showpassword` de `"no"` para `"yes"`.

Como a operação XOR é reversível (`A XOR B = C` implica `A XOR C = B`), utilizei um ataque de **Known Plaintext** (Texto Plano Conhecido). Eu possuía:

1.  **O Texto Plano:** O JSON padrão que vi no código (`{"showpassword":"no"...}`).
2.  **O Texto Cifrado:** O cookie gerado pelo site para esse padrão.

Criei um script PHP para fazer o XOR entre esses dois valores e descobrir a chave secreta utilizada pelo servidor.

![Cálculo do XOR e geração do cookie](images/natas11-cookie.png)

O script revelou que a chave de criptografia era `eDWo`.
Com a chave em mãos, o próprio script gerou um novo cookie malicioso contendo o JSON com `"showpassword"=>"yes"`.

Injetei este novo cookie no navegador (substituindo o valor original na aba Application \> Cookies), recarreguei a página e a senha foi revelada.

![Flag capturada e cookie modificado no navegador](images/natas11-flag.png)

### Raciocínio (Vulnerabilidade)

A vulnerabilidade é o uso inseguro de criptografia (Weak Cryptography). O algoritmo XOR é rápido, mas não oferece confidencialidade se a chave for reutilizada e o atacante tiver acesso a uma amostra do texto original e do texto cifrado. Além disso, a chave estava "hardcoded" no servidor, mas pôde ser deduzida matematicamente sem acesso ao código-fonte, apenas pela análise dos dados de saída.

### Pesquisa

Pesquisei sobre **"XOR Cipher Known Plaintext Attack"** e **"PHP json\_encode vulnerability"**.
