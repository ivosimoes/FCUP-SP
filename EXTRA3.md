# Apply For Flag II

  Este desafio é muito semelhante a outro que já resolvemos na semana 10. Trata-se de um site onde temos que fazer um pedido para obter a flag do admin. Porém, o enunciado do desafio diz-nos que, desta vez, não vamos conseguir XSS. Vamos ver se isso é verdade:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/547906e5-3eaf-4595-9f51-fd02e66fa73f)
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/9a8c1788-b666-4d50-89ad-3ee5c1eed76d)

  Só com este teste já observámos várias coisas: o site é suscetível a XSS, o ID do nosso pedido aparece logo na páginal inicial de submissão de pedido e após submissão, a página entra num ciclo de refresh constante, com um aviso a dizer se o admin aceitou o nosso pedido ou não. Vamos agora ver como é que podemos chegar à flag:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/469caf40-389b-4bad-882d-ea82dc3bf53f)

  Ao que parece, a flag está bloqueada atrás do botão "Give the flag", que nos redireciona para o endpoint /accepted se o admin aprovar o nosso pedido. Assim sendo, precisamos de um script que faça o admin clicar neste botão para obter a flag:
  
  ```html
  <form method="POST" action="http://ctf-sp.dcc.fc.up.pt:5005/request/<request_id>/approve" role="form">          
  <div class="submit">                  
    <input type="submit" id="giveflag" value="Give the flag">   
  </div>  
  </form>    

  <script type="text/javascript">         
   document.querySelector('#giveflag').click();  
  </script>
  ```

  Este script vai ao pedido <requestID> e ativa o botão "Give the flag" que, como vimos anteriormente, está desativado. A parte do script faz com que o admin clique no botão quando vai dar refresh, levando este a aceitar o pedido. Vamos testar o script:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/1fa093aa-b6b4-49ef-8960-71552f0c9caa)

  Por algum motivo, não nos deu acesso à página. Após pesquisar um pouco, descobrimos que isto pode ser causado por execução de JavaScript. Recorrendo a uma extensão, desligámos a execução de JavaScript e tentámos outra vez:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/c7cc0e2c-1935-445b-961e-ddd807f1b2b7)
