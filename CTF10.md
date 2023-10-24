# CTF 10

## Desafio 1
  
  Neste desafio temos de fazer com que o administrador dê a flag sem ele carregar no botão que faz essa ação. De acordo com o enunciado, o administrador, num espaço máximo de 2 minutos, irá carregar no botão "Mark request as read" em vez do botão que fornece a flag, negando assim o pedido. Sabendo esta informação, podemos criar um script para que quando o administrador carregar nesse botão, execute a ação do outro botão. Para isso foi criado o seguinte código:

    <script>
        window.onload = function() {
            var flag = document.getElementById('giveflag'); 
            var button = document.getElementById('markAsRead');  
            button.disabled = false; 
            button.onclick = flag.click(); 
            button.parentElement.parentElement.action=""  
        }  
    </script>
    
  Com a execução deste script descobrimos a flag: flag{71b875bda298d447ed33e2fbefb9bbb2}.
  
  <img width="524" alt="image" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/66d8621e-7736-4e0f-8c00-44dec25c1aff">

## Desafio 2

### Tarefas

- Que funcionalidades é que estão acessiveis a um utilizador sem este estar autenticado?

  Sem estar autenticado, um utilizador pode fazer um teste à velocidade da network e ping um host.

- Das funcionalidade que identificaste e do feedback que tiveste da sua utilização, pensa como é que estas podem estar implementatadas no servidor. Será que estão a utilizar algum utilitário linux?

  Sim, o comando ping deve utilizar um utilitário linux.

- Se sim, que vulnerabilidades podem estar presentes na chamada deste utilitário?

  Algumas das variáveis possíveis são:
  
  - Injeção de comandos: Se a caixa de input não for adequadamente validada ou filtrada, um invasor pode inserir caracteres especiais ou sequências maliciosas que podem ser interpretadas como comandos pelo sistema operativo. Isso permitiria que o invasor execute comandos arbitrários no servidor.

  - Acesso não autorizado: Se a caixa de input permitir que o invasor especifique caminhos de arquivo arbitrários, ele poderia tentar acessar arquivos do sistema que não deveriam ser acessíveis, expondo informações sensíveis ou explorando vulnerabilidades de segurança.

- Verifica se existe alguma vulnerabilidade nesta funcionalidade.

  Tentando explorar o ping tentámos pesquisar por 'google.com' o que resultou no resultado apresentado na imagem seguinte:
  
  <img width="539" alt="image" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/e8247d65-25c8-4d27-8558-20614f68eefa">
 
  A impressão feita é muito parecida com a de um terminal pelo que tentámos o comando '; ls'. O resultado está na próxima imagem:
 
  <img width="542" alt="image" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/a5354343-c8e5-4da2-8136-1ea4b2dc0687">
  
  Como o comando funciona podemos concluir que o input não é sanitized.

- Identificada a vulnerabilidade, utiliza-a para aceder à flag que se encontra no ficheiro /flags/flag.txt.

  Utilizando um input como o que seria usado num terminal testamos então o seguinte código para obter a flag:
  
      ; cat /flags/flag.txt ; cd /flags; cat flag.txt
  
  <img width="532" alt="image" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/b87a559b-dbdc-4678-b531-bd640427ce6e">
  
  <img width="539" alt="image" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/31e65625-96ec-4cb8-a96d-71c190b0b528">

  Tal como verificamos na última imagem podemos verificar que a flag é a seguinte: flag{74332bdc6d95fb88e8b18fa424d051e0}.
