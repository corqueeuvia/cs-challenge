# :page_facing_up: relatório do código
### tudo sobre o processo de criação :brain:

:dart: **a meta:** criar uma `função` que retorne o id do funcionário que atende o maior número de clientes

:scroll: **as regras:**
* todos os CSs têm níveis diferentes
* não há limite de clientes por CS
* clientes podem ficar sem serem atendidos
* clientes podem ter o mesmo tamanho
* 0 < n < 1.000
* 0 < m < 1.000.000
* 0 < id do cs < 1.000
* 0 < id do cliente < 1.000.000
* 0 < nível do cs < 10.000
* 0 < tamanho do cliente < 100.000
* valor máximo de t = n/2 arredondado para baixo

## :gear: o processo
### `filtrando os CSs disponíveis:` 
comecei tirando de cena os funcionários de folga para designar clientes apenas aos disponíveis.

tive problemas usando o `for` porque o return encerrava o laço, mas não pude criar a variável fora e dar o `.push()` dentro do for porque o `.filter()` precisava de um retorno verdadeiro ou falso

[o resultado foi esse](https://jsfiddle.net/y09pf426/)

enviei para meus amigos e um deles me mostrou como resolver usando `.find()` no lugar do `for`:

`if (!csAway.find(element => element === employee.id)) return true;`

### `designando clientes aos CSs:`
criei uma propriedade `clients` para cada CS disponível recebendo um `array` vazio, onde eu poderia inserir os ids dos clientes que ele fosse atender para, no final, contar quantos clientes cada CS tem

consegui fazer a verificação de `scoreCS >= scoreCliente`, mas o mesmo cliente estava sendo designado para mais de um CS

[o resultado foi esse](https://jsfiddle.net/1sj2xz9o/)

mais uma vez contei com a ajuda do meu amigo, mas não chegamos a um bom resultado.. só depois de uma boa noite de sono eu consegui, com o *insight* que ele havia me passado: criar uma cópia da lista de clientes e trabalhar com ela na comparação

depois de cada "match", fui alterando o score do cliente que já havia sido designado para um número **maior** do que o maior score possível dos meus funcionários (100.000), assim ele não satisfaria a condição novamente para outro funcionário.

*spoiler alert: um micro deslize nessa etapa gerou um erro grave depois, que levei um certo tempo para diagnosticar!*

[o resultado foi esse](https://jsfiddle.net/qsL738xh/)

### `contando os clientes dos CSs:`

usei um `.reduce()` na minha lista de funcionários para chegar àquele com maior número de clientes designados, mas comecei a ter problemas quando usava as funções geradoras de dados, só passava no cenário 1

[o resultado era esse](https://jsfiddle.net/jkob5hmc/)

pensei que o problema poderia ser como as funções retornavam os dados, então testei todas elas e analisei seus retornos com amostras iguais as dos cenários de teste. não era esse o problema.

pensei que o problema poderia ser o `if else if` encadeado dentro do `reduce`, mas não encontrei nada que validasse essa ideia. fiz também testes trocando por um `if`ternário e, apesar de não ter mais o mesmo erro, não conseguia satisfazer a condição em que dois funcionários tivessem o maior número de clientes (para retornar 0).

evoquei artilharia pesada: pedi ajuda do meu professor de JS, que em 10 minutos enxergou o problema: o tipo de dado que eu estava retornando no id dentro do `reduce`! 

por isso meu erro era `can't read length poperty of null`! como pode ver na [linha 34](https://jsfiddle.net/jkob5hmc/), que retorna 0 ao invés de um objeto.

depois disso funcionou! estava conseguindo contar meus clientes, mas ainda não estava tendo os resultados esperados pelo programa...

### `debugando os cenários`
a essa altura eu estava com erro em 2 cenários: 3 e 7.

o cenário 3 é covardemente imenso, então comecei pelo 7.

recebia o retorno de que o funcionário 1 tinha mais clientes, sendo que o esperado seria o funcionário 3.

com o `console.log()` eu vi quais clientes estavam sendo designados a cada funcionário e, designando manualmente para comparar percebi que havia clientes do 1 que deveriam estar sendo designados ao 3. passando um breve teste de mesa no trecho de designar clientes do código entendi: faltou organizar meus funcionários disponíveis em ordem crescente de score.

sem isso os funcionários designados não eram necessariamente os...
> "de capacidade de atendimento mais próxima (maior) ao tamanho do cliente"

...como sugeria o enunciado.

um `.sort()` resolveu meus problemas!

mas gerou erro em mais 2 cenários.

muito *debug* depois, segundos antes do meu último neurônio derreter, analisando os relatórios de erro do terminal eu percebi que a designação de clientes não estava correta em todos os cenários.

eu citei um mini deslize ali em cima, né?

`customersCopy[j].score = 100.000;`

esta inocente linha 57 (naquela versão) atrapalhava a comparação dos scores por motivos de:

[olha esse console.log!](https://jsfiddle.net/hszn6tar/)

corrigido um ponto (literal) faltava só um ponto: cenário 5

### `o cenário 5`
finalmente tomei um tempo para olhar para o erro do cenário 5, que tem um grande volume de dados...

a primeira coisa que percebi foi que todos os 10.000 clientes estavam sendo designados a um único funcionário, então fui investigar o motivo.

analisando as funções geradoras vi que tinha apenas clientes de score 998, enquanto meus funcionários tinham o score igual ao seu id, indo de 1 a 999.

a princípio achei que meus clientes deveriam estar divididos entre o 998 e o 999, mas vi que da forma como minha lógica de atribuição está estruturada, os clientes são designados ao funcionário de menor score possível entre aqueles de score maior que o deles. ou seja: o 998 realmente pegaria todos os 10 mil clientes.

fui verificar o que o teste esperava de mim e ele esperava um retorno 999, que é id do funcionário que está de folga!

fiquei confuso e perguntei para a carolina se era um erro do teste ou uma pegadinha que eu não estava enxergando, mas ela obviamente me disse que esse tipo de reposta seria injusta com outras pessoas concorrendo à vaga, então voltei a analisar o enunciado e tentar resolver o erro deste último cenário.

#### falha minha, falha nossa?
> eu poderia estar errado e ter entendido mal a parte de abstenções
> talvez o enunciado só tivesse mencionado os funcionários de folga para que não fosse feita nenhuma designação SOMENTE no caso de mais da metade deles estar de folga, mas que para fins de análise do balanceamento de CS os funcionários de folga pudessem ser inclusos na designação, apenas a fim de saber qual nível de CS está sendo mais requisitado
> nope. removi a parte da função que cria `availableEmployees` e mantive apenas a parte que os ordena crescentemente em função do score, mas isso gerou erro nos cenários 1 e 3. e no cenário 1 é muito fácil observar o *output* desejado, então descartei essa hipótese
> 
> **os funcionários de folga realmente deveriam ser desconsiderados da atribuição**
>
> enfim, entrego meu desafio concluído sem satisfazer o cenário 3, porém, gostaria muito de saber se o teste está realmente esperando um reasultado impossível e, caso não, onde está a falha da minha lógica ou interpretação que possibilita fazer da forma correta

## agradecimentos
* sempre à minha noiva, [sarah](https://www.linkedin.com/in/sarahnani/), em primeiro lugar: por estar ao meu lado e ser todo o suporte que preciso nas minhas empreitadas e por conversar sobre meus códigos para trazer uma nova perspectiva
* depois à [carolina](https://www.linkedin.com/in/carolinasilvagc/) da rd, que tratou do meu caso atípico com muita atenção e sempre disposta a ajudar
* também ao meu amigo, [rodrigo](https://www.linkedin.com/in/rodrigozaum/), citado no relatório: um ser humano GIGANTE de humanidade, com uma inteligência ainda maior!
*  ao meu professor [fábio](https://www.linkedin.com/in/fabio1990henrique/), que em 6 meses me tirou de zero JS até este ponto em que me encontro e está sempre disposto a debugar e ajudar quem quer que tenha curiosidade o suficiente pra encarar algo que não sabe ainda

