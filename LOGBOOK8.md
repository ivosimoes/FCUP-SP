# SQL Injection Attack Lab

  Antes de começar as tarefas, tal como na semana anterior, temos que começar por construir e iniciar os containers:
  
   ![imagem](https://user-images.githubusercontent.com/126570489/234631711-8f32712b-6c57-4e18-b745-74eaf7bd13b2.png)
   ![imagem](https://user-images.githubusercontent.com/126570489/234631854-b303c1b8-2d78-4f24-8266-fa2616d24aab.png)
   ![imagem](https://user-images.githubusercontent.com/126570489/234632337-83a8529b-cff0-4106-ae04-6699683edab2.png)

  Agora que estamos no container do MySQL, podemos prosseguir para as tarefas.
  
## Task 1: Get Familiar with SQL Statements

  Esta tarefa é bastante simples: usar o select do MySQL para imprimir os dados da Alice. Como já estamos familiarizados com o MySQL (dado ao que aprendemos em Bases de Dados), foi relativamente fácil chegar a uma prompt que imprimisse a informação desejada:
  
   ![imagem](https://user-images.githubusercontent.com/126570489/234636409-9f87c668-8e58-431f-b886-57efe77ccd76.png)
   
  Neste caso, usamos o * para ir buscar informações a todas as colunas e aplicamos a condição Name='Alice' que só imprime todas as colunas nas quais o Name seja igual ao especificado.
  
## Task 2: SQL Injection Attack on SELECT Statement
  
### Task 2.1: SQL Injection Attack from webpage

  Nesta sub-tarefa vamos tentar aceder à conta do admin sem saber a sua password. Uma ideia que tivemos consiste em aproveitar-nos da vulnerabilidade presente na variável 'input_uname' que é usada na query sem ser verificada, fornecendo uma maneira de injetar código SQL adicional na query.
  
    - Indicação da vulnerabilidade no guião:
    
   ![imagem](https://user-images.githubusercontent.com/126570489/234662230-4d28647b-cd7f-4f3a-b650-ba61264e41e6.png)

  Após diversas tentativas, conseguimos arranjar um input que nos leva até à conta do admin:
  
   ![imagem](https://user-images.githubusercontent.com/126570489/234675170-9968d72c-0a4e-4e61-9169-bd02136f925e.png)
   ![imagem](https://user-images.githubusercontent.com/126570489/234675501-c10930a4-bb72-4e8b-b7dd-b9299476e2de.png)

  Para entender o input usado é necessário entender a estrutura da query. Vamos percorrer a estrutura do código e do input passo a passo:
  
  1. Primeiro apóstrofe (') - No MySQL, quando queremos usar strings, temos que recorrer a apóstrofes. Neste caso, para fazer a query sobre o Username, o input dado no site é posto dentro de apóstrofes, sinalizando ao SQL que este input é uma string. Ao colocarmos um apóstrofe no input, estaremos a sinalizar ao SQL que a string do username terminou e que o resto é código SQL;

  2. OR Name='Admin' - Devido ao ponto anterior, passámos a ter a condição Name='' que resultaria num empty set. Para resolver este problema, basta adicionar outra condição que nos permita obter um resultado válido e, para tal, podemos usar o operador OR que, quando combinado com Name='Admin', possibilita o acesso à conta do user que desejamos.

  3. Comentário (--) - Em MySQL, o operador -- é usado para introduzir um comentário. No contexto deste input, comentar a condição que vem a seguir facilita-nos a vida pois assim deixa de ser necessário introduzir a password associada à conta do admin.

  4. Segundo apóstrofe - Juntando todos os passos até agora, a nossa query seria "SELECT id, name, eid, salary, birth, ssn, address, email, nickname, Password FROM credential WHERE name= '' OR Name='Admin' --' and Password=’$hashed_pwd’". Se olharmos com atenção, veremos que existe um apóstrofe a mais a seguir ao comentário, o que pode causar confusão ao SQL. Para evitar o surgimento de um erro na query, podemos simplesmente adicionar um outro apóstrofe, indicando que o apóstrofe pertence, de facto, ao comentário.
  
### Task 2.2: SQL Injection Attack from command line
  
  Tal como diz no guião, esta sub-tarefa resume-se a fazer o mesmo que fizemos na tarefa anterior, só que, em vez de fazermos a injeção no site, vamos fazê-la no terminal, recorrendo ao comando 'curl'. O comando 'curl' envia um pedido a um site e imprime o resultado do pedido feito na shell. Uma maneira de fazer a sub-tarefa seria a seguinte:
  
   ![imagem](https://user-images.githubusercontent.com/126570489/235179960-c4c5839e-1883-4c8c-8229-6a3d4b077cc0.png)
   ![imagem](https://user-images.githubusercontent.com/126570489/235180195-563d43f7-1015-491f-b408-efdfbf357324.png)

  No que toca ao input, seguimos o modelo apresentado no guião, onde definimos o conteúdo do campo Username através da string "%27%20OR%20Name%3d%27Admin%27%20%2d%2d%20%27". Esta string é a mesma que foi usada na sub-tarefa anterior, alterando apenas os carateres especiais para a sua codifição em UTF-8, que constam na lista seguinte:
  
  1. Apóstrofe (') = %27
  2. Espaço ( ) = %20
  3. Igual (=) = %3d
  4. Hífen (-) = %2d

  O output é constituído por código HTML da página apenas acessível pelo administrador. Na segunda imagem, conseguimos ver o código da tabela que está presente na página do administrador, provando assim que conseguimos aceder à conta Admin através do terminal.
  
### Task 2.3: Append a new SQL statement

  Queremos agora averiguar se o site nos deixa correr múltiplas queries. Para tal, vamos tentar adicionar uma query que apaga dados da tabela (DELETE FROM credential WHERE Name='Alice') após a query SELECT usando o operador ';' do SQL. O processo anteriormente descrito resultou no seguinte:
  
   ![imagem](https://user-images.githubusercontent.com/126570489/235185553-b9caa9fa-3198-4c4c-8127-5836abe06dc2.png)

  Apesar do código SQL estar bem formulado, a query resultou num erro. Porquê? Vamos colocar aqui umas imagens que podem responder à nossa pergunta:
  
    - Código PHP da página:
    
   ![imagem](https://user-images.githubusercontent.com/126570489/235187210-74149659-4781-499e-b543-3014f88205e5.png)

    - Manual da função query do PHP:
    
   ![imagem](https://user-images.githubusercontent.com/126570489/235189620-baa7b8d5-9106-43b6-815f-9b15fc53fe16.png)

  Como se pode obsevar na primeira imagem, o site recorre à função query() do mysqli do PHP para fazer a query na base de dados. Esta função permite apenas fazer 1 query sobre a base de dados. Como, no caso desta tarefa, estamos a tentar executar 2 queries, o programa vai retornar um erro quando chega à segunda e ambas as queries acabam por não ter efeito. Para resolver este problema, poderíamos usar a função multi_query(), também existente no mysqli:
  
    - Manual da função multi_query:
     
   ![imagem](https://user-images.githubusercontent.com/126570489/235190880-0e8aa7e7-1eeb-4716-a443-e9334c588faf.png)

  Nota: Apesar de existirem 2 classes (PDO e mysqli) que possuem o método query(), determinamos a que está a ser usada através da função que cria uma ligação ao SQL (getDB), como se pode ver nesta imagem:
  
   ![imagem](https://user-images.githubusercontent.com/126570489/235192578-abeb1abc-e2a3-42c2-9d14-f89296030b27.png)

  Fontes: query() - https://www.php.net/manual/en/mysqli.query.php
          multi_query() - https://www.php.net/manual/en/mysqli.multi-query.php

## Task 3: SQL Injection Attack on UPDATE Statement

### Task 3.1: Modify your own salary

  À semelhança da tarefa anterior, nesta sub-tarefa temos de manipular os operadores de SQL para adicionar/modificar a query UPDATE. Uma maneira de alterar o salário da Alice seria adicionar "123', salary=999999 WHERE Name='Alice' #" ao campo Phone Number tal como mostrado nas seguintes imagens:
  
   ![imagem](https://user-images.githubusercontent.com/126570489/235207632-1449fed1-e623-411d-931b-f0ce4ba7dc01.png)
   ![imagem](https://user-images.githubusercontent.com/126570489/235207713-c63355a5-ae83-44ed-b404-a8bc4ecff026.png)

  Vamos percorrer o input passo a passo tal como em 2.1:
  
  1. Escolhemos o campo Phone Number devido a ser o campo que se localiza imediatamente antes ao WHERE, sendo assim possível editar o salário e escolher a pessoa a quem ele é alterado. Como temos que comentar o resto da linha devido ao apóstrofe já referido anteriormente, é necessário também editar a condição WHERE, daí ser necessário escolher a pessoa a quem o salário é alterado;

  2. A parte "123'" da string edita o campo Phone Number e fecha essa mesma edição com o apóstrofe, sendo depois possível alterar o salário;

  3. O salário é alterado com ", salary=999999" que foi adicionado a seguir ao Phone Number, usando uma vírgula para indicar a inserção de um novo dado a ser alterado;

  4. A condição "WHERE Name='Alice'" especifica que os dados anteriores devem ser alterados para o funcionário com nome Alice;

  5. O operador '#' marca um comentário que acaba ao mudar de linha. Tentámos usar os 2 hífens '--' para inserir um comentário, mas, por algum motivo, não funcionou. Após uma breve pesquisa, vimos que '#' também serve para inserir um comentário e, no nosso caso, realiza a alteração;

### Task 3.2: Modify other people’ salary

  Esta sub-tarefa é praticamente igual à anterior. A diferença é que agora queremos alterar o salário do Boby para 1, o que pode ser alcançado alterando a string anterior de acordo com os novos parâmetros: "123', salary=1 WHERE Name='Boby' #". Recorrendo à técnica anterior, vamos tentar fazer a modificação:
  
   ![imagem](https://user-images.githubusercontent.com/126570489/235211921-dfdddb49-de49-4de0-9333-529df94994de.png)

  Como as alterações à conta do Boby não são visíveis devido a não estarmos na conta do Admin, vamos à conta do Admin ver se realmente alterámos o salário:
  
  ![imagem](https://user-images.githubusercontent.com/126570489/235213161-f55a8647-7ddc-4bd3-83ab-fb187a84a72d.png)

  Recorrendo à vulnerabilidade que explorámos em 2.1, conseguimos ver que o salário do Boby foi alterado com sucesso através da conta da Alice.

### Task 3.3: Modify other people’ password

  Recorrendo ao input usado em 3.2 e fazendo uma pequena alteração, vamos tentar alterar a password do Boby. A única alteração que precisamos de fazer é alterar o campo Password, acrescentando uma password à nossa escolha. Decidimos usar a password "hello123". Vamos ver se resulta:
  
   ![imagem](https://user-images.githubusercontent.com/126570489/235217866-defd780a-a875-4b0d-a82f-af774359b08f.png)
   ![imagem](https://user-images.githubusercontent.com/126570489/235218360-5b0b5cf1-9e8d-4188-8b2d-2a0c804de2fd.png)
   ![imagem](https://user-images.githubusercontent.com/126570489/235218478-d535dc2e-2839-4075-aff5-a53074d593ef.png)
   ![imagem](https://user-images.githubusercontent.com/126570489/235218548-c2873bf2-559c-476c-9c55-05ec05d3ca64.png)

  Após efetuar as alterações, o site apresenta uma mensagem a dizer que "as credenciais introduzidas não existem", possivelmente sinalizando que uma password foi alterada com sucesso. Seguidamente, guardámos a password junto com o username do Boby no Firefox como mostra a segunda imagem. As últimas 2 imagens mostram uma tentativa de dar login na conta do Boby com as credenciais que guardámos no browser, que foi bem sucedida.
  
