# Resumo da análise inicial do dataset UFPR-ALPR

## Contexto e objetivo

O dataset está armazenado em `UFPR-ALPR dataset/`. A análise inicial foi realizada no notebook `src/analise_dataset.ipynb`, para entender a organização dos arquivos e apoiar o desenvolvimento de um pipeline de localização e reconhecimento de placas.

O objetivo do sistema é receber uma imagem e retornar o texto da placa. A abordagem inicial utiliza um detector, um recorte feito por código a partir da imagem original e um reconhecedor de caracteres.

## Informações declaradas pelo dataset

Segundo o `README.txt` que acompanha os dados:

- Há 4.500 imagens de 150 veículos, capturadas com o veículo e a câmera em movimento.
- As imagens são PNG, com resolução de 1920 × 1080 pixels.
- Foram utilizadas três câmeras: GoPro Hero4 Silver, Huawei P9 Lite e iPhone 7 Plus.
- As anotações incluem informações do veículo, texto da placa, quatro cantos da placa e posições dos caracteres.

Essas informações vêm da documentação; a resolução, as câmeras e todos os campos não foram auditados em cada arquivo.

## Organização e contagens verificadas

Os arquivos estão separados em `training`, `validation` e `testing`. Cada divisão contém subpastas de tracks, com imagens `.png` e anotações `.txt` de mesmo nome-base.

| Divisão | Imagens | Anotações | Tracks | Textos de placas distintos |
|---|---:|---:|---:|---:|
| Treino | 1.800 | 1.800 | 60 | 60 |
| Validação | 900 | 900 | 30 | 30 |
| Teste | 1.800 | 1.800 | 60 | 60 |

Foram encontradas 4.500 imagens e 4.500 anotações. Não foram encontrados pares ausentes ao comparar o caminho relativo e o nome-base dos arquivos dentro de cada divisão.

Esse pareamento verifica a existência dos arquivos correspondentes, mas não garante que o conteúdo de cada anotação descreva corretamente a imagem.

Há 30 imagens por track em média. Não foi verificado se todos os tracks têm exatamente essa quantidade, nem se cada track corresponde a uma única placa.

## Inspeção de uma amostra

Foi lida a anotação de `training/track0001/track0001[01].txt`. Foram extraídos o texto `AXV8804` e os quatro cantos da placa:

```text
(912, 526), (986, 531), (983, 553), (912, 550)
```

O contorno foi desenhado sobre a imagem correspondente e a inspeção visual foi considerada adequada. Essa conferência se restringiu a uma amostra e não valida todas as anotações do dataset.

## Placas compartilhadas entre divisões

A comparação dos textos anotados, removendo apenas espaços nas extremidades, apresentou:

| Comparação | Quantidade de placas compartilhadas |
|---|---:|
| Treino × validação | 0 |
| Treino × teste | 2 |
| Validação × teste | 0 |

As duas placas compartilhadas entre treino e teste aparecem nestes tracks:

| Placa | Track no treino | Imagens no treino | Track no teste | Imagens no teste |
|---|---|---:|---|---:|
| AVA1466 | track0011 | 30 | track0140 | 30 |
| BBO8514 | track0059 | 30 | track0110 | 30 |

Esse resultado é um sinal de possível sobreposição de veículos entre treino e teste. Não comprova imagens duplicadas, continuidade de captura ou erro de anotação. A comparação visual desses tracks e uma busca por duplicatas não foram concluídas nesta análise.

Se os mesmos veículos estiverem presentes nas duas divisões, a avaliação pode ser menos representativa do desempenho em veículos nunca vistos. Essa limitação deve ser considerada ao interpretar resultados futuros.

## Decisões e limites desta etapa

- Encerrar a análise exploratória inicial neste ponto e registrar os achados.
- Manter os arquivos e as divisões originais, sem remover ou mover os tracks compartilhados.
- Não considerar o dataset completamente validado: não foram realizadas auditorias completas de legibilidade das imagens, geometria das anotações, tamanhos das placas, distribuição dos caracteres ou duplicatas.
- Tratar a semelhança entre imagens como uma observação inicial; sua intensidade não foi quantificada.
- O dataset adicional com placas Mercosul, solicitado ao autor, ainda não foi analisado. Sua suficiência e eventual combinação com estes dados permanecem em aberto.

As verificações realizadas permitem descrever a estrutura e registrar uma limitação concreta na separação dos dados. Elas não determinam, por si só, a qualidade final ou a suficiência do dataset para o sistema pretendido.
