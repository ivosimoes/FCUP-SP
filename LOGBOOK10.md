# Cross-Site Scripting (XSS) Attack Lab (Web Application: Elgg)

## Task 1: Posting a Malicious Message to Display an Alert Window

  O perfil escolhido foi Alice e será esse o utilizador escolhido como nosso ao longo dos exercícios. O programa JavaScript fornecido na ficha (<script>alert("XSS");</script>) foi colocado no campo "Brief Description", tal como se pode verificar na primeira imagem. Após a informação ser guardada o alerta aparece no perfil, tal como podemos ver na segunda imagem. Mantemos a restrição do campo como "Public" porque não há restrições impostas na ficha. 
  
  <img width="538" alt="Print1" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/e56f63fd-b7d2-4e38-8770-9da6ca875cbc">

  <img width="542" alt="Print2" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/1df7f99c-0fde-4a7f-8b20-5e0aadb99bcc">
  
## Task 2: Posting a Malicious Message to Display Cookies

  O método alert() de JavaScript permite mostrar uma mensagem de texto (tal como foi feito na tarefa anterior) ou alguma informação requesitada, como neste caso.
  Para completar esta tarefa foi apenas alterado o texto do método alert() para mostrar as cookies do utilizador tal como é mostrado na primeira imagem. Tal como podemos ver na segunda imagem, o código está a funcionar.
  
  <img width="539" alt="Print3" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/6bc299fe-0441-4389-a4c9-352bd47dcbc2">

  <img width="545" alt="Print4" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/f1d11da1-74de-4530-9c09-d9c7b55b7526">
  
## Task 3: Stealing Cookies from the Victim’s Machine

  De maneira completar a tarefa com o perfil escolhido, colocámos o texto da ficha no campo "About me" na secção "Edit HTML" tal como se pode verificar na primeira imagem. Como podemos verificar na segunda imagem, um dos endereços IP do computador é 10.9.0.1 logo é para a porta 5555 desse endereço que iremos mandar as cookies recolhidas com a abertura do perfil por utilizadores. O programa funciona tal como evidencia a última imagem.
  
  <img width="536" alt="Print5" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/a48430c5-54bf-423d-85f4-64cacb6ac775">
  
  <img width="442" alt="Print6" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/658b55a5-4d8d-4fc5-a5b4-bb03b0242c78">

  <img width="532" alt="Print7" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/3eb0fd6b-16e8-4e6c-a5cf-5ac8bc72a0fe">
  
## Task 4: Becoming the Victim’s Friend

  O código completo em JavaScript que completa a tarefa é o apresentado na imagem abaixo:
  
  <img width="530" alt="image" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/c40a27ae-868b-4dd1-81ba-ca311e1d1f21">
  
### Question 1 Explain the purpose of Lines ➀ and ➁, why are they needed?

   Usando a extensão HTTP Header live, vemos que surge um novo endereço quando mandamos um pedido de amizade do nosso utilizador para o utilizador Samy, tal como pode ser verificado na imagem abaixo. O endereço é http://www.seed-server.com/action/friends/add?friend=59&__elgg_ts=1684538279&__elgg_token=iEzhlNrBrQPvFn_vhyNyZg&__elgg_ts=1684538279&__elgg_token=iEzhlNrBrQPvFn_vhyNyZg .
  
  <img width="545" alt="image" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/b464f83c-9a88-49e8-861c-6c579f50d933">
  
   Desta maneira e com as variáveis ts (linha ➀) e token (linha ➁) conseguimos formar a estrutura do endereço, que tem a seguinte forma: http://www.seed-server.com/action/friends/add?friend=59 + ts + token + ts + token.
  
### Question 2: If the Elgg application only provide the Editor mode for the "About Me" field, i.e., you cannot switch to the Text mode, can you still launch a successful attack?

  Sim, é possivel. Se inspecionarmos a página e inserimos o código html no inspetor é possível executar um ataque, tal como da maneira anterior. O processo feito está mostrado na imagem a baixo.
  
  <img width="537" alt="image" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/e56a06ff-8600-412b-8fbe-204e2e4ac77e">

  
  
