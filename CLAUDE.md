# Guia do projeto: Tabuada em Cores (multiplicação de 2 a 10, para crianças de 7 a 10 anos)

Este arquivo resume as decisões, o funcionamento e as lições do jogo `index.html`, para que uma nova
conversa possa continuar este jogo. O jogo é irmão de `classes-gramaticais`: mesma arquitetura e
mesmas mecânicas, adaptadas para a tabuada.

## Objetivo do jogo
Fazer a criança **saber a tabuada de cor e rápido**: muitas repetições curtas, **uma cor fixa por
tabuada**, foco no que ela ainda não sabe e **fluência** (acertar rápido, não só acertar).
Público: 7 a 10 anos. Textos curtos e simples, em português do Brasil. Fatores de 1 a 10.

## Arquitetura (manter assim)
- **Um único arquivo `index.html`**, com HTML, CSS e JavaScript juntos, sem backend e sem dependências
  (só Google Fonts). Abre em qualquer navegador e pode ser hospedado de graça no GitHub Pages.
- **Sem IA durante o jogo.** As contas são calculadas, então não há resposta errada no banco. O que é
  escrito à mão são as **estratégias** (`TIP`, `strat()`, `SPECIAL`) e as **pegadinhas** (`CONF`).
- **Progresso salvo em `localStorage`**:
  - `multiplicacao.v1`: estado (nível, pontos, domínio, preferências);
  - `multiplicacao.hist.v1`: histórico completo de respostas.

## Conteúdo
- `FACTS`: 90 contas `a × b`, com a tabuada `a` de 2 a 10 e o multiplicador `b` de 1 a 10.
  Cada conta tem `id` (`7x8`, na ordem em que aparece) e `k` (par ordenado, `7x8` para 7×8 e 8×7).
  **O domínio usa `k`**: acertar 7×8 também conta para 8×7 (comutatividade).
- `TB`: cor fixa de cada tabuada (2 azul, 3 verde, 4 roxo, 5 laranja, 6 rosa, 7 vermelho, 8 amarelo,
  9 turquesa, 10 marrom; 1 cinza só na tabela). `color(a,b)` escurece o tom conforme o multiplicador.
- `strat(x)`: estratégia de 1 linha. Usa o fator mais fácil (ordem `EASE`: 1, 10, 2, 5, 9, 4, 3, 6, 8, 7)
  e deriva a conta das "âncoras" (×1, ×2, ×5, ×10): "vezes 6 é vezes 5 e mais uma vez: 35 + 7 = 42".
  Com `hide=true` (dica antes de responder), para no "= ?" e não mostra notas que entregam o resultado.
- `stratL(x)`: estratégia do modo "número que falta" (`a × ? = p`). Ensina a achar o **?**, e não a
  calcular `p`, que já está na tela: atalho pelas âncoras (×1, ×2, ×5, ×10, ou "fica entre 5 e 10"
  comparando com a × 5 e a × 10) e contagem de a em a com o número de pulos. Com `hide=true` não entrega
  o número. **Lição:** cada modo de pergunta precisa da sua própria explicação; reaproveitar a do
  resultado no modo inverso confunde.
- `SPECIAL`: notas para as contas que mais confundem (7×8 "5, 6, 7, 8", 6×8 e 7×7, 6×9 e 7×8…).
- `CONF`: pegadinhas por resultado (48↔49, 54↔56, 63↔64…). `optionsR()` usa, nesta ordem: `CONF`, as
  contas vizinhas (a×(b±1), (a±1)×b), a soma a+b, os algarismos trocados (não quando termina em 0: 30 → "03")
  e, por fim, outros produtos próximos. As opções ficam em ordem crescente, mas **a posição da certa é
  sorteada**: primeiro sorteia quantas pegadinhas ficam abaixo e quantas acima. **Lição:** com pegadinhas
  "vizinhas" e ordem crescente, a certa caía sempre no meio e a criança aprendia a posição, não a conta.
  Só o 10 × 10 fica sempre por último, porque não há produto acima de 100.
- `diagnose()` explica o erro: soma no lugar de vezes, conta vizinha ou algarismos trocados.

## Mecânicas (vindas do jogo de classes gramaticais)
1. **Cores fixas por tabuada.** A conta é pintada assim que é respondida, e a **tabela 10×10** ao lado
   (a "legenda") vai ficando mais colorida conforme o domínio. Só mostra o número das contas já
   dominadas (etapa 2+), e a conta atual aparece como "?" nos dois lados da diagonal.
2. **Rodada de 8 contas** (no lugar da "frase"). Ela sorteia uma tabuada principal (peso ×3) e completa
   com outras do nível. **Ordem recomendada: fáceis primeiro** (as que têm ×1, ×2, ×5 ou ×10), porque
   as difíceis se derivam delas. Também existe a ordem "misturada".
3. **Feedback instantâneo.** Acerto rápido avança em 0,45 s. Acerto lento mostra o tempo e avança em
   1,6 s. Erro mostra a conta certa, o diagnóstico e a estratégia; continua com Enter.
4. **Fluência (novidade).** Uma barra de tempo enche durante a pergunta. Os limites ficam em `LIMIT`:
   6 s para resultado e para o número que falta, 8 s para algarismos e escrita, ×0,75 nos níveis
   "Mestre". **Só acerto rápido e sem dica sobe no domínio.** Acerto lento conta para a precisão, mas
   não sobe nem zera o domínio.
5. **12 níveis**, com subida automática (85% nas últimas 20 respostas), descida (abaixo de 50%) e
   seleção manual:
   - 1–6, modo **R** (escolher o resultado): 2/5/10 com 4 opções → +3 e 4 com 6 → +6 com 8 →
     7/8/9 em foco com 8 → todas com 10 → pegadinhas (`TRICKY` ×4);
   - 7, modo **D** (algarismo por algarismo: dezena, depois unidade; é a "árvore de classificação" deste
     jogo, sempre com as 10 opções e a tecla igual ao algarismo);
   - 8, modo **L** (número que falta: `7 × ? = 56`, opções de 1 a 10, a tecla 0 vale 10);
   - 9, D e L sem cores; 10, Mestre (sem cores, sem dica, mais pegadinhas, menos tempo);
   - 11–12, **escrita** (WD/WL): digita o resultado, ou o número que falta, das contas que já domina
     (etapa 3+ no 11, 2+ no 12); o resto segue com opções.
6. **Domínio por etapas**: acertar rápido 5→3→2→1→1→1 vezes seguidas faz a conta ser pulada
   5→10→20→40→80→160 vezes. Um erro volta ao início. Mais de 5 dias sem responder faz a conta voltar.
   Cada modo (R, D, L) tem contagem própria (`R:3x7`, `D:3x7`, `L:3x7`); a escrita usa a do modo de base.
   Contas puladas aparecem já reveladas na rodada (até 2, com "★ já domina"). Se quase tudo estiver
   dominado, a rodada é completada mesmo assim (no mínimo 4 contas), marcadas como revisão.
7. **Histórico completo**: `[data, tabuada, multiplicador, modo, certo, resposta, acerto, nível, dica, ms,
   rápido]`. Estatísticas por tabuada, domínio por modo, "contas para revisar" e "Copiar CSV".
8. **Teclado: a criança digita o número da resposta**, não a posição do botão (os botões não têm número
   de posição). 10 = <kbd>1</kbd> <kbd>0</kbd>. Se o que foi digitado só pode ser uma opção, responde na
   hora; se ainda pode virar outra (digitou 1 e existe 10), mostra o 1 no "?" e espera o próximo
   algarismo, o Enter ou 1,2 s (`typeKey`). Essa espera não conta no tempo (`kTime`). Backspace apaga.
   Enter/Espaço continuam, H pede dica e L mostra ou esconde a tabela. **Lição:** responder pela posição
   confundia: para responder 10, a criança apertava 1 e o jogo marcava o primeiro botão.
9. **Proteção contra clique duplo**: ignora o segundo clique em menos de 320 ms e cliques em botões que
   acabaram de aparecer (por exemplo, o próximo algarismo). O teclado não sofre esse bloqueio.
10. **Dica**: elimina metade das opções erradas e mostra a estratégia sem o resultado. O acerto com
    dica vale metade dos pontos e não conta para o domínio.

## Como testar
- Playwright já vem instalado no ambiente de nuvem: `require('playwright')` funciona direto.
- O jogo expõe `window.__T = {strat, stratL, optionsR, checkFacts}` para os testes.
- Testes que valeram a pena:
  - para todas as 90 contas: a estratégia contém o resultado, toda conta escrita nela ("35 + 7 = 42",
    "7 × 8 = 56") está certa, `stratL` dá o número que falta (e a versão de dica não o entrega), e `optionsR` devolve n opções únicas entre 1 e 100 com a certa incluída;
  - jogar os 12 níveis pelo teclado, lendo a conta em `#qBig`, com um erro proposital por nível e sem
    erros no console;
  - semear o `localStorage`: tudo dominado (a rodada não pode ficar vazia), resposta lenta (não mexe no
    domínio) e escrita no nível 11 (etapa 3 em `D:`).
- Testar também: a posição da certa em `optionsR` é uniforme (≈ 1/n para n = 4, 6, 8, 10); digitar "1" e
  "0" responde 10; "1" sozinho espera e responde 1 sem contar a espera; dois algarismos no modo resultado.
- Cuidado com nomes de classe CSS: o espaço da resposta usa `.slot.fill`, porque `.done` já é o texto
  pequeno e cinza (o número respondido ficou minúsculo por causa disso).
- Nos testes, espaçar os cliques em mais de 320 ms, ou usar o teclado. Bloquear as Google Fonts no
  teste gera um erro de console esperado ("Failed to load resource").

## Publicação
- GitHub Pages: repositório público com `index.html` na raiz → Settings → Pages → branch `main`, pasta `/`.
- A sessão do Claude não consegue criar repositórios nem ativar o Pages: o usuário cria o repositório e
  liga o Pages. Para o Claude enviar código, o app do Claude no GitHub precisa ter acesso ao repositório
  (github.com/settings/installations → Claude → Configure).

## Ideias para depois
- Tabuada até 12, divisão como modo próprio (a partir do "número que falta"), modo "família de fatos"
  (3 × 7, 7 × 3, 21 ÷ 3 e 21 ÷ 7 juntos), sons opcionais de acerto e perfis para mais de uma criança no
  mesmo navegador.
