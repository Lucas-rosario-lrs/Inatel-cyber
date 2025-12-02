Natas Level 0
Objetivo
Acessar a senha para o nível 1. O nível começa com uma página simples contendo apenas texto.

Solução
Ao acessar o site, é solicitado o login inicial (natas0/natas0).

A página informa que a senha está escondida nela mesma, mas não há nada visível na interface do usuário. A primeira ação em qualquer teste de segurança web é verificar como a página foi construída.

Para isso, inspecionei o código-fonte da página (botão direito -> Ver código-fonte ou Ctrl+U).

A senha foi encontrada dentro de um comentário HTML.

Raciocínio (Vulnerabilidade)
A falha aqui é a exposição de informações sensíveis no lado do cliente (Client-Side). Comentários no código HTML (``) não são renderizados pelo navegador visualmente, mas são baixados para a máquina do usuário. Desenvolvedores frequentemente usam comentários para notas internas e esquecem de removê-los em produção.

Pesquisa
Pesquisei sobre "HTML Comments sensitive data" e como inspecionar elementos no navegador (DevTools).

Natas Level 1
Objetivo
Acessar a senha para o nível 2. A página é idêntica à anterior, mas com uma restrição de interface.

Solução
Ao tentar clicar com o botão direito para ver o código-fonte, um alerta JavaScript bloqueia a ação.

No entanto, o bloqueio do botão direito é apenas um evento de interface (UI). O navegador ainda possui o código HTML carregado. Para contornar isso, utilizei o atalho de teclado Ctrl + U (ou digitei view-source: antes da URL).

Ao acessar o código-fonte, a senha estava novamente em um comentário.

Raciocínio (Vulnerabilidade)
A "segurança" implementada aqui depende exclusivamente de restrições no navegador (JavaScript). Como o código roda no computador do cliente, o usuário tem controle total sobre ele. Bloquear o botão direito ou desabilitar teclas não impede que o usuário veja o código que já foi enviado pelo servidor.

Pesquisa
Pesquisei sobre "Bypass right click disable JavaScript" e "View source shortcut chrome".

Natas Level 2
Objetivo
Acessar a senha para o nível 3. A página informa que não há nada nela.

Solução
Acessando a página, não há links visíveis ou texto contendo a senha.

Ao inspecionar o código-fonte, notei que havia uma tag de imagem <img src="files/pixel.png">. Mesmo que a imagem seja um pixel quase invisível, ela indica a existência de um diretório chamado /files/.

A URL da imagem confirma o caminho relativo:

Ao tentar acessar o diretório raiz onde a imagem está hospedada (/files/), o servidor retornou uma lista de todos os arquivos presentes naquela pasta.

Encontrei um arquivo chamado users.txt. Ao abri-lo, a lista de usuários e senhas foi revelada.

Raciocínio (Vulnerabilidade)
A vulnerabilidade explorada é conhecida como Directory Listing (Listagem de Diretório). Quando um servidor web não encontra um arquivo de índice padrão (como index.html ou index.php) em uma pasta e a listagem de diretórios não está desativada na configuração, ele mostra todos os arquivos contidos ali. Isso permite que atacantes descubram arquivos que não deveriam ser públicos, baseando-se apenas na estrutura de pastas revelada por outros elementos (neste caso, a imagem).

Pesquisa
Pesquisei sobre "Apache Directory Listing vulnerability" e "Security by obscurity hidden files".

Natas Level 3
Objetivo
Acessar a senha para o nível 4. A página inicial apresenta apenas a mensagem "There is nothing on this page" (Não há nada nesta página).

Solução
Ao acessar o site, o conteúdo visível é inexistente.

Seguindo a metodologia padrão, inspecionei o código-fonte da página. Encontrei um comentário intrigante: "No more information leaks!! Not even Google will find it this time..." (Sem mais vazamentos de informação!! Nem mesmo o Google vai encontrar desta vez...).

A menção ao Google refere-se a mecanismos de busca (crawlers). A forma padrão de instruir robôs de busca a não indexarem ou não acessarem certas partes de um site é através do arquivo robots.txt.

Acessei http://natas3.natas.labs.overthewire.org/robots.txt e encontrei uma regra de "Disallow" (Não permitir) para o diretório /s3cr3t/.

Ao navegar para esse diretório proibido (/s3cr3t/), o servidor listou os arquivos contidos nele, revelando o arquivo users.txt.

Abrindo o arquivo, obtive a credencial para o próximo nível.

Raciocínio (Vulnerabilidade)
A vulnerabilidade aqui é a Security by Obscurity (Segurança por Obscuridade) e o vazamento de informações via robots.txt. O arquivo robots.txt é público e serve apenas como um guia voluntário para indexadores de busca; ele não impõe controle de acesso real. Atacantes frequentemente verificam esse arquivo para descobrir quais diretórios os administradores consideram sensíveis ou privados.

Pesquisa
Pesquisei sobre "robots.txt security risk" e "web crawler directives".

Natas Level 4
Objetivo
Acessar a senha para o nível 5. O acesso é negado com base na origem da visita.

Solução
Ao acessar a página, recebi uma mensagem de erro informando que o acesso é proibido porque estou visitando a partir do endereço atual (natas4...), enquanto usuários autorizados devem vir exclusivamente de http://natas5.natas.labs.overthewire.org/.

Isso indica uma verificação baseada no cabeçalho HTTP Referer. Para manipular essa requisição, utilizei a ferramenta Burp Suite. Configurei o proxy para interceptar o tráfego.

Enviei a requisição para o módulo "Repeater" do Burp Suite e alterei manualmente o cabeçalho Referer para o valor exigido pelo desafio: http://natas5.natas.labs.overthewire.org/.

Ao reenviar a requisição modificada, o servidor validou a origem falsa e retornou a página contendo a senha.

Raciocínio (Vulnerabilidade)
A vulnerabilidade é a Insecure Referer Checking (Verificação Insegura de Referer). O cabeçalho Referer é enviado pelo navegador do cliente e pode ser facilmente modificado ou falsificado por qualquer usuário. Servidores nunca devem confiar nesse cabeçalho para autenticação ou controle de acesso (Authorization).

Pesquisa
Pesquisei sobre "HTTP Referer header spoofing", "Burp Suite Repeater tutorial" e "Modifying HTTP headers".

Natas Level 5
Objetivo
Acessar a senha para o nível 6. O site informa que o usuário não está logado.

Solução
A página exibe a mensagem "Access disallowed. You are not logged in" (Acesso negado. Você não está logado). Não há formulário de login visível.

A autenticação web frequentemente depende de Cookies ou Sessões. Abri as ferramentas de desenvolvedor (F12) e naveguei até a aba Application (ou Storage) para inspecionar os cookies armazenados pelo site.

Encontrei um cookie chamado loggedin com o valor 0. Isso sugere uma lógica booleana simples (0 para não logado, 1 para logado).

Editei o valor do cookie loggedin de 0 para 1 diretamente no navegador e recarreguei a página (F5). O servidor leu o novo valor do cookie, assumiu que eu estava autenticado e exibiu a senha.

Raciocínio (Vulnerabilidade)
A falha é a Insecure Client-Side Storage e Broken Authentication (Autenticação Quebrada). O estado de autenticação foi confiado inteiramente a um dado armazenado no cliente (o cookie), sem validação criptográfica ou verificação no lado do servidor (Server-Side Session). O servidor confiou cegamente no dado enviado pelo usuário.

Pesquisa
Pesquisei sobre "Cookie manipulation attacks", "Client-side authentication vulnerabilities" e "Editing cookies in DevTools".

Natas Level 6
Objetivo
Acessar a senha para o nível 7. O site apresenta um formulário solicitando um segredo ("Input secret").

Solução
Ao tentar submeter qualquer valor aleatório no formulário, o sistema retorna "Wrong secret".

Cliquei no link "View sourcecode" para analisar a lógica de validação no back-end.

O código PHP revela que o segredo esperado é comparado com uma variável $secret. Essa variável é definida em um arquivo externo incluído no início do script: include "includes/secret.inc";.

Como o caminho é relativo e está dentro da estrutura web pública, tentei acessar o arquivo diretamente pelo navegador alterando a URL para /includes/secret.inc.

Ao acessar o arquivo, o navegador executou o PHP (que não imprime nada visualmente), resultando em uma página em branco. Para ver o conteúdo da variável, visualizei o código-fonte dessa página (Ctrl+U).

Copiei o segredo revelado (FOEIUWGHFEEUHOFUOIU), voltei à página inicial do nível e o submeti no formulário.

Raciocínio (Vulnerabilidade)
A falha aqui é a Information Disclosure (Divulgação de Informação) causada por má configuração de permissões de arquivo. Arquivos de configuração ou inclusão (como .inc ou .php) que contêm credenciais não devem estar acessíveis diretamente via URL. Se o servidor web não estiver configurado para negar acesso a esses arquivos, qualquer usuário pode baixá-los e ler seu conteúdo.

Pesquisa
Pesquisei sobre "Preventing direct access to include files" e "Sensitive data exposure in source code".

Natas Level 7
Objetivo
Acessar a senha para o nível 8. O site contém links para "Home" e "About".

Solução
Navegando pelos links, notei que a URL mudava através de um parâmetro page (ex: index.php?page=home). Isso sugere que o conteúdo está sendo carregado dinamicamente.

Ao inspecionar o código-fonte, encontrei um comentário deixado pelos desenvolvedores contendo uma dica crucial: o caminho absoluto onde a senha reside.

A dica informa: password for webuser natas8 is in /etc/natas_webpass/natas8.

Sabendo que o parâmetro page carrega arquivos, testei a vulnerabilidade de Inclusão de Arquivo Local (LFI) inserindo o caminho absoluto fornecido na URL: index.php?page=/etc/natas_webpass/natas8.

O servidor processou o caminho e exibiu o conteúdo do arquivo de senha diretamente na página.

Raciocínio (Vulnerabilidade)
A vulnerabilidade explorada é LFI (Local File Inclusion). Ela ocorre quando uma aplicação aceita entrada do usuário (neste caso, o parâmetro page) e a utiliza em funções de sistema de arquivos (como include ou require no PHP) sem validação ou sanitização adequada. Isso permite que um atacante leia arquivos sensíveis fora do diretório web root.

Pesquisa
Pesquisei sobre "PHP Local File Inclusion LFI payloads" e "LFI /etc/passwd path traversal".

Natas Level 8
Objetivo
Acessar a senha para o nível 9. O site solicita novamente um "secret".

Solução
A página inicial contém um formulário de entrada.

Ao analisar o código-fonte ("View sourcecode"), identifiquei a função encodeSecret, que ofusca o segredo antes de compará-lo com um hash hardcoded.

A lógica de encriptação é: bin2hex(strrev(base64_encode($secret))). Como todas essas funções são reversíveis e não utilizam uma chave privada, é possível recuperar o texto original invertendo a ordem das operações:

hex2bin

strrev

base64_decode

Criei um script PHP simples para automatizar a decodificação da string hexadecimal fornecida no código (3d3d516343746d4d6d6c315669563362).

O script retornou o valor oubWYf2kBq. Submeti este valor no formulário do desafio e obtive acesso.

Raciocínio (Vulnerabilidade)
A falha é o uso de Encoding como Criptografia (Security by Obscurity). Codificação (Base64, Hex) apenas muda o formato dos dados, não os protege. Sem o uso de algoritmos de criptografia robustos com chaves secretas (como AES), qualquer pessoa que tenha acesso ao algoritmo (visível no código-fonte) pode reverter o processo e recuperar os dados originais.

Pesquisa
Pesquisei sobre "Difference between encoding and encryption" e "PHP reverse hex2bin base64".

Natas Level 9
Objetivo
Acessar a senha para o nível 10. O site oferece uma ferramenta de busca para encontrar palavras contendo uma string específica.

Solução
A interface apresenta um campo de entrada simples: "Find words containing".

Ao analisar o código-fonte ("View sourcecode"), identifiquei que a entrada do usuário é passada diretamente para uma função de sistema sem sanitização adequada.

O código vulnerável é: passthru("grep -i $key dictionary.txt");.

A função passthru executa um comando diretamente no shell do sistema operacional. Como não há validação, é possível utilizar caracteres especiais do shell para manipular o comando. O caractere ; (ponto e vírgula) no Linux funciona como um separador, permitindo executar múltiplos comandos na mesma linha.

Construí o payload para:

Fechar o comando grep original.

Executar o comando cat para ler o arquivo da senha.

Payload utilizado: ; cat /etc/natas_webpass/natas10

O comando final executado pelo servidor tornou-se: grep -i ; cat /etc/natas_webpass/natas10 dictionary.txt.

Ao submeter a busca, o servidor executou o comando injetado e retornou a senha.

Raciocínio (Vulnerabilidade)
A vulnerabilidade é Command Injection (Injeção de Comando). Ela ocorre quando uma aplicação concatena dados não confiáveis (input do usuário) diretamente em uma string de comando do sistema operacional. Isso permite que o atacante execute comandos arbitrários no servidor com os privilégios da aplicação web.

Pesquisa
Pesquisei sobre "PHP passthru command injection" e "Linux shell command separators".

Natas Level 10
Objetivo
Acessar a senha para o nível 11. O funcionamento é similar ao nível anterior, mas com uma mensagem avisando sobre novos filtros de segurança.

Solução
A interface avisa: "For security reasons, we now filter on certain characters".

Analisando o código-fonte, notei que uma verificação com Expressão Regular (preg_match) foi adicionada para bloquear caracteres específicos que permitiriam encadear comandos.

A linha if(preg_match('/[;|&]/',$key)) bloqueia o uso de ;, | e &. Isso impede a técnica usada no nível 9 (terminar um comando e começar outro).

No entanto, o comando base ainda é grep -i $key dictionary.txt. A vulnerabilidade persiste porque podemos manipular os argumentos do comando grep, não o comando em si. O utilitário grep aceita múltiplos nomes de arquivos para busca.

A estratégia foi forçar o grep a procurar no arquivo da senha em vez de (ou além de) procurar no dicionário. Usei o payload .* /etc/natas_webpass/natas11.

.*: Uma regex que corresponde a qualquer conteúdo (garantindo que o grep imprima o conteúdo do arquivo).

/etc/natas_webpass/natas11: O caminho do arquivo que queremos ler, passado como segundo argumento para o grep.

O comando executado pelo servidor tornou-se: grep -i .* /etc/natas_webpass/natas11 dictionary.txt.

Isso fez com que o grep imprimisse todo o conteúdo do arquivo de senha.

Raciocínio (Vulnerabilidade)
A vulnerabilidade é Argument Injection (Injeção de Argumento). Mesmo que caracteres de controle de fluxo (como ; ou |) sejam filtrados, se o atacante puder controlar os argumentos passados para um executável, ele pode alterar o comportamento do programa para acessar arquivos ou realizar ações não intencionais. "Blacklists" (listas de caracteres proibidos) são frequentemente ineficazes pois raramente cobrem todos os vetores de ataque.

Pesquisa
Pesquisei sobre "Grep argument injection" e "Bypassing command injection filters".
