# ❄️ Snow Party - Capy NPC Guia com Inteligência Artificial (Groq/Llama 3.1)

Este repositório contém o sistema de inteligência artificial e pathfinding do NPC **Capy**, a capivara guia turística do jogo **Snow Party** no Roblox. O projeto une conversação contextualizada em tempo real com movimentação automatizada pelo mapa.

---

## 🔎 O que tem no projeto? (Spoilers & Recursos)

Se você veio dar uma espiada no código, aqui está o resumo das mecânicas principais implementadas:

* 🤖 **Cérebro Atualizado (Groq API):** Integração direta com o modelo `llama-3.1-8b-instant`, garantindo respostas ultra-rápidas em português para os jogadores.
* 🗺️ **Contexto Estrito (Sem Alucinações):** O NPC foi treinado via *system prompt* para conhecer **apenas** os locais reais do mapa (*Área da Piscina, Salão Principal, Capy Store, Área de Eventos e Spawn*), impedindo a IA de inventar lugares que não existem no jogo.
* 🚀 **Gatilho de Movimento Automático (O Grande Trunfo):** Quando um jogador pede para ser levado a um lugar, a IA injeta uma "tag oculta" na resposta (ex: `[IR_PARA:Piscina]`). O script do Roblox intercepta essa tag, apaga ela do balão de fala para o jogador não ver, e ativa o sistema de caminhada sozinho.
* 🛤️ **Pathfinding Avançado:** Uso do `PathfindingService` do Roblox para fazer o NPC desviar de obstáculos e pular plataformas de forma inteligente até chegar ao ponto turístico.
* 👥 **Proximidade Realista:** O NPC possui um limitador de distância de 15 *studs*. Ele não vai responder a jogadores que estiverem "gritando" do outro lado do mapa.

---

## 🛠️ Como o fluxo funciona por trás dos panos?

1. O Jogador digita no chat perto da Capy: *"Me leva pro seu lugar favorito?"*
2. O script detecta a proximidade e envia a mensagem para a API do Groq.
3. A IA processa e responde: *"A área da piscina é o meu lugar favorito! Vamos lá! [IR_PARA:Piscina]"*
4. O script limpa o texto para exibir apenas a frase fofa no balão azul.
5. O script lê a tag `Piscina`, calcula a rota física e faz o NPC andar até o `PiscinaPoint` no
