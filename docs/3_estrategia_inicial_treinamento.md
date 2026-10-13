# Decisão inicial: divisão dos dados e estratégia dos modelos

Data: 29/09/2026

Status: estratégia atualizada em 13/10/2026, após a análise de UFPR e RodoSol e confirmação de Igor. O treinamento anterior foi cancelado. A próxima etapa é preparar os dados e treinar primeiro o detector.

## Decisão de 13/10/2026: uso dos dois datasets

Manter as divisões oficiais de ambos os datasets: 40% treino, 20% validação e 40% teste. A alternativa 70/15/15 não foi adotada. Não redistribuir imagens, tracks ou placas nesta preparação.

| Origem | Treino | Validação | Teste | Total |
|---|---:|---:|---:|---:|
| UFPR | 1.800 | 900 | 1.800 | 4.500 |
| RodoSol | 8.000 | 4.000 | 8.000 | 20.000 |
| Ambos | 9.800 | 4.900 | 9.800 | 24.500 |

Comparar três experimentos do detector, todos iniciados dos mesmos pesos pré-treinados, sem continuar os pesos de um experimento no outro:

1. Treinar apenas com o treino UFPR.
2. Treinar apenas com o treino RodoSol.
3. Treinar com a união dos dois treinos, inicialmente na proporção natural dos dados.

Avaliar cada candidato nas duas validações, apresentando resultados separados por origem. A validação concatenada do YAML combinado é auxiliar: sua média não substitui a avaliação de cada dataset. Cerca de 82% das imagens de treino combinado vêm do RodoSol. Não adotar reamostragem 50/50 sem evidência experimental.

Manter comparáveis a configuração inicial e os critérios de seleção; registrar semente, versões, resolução, batch, épocas efetivas, passos de otimização e tempo. Mesmas épocas não equivalem a mesmo custo quando o tamanho dos dados difere. Confirmar os parâmetros e o critério de escolha do checkpoint antes de executar os treinos; esta decisão não fixa esses valores.

Usar a validação para escolher parâmetros e configurações. Reservar os testes para a avaliação final do protocolo definido, com resultados separados por dataset e pelos grupos pertinentes (tipo, layout anotado e tamanho de placa). Não considerar o resultado agregado suficiente para escolher o melhor sistema.

### Limitações conhecidas e avaliações complementares

- UFPR: AVA1466 e BBO8514 aparecem no treino e teste. Preservar o protocolo e apresentar também resultado sem os tracks de teste dessas placas, conforme detalhado adiante. Há um par de imagens idênticas dentro da validação (`track0090[14]` e `[15]`); declarar essa repetição e, na avaliação complementar, contar a imagem duplicada uma única vez.
- RodoSol: HDE1F59 aparece em três imagens de treino e três de validação. Preservar os arquivos oficiais, mas apresentar também validação sem essas três imagens e usar esse resultado complementar para orientar escolhas. O filtro é de avaliação, não uma alteração do dataset oficial. O YAML padrão continua apontando para a validação completa.
- Examinar candidatos a semelhança entre divisões antes do treinamento. Igualdade de dHash não confirma duplicação e não autoriza excluir imagens automaticamente.
- No RodoSol, algumas cenas contêm placas adicionais visíveis sem anotação. Ainda é necessário definir se o sistema deve encontrar a placa do veículo principal ou todas as placas visíveis; a conversão preserva somente o alvo anotado, sem inventar labels.
- A definição do cenário final de uso permanece em aberto. Uma mistura de datasets não garante generalização para uma câmera ou ambiente novo.

### Preparação YOLO aprovada

Usar dois notebooks independentes, sem módulos auxiliares ou relatórios adicionais:

- `src/preparacao_yolo_UFPR.ipynb`: preservar as divisões e tracks de `datasets_brutos/UFPR-ALPR dataset`; preparar `datasets_yolo/UFPR`.
- `src/preparacao_yolo_RodoSol.ipynb`: seguir exatamente `split.txt` de `datasets_brutos/tbFcZE-RodoSol-ALPR`; preparar `datasets_yolo/RodoSol`.

Cada dataset terá `images/<divisão>/<grupo>/...`, `labels/<divisão>/<grupo>/...` e `dataset.yaml`. O grupo é o track no UFPR e a pasta de tipo/layout no RodoSol. Manter imagens originais sem redimensionar e sem mover ou alterar os brutos. Usar dimensões reais, inclusive as 61 imagens UFPR com altura 1088.

Uma única classe, `0: placa`, representa todos os layouts e tipos de veículo. Converter os quatro cantos para a caixa alinhada aos eixos e normalizar centro, largura e altura pelas dimensões da imagem. O texto da placa não é alvo do detector. Não corrigir silenciosamente anotações inválidas: interromper e informar os arquivos envolvidos.

O notebook RodoSol também gera `datasets_yolo/combinado.yaml`, referenciando as pastas das duas origens, sem copiar novamente as imagens. Executar primeiro o notebook UFPR. Os YAMLs individuais e o combinado usam caminhos absolutos calculados a partir do projeto; ao mover o projeto, executar novamente os notebooks para atualizar esses caminhos.

Verificar pares, totais por divisão, correspondência das imagens copiadas e caixas YOLO contra as anotações originais. Mostrar exemplos visuais nos próprios notebooks. A preparação não executa treinamento nem altera os notebooks de análise.

## Objetivo

Construir um pipeline que receba uma imagem e retorne o texto da placa. A métrica principal será a proporção de placas transcritas integralmente de forma correta.

## Divisão dos dados

Manter inicialmente a divisão oficial do UFPR-ALPR:

| Divisão | Proporção | Imagens | Tracks | Finalidade |
|---|---:|---:|---:|---|
| Treino | 40% | 1.800 | 60 | Ajustar os pesos dos modelos que precisarem de treinamento |
| Validação | 20% | 900 | 30 | Comparar configurações, analisar erros e orientar decisões |
| Teste | 40% | 1.800 | 60 | Avaliar o pipeline ao final, após definir as configurações |

Essa divisão não é considerada ideal por princípio. A decisão é estabelecer primeiro uma referência com a organização existente, sem redistribuir os dados antes de medir o desempenho.

A disponibilidade de um OCR pré-treinado pode reduzir a necessidade de treinar o reconhecedor com este dataset, mas não elimina a necessidade de diversidade no treino do detector. O número de imagens também não equivale ao número de exemplos independentes: imagens de uma mesma sequência podem ser muito semelhantes.

Não utilizar os resultados do teste para escolher modelos, ajustar parâmetros ou decidir quando interromper o treinamento.

### Sobreposição conhecida entre treino e teste

A análise inicial encontrou duas placas compartilhadas:

| Placa | Track no treino | Track no teste |
|---|---|---|
| AVA1466 | track0011 | track0140 |
| BBO8514 | track0059 | track0110 |

Cada um desses tracks contém 30 imagens. A sobreposição foi identificada pelos textos anotados; não comprova imagens duplicadas nem continuidade entre capturas.

Preservar esses arquivos na divisão oficial e declarar a limitação. Na avaliação final, apresentar o resultado completo e os resultados separados para os tracks com placas compartilhadas e para os demais. A ausência de texto compartilhado, por si só, não comprova ausência de todo tipo de vazamento.

## Pipeline proposto

```text
Imagem original
    ↓
Detector YOLO26n ajustado para a classe placa
    ↓
Coordenadas da placa
    ↓
Recorte por código a partir da imagem original
    ↓
Reconhecedor de texto pré-treinado
    ↓
Texto da placa
```

O recorte é uma operação de código, não um terceiro modelo. Se a imagem for redimensionada para a detecção, as coordenadas deverão ser mapeadas para a imagem original antes do recorte.

### Detector

- Iniciar com YOLO26n pré-treinado e fazer ajuste fino para uma única classe: `placa`.
- Converter os quatro cantos anotados em uma caixa retangular usando os menores e maiores valores de `x` e `y`.
- Nesta abordagem inicial, o detector prevê uma caixa retangular; não prevê os quatro cantos nem realiza correção de perspectiva.
- Escolher resolução, tamanho do lote e demais parâmetros após verificar o ambiente e acompanhar a validação. Esses valores ainda não estão definidos.

YOLO26n é o ponto de partida, não um vencedor já demonstrado neste dataset. Resultados publicados em outros dados e equipamentos não garantem o mesmo desempenho nas nossas placas.

### Reconhecedor

- Avaliar inicialmente um reconhecedor pré-treinado disponível no PaddleOCR, sem ajuste em placas.
- A biblioteca foi escolhida como candidata; o modelo específico e sua versão ainda serão definidos.
- Usar primeiro recortes produzidos a partir das anotações de validação, para observar a leitura sem misturar erros de localização do detector.
- Medir o acerto da placa inteira e examinar os erros antes de decidir se é necessário ajuste fino.

Os dois modelos não precisam ser treinados com o mesmo dataset. O reconhecedor pode aproveitar aprendizado anterior em outros dados de texto, desde que o vocabulário e as características das imagens sejam adequados à tarefa.

Mesmo sem o carro e a paisagem, o recorte contém características próprias das placas: fonte, reflexos, parafusos, desfoque, perspectiva, resolução e possíveis layouts em duas linhas. Portanto, o treinamento genérico em texto não garante bom desempenho nesse domínio.

Se for necessário ajustar o reconhecedor, utilizar apenas dados de treino para atualizar seus pesos e a validação para orientar as escolhas. Dados externos ou sintéticos poderão ser considerados posteriormente, com origem, adequação e separação documentadas.

## Sequência de avaliação

1. Ajustar e avaliar o detector usando treino e validação.
2. Avaliar o OCR pré-treinado nos recortes da validação obtidos pelas anotações.
3. Decidir, com base nos erros de validação, se o reconhecedor precisa de ajuste específico.
4. Integrar os componentes e comparar a leitura dos recortes anotados com a dos recortes produzidos pelo detector.
5. Após definir o pipeline e suas configurações, realizar a avaliação final no teste e reportar a sobreposição conhecida.

A avaliação do pipeline completo deve contabilizar falhas de detecção; não deve considerar apenas as placas que o detector conseguiu encontrar.

Podemos ter dois modelos em execução e precisar treinar apenas o detector. A necessidade de treinar o reconhecedor será uma conclusão experimental.

## Critérios para revisar a decisão

- Revisar a estratégia do reconhecedor se o OCR pronto apresentar erros relevantes nos recortes anotados da validação.
- Investigar dados, resolução, recortes e configuração do detector se ele limitar o resultado do pipeline.
- Considerar uma nova divisão somente mediante justificativa experimental. Nesse caso, preservar tracks inteiros, agrupar tracks de mesma placa, registrar o manifesto e a semente da divisão e identificar os resultados como um novo protocolo, não diretamente comparável ao oficial.
- Reavaliar a estratégia quando o dataset Mercosul solicitado estiver disponível e tiver sido analisado.

Uma nova divisão que reutilize exemplos de um teste já avaliado não deve ser apresentada como um teste inteiramente intocado.

## Referências para consulta

- [Documentação oficial do YOLO26](https://docs.ultralytics.com/models/yolo26/)
- [Comparação oficial YOLO26 e YOLO11](https://docs.ultralytics.com/compare/yolo26-vs-yolo11/)
- [Módulo de reconhecimento de texto do PaddleOCR](https://paddlepaddle.github.io/PaddleOCR/main/en/version3.x/module_usage/text_recognition.html)

As comparações da Ultralytics são publicadas pelos próprios desenvolvedores. Servem como referência inicial, sem substituir a avaliação no nosso problema.
