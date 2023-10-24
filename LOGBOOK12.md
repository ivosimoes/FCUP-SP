# Public-Key Infrastructure (PKI) Lab

## Task 1: Becoming a Certificate Authority (CA)
  
  Começamos por copiar openssl.cnf para o diretório de trabalho para fazer as mudanças necessárias.
  
  <img width="542" alt="cap2" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/c9bc79c7-4d7a-4525-88db-9c91b6ecfbf9">
  
  Tal como pedido, tiramos o simbolo de comentário na linha relativa a "unique subject", tal como mostra a próxima imagem:
  
  <img width="515" alt="cap1" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/44b90b0b-278d-412c-b3ff-48387d17466e">
  
  De seguida foram criados os ficheiros index.txt e serial tal como foi pedido. Seguidamente criamos o certificado CA que será completamente confiável.
  
  <img width="536" alt="cap3" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/9a68c399-c29a-44da-9e7b-094cddec27dc">
  
  Após todos este passos é possível responder às perguntas colocadas no ficheiro para esta tarefa:
  - What part of the certificate indicates this is a CA’s certificate?
     
     Tal como é mostrado na imagem, o valor de CA é verdade pelo que se trata de uma certificado CA.
     
     <img width="540" alt="image" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/44c29da5-ea53-4c4b-b886-4cfac93b4c2a">
     
  - What part of the certificate indicates this is a self-signed certificate?
     
     "Issuer" e "Receiver" são os mesmos.
     
     <img width="536" alt="image" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/78fd3a7d-b266-491a-aae9-5cd076f0e4d3">
     
  - In the RSA algorithm, we have a public exponent e, a private exponent d, a modulus n, and two secret numbers p and q, such that n = pq. Please identify the values for these elements in your certificate and key files.
     
     O módulo n é:
     
     <img width="231" alt="image" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/172ff189-3cc7-4613-b77b-a9ba877d9979">
     
     Os expoentes público (e) e privado (d) são, por esta ordem, os seguintes:
     
     <img width="227" alt="image" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/92d48d3e-0bd3-4fbb-950d-d141c45dd2e8">
     
     Os números secretos p e q são:
     
     <img width="233" alt="image" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/54e3dca2-35f7-45eb-ab98-b54996d8985c">

## Task 2: Generating a Certificate Request for Your Web Server
  
  Tal como pedido na tarefa, foi criado um certificado de chave pública CA para a empresa bank32.com com possibilidade para dois nomes alternativos.

  <img width="532" alt="image" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/f751866a-8019-4c22-955f-6b7566754594">
  
## Task 3: Generating a Certificate for your server
  
  Para esta tarefa, com o código fornecido pelo enunciado e a alteração necessária no openssl.cnf tornamos o CSR criado anteriormente num certificado auto-assinado.
  
  <img width="534" alt="image" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/a5e98e81-662c-45e5-aed3-1fafb449ad88">
  
  <img width="484" alt="image" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/771e096d-1987-43b8-8e9f-994ce150bc67">
  
## Task 4: Deploying Certificate in an Apache-Based HTTPS Website
  
  Nesta tarefa vamos tentar montar o nosso próprio site com os certificados que obtemos anteriormente. O primeiro passo será modificar o ficheiro bank32_apache_ssl.conf para o seguinte:

  ```cfg
  <VirtualHost *:443> 
      DocumentRoot /var/www/SP2023
      ServerName www.SP2023.com
      ServerAlias www.SP2023A.com
      ServerAlias www.SP2023B.com
      DirectoryIndex index.html
      SSLEngine On 
      SSLCertificateFile /certs/SP2023.crt
      SSLCertificateKeyFile /certs/SP2023.key
  </VirtualHost>

  <VirtualHost *:80> 
      DocumentRoot /var/www/SP2023
      ServerName www.SP2023.com
      DirectoryIndex index_red.html
  </VirtualHost>

  # Set the following gloal entry to suppress an annoying warning message
  ServerName localhost
  ```
  
  Temos também que mover os ceritificados que criámos para /certs e criar uma pasta /var/www/SP2023 para ter código para o nosso site:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/d0f0d85d-8fee-422a-b52a-10b7d39a61c7)
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/5ccac44f-a8b1-438c-97b5-b4cc513f55ce)

  Ainda assim o site não funciona. Isto deve-se ao facto do emissor do certificado ser desconhecido por parte do browser, levando-o a assumir a presença de malware:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/84c38f41-63a0-4221-9c2d-87c8eb30bd34)

  Para resolver este problema, temos que dar a conhecer ao browser que somos uma CA (Certificate Authority):
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/d2a4a69a-b1ca-4a58-be60-8890c8947f6b)

  Como estamos registados como uma CA, o nosso site pode ser visto agora sem problemas:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/e0f028a3-01fc-482c-b825-b02c20e7fbc9)
  
## Task 5: Launching a Man-In-The-Middle Attack
  
  Começemos por adicionar um site a /etc/hosts:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/8c9d2ec8-e465-4ebf-b38c-7147a4744864)

  Vamos agora editar o ficheiro de configuração do nosso site:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/a37e0043-9180-47e1-b943-ac430c1e6038)

  Tentemos agora aceder a https://www.mitm.com:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/d2ce9f4d-75f6-4f58-8da7-43c6bb128f69)

  Como o nosso certificado só abrange o domain sp2023, este site irá ser considerado inseguro, tendo-se assim invocado um MITM.
  
## Task 6: Launching a Man-In-The-Middle Attack with a Compromised CA
  
  Para o ataque vamos precisar de um certificado para o site MITM:

  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/252e124c-086e-481f-b245-32df1b6da91c)
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/8bbb6ae9-028e-448e-835b-5f71a9f83baa)

  Depois de enviar os novos certificados para o container, vamos alterar mais uma vez o config para o seguinte:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/a4e25301-51bd-4f86-b016-e9a5aeb121de)
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/5e0dc219-dd7c-4964-993b-f59677f18263)
  
  Agora temos de reiniciar o container e recomeçar o apache2:

  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/8ceb274c-6228-4410-bb01-4126525ebc9f)
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/7d9ae323-ea69-4137-83be-75d59a35e0e0)
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/e5b7e5d3-35a3-4ae1-a8b7-7f79f77a1e98)

  Feito isto, devemos conseguir aceder ao site www.mitm.com sem problemas:

  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/8cf8387d-a456-4ff4-83c2-4edfd820f9d7)
