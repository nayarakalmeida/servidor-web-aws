# servidor-web-aws

Servidor web em EC2 com Apache, domínio próprio e HTTPS via Let's Encrypt.

Site: https://nayaraalmeida.com.br

## Como foi montado

| Camada | O que foi usado |
|---|---|
| Servidor | Instância EC2 com Ubuntu (AWS, us-east-1) e Elastic IP |
| Servidor web | Apache 2.4, servindo a página de portfólio em /var/www/html |
| DNS | Registros A no Registro.br para o domínio e para o www |
| HTTPS | Certificado Let's Encrypt emitido e renovado pelo Certbot, com redirecionamento de HTTP para HTTPS |
| Rede (fora do servidor) | Grupo de segurança da AWS com as portas 80 e 443 abertas e a 22 (SSH) restrita ao meu IP |
| Firewall (dentro do servidor) | ufw ativo, liberando só as portas 22, 80 e 443 |
| Custos | AWS Budgets com orçamento mensal e alerta por e-mail |

## Por que cada decisão

### Elastic IP
Usei um Elastic IP para o servidor manter sempre o mesmo endereço, mesmo se a instância for parada e iniciada de novo. Assim o domínio e o meu acesso por SSH continuam apontando para o lugar certo.

### Porta 22 (SSH)
Deixei restrita ao meu IP por ser a porta de administração do servidor. Mesmo exigindo a chave para entrar, aberta para todos ela receberia várias tentativas automáticas o tempo todo.

### Portas 80 e 443
Deixo abertas para qualquer IP porque é através delas que os visitantes entram no site.

### Registros A com e sem www
Criei os dois para que a página seja encontrada tanto com www quanto sem.

### HTTPS
O HTTPS garante que ninguém no caminho possa ler ou alterar o que chega ao visitante, e o certificado comprova que o site é meu. Sem ele, o navegador mostraria o aviso de "Não seguro".

### Porta 80 depois do HTTPS
Continua aberta porque é por ela que a Let's Encrypt confere o domínio para emitir e renovar o certificado. Além disso, ela redireciona quem entra sem https para a página segura.

### Firewall do servidor (ufw)
O ufw é uma segunda camada de segurança, que filtra o tráfego dentro do próprio Ubuntu. O grupo de segurança é o firewall da AWS e filtra antes de a conexão chegar à máquina, do lado de fora da instância. Assim, uma conexão de fora passa primeiro pelo grupo de segurança, depois pelo ufw, e só então chega ao serviço.

Ele libera só três portas:

- **22 (SSH):** acesso remoto e criptografado para a administração do servidor
- **80 (HTTP):** encaminha para a versão segura quem entra sem https, e é por ela que a Let's Encrypt valida o domínio para emitir e renovar o certificado
- **443 (HTTPS):** a versão criptografada do site, por onde o visitante entra

No ufw as três ficam abertas para qualquer IP. Quem restringe a 22 ao meu IP é o grupo de segurança, que é a camada de fora.

Liberei a 22 antes de ativar o firewall porque, ao ser ligado, o ufw fecha todas as entradas que não têm regra. Sem essa regra, minha conexão SSH cairia e eu ficaria sem acesso ao servidor. Antes de ativar, conferi as regras com `sudo ufw show added` e, depois, testei de fora a cada porta liberada.

## Arquivos

- `index.html` é a página de portfólio servida pelo Apache

## Laboratórios de quebra

Quebrei o servidor de propósito para treinar o diagnóstico: provocar a falha, encontrar a evidência e desfazer.

### 1. Erro de digitação na configuração do Apache

- **Quebra:** troquei `DocumentRoot` por `DocumentRot` no arquivo de configuração do site, depois de fazer uma cópia de segurança
- **Sintoma:** o Apache não reiniciou e o site saiu do ar
- **Evidência:** o `systemctl status apache2` apontou erro de sintaxe na linha 12, com o comando inválido `DocumentRot`
- **Correção:** corrigi a linha, conferi com `apache2ctl configtest` (Syntax OK) e reiniciei
- **Lição:** rodar o `configtest` sempre antes de reiniciar

### 2. Porta 80 bloqueada no firewall

- **Quebra:** bloqueei a porta 80 no ufw com `sudo ufw deny 80`
- **Sintoma:** o acesso por http falhou, enquanto o https continuou respondendo com 200
- **Evidência:** testei de fora com `curl.exe -I`, e o ensaio de renovação (`sudo certbot renew --dry-run`) falhou com timeout ao conectar
- **Correção:** liberei a 80 de novo e conferi o redirecionamento e a renovação
- **Lição:** a porta 80 continua necessária depois do HTTPS, para o redirecionamento e para a validação do certificado

### 3. Porta 80 ocupada por outro programa

- **Quebra:** parei o Apache e coloquei um servidor web mínimo do Python escutando na porta 80, a partir de uma pasta vazia
- **Sintoma:** o Apache não ligou, e o site saiu do ar também no HTTPS, mesmo com a 443 livre
- **Evidência:** o `systemctl status apache2` mostrou "Address already in use" na porta 80, e o `sudo ss -tlnp` mostrou o `python3` escutando nela
- **Correção:** encerrei o Python, liguei o Apache e conferi com `ss -tlnp` e `curl.exe -I` (200)
- **Lição:** o Apache é um programa só nas duas portas. Se uma falha, ele não sobe, e a 443 cai junto

### 4. Pasta do site sem permissão

- **Quebra:** mudei a permissão de `/var/www/html` de 755 para 700, tirando a permissão de entrada do usuário do Apache (`www-data`)
- **Sintoma:** o site respondeu 403 Forbidden, com o Apache ligado
- **Evidência:** o `ls -ld` mostrou `drwx------`, e o `error.log` registrou "Permission denied" por falta de permissão de busca na pasta
- **Correção:** voltei para 755 e conferi com `curl.exe -I` (200)
- **Lição:** o Apache roda como `www-data`, não como root, e a pasta do site precisa deixar esse usuário entrar e ler

## Status

No ar desde 22/10/2026.
