

# HERANÇA ROUBADA — Stealth Horror 3D

##  Sobre o Projeto

Este projeto consiste no desenvolvimento de um **Jogo 3D de Stealth Horror** intitulado **"Herança Roubada"**, construído inteiramente do zero utilizando **WebGL 2.0 Puro** (computação gráfica de baixo nível) sem o auxílio de motores ou bibliotecas de alto nível (como *Three.js* ou *Babylon.js*). A única dependência externa para cálculos de matrizes é a biblioteca auxiliar `gl-matrix.js`.

O trabalho atende integralmente a todos os critérios exigidos na segunda avaliação parcial da disciplina de **Computação Gráfica**, englobando transformações lineares, projeção perspectiva, sistema de iluminação matemática avançado (Reflexão de Phong e Spotlight), texturização procedural, e um *parser* customizado de arquivos vetoriais gráficos no formato OBJ e materiais MTL.

---

##  Equipe e Contexto Acadêmico

* **Instituição:** Universidade Estadual do Ceará (UECE)
* **Disciplina:** Computação Gráfica
* **Professor:** Matheus
* **Trabalho:** Trabalho 2 — Projeto 3D (Avaliação Parcial 2)
* **Desenvolvedor:** Hyan
* **Tester / Garantia de Qualidade:** Yasmin

---

##  A Lore do Jogo & Loop de Gameplay

### O Enredo
Um velho ganancioso e sem escrúpulos roubou a herança legítima da sua família e a trancou em um cofre blindado nos fundos de sua residência. Sabendo disso, você decide invadir a propriedade dele estrategicamente durante a calada da noite para recuperar o que é seu por direito.

### O Objetivo
Você deve se infiltrar na casa, explorar os cômodos em busca de **3 pistas textuais** espalhadas que revelam, por meio de charadas matemáticas, a combinação exata de uma **senha numérica de 3 dígitos**. Após descriptografar as pistas, você deve abrir o cofre, coletar as **3 pilhas de ouro** da sua herança — transportando uma por uma — e guardá-las em segurança no porta-malas do seu **carro de fuga** (um Chevrolet Camaro) estacionado na rua.

### Condições de Derrota e Vitória
1. **Flagrante (O Despertar do Velho):** O dono da casa está dormindo no quarto principal. Caso você faça muito barulho dentro da casa (barra de ruído atinja **100%**) ou erre a senha do cofre **3 vezes consecutivas**, o velho acordará e começará a caçá-lo implacavelmente utilizando um sistema de navegação e colisão tridimensional. Se ele te tocar, é Game Over.
2. **Cercado (Amanhecer):** Você tem exatamente **90 segundos** antes do sol nascer. Se o tempo se esgotar na rua sem ter completado a missão, ocorre uma transição cinematográfica de dia/noite e a patrulha policial local efetuará a sua prisão.
3. **Fuga de Sucesso:** Coletar as 3 pilhas e chegar ao carro antes do amanhecer ativa a cutscene de vitória, com o veículo acelerando rumo à liberdade.

---

##  Controles e Interação

O jogo utiliza um mapeamento de controles intuitivo no estilo de jogos em primeira pessoa (FPS) através da API nativa do navegador:

* **Movimentação da Câmera:** Movimento do mouse (utiliza a API `Pointer Lock` para prender o cursor e permitir rotação infinita de 360°).
* **Movimentação do Jogador:** Teclas `W` (Frente), `S` (Trás), `A` (Esquerda) e `D` (Direita).
* **Lanterna Tática:** Tecla `X` (Alterna o facho de luz da lanterna dinâmica para iluminar áreas escuras).
* **Ações e Coleta:** Tecla `E` (Utilizada para ler papéis com dicas, destravar a UI do cofre, coletar o ouro e depositar no carro).
* **Transição de Ambientes:** Tecla `F` (Utilizada para entrar/sair da casa pela porta principal e interagir com as portas internas dos cômodos).
* **Pular Cutscene Inicial:** Tecla `F` (Acelera a introdução do velho chegando na residência).
* **Pausar o Jogo:** Teclas `ESC` ou `P` (Abre o menu de pausa, bloqueia o tempo e libera o cursor do mouse).

---

## 🛠️ Requisitos Técnicos Implementados

Abaixo está o detalhamento científico e técnico de como as teorias de Computação Gráfica foram implementadas de maneira manual no código-fonte do sistema.

### A) Projeção Perspectiva e Câmera Vetorial
O motor gráfico implementa uma câmera em primeira pessoa real baseada em vetores tridimensionais (Posição, Direção e Cima):
* **Cálculo Vetorial de Direção:** Utiliza as coordenadas esféricas (*Yaw* e *Pitch*) mapeadas a partir do mouse para computar o vetor unitário frontal (`camFront`):
    $$\Delta X = \cos(	ext{yaw}) \cdot \cos(	ext{pitch})$$
    $$\Delta Y = \sin(	ext{pitch})$$
    $$\Delta Z = \sin(	ext{yaw}) \cdot \cos(	ext{pitch})$$
* **Matrizes Computadas:** As matrizes *View* e *Perspective* são geradas manualmente via algoritmos de álgebra linear computacional (`manualLookAt` e `manualPerspective`), cruzando vetores e controlando o campo de visão de 45°.

### B) Iluminação Híbrida (Phong e Spotlight Direcional)
Totalmente calculada no *Fragment Shader* (`fs`), englobando fontes globais e locais:
1. **Componente Ambiente e Difusa:** Calcula a dispersão de luz aplicando a Lei de Lambert via produto escalar entre a normal e a direção da luz.
2. **Componente Especular:** Simula brilho reflexivo de superfícies elevando o produto escalar do vetor de visão e reflexão a um fator de rugosidade ($32.0$).
3. **Spotlight Direcional (Lanterna):** O sistema calcula dinamicamente um cone de luz acoplado à câmera do jogador. A intensidade da luz ($I$) é gerada verificando o cosseno do ângulo ($	heta$) do fragmento em relação à direção da câmera, utilizando atenuação por distância e bordas suaves (*Smooth Edges*) controladas pelos ângulos internos (`cutOff`) e externos (`outerCutOff`):
    $$I = rac{	heta - 	ext{outerCutOff}}{	ext{cutOff} - 	ext{outerCutOff}}$$

### C) Transformações Geométricas e Direção Cinematográfica
O posicionamento de cada elemento utiliza matrizes homogêneas de translação, rotação e escala (`u_model`), expandidas com interpolações temporais:
* **Animação de Portas:** O ângulo interage progressivamente em direção a um `targetAngle`.
* **Animação Esquelética e Wave Functions:** A caminhada do policial e do velho utiliza funções senoidais baseadas no relógio de execução (`now`) para alternar a rotação dos braços e pernas de forma orgânica.
* **Cutscenes com Controle de Matriz:** A câmera é sequestrada do jogador e manipulada matematicamente, alterando *Yaw* e *Pitch* automaticamente. A cutscene de derrota por amanhecer calcula o arco do vetor de visão para focar na Lua, reduz a iluminação ambiente e rotaciona bruscamente para focar no Sol, revelando a chegada do Policial.

### D) Mapeamento Procedimental de Texturas e Cores Sólidas
Sem bibliotecas de alto nível, o jogo renderiza padrões via HTML5 Canvas 2D em memória e injeta os bytes brutos no WebGL:
* **Gerador de Padrões:** Texturas de asfalto, calçadas, troncos de árvores, folhas, porcelana, cama e grama são desenhadas no canvas e convertidas com mipmapping ativado.
* **Bifurcação de Rendering:** O Fragment Shader aceita desativar amostragem UV utilizando branches booleanas (`u_useSolid`), permitindo desenhar astros (como o Sol) com cores emissivas sólidas injetadas via `vec4`.

### E) Parser Assíncrono de Modelos 3D e Materiais (OBJ / MTL)
O motor lê strings cruas de arquivos `.obj` gerados externamente, reconstruindo geometrias e mapeamentos em tempo real:
* **Identificadores Analisados:** Processamento vetorial de vértices (`v`), texturas (`vt`), normais (`vn`) e triangulação de faces complexas (`f`).
* **Complexidade Multi-Buffer:** Diferente de leitores primitivos, este parser mapeia a tag `usemtl` para vincular automaticamente a texturas lidas de arquivos `.mtl` complementares, agrupando vértices em arrays distintos (VAOs e VBOs independentes) para que modelos complexos como o `vovo.obj` e o `Chevrolet_Camaro_SS_High.obj` sejam desenhados em múltiplas chamadas (`gl.drawArrays`), preservando cores e materiais de fábrica.

---

##  Arquitetura Estrutural do Código

O arquivo fonte principal está organizado de forma linear, seguindo a topologia de pipelines WebGL:
1. **Declaração de Shaders:** Compilação em tempo de execução dos Shaders (GLSL `300 es`).
2. **Mapeamento de Localização de Uniforms:** Captura das referências de memória na GPU das variáveis de matrizes, cones de luz (`u_flashlightOn`, `u_camFront`) e flags de renderização.
3. **Instanciação dos Motores de Parser:** Requisições assíncronas de arquivos vetoriais para os atores principais e o veículo de fuga.
4. **Level Design Avançado:** Array estruturado com caixas de colisão das paredes internas, expansão do quarteirão vizinho (Casas da Vila) e posicionamento das rodovias e vegetações.
5. **Loop Principal de Renderização (`render`):** Calcula o tempo delta, resolve físicas de colisão de eixo X e Z, gerencia a máquina de estados (`INTRO`, `JOGANDO`, `CUTSCENE`, `PAUSADO`) e submete os arrays de triângulos para a placa de vídeo.

---

##  Como Executar o Projeto

Devido às diretivas de segurança padrão do navegador (*Same-Origin Policy*), o carregamento de arquivos externos (`.obj` e `.mtl`) exige a execução através de um servidor HTTP local.

### Passos para Execução:
1. Certifique-se de que o arquivo principal `HerancaRoubada.html`, as texturas e os modelos tridimensionais (`vovo.obj`, `Chevrolet_Camaro_SS_High.obj`) e seus respectivos arquivos `.mtl` estejam na mesma pasta.
2. Abra o diretório no editor de código **VS Code**.
3. Instale a extensão **Live Server** (caso ainda não possua).
4. Clique com o botão direito no arquivo `.html` e selecione **"Open with Live Server"**.
5. O jogo iniciará automaticamente em seu navegador. Dê um clique na tela inicial para liberar a captura do ponteiro do mouse e habilitar o áudio/controles.
README.md
Exibindo README.md.