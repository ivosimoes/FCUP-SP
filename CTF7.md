# Desafio 1

  Para estes desafios, vamos explorar programas com vulnerabilidades String Format. Neste primeiro é nos apresentado um programa que começa por declarar uma variável global que irá conter a flag quando a função 'load_flag()' for chamada na main. A seguir, a main irá criar um buffer de 32 bytes que será preenchido pelo input do user e será imprimido pelo printf, onde se encontra uma vulnerabilidade.

  Normalmente, quando queremos imprimir variáveis com o printf, temos que especificar o formato da variável (%d, %s, %f, %x, etc.). Quando não especificamos estes formatos, o programa continua a verificar o que imprimir, apesar do código fonte não ter lá o formato da variável. Ou seja, no caso deste desafio, mesmo não tendo especificado o formato da variável a imprimir, podemos colocar esses formatadores no input e fazer com que o programa imprima valores que não deveria imprimir.
  
  Outro aspeto importante que necessitamos ter em conta é o local em memória onde está guardada a informação que está a ser imprimida. Isto é importante para os formatadores strings (%s). Quando usamos um formatador string no printf, o programa espera um endereço onde possa ir buscar o conteúdo da variável, ao contrário, por exemplo, do %d que vai buscar o próximo valor que encontrar na stack do tipo inteiro decimal.
  
  Com os conhecimentos que adquirimos sobre o programa, vamos tentar chegar à flag. Eis os passos que seguimos para chegar lá:

  1. Descobrir onde a flag está guardada. Como a flag foi guardada numa variável global, o seu endereço varia um bocado de uma variável normal. Para tal, vamos recorrer ao GDB, colocando um breakpoint na main e recorrendo ao comando "print &flag" para saber o endereço da flag:
  
  ![imagem](https://user-images.githubusercontent.com/126570489/233655099-8db529f5-0a71-4c10-b204-3d865f516551.png)
  ![imagem](https://user-images.githubusercontent.com/126570489/233655212-876f348a-033b-42f0-b829-c5e969973f58.png)
  ![imagem](https://user-images.githubusercontent.com/126570489/233655830-da766979-daab-40f8-91f6-178736ce9e9e.png)

  2. Com o endereço da flag (0x0804c060), vamos tentar arranjar maneira de fornecer este endereço a um formatador string de modo a ele imprimir o valor que reside nesse local da memória. Para isso, vamos descobrir quando é que o programa começa a ler o conteúdo do buffer (visto que é o único lugar de memória onde conseguimos escrever):

  ![imagem](https://user-images.githubusercontent.com/126570489/233662781-4fb65298-92c8-4e94-981b-e5a9e427f9b3.png)

  Pela imagem conseguimos ver que o programa vai de imediato buscar o conteúdo que reside no início da string; o conteúdo que estávamos à espera que fosse imprimido pelo %x ao colocar "AAAA" no início do buffer é precisamente a codificação de cada 'A' em hexadecimal (0x41) repetida 4 vezes (0x41414141).
  
  3. Sabendo que o conteúdo no início do buffer é a primeira coisa que o programa lê ao usar um formatador, podemos colocar o endereço da flag no início do buffer de modo a %s ler este endereço e imprimir o conteúdo que lá se encontra. Uma maneira de fazer isto é, mais uma vez, com o python2 (o processo já foi explicado previamente no CTF5 e no LOGBOOK7 na Tarefa 2.A):

  ![imagem](https://user-images.githubusercontent.com/126570489/233665208-c3cc9d69-2e1d-4e12-9f5e-fb2413136927.png)
  
# Desafio 2

  O segundo desafio difere ligeiramente do primeiro; em vez de escrever e aceder um endereço, queremos modificar o valor lá guardado. Um formatador que nos pode ajudar neste desafio é o %n. O %n funciona de modo semelhante ao %s: temos que fornecer um endereço como argumento, mas, em vez de imprimir o conteúdo, este formatador guarda o número de carateres no input.
  
  Porém o %n não é suficiente. Neste desafio continuamos a ter um buffer de 32 bytes e, para passar na condição que dá acesso à flag, precisamos de escrever o equivalente a 0xbeef em número de carateres (que se traduz para 48879 em decimal). O problema é que o buffer não tem tamanho suficiente para armazenar 48879 carateres e, como tal, não podemos simplesmente escrever milhares de carateres no input. Uma alternativa é usar um método semelhante no LOGBOOK7 na tarefa 3.A, que consiste em escrever o número de carateres que necessitamos no meio de um formatador (e.g. %48879x).
  
  Recorrendo a estas técnicas, vamos tentar chegar à flag, descrevendo o caminho passo a passo:
  
  1. Tal como no desafio anterior, primeiro queremos saber onde se localiza a key que vamos modificar. Como a key também é uma variável global, vamos recorrer ao GDB:

  ![imagem](https://user-images.githubusercontent.com/126570489/233672758-bc300db4-1da3-48ba-85d7-fcda0eaf3d3b.png)
  ![imagem](https://user-images.githubusercontent.com/126570489/233672827-3ffe2459-cf54-4bde-832f-51df235db0ba.png)
  ![imagem](https://user-images.githubusercontent.com/126570489/233672899-3bf3d246-5823-4931-92c7-a03bf544c06d.png)

  2. Ao contrário do desafio anterior, em vez de nos dirigirmos logo para o programa, vamos tentar descobrir qual o valor quantos espaços temos que adicionar para a key ficar com o valor 0xbeef. O motivo de fazermos isto já foi referido no LOGBOOK7 na tarefa 3.A: apesar de termos feito uma mera estimativa do valor de carateres a imprimir no output, a quantidade de espaços não coincide com esta estimativa, havendo necessidade de fazer uns acertos. Para tal, vamos permanecer no GDB, usando um segundo terminal para criar e editar a payload:

  ![imagem](https://user-images.githubusercontent.com/126570489/233682154-7f662537-6b85-40c5-9bb0-0bed368605b9.png)
  ![imagem](https://user-images.githubusercontent.com/126570489/233682277-b1985045-f6e1-416a-a1fc-102b0dcebcb3.png)
  
  2.1 Inicialmente, vamos tentar criar uma payload com 48879 espaços. É expectável que este valor não esteja correto devido à presença dos 4 A's inicialmente (para submeter colocar o formatador %x que nos permite criar os espaços) e do endereço do valor que temos que editar. Se tivermos em conta os A's e o endereço, teremos que adicionar 48871 (menos 8 espaços correspondentes aos 8 bytes já presentes) para chegar ao valor pretendido:
  
  ![imagem](https://user-images.githubusercontent.com/126570489/233683093-40be2574-36b4-47ec-9c3a-5faa2f4dc312.png)
  ![imagem](https://user-images.githubusercontent.com/126570489/233682693-36f0d7ca-b663-445d-89a7-5052fc8a8965.png)
  
  2.2 A primeira tentativa confirma as nossas previsões: o valor é, de facto, 8 unidades mais alto do que o suposto. Vamos agora tentar com o número de espaços 8 unidades menor:
  
  ![imagem](https://user-images.githubusercontent.com/126570489/233683333-979acf7b-5a10-45a0-84a0-b19a4b9b8d44.png)
  ![imagem](https://user-images.githubusercontent.com/126570489/233682830-7d29d980-6893-4e87-9a3f-9c779d779e1c.png)
  
  3. Agora que temos a payload certa, vamos testá-la no servidor:
  
  ![imagem](https://user-images.githubusercontent.com/126570489/233688554-6baad801-6525-4491-8e2f-97b7693edf08.png)
  ![imagem](https://user-images.githubusercontent.com/126570489/233688637-c3d91037-0031-4e0f-8ffd-ed245df4b4f8.png)

  Parece que conseguimos modificar a key com sucesso, só que, em vez de obtermos uma flag, fomos redirecionados para um terminal shell.

  4. Para extrair a flag do terminal shell, precisamos de interagir com o terminal de modo a vermos o conteúdo de flag.txt. Até agora tivemos a fornecer o input através de um ficheiro que criámos com o python2, mas, quando fazemos isso, o programa vai buscar todo o seu input ao ficheiro, não aceitando outro input. Como precisamos de fornecer input à função scanf e ao terminal, temos que inserir o input de modo a que o programa aceite a payload para o scanf e o resto para o terminal. Devido a não podermos simplesmente copiar e colar o input devido à payload, vamos recorrer a uma opção do 'cat' que nos permite inserir input mesmo após chegar ao fim do ficheiro:

  ![imagem](https://user-images.githubusercontent.com/126570489/233708773-e49501f6-2ae4-4f01-bda2-d0f42d382a09.png)
  ![imagem](https://user-images.githubusercontent.com/126570489/233708862-b4f6e7bb-70c5-4a54-be24-444b1038c4ed.png)

  (Nota: a técnica do cat foi encontrada em https://security.stackexchange.com/questions/73878/program-exiting-after-executing-int-0x80-instruction-when-running-shellcode/120501#120501)
