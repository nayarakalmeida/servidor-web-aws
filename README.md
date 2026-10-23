\# servidor-web-aws



Servidor web em EC2 com Apache, domínio próprio e HTTPS via Let's Encrypt.



Site: https://nayaraalmeida.com.br



\## Como foi montado



| Camada | O que foi usado |

|---|---|

| Servidor | Instância EC2 com Ubuntu (AWS, us-east-1) e Elastic IP |

| Servidor web | Apache 2.4, servindo a página em /var/www/html |

| DNS | Registros A no Registro.br para o domínio e para o www |

| HTTPS | Certificado Let's Encrypt emitido e renovado pelo Certbot, com redirecionamento de HTTP para HTTPS |

| Rede | Grupo de segurança com as portas 80 e 443 abertas e a 22 (SSH) restrita ao meu IP |



\## Arquivos



\- `index.html`: a página servida pelo Apache



\## Status



No ar desde 22/10/2026. Próximo passo: documentar o porquê de cada decisão de configuração.

