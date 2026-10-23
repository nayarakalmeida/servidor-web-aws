# servidor-web-aws

Servidor web em EC2 com Apache, domínio próprio e HTTPS via Let's Encrypt.

Site: https://nayaraalmeida.com.br

## Como foi montado

| Camada | O que foi usado |
|---|---|
| Servidor | Instância EC2 com Ubuntu (AWS, us-east-1) e Elastic IP |
| Servidor web | Apache 2.4, servindo a página em /var/www/html |
| DNS | Registros A no Registro.br para o domínio e para o www |
| HTTPS | Certificado Let's Encrypt emitido e renovado pelo Certbot, com redirecionamento de HTTP para HTTPS |
| Rede | Grupo de segurança com as portas 80 e 443 abertas e a 22 (SSH) restrita ao meu IP |

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
Por enquanto quem controla as portas é o grupo de segurança da AWS, então deixei o ufw desligado. Quando ligado, ele bloqueia tudo o que não foi liberado antes, e eu poderia perder o acesso ao servidor se esquecesse da porta 22. Ativar o ufw como segunda camada fica para uma próxima etapa.

## Arquivos

- `index.html` é a página servida pelo Apache

## Status

No ar desde 22/10/2026.