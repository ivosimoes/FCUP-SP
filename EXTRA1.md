# Secure WP Hosting

  Neste desafio é nos apresentado um site que assemelha a uma loja de instâncias WordPress. Incialmente, não parece existir algo de errado com o site, mas, após investigar o site mais a fundo, descobrimos algo que pareceu ser uma dica:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/587b2b22-a466-4246-8e79-8639f7885571)

  Ao que parece, os donos do site não atualizam os plugins e as instâncias de WordPress frequentemente, pelo que podem existir versões desatualizadas (e vulneráveis) de plugins ou do WordPress em si. Após analisar o produto no qual foi postada esta review, confirmamos a nossa suspeita:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/91204419-533a-44ec-b622-e51b761da8b6)

  Neste caso, tanto o WordPress como os plugins estão extremamente desatualizados, aumentando a probabilidade de existir vulnerabilidades críticas associadas a estes. Com esta informação, podemos procurar por CVEs que comentem a existência de vulnerabilidades à volta destes plugins. Alguns exemplos são:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/f1ae0236-c6d3-4077-af14-7ec724c08ab3)
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/94c74f34-af0d-4c6d-a0aa-a21fb63fe323)

  Dos possíveis candidatos apresentados acima, pareceu-nos que o terceiro (CVE-2021-34646) é o mais favorecido, devido a ter uma pontuação mais alta, menos requisitos para ser "executado" e ser mais simples em geral. Agora precisamos de um exemplo ou PoC (Proof of Concept) do CVE para pô-lo em prática. Através de uma simples pesquisa pelo CVE, encontrámos uma página no GitHub que apresenta um script Python exemplo para explorar esta vulnerabilidade:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/9b6f1166-7407-4849-9502-a12d35859284)
  (Fonte: https://github.com/motikan2010/CVE-2021-34646)

  Na página vemos o tal script junto com uma explicação de como usá-lo e como funciona. Seguindo as instruções apresentadas, vamos editar o script Python para tentar dar bypass à autenticação de admin. Primeiro precisamos de saber qual o user_id do admin. Por norma, o valor do user_id para o admin é 1, mas, para confirmar, podemos aceder a alguns detalhes dos utilizadores do site CTF:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/3fa36af4-aa4b-4d32-8459-cc18d68f1eaa)
  (Fonte: http://ctf-sp.dcc.fc.up.pt:5001/wp-json/wp/v2/users)
  
  Agora que temos certeza do user_id do admin, vamos proceder à edição do script. Um exemplo do seu uso para o nosso caso seria o seguinte:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/053bd1a4-71a9-43be-afd5-15e181781fcd)
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/0a4dd87b-b1e7-4bfd-9928-402b96dd0f51)

  O link gerado pelo script deve-nos dar acesso à conta do admin. Vamos ver se esse é realmente o caso:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/31de5454-1d75-4c39-b77c-bae4f2b7b0f5)
  (Fonte: http://ctf-sp.dcc.fc.up.pt:5001?wcj_verify_email=eyJpZCI6IjEiLCJjb2RlIjoiZTEzMjIxNDU0ODY4ZjA0NjU5OWQ5YmY4MzVjN2ZlZTAifQ%3D%3D)
  
  Acedemos à conta do admin com sucesso! Só nos resta procurar pela flag. Após uns largos minutos, encontrámo-la no meio de uma mensagem:

  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/27553461-2e8b-4da2-92e9-d26ecd0f241c)
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/eaaa58fc-b278-4d87-865a-0f774582a82f)
  (Fonte: http://ctf-sp.dcc.fc.up.pt:5001/wp-admin/post.php?post=11&action=edit)  
