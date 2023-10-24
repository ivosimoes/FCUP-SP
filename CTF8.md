# Desafio 1

  Este primeiro desafio pede-nos para dar login na conta do admin sem saber a sua password. Uma técnica que podemos utilizar está presente e já foi explicada no LOGBOOK8:
  
   ![imagem](https://user-images.githubusercontent.com/126570489/235238272-dffd1358-8b52-4075-b3cf-ff7cc939b3c1.png)
   ![imagem](https://user-images.githubusercontent.com/126570489/235238333-8eea4ef7-0d47-455f-b8de-c176f434d654.png)

  Aparentemente, a string "' OR username='Admin' --'" resultou. Esta string foi feita com base na string presente na tarefa 2.1 no LOGBOOK8.
  
# Desafio 2

  Desta vez, é nos apresentado um desafio que é semelhante a um da semana 5, só que desta vez uma das medidas de segurança está ativa:
  
   ![imagem](https://user-images.githubusercontent.com/126570489/236283541-e58b32d8-9ac4-484c-9fc7-30154fa913ab.png)

  Na imagem acima vemos que o PIE está ligado. O PIE utiliza endereços aleatórios a cada execução do programa para guardar a sua stack, dificultando ataques que lidem com endereços. Como queremos realizar um ataque com shellcode, não podemos simplesmente escrever o endereço do buffer no return que encontrarmos numa das execuções do programa. Antes de prosseguir, vamos dar uma vista de olhos ao programa:
  
  ![imagem](https://user-images.githubusercontent.com/126570489/236284689-3735ec5c-0238-4978-a4a7-b2b93b5ddf5f.png)

  Mesmo o PIE estando ligado, é possível explorar uma vulnerabilidade de buffer overflow no programa quando o gets() é usado. É de notar que o endereço que o programa imprime já nos corta imenso trabalho como veremos a seguir. Vamos percorrer uma solução para este desafio passo a passo:
  
  1. Encontrar a distância entre o buffer e o return address. Este passo é fundamental num ataque de buffer overflow. Para executar o shellcode, necessitamos de o armazenar e de o executar, o que pode ser feito com um return, pelo que temos que descobrir onde este se situa na main:

   ![imagem](https://user-images.githubusercontent.com/126570489/236291363-a2b0d6b9-2740-451d-9b38-bd8266476625.png)

  Com a localização do buffer (0xffffd100) e do eip (0xffffd16c) descobrimos que a distância que procuramos é 0xffffd16c-0xfffffd100 = 0x6c = 108.
  
  2. Armazenar shellcode no buffer e preenchê-lo até chegar ao return. No ponto anterior descobrimos que temos de ocupar 108 bytes até chegar ao return address. 27 destes bytes (os iniciais) vão ser ocupados por shellcode, enquanto que os restantes 81 vão ser ocupados por lixo:

   ![imagem](https://user-images.githubusercontent.com/126570489/236295438-9be86187-1548-43df-b5d2-303eeccd2ded.png)

  3. Obter o endereço do buffer e colocá-lo no return. Se olharmos para o código do input ou para a execução do programa, veremos que o endereço do buffer é sempre imprimido a cada execução. Recorrendo a umas funções da biblioteca pwn, podemos extrair este endereço do output do programa:

  ![imagem](https://user-images.githubusercontent.com/126570489/236296472-d3ace08f-d791-4ed4-b4be-9ca44752d378.png)

  O segundo recvline extrai o endereço do buffer da segunda linha de output. A terceira instrução elimina o ponto final que ficou no endereço anteriormente. A quarta instrução converte o endereço em base 16 para um inteiro em base 10 para que este seja convertido para hexadecimal na quinta instrução, que junta o shellcode, o lixo e o endereço.
  
  Recorrendo ao método descrito, foi possível aceder à flag com sucesso:
  
   ![imagem](https://user-images.githubusercontent.com/126570489/236299533-f5005eeb-4437-4d94-a491-94aaf47d9712.png)
   
  Nota: A solução apresentada foi baseada numa solução fornecida em https://corruptedprotocol.medium.com/elf-x64-stack-buffer-overflow-pie-rootme-app-system-842bab495fb6
