# Decisão inicial: divisão dos dados e estratégia dos modelos

Data: 29/09/2026

Status: estratégia inicial aprovada, sujeita a revisão com base nos resultados de validação.

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
