# Minado — campo minado em 31 fases

Jogo **single-file** (`index.html`) da suíte pessoal. PWA instalável, **100% offline**
(sem Supabase, sem login) — o progresso fica no `localStorage` do aparelho.

- **Live:** https://dkonrad88.github.io/minado/ (GitHub Pages, publica no push da `main`)
- **Repo:** `dKonrad88/minado` — trabalhar sempre no `index.html`
- **Na Biblioteca:** entrada no array `APPS` de `dKonrad88/apps` (`index.html`)
- **Idioma:** PT-BR. Confirmar decisões de produto com o Diego, mas **commit + push é autônomo**.
- É jogo **só pra diversão**, como o caça-níquel e o River Raid — não é pra vender.

## O jogo

**31 fases** (`FASES`) numa jornada que sai do quintal e termina no espaço. A tabela é compacta:
cada fase aponta um **bioma** (`BIOMAS`: paleta + cenário + trilha), um número de minas, o par de
tempo e, quando tem, `forma`, `rochas`, `mel` e `regra`. `dicas` sai do tamanho do campo.

**Campos com forma** (`FORMAS`): a função da forma decide quais casas existem — as outras viram
`c.fora` (buraco invisível). Tem círculo, losango, anel, cruz, ampulheta, taça, onda, escada,
seta, tijolo e trevo. `viz()` **ignora buraco e rocha**, então contagem, flood, chord e solver
herdam a forma de graça. A área real fica em `dentro` (não use `cel.length`).

⚠️ **Cada fase tem cara e som próprios** (v3.2): o bioma é só o ponto de partida. Cada fase carrega
`h` (giro de matiz), `dl` (luz), `sf` (afinação/saturação) e o seu `deco`; `gira()` converte a
paleta pra HSL e desloca. Sem isso as 3 primeiras fases eram todas verdes com o mesmo ruído — foi
reclamação dele. Ao criar fase nova, **sempre** dar h/sf/deco diferentes das vizinhas.
Atenção ao sentido do giro: verde (~140°) vai pro amarelo com h **negativo**.

**As regras entram em cena** (v3.1): `bicho(emoji,casa,aoChegar)` faz o personagem voar até a casa,
agir e sair; `ondaColuna(x,aoPassar)` varre uma coluna. A galinha cisca, o fantasma leva o número,
o vulcão jorra lava e abre uma poça, o floco recongela, a maré passa como onda. Nada de banner seco.

**Obstáculos** são **temáticos e pontuais** — 10 fases das 31 têm, e o emoji vem do bioma
(`BIOMAS[b].bloco` / `.grude`): tronco na mata, coral no mar, porta no casarão, coluna nas ruínas,
engrenagem na fábrica… ⚠️ **A dificuldade não inventa obstáculo**: ela só engrossa o que a fase
já tem (`rochas: -1` no Brisa chega a zerar).

**Obstáculos**: `c.rocha` (sólida, não abre, não tem mina), `c.mel` (o 1º toque só limpa o mel)
e `c.premio` (presente: +1 💡 ao abrir). `espalharObstaculos()` sorteia e testa `conexo()` —
rocha nunca pode partir o campo em dois, senão o pedaço isolado exigiria adivinhação.

**Tela de fim de fase** (v3.4): `caixa()` aceita `num` (três caixinhas: tempo / jogadas / vidas)
e `tags` (etiquetas `bom`/`ruim`/`ouro`). Era um parágrafo de 4 linhas e virou painel — se for
acrescentar informação ali, vira etiqueta, não frase.

**Os 4 modos são de exército** (v3.5, pedido dele): 🎒 Treinamento · 🧭 Patrulha · 🛡️ Operação ·
⚡ Linha de Frente. Além de vida/mina/dica/tempo, cada um tem `abre` (anéis protegidos na 1ª
cavada: 2/1/1/0) e a dose de surpresas. Medido: a 1ª cavada abre 31 / 19 / 16 / 12 casas.
⚠️ **O primeiro toque nunca pode ganhar a fase** (v3.9): em campo pequeno o flood abria TODAS as
casas seguras de uma vez — 28% das partidas no Quintal/Treinamento. `gerarJusto()` rejeita sorteio
cujo `floodConta(seguro)` passe do **teto** da dificuldade (`teto:` .75/.60/.45/.35 das seguras,
piso de 11 e folga mínima de 4 casas); `minasDe()` ganhou **piso de 3 minas ou 9% da área**; e o
último recurso **mura um canto** (parede de minas em volta) para garantir casa fechada.
Medido em 960 partidas nas fases pequenas: **0 vitórias no primeiro clique**.
Medido em 124 partidas (31×4): abre 49/35/29/24% do campo e 100/94/87/74% sem chute.

⚠️ **Regra é raridade, não relógio** (v3.6): nada de `setInterval`. `talvezRegra()` roda **depois
de cada jogada sua** — 28% de chance, espera de 3-6 jogadas entre uma e outra, teto de 1-3 por
fase e **30% de chance de a fase não ter nenhuma**. Assim ficar parado não resolve o campo (era o
caso da galinha, que ciscava a cada 30s e terminava a fase sozinha).
⚠️ **Nada de efeito fora da fase**: `irPara()` limpa confete, bichos, onda e partículas, e o evento
sonoro só toca com a tela do jogo ativa — confete continuava caindo sobre o mapa.

**A regra da fase é sorteada a cada partida** (`POOL[bioma]`, a assinatura da fase pesa o dobro),
e o texto do aviso é montado por `FALA[regra](tema)` — por isso jogar o Quintal duas vezes não dá
a mesma coisa.

**Trilha = só fundo** (v3.8): melodia em loop foi REMOVIDA — ele achou enjoativa ("tipo piano").
Ficaram duas camadas: a **cama de ruído** (`som.t/f/q/lfo` + `tf` filtro, `prof` varredura, `tr/trp`
tremolo e `dr` fio grave por bioma) e o **bicho** ocasional. 31 texturas distintas.
⚠️ Não reintroduzir melodia contínua.

**Cada fase tem um bicho sonoro** (`tema.som.ev`): motivo curto que toca a cada 4-17s — pássaro,
galinha, abelha, coruja, gota, buzina… É o que faz perceber a diferença entre fases; só mudar a
frequência do ruído não bastava (reclamação dele).

**Surpresas** (`c.surp`, invisíveis até tocar; `BONS`/`RUINS`): presente (+1 💡), sopro (abre a
vizinhança), alarme (expõe uma mina) · grude (vira `mel=2`, que quebra **e abre** no toque
seguinte — `mel=1` é o grude visível, que só limpa), desabamento (a casa vira rocha, `dentro--`,
e **só acontece se `conexo()` continuar true**) e fumaça (oculta 6 números por 9s).
É o principal diferencial entre dificuldades hoje: Brisa 4 boas / 0 ruins · Na Medida 2/1 ·
Suor Frio 1/3 · Relâmpago 0/5.

**A dificuldade muda o terreno também**: Brisa dá 2 presentes; Suor Frio joga 4 rochas e 3 mel;
Relâmpago, 8 rochas e 6 mel. `areaDe()`/`minasDe()` já descontam isso.

**2 chefes** (`CHEFES=[15,30]`) ocupam duas colunas no mapa e fecham a grade (29×1 + 2×2 = 33).

Regras que fogem do campo minado clássico:

- **A primeira jogada nunca explode:** as minas só são sorteadas depois do 1º toque,
  excluindo a casa tocada e as 8 vizinhas (`gerar(seguro)`), então sempre abre uma clareira.
- **Dicas** (1 a 3 por fase) são um **detector apontado pelo jogador**: toca no chip 💡 e a casa
  que você escolher conta a verdade — abre se for segura, ou **expõe a bomba** (mesmo `c.show` do
  Revelar) se for mina. **Nunca explode nem custa vida.** Casa já aberta não gasta. Teto de 2 ⭐.
  ⚠️ Nunca usar a classe CSS `.conf` numa célula: ela é do **confete** (`position:absolute;top:-20px`)
  e joga a casa pra fora do tabuleiro. Esse bug chegou a ser publicado (v1.7).
- **Sem bandeiras.** O modo Certeza 🚩 foi removido em 22/09/2026 a pedido dele (junto com a
  dúvida amarela, que já tinha saído). Sobraram **dois modos: ⛏️ Cavar e 💣 Revelar**, e
  `c.fl` saiu do modelo. Mina "achada" = `c.show` (exposta) ou `c.boom` (pisada); é isso que
  o contador 💣 e o chord usam.
- **Toque longo (ou botão direito) faz o contrário do modo ligado** (`oposto()`): no Cavar ele
  aposta, no Revelar ele cava. É o atalho pra não ficar trocando de botão.
- **A fase acaba de dois jeitos**: abrir tudo que é seguro **ou achar todas as minas**
  (`contaAchadas()===minas`) — dá pra vencer só apostando, sem limpar o campo.
- **💣 Revelar** (`apostar()`): você aponta onde acha que tem bomba. Acertou, ela fica **exposta**
  (`c.show`, com `fl=1` — já conta no contador e no chord, e não dá pra mexer). Errou, a casa
  abre e custa **1 ❤️** (e zera a sequência de fases limpas). Antes do 1º toque não deixa apostar.
- **Chord:** tocar num número que já tem todas as bandeiras abre a vizinhança.
- **Derrota em 3 tempos:** mina pisada acende → minas surgem em ondas → pausa pra olhar
  → estouro geral (clarão + tremor). Um toque pula. No fim, "👀 Ver o campo" remonta
  o tabuleiro inteiro com todos os números.

## Estrutura

- `index.html` — o jogo inteiro (HTML/CSS/JS, sem build, sem CDN).
- `manifest.json` / `sw.js` — PWA. O SW é **network-first** na navegação (pega a versão
  nova quando online, abre offline pelo cache). Subir `CACHE='minado-vN'` força limpeza.
- `.nojekyll` — obrigatório na suíte (o build Jekyll já travou publicação por dias no `viagem`).
- Ícones gerados com PIL (grade 3×3 + bandeira vermelha).

## Ajustes dentro do jogo

Rodapé do mapa → **⚙️ Ajustes**: som/vibração, **🔄 Atualizar o jogo**, **↩️ Recomeçar a
jornada** (volta pro Quintal com o coração cheio, guardando estrelas) e **🗑️ Apagar tudo**
(as 4 dificuldades, como na primeira vez que abriu).

⚠️ As confirmações são um modal do próprio app (`confirmar()`), **nunca `confirm()`** —
o diálogo nativo é engolido sem aviso em PWA dentro de iframe, e foi por isso que o Diego
não conseguiu zerar o progresso no celular.
O Atualizar desregistra o service worker, apaga os caches e recarrega com `?v=<timestamp>` —
é o caminho pro Diego pegar no celular o que foi mudado aqui sem esperar cache.
Subir `VERSAO` no topo do `<script>` a cada mudança publicada (hoje: 4.2).

## ⚠️ Invariantes que a v4.0 firmou (não desfazer)

Vieram de uma varredura completa + testes em execução. Cada um já foi bug de verdade.

**A fase só acaba uma vez.** `checar()` começa com `if(!vivo)return` e compara com `>=`
(não `===`). `vencer()` e `perder()` começam com `if(!vivo)return` + `limparAgenda()`.
Sem isso, qualquer callback de raridade ainda na agenda chamava `vencer()` de novo e
contava tudo em dobro: vidas, `p.limpas`, `S().fases`, tempo e um segundo modal.
Motivo raiz: `vencer()` expõe todas as minas, então `contaAchadas()===minas` fica
**permanentemente verdadeiro** depois da primeira vitória.

**A partida salva tem que guardar a REGRA.** `viz()` muda de topologia conforme
`regra` (`orbita` faz a vizinhança dar a volta), e `retomar()` recalcula todos os `n`.
Salvar sem a regra fazia `abrirFase()` sortear outra e **renumerar o tabuleiro que o
jogador já tinha deduzido**. Por isso `abrirFase(i, regraFixa)` aceita a regra forçada —
tem que ser no argumento, não depois, senão o banner e o `regraTeto` saem errados.
O save também guarda `jogadas`, `gelo`, `oculto`, `mente`, `mel` (0/1/2), o **tipo** da
surpresa (índice em `SURPS`, ordem FIXA) e o orçamento de regra (`ru`/`re`/`rt`).
Tem `v:VERSAO` e `n:cel.length`; `retomar()` recusa save de tamanho diferente em vez de
montar uma fase sem mina.

**Quem mexe no campo depois de um `ag()` checa `vivo` e `gerado` de novo.** Vale para
todo callback de `bicho()`/`ondaColuna()`. Cada raridade devolve `true`/`false`, e
`talvezRegra()` só gasta o uso da fase quando ela **realmente** agiu — senão a maré
queimava o único uso da fase sem aparecer.

**`abertas` e `dentro` andam juntos.** O desabamento (`surp:'desaba'`) numa casa já
aberta pelo flood precisa de `abertas--` junto do `dentro--`; sem isso a vitória por
limpeza ficava impossível para sempre.

**`acabou` é por dificuldade** (`acabouDif`). Global, ele jogava o jogador na fase 1 de
uma dificuldade intacta.

**Voltar do segundo plano re-arma `agendarEvento()`.** `pararAmbiente()` mata `evTimer`
e só `abrirFase()` religava — o bicho da fase calava para sempre depois de uma
notificação no celular.

**Nada de `tickRegra`.** Foi removido na v4.0. A regra é sorteada nas jogadas
(`talvezRegra`), nunca por relógio.

## ⚠️ A origem é compartilhada com a suíte inteira (v4.2)

Todos os apps do Diego moram em `dkonrad88.github.io/<app>/` — **mesma origem**.
Logo `caches.keys()` e `navigator.serviceWorker.getRegistrations()` enxergam os
outros apps. O `activate` do sw.js apagava **todo** cache com nome diferente do seu,
e o botão Atualizar desregistrava **todo** service worker da origem: o Minado
derrubava o River Raid, o HUB, o Viagem… Hoje ambos filtram por `minado`.
Se copiar esse sw.js para outro app, trocar o prefixo do filtro junto com o CACHE.

## ⚠️ Outras travas da v4.2

- **Bomba pisada não cobra de novo.** `cavar()` tem que barrar `c.boom` — a casa
  fica `ab=false, show=false, boom=true`, então sem essa guarda dava para tocar na
  mesma mina e drenar a jornada inteira.
- **Botão de modal sempre fecha o modal.** `caixa()` embrulha o handler com
  `modal.classList.remove('on')`; `irPara()` não fecha o `#modal` (ele não é `.screen`),
  então "Voltar ao mapa" deixava o jogador preso.
- **`usouDica` é um booleano da fase**, não `dicasDe()-dicas>0`: um presente repondo a
  dica gasta apagava o teto de 2 ⭐. Vai no save (`ud`).
- **Fechar a jornada é uma vez só** (`p.fechou`, zerado em `novaJornada()`), senão
  rejogar a fase 31 farmava jornada e vidas. E a reposição usa `Math.max` — nunca
  tira vida de quem juntou prêmio.
- **O relógio para no segundo plano** (senão come as estrelas de quem atende o celular).

## Trilha (v4.1) — o bioma dá o timbre, a fase dá o resto

Ele reclamou **três vezes** que as fases soavam igual ("circo tem que soar como circo").
A causa era estrutural: o `som` era só do bioma e a fase só escalava a frequência do
filtro (`sf`) — 31 fases com **6 timbres**.

Hoje: `BIOMAS[x].som` define o timbre do bioma e `TRILHA[i]` (tabela nova, antes de
`FASES`) dá a **cada** fase o seu `ev` (o bicho, som ocasional do cenário) e um `s`
opcional que torce o caráter da cama. O `.map()` aplica `sf`, depois `Object.assign(som, tr.s)`.

`tocarAmbiente()` entende: `tf/f/q/lfo/prof` (camada de ruído filtrada), `n2` (2ª camada),
`acorde`+`tipoOsc`+`ag` (acorde PARADO), `bat` (desafinagem em cents → batimento),
`tr/trp/trt` (tremolo, com `trt:'square'` cortando seco para máquina).

⚠️ **`acorde` é cama, não melodia.** As notas nascem juntas e não mudam até a fase acabar.
Ele mandou tirar a melodia na v3.8 ("tipo piano, enjoativo") — nunca sequenciar notas na
cama. O `ev` é um motivo curto e esparso e ele aceitou; não crescer isso.

⚠️ **Como medir se mudou de verdade:** renderizar num `OfflineAudioContext` e comparar
*espectro* **e** *pulsação* (espectro do envelope). Só o espectro não serve — circo e
fábrica dão 99% de parecido nele e soam completamente diferentes. E **descartar os
primeiros ~3 s**: a rampa de entrada de 2,5 s domina a medição do envelope e faz tudo
parecer idêntico. Hoje: mediana 29%, nenhuma dupla do mesmo bioma acima de 88%.

## Como testar (workflow da suíte)

Não precisa de login → dá pra testar direto no browser.

1. Editar `index.html`.
2. Copiar pra `<scratchpad>/minado/` e servir com um `server.py` mínimo (o TCC bloqueia
   ler `~/Documents` pelo preview). Abrir no Browser pane, `resize_window` mobile.
3. **Teste de lógica via `javascript_tool`** (o que pegou bugs de verdade): rodar N partidas
   por fase conferindo nº de minas, contagem dos vizinhos, 1ª jogada segura e detecção de
   vitória; e o ciclo das marcas. Ver o console — foi assim que apareceu um `$('#btSom')`
   morto que derrubava o registro do service worker.
4. **commit + push na `main`** e **conferir o site publicado com `curl`/`gh api`**, não só o push.
