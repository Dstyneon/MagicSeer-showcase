# MagicSeer — Reconhecimento e Precificação de Cartas de Magic: The Gathering

**Demo ao vivo:** https://deployvercel-sable.vercel.app

Aplicação web que identifica cartas de *Magic: The Gathering* pela câmera do celular e mostra o
valor de mercado na hora, em português. Em produção, com usuários reais (teste inicial: 57
leituras em 8 aparelhos diferentes, 100% de acerto).

> O código-fonte deste projeto é privado. Este repositório existe para apresentar o projeto —
> arquitetura, decisões técnicas e resultados — com screenshots e a demo publicada. Se quiser ver
> trechos de código específicos, me chame.

![Tela inicial do MagicSeer](home.jpg)

## O que o app faz

- Aponta a câmera (ou tira uma foto) e o app reconhece a carta sozinho, mostra nome, edição e
  o menor preço encontrado no mercado brasileiro.
- Modo "Bullet": reconhece várias cartas em sequência, uma atrás da outra.
- Guarda coleções, calcula valor total e exporta a lista.
- Contador de vida completo pra partidas.

## Como foi construído

**Reconhecimento por imagem** — pipeline híbrido:
- **CLIP** (embeddings semânticos, PyTorch) sobre um índice próprio de **69 mil artes** de cartas.
- **ORB** (verificação por características locais) como conferência cruzada do resultado do CLIP.
- OCR de reserva rodando em paralelo, pra quando a imagem não é boa o bastante pro reconhecimento
  visual.
- Detecção de contorno, retificação por homografia (endireita a carta na imagem) e correção de
  exposição separada pra CLIP e pra ORB.
- Suporte a layouts especiais do jogo: cartas saga, class, split, borderless, dupla face.

**Precificação para o mercado brasileiro** — os preços de referência (Scryfall/TCGplayer) são
internacionais; o app converte pro menor preço real praticado no Brasil usando um modelo de
*gradient boosting*, calibrado e avaliado com mais de 3.000 preços reais coletados à mão. O modelo
também considera efeito de liquidez, popularidade da carta, tipo de coleção e idade da impressão.

**Base de preços própria** — cerca de 1.400 cartas e mais de 7.000 edições, com mínimo, médio e
máximo (normal e foil), alimentada automaticamente por cada busca de usuário e por uma recoleta
semanal agendada, guardando o histórico pra análise de série temporal.

**Coleta e integração de dados** — web scraping resiliente (navegação automatizada com contorno de
anti-bot, mais um caminho rápido por parsing de JSON embutido quando disponível), integração com a
API pública do Scryfall (110 mil cartas), e *fuzzy matching* pra casar nomes de edições em
português com os nomes oficiais em inglês.

**Operação 24/7** — um *watchdog* próprio monitora os serviços, reinicia o que cair, troca a URL do
túnel (Cloudflare) quando necessário e republica o site sozinho, sem intervenção manual. Logging
estruturado foi usado em diagnósticos reais, incluindo mais de uma análise de causa raiz feita
direto em log de produção.

**Qualidade** — harness de regressão com mais de 1.000 casos reais: nenhuma mudança no
reconhecimento ou na precificação sobe sem ser comparada contra o resultado anterior ("nada sobe
sem comparar limpo"). Toda mudança relevante é validada por um experimento medido antes do deploy.

## Stack

Python (reconhecimento, precificação, coleta de dados) · PyTorch / CLIP (open_clip) · OpenCV ·
scikit-learn (gradient boosting) · FastAPI · Scryfall API · web scraping automatizado ·
HTML/CSS/JS (frontend, captura de câmera no navegador) · Vercel (deploy) · Cloudflare Tunnel.

## Desenvolvimento assistido por IA

Projeto construído em parceria com agentes de IA (Claude Code, Anthropic) no ciclo completo:
planejamento e execução de rodadas de melhoria, automação de coletas e experimentos — inclusive
execução autônoma noturna supervisionada —, análise de logs de produção, calibração de modelos,
recuperação de incidentes e documentação contínua. A prática usada foi dirigir a IA com critérios
de aceite mensuráveis (o harness de regressão, avaliação fora da amostra) e auditar os resultados
antes de cada publicação, em vez de aceitar o que a IA entrega sem checar.

---

Contato: [Antonio Naranjo](https://www.linkedin.com/in/antoniojnc) · totyjnc@gmail.com
