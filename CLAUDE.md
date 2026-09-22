# Minado 11 — campo minado em 11 fases

Jogo **single-file** (`index.html`) da suíte pessoal. PWA instalável, **100% offline**
(sem Supabase, sem login) — o progresso fica no `localStorage` do aparelho.

- **Live:** https://dkonrad88.github.io/minado/ (GitHub Pages, publica no push da `main`)
- **Repo:** `dKonrad88/minado` — trabalhar sempre no `index.html`
- **Na Biblioteca:** entrada no array `APPS` de `dKonrad88/apps` (`index.html`)
- **Idioma:** PT-BR. Confirmar decisões de produto com o Diego, mas **commit + push é autônomo**.
- É jogo **só pra diversão**, como o caça-níquel e o River Raid — não é pra vender.

## O jogo

11 fases em dificuldade crescente (5×5 → 11×14), cada uma vale até 3 ⭐ pelo tempo.
Desbloqueio sequencial, 33 ⭐ por dificuldade.

**4 dificuldades** (`DIFS`), escolhidas na faixa no topo do mapa. Cada uma tem **vidas**,
multiplicador de minas, ajuste de dicas e de tempo-par — e **progresso próprio**
(`st.prog[dif] = {max, est, rec}`; `P()` devolve o da atual):

| | vidas | minas | dicas | par |
|---|---|---|---|---|
| 🍃 Brisa | 7 | ×0,75 | +1 | ×1,6 |
| 🎯 Na Medida | 5 | ×1,00 | — | ×1,0 |
| 🥶 Suor Frio | 4 | ×1,20 | — | ×0,8 |
| 💀 Sem Volta | 3 | ×1,40 | −1 | ×0,65 |

`minasDe()` põe um teto de **33% do campo** e nunca passa de `total-9` (senão a 1ª jogada
segura fica impossível). Na fase 11 isso dá 29 / 38 / 46 / 50 minas.

**Vidas são da JORNADA inteira**, não da fase (regra dada pelo Diego em 20/09/2026 —
não voltar a recarregar por fase). Ficam em `P().vidas`, salvas a cada bomba.

- Pisar numa mina custa 1 ❤️ e **não acaba o jogo**: a casa vira 💥 travada (`c.boom`, com
  `fl=1`, então já conta como marcada no contador e no chord) e a partida segue.
- Vencer uma fase **não devolve vida**. Chegar a 0 ❤️ ainda deixa jogar — é a última chance.
- **A bomba seguinte a 0 ❤️ encerra**: com 5 vidas, o fim vem na 6ª bomba (`pisar()` testa
  `p.vidas<=0` ANTES de descontar; mexer nessa ordem quebra a regra que ele pediu).
- Fim de jogo = `perder()` marca `acabou=true`, guarda `faseMorte` e chama `novaJornada()`
  na hora: `max=0` (as 11 trancam) e coração cheio. `abrirFase` força a fase 1 enquanto
  `acabou` estiver ligado, senão o ↻ durante a animação escaparia do castigo.
  **As estrelas e os recordes ficam** — são a marca pessoal, não o avanço.
- Zerar as 11 fecha a jornada: devolve as vidas e **mantém tudo destravado** (pra poder rejogar).

**Volume** (`st.vol`, 0-3: mudo/baixo/normal/alto, `VOLS[]`): o ganho de cada bip é multiplicado
por 1.3–4.2 e limitado a .62. ⚠️ No iOS o AudioContext nasce **suspenso** — `acordarSom()` roda
em todo `pointerdown` (capture) com `resume()` + buffer silencioso; sem isso o jogo fica mudo no
iPhone, que foi o que aconteceu.

**Contador de jogadas** (`jogadas`): chip 👆 no placar, contado em `cavar`/`apostar`/dica,
mostrado no fim da fase e somado nas estatísticas.

**Metas de ⭐ sempre à vista**: a linha `#metas` abaixo do placar mostra `⭐⭐⭐ até 1:50 ·
⭐⭐ até 3:40 · depois ⭐` (ou o teto de 2 ⭐ quando gastou dica), e o chip do relógio muda de
cor conforme a estrela vigente — ouro dentro do par, prata até o dobro, bronze depois.

**Campo sem chute** (`gerarJusto`): sorteia e roda `resolvivel()` — um solver que simula um
jogador que só deduz (regras diretas + subconjunto para o 1-2-1 + contagem global de minas).
Campo que exigiria adivinhação é descartado; até 6000 sorteios ou 450ms, e aí segue o último.
Taxa medida: **Brisa e Na Medida 100%**, Suor Frio ~87%, Sem Volta ~75% (densidade de 32% é
dura demais). Campos crus sem esse filtro: só 28% são justos.
⚠️ `gerar()` **precisa limpar `m`/`n` no início** — é chamado centenas de vezes seguidas.

**Vida extra**: 3 fases seguidas sem pisar em bomba dão +1 ❤️ (`P().limpas`, zera ao pisar).

**Partida em andamento** (`CHAVE_P`): `salvarPartida()` roda dentro de `atualizarHud()` e no
`visibilitychange`. O card grande de "Continuar" foi removido em 22/09/2026 (ocupava espaço):
a fase em andamento ganha a classe `.curso` no próprio card do mapa, com `▸ tempo`, e tocar nela
retoma. `vencer()`/`perder()` limpam.

⚠️ **Safe area só no `#app`** (v2.2): no celular ele é `position:fixed; inset:0` — não depende do
`100dvh`, que no PWA do iPhone reportava menos do que a tela — e o `padding: env(...)` fica só nele.
Nenhum filho pode somar `env(safe-area-inset-bottom)` de novo, senão a margem de baixo entra duas
vezes e sobra uma faixa morta embaixo (foi o que aconteceu no iPhone do Diego).

**Visual (v2.1, "liquid glass")**: a classe `.vidro` (blur + saturate + brilho na borda via `::after`)
vai **só onde a Apple põe vidro** — segmented de dificuldade, chips do placar, barra de ações,
rodapé do mapa, banner e sheets. **Nunca nas células nem nos cards de fase**, que são conteúdo.
A barra de ações e o rodapé do mapa são `position:absolute` e o conteúdo rola por baixo deles.
⚠️ Por isso `ajustar()` **subtrai os paddings** de `wrap` (clientHeight os inclui, e o de baixo é
o espaço reservado da barra) — sem isso a fase 11 fica escondida atrás dos botões.

**Regras de cenário** (`FASES[i].regra` + `aviso`, mostrado num banner na entrada):
`mare` (Praia, 45s, abre as casas seguras da coluna mais à esquerda), `gelo` (Geleira, 13s,
casa vazia volta a parecer fechada — só visual, `c.ab` continua true e o toque só descongela),
`orbita` (bordas ligadas: `viz()` faz wrap — o solver herda isso de graça) e `rei` (Covil:
uma mina custa 2 ❤️). As outras 7 fases ficam limpas de propósito.

**Ambiente animado** (`tema.amb`): 4-14 partículas em CSS (`cai`/`sobe`/`flutua`/`pisca`),
opacidade ~.42, com `prefers-reduced-motion` respeitado.

**Estatísticas** (`st.sta` via `S()`): fases, jornadas, bombas, dicas, tempo, vidas ganhas e
a fase que mais mata. Não zeram com a jornada.

**Tema por fase** (`FASES[i].tema`): `aplicarTema()` troca as variáveis CSS `--tampa/--tampa2/
--aberta`, o fundo da tela do jogo (`ceu` + `luz`) e enche o `#cenario` com 5 emojis
(`deco`) a 13% de opacidade nos cantos. O card do mapa herda o mesmo tom. Regra: **levemente**
temático — `ab` (casa aberta) tem que continuar escuro ou os números perdem contraste.

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
Subir `VERSAO` no topo do `<script>` a cada mudança publicada (hoje: 2.2).

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
