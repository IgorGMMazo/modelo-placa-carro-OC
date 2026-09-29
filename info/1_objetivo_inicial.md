# Objetivo Inicial do Projeto

 - Desenvolver e avaliar um pipeline que receba uma imagem e retorne a sequência de caracteres da placa. A abordagem inicial terá _localização_ da placa seguida de _reconhecimento_ de texto. O principal critério de sucesso será a proporção de placas transcritas integralmente de forma correta.

## Por que essa abordagem foi escolhida?

 - Reconhecer a foto por inteiro não é errado e separar em etapas não garante um melhor resultado, entretanto, a modularização de um problema é uma boa forma de isolar problemas e futuros diagnósticos. Sem contar no fator de que por ser um projeto do CEIA e atuar na área de pesquisa a ideia central é o aprendizado.

 - Mas tecnicamente falando se usarmos a inferência sem essa dupla etapa, o modelo que antes seria para inferir caracteres se torna um modelo que além disso , também aprendeu a identificar onde ler e o que ler, podendo tirar um pouco de sua especificidade. Sem contar que a abordagem dupla nos dá mais possibilidades. Se nós mudarmos o tamanho da imagem para porcessá-las , as placas também diminuirão o que afetaria o treino , nessa dupla etapa podemos diminuir para identificar e depois com as coordenadas coletadas , pegar da imagem original a placa para treino.