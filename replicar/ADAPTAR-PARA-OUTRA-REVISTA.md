# Adaptação para outra revista

O código de interface, gráficos e cálculos pode servir como ponto de partida para outras revistas. Ele reflete um fluxo editorial específico: reutilizá-lo exige compatibilizar os campos e revisar o significado dos indicadores. O pacote entregue é uma demonstração independente, sem conexão às fontes da revista original.

## O que está incluído

`dist/metrics.js` reúne as regras e denominadores; `dist/app.js` monta a página e aplica filtros; `dist/charts.js` e `dist/authorship-charts.js` desenham gráficos; `dist/br-uf-paths.js` contém a malha simplificada do mapa. `dist/demo-data.js` contém apenas registros fictícios gerados por `scripts/gerar_dados_ficticios.py`.

O arquivo portátil `Abrir-demonstracao.html` incorpora todo o conteúdo da demonstração. Ele não lê Excel ou Google Sheets. O importador incluído em `adaptacao/` converte uma planilha em JSON para apoiar uma nova integração; sua saída não substitui automaticamente a base da demonstração.

## Estrutura da planilha

Use `modelos/Modelo-Artigos-FICTICIO.xlsx`. O importador lê somente a aba **Artigos**, com os cabeçalhos na linha 1 e um artigo por linha. A ordem das colunas pode mudar: a correspondência é pelo cabeçalho. Espaços repetidos, asteriscos e acentos são normalizados para localizar as colunas. Preserve os nomes do modelo sempre que possível; a lista exata consumida pelo importador está no dicionário `HEADERS` de `adaptacao/importador_artigos.py`.

Os cabeçalhos consumidos devem existir mesmo quando várias células estão vazias. As três colunas opcionais são **EIC**, **Dias para a primeira decisão** e **Revisores com emissão**. Colunas adicionais, como os convites individuais aos editores e os nomes de autores, são ignoradas pelo importador atual.

| Grupo | Campos consumidos |
|---|---|
| Identificação | Código; Título; Tipo de artigo; EIC |
| Submissão | Data da submissão; Data da adequação |
| Autoria | Primeiro autor é mulher?; Algum autor é mulher?; Instituição do 1o. autor (sigla); UF da instituição do 1o. autor |
| Desfecho | Status do momento; Decisão editorial final; Data da decisão editorial final |
| Publicação | Data do envio à Zeppelini; Data da publicação na SciELO |
| Revisão | Algum EA já indicou revisores?; Data dos primeiros convites aos revisores; Data de conclusão da etapa de revisores (1a. rodada); No total, quantos revisores já foram convidados?; Quantos revisores já aceitaram?; Revisores com emissão |
| Primeira decisão | Dias para a primeira decisão |

O cabeçalho alternativo **EA já indicou revisores?** também é aceito. A coluna **Revisores com emissão** é importada e preservada, mas não possui um indicador próprio nesta versão.

O código deve ser único. O prefixo `AAAA-` define o ano da coorte; na ausência dele, o importador usa o ano da submissão ou o ano passado explicitamente com `--ano`. Cada arquivo deve conter uma única coorte. Duplicidades são preservadas e sinalizadas para conferência, não removidas silenciosamente.

Datas devem ser células de data ou texto `dd/mm/aaaa` / `aaaa-mm-dd`. Fórmulas precisam ter resultado calculado e salvo no arquivo Excel: o importador lê o valor armazenado. Contagens aceitam inteiros não negativos; zero é um valor informado e vazio é ausência. Dias para a primeira decisão aceita número não negativo. As classificações de autoria e indicação de revisores usam **Sim**, **Não** ou vazio.

## Categorias editoriais e cálculos

As decisões finais reconhecidas são **Aprovado**, **Recusa pelos ECs sem tramitação**, **Recusa pelo EA sem tramitação**, **Recusa após revisão**, **Arquivado** e **Ainda está em análise**. Se outra revista usar categorias diferentes, adapte explicitamente as constantes e regras em `metrics.js`.

- **Arquivados:** status ou decisão `Arquivado` exclui o artigo das proporções, médias e análises de autoria e instituição. O total recebido, o card de arquivados e a comparação anual continuam mostrando esses registros.
- **Marco efetivo:** data da adequação, quando preenchida; caso contrário, data da submissão. Ausências e intervalos inválidos não viram zero.
- **Em análise:** status preenchido diferente de `Arquivado`, `Concluído recusado` e `Concluído publicado`, após excluir arquivados. Essa regra pode incluir artigos já aprovados em produção; por isso existe o quadro de conferência de sobreposições.
- **Resultados anuais:** cada artigo é contado uma vez, priorizando decisão final e arquivamento. Os cards, por sua vez, preservam seus critérios individuais.
- **Aprovação por UF/instituição:** aprovados divididos pelos artigos com decisão final da mesma categoria, isto é, aprovados + recusados. Os sem decisão e os arquivados não entram nessa taxa.
- **Aprovação por grupo de autoria:** aprovados do grupo divididos por todos os artigos do mesmo grupo, sem arquivados, inclusive os ainda sem decisão. Esse denominador é diferente do utilizado por UF/instituição.
- **Classificação de autoria:** neste modelo, “Não” em primeira autora mulher representa primeiro autor homem; “Não” em alguma autora mulher representa autoria somente masculina. “Sim” na segunda pergunta não permite saber se há algum homem. A classificação e a coleta devem ser definidas pela revista que adaptar o painel.
- **Prazos:** dias corridos e apenas intervalos válidos. Submissão–publicação e aprovação–publicação usam aprovados; primeiros convites–conclusão da revisão representa a primeira rodada e inclui espera pelo aceite.
- **Tipos de artigo:** os filtros de prazos são independentes por quadro/figura, com padrão `Número regular`; `Todos` inclui tipos ausentes. Adapte o padrão se a revista não usar essa categoria.
- **Editores:** configure `EICS` e `ACTIVE_EICS` em `metrics.js`. As filas usam decisão `Ainda está em análise`. Nesta demonstração há somente quatro nomes fictícios. Não há associação automática de grafias diferentes ou responsabilidade compartilhada.
- **Mapa:** volume absoluto de submissões, excluídos arquivados, em cada UF; não é uma taxa por população. Países e valores não reconhecidos como UF ficam fora do desenho e são informados ao lado. Instituições são agrupadas pelo texto informado, sem conciliação automática de siglas.

### Alertas presentes

Cada artigo aparece uma vez, acumulando todos os motivos aplicáveis. Os prazos são avaliados até a data de referência selecionada; datas ausentes, inválidas ou futuras impedem avaliar o critério correspondente.

| Situação atual | Condição |
|---|---|
| Aguardando EC — etapa 1 / primeira análise | Pelo menos 14 dias desde o marco efetivo |
| Aguardando EA — etapa 1 / definindo revisores | Pelo menos 21 dias desde o marco efetivo |
| Aguardando revisores — etapa 1 / 1ª rodada | Pelo menos 46 dias desde os primeiros convites |
| Aguardando EA — etapa 2 / já com pareceres | Pelo menos 10 dias desde a conclusão da 1ª rodada |
| Aguardando revisores — etapa 1 / 1ª rodada | EA indicou revisores = Sim; pelo menos 14 dias desde os primeiros convites; aceites = zero ou em branco |

A lista separada de 120 dias usa o total desde o marco efetivo, entre artigos em análise. As grafias aceitas dos status estão em `ALERT_STAGE_ALIASES` e `stageStatus`, no código. Etapas posteriores não são tratadas como primeira rodada.

## Experimentar o importador

Com Python 3 e a dependência `openpyxl` instalada conforme `adaptacao/requirements.txt`, execute na pasta do pacote:

```sh
python3 adaptacao/importador_artigos.py modelos/Modelo-Artigos-FICTICIO.xlsx --ano 2026 --saida exemplo-convertido.json
```

O comando lê somente o modelo e cria um JSON separado. Recusa sobrescrever uma saída existente. O JSON possui `records`, `sources`, `schemaVersion` e configurações; cada registro contém os campos normalizados e observações em `issues`. O código de importação é independente e não acessa Google nem servidores.

## Uma implantação real é um projeto separado

Para usar dados confidenciais de outra revista, prepare uma instância própria, com identidade visual, autenticação, conexão às suas planilhas e revisão das regras. O backend deve ler as fontes com permissões próprias e fornecer os dados somente a usuários autorizados. A implementação dessa conexão e da hospedagem não faz parte deste pacote local.

O HTML portátil contém a base completa e pode ser inspecionado por quem o receber. Ele é adequado para esta base fictícia; não acrescente dados confidenciais a uma cópia destinada a divulgação. Uma senha desenhada apenas no JavaScript não protege os dados incorporados ao HTML.

O script é compartilhável como ponto de partida técnico, acompanhado desta ressalva sobre estrutura e critérios. Para publicação aberta ou redistribuição formal, o responsável deve escolher a licença do código e verificar os direitos dos elementos de marca. A demonstração não concede uso da marca epidemio como identidade de outra revista.
