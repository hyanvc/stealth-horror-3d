# HERANÇA ROUBADA — Stealth Horror 3D

##  Sobre o Projeto

Este projeto consiste no desenvolvimento de um **Jogo 3D de Stealth Horror** intitulado **"Herança Roubada"**, construído inteiramente do zero utilizando **WebGL 2.0 Puro** (computação gráfica de baixo nível) sem o auxílio de motores ou bibliotecas de alto nível (como *Three.js* ou *Babylon.js*). A única dependência externa para cálculos de matrizes é a biblioteca auxiliar `gl-matrix.js`.

O trabalho atende integralmente a todos os critérios exigidos na segunda avaliação parcial da disciplina de **Computação Gráfica**, englobando transformações lineares, projeção perspectiva, sistema de iluminação matemática baseado no modelo de reflexão de Phong, texturização avançada e um *parser* customizado de arquivos vetoriais gráficos no formato OBJ.

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
Você deve se infiltrar na casa, explorar os cômodos em busca de **3 pistas textuais** espalhadas que revelam, por meio de charadas matemáticas, a combinação exata de uma **senha numérica de 3 dígitos**. Após descriptografar as pistas, você deve abrir o cofre, coletar as **3 pilhas de ouro** da sua herança — transportando uma por uma — e guardá-las em segurança no porta-malas do seu **carro de fuga** estacionado na rua.

### Condições de Derrota (Game Over)
1.  **Flagrante (O Despertar do Velho):** O dono da casa está dormindo no quarto principal. Caso você faça muito barulho dentro da casa (correndo sem cautela) e a barra de ruído atinja o limite máximo (**100%**), ou se você errar a senha do cofre por **3 vezes consecutivas**, o velho acordará em estado de alerta máximo e começará a caçá-lo implacavelmente. Se ele te tocar, você é pego em flagrante.
2.  **Cercado (Amanhecer):** Você tem exatamente **90 segundos** antes do sol nascer. Se o tempo se esgotar enquanto você estiver dentro da casa, o velho acorda. Se o tempo acabar e você estiver na rua sem ter completado a missão, o dia clareará e a patrulha policial local efetuará a sua prisão em flagrante.

---

##  Controles e Interação

O jogo utiliza um mapeamento de controles intuitivo no estilo de jogos em primeira pessoa (FPS) através da API nativa do navegador:

* **Movimentação da Câmera:** Movimento do mouse (utiliza a API `Pointer Lock` para prender o cursor na tela e permitir rotação infinita de 360°).
* **Movimentação do Jogador:** Teclas `W` (Frente), `S` (Trás), `A` (Esquerda) e `D` (Direita).
* **Ações e Coleta:** Tecla `E` (Utilizada para ler papéis com dicas, abrir a interface numérica do cofre, interagir para coletar as pilhas de ouro e depositá-las no carro).
* **Transição de Ambientes:** Tecla `F` (Utilizada para entrar ou sair da casa pela porta principal e para abrir/fechar as portas internas dos cômodos).
* **Pular Cutscene Inicial:** Tecla `F` (Durante a introdução cinematográfica do velho chegando em casa).
* **Pausar o Jogo:** Teclas `ESC` ou `P` (Abre o menu de pausa e libera o cursor do mouse).

---

##  Requisitos Técnicos Implementados

Abaixo está o detalhamento científico e técnico de como as teorias de Computação Gráfica foram implementadas de maneira manual no código-fonte do sistema.

### A) Projeção Perspectiva e Câmera Vetorial
O motor gráfico implementa uma câmera em primeira pessoa real baseada em vetores tridimensionais (Posição, Direção e Cima):
* **Cálculo Vetorial de Direção:** Utiliza as coordenadas esféricas (*Yaw* e *Pitch*) mapeadas a partir do deslocamento horizontal e vertical do mouse para computar o vetor unitário frontal (`camFront`) através das equações:
    $$\Delta X = \cos(	ext{yaw}) \cdot \cos(	ext{pitch})$$
    $$\Delta Y = \sin(	ext{pitch})$$
    $$\Delta Z = \sin(	ext{yaw}) \cdot \cos(	ext{pitch})$$
* **Matriz de Visualização (View Matrix):** Desenvolvida manualmente através da função `manualLookAt`, baseando-se no produto vetorial (*cross product*) entre a direção da câmera e o vetor de inclinação para derivar os eixos locais do espaço da câmera (U, V, N).
* **Matriz de Projeção Perspectiva:** Gerada manualmente via `manualPerspective`, mapeando o campo de visão (*Field of View* de 45°), a razão de aspecto da janela (*aspect ratio*) e os planos de corte próximo (*Near* a 0.1) e distante (*Far* a 100.0).

### B) Sistema de Iluminação Matemática (Modelo de Reflexão de Phong)
Implementado em sua totalidade dentro do *Fragment Shader* (`fs`), calculando a iluminação estática e dinâmica pixel a pixel:
1.  **Componente Ambiente:** Uma constante de iluminação básica (`u_ambientLvl`) que garante a visibilidade mínima do cenário escuro na calada da noite.
2.  **Componente Difusa:** Calcula a dispersão de luz na superfície rugosa utilizando a Lei dos Cossenos de Lambert, aplicando o produto escalar entre o vetor normal unitário do fragmento (`v_norm`) e a direção da fonte de luz (`lightDir`):
    $$I_{	ext{diff}} = \max( ec{N} \cdot  ec{L}, 0.0) \cdot 	ext{u\_lightCol}$$
3.  **Componente Especular:** Simula o brilho reflexivo de superfícies polidas (como o ouro e o cofre). Calcula o vetor de reflexão da luz (`reflect`) e realiza o produto escalar com a direção de visão do observador (`viewDir`), elevado a um coeficiente de rugosidade/brilho de valor `32.0`:
    $$I_{	ext{spec}} = 0.5 \cdot \max( ec{V} \cdot  ec{R}, 0.0)^{32} \cdot 	ext{u\_lightCol}$$
* **Fonte de Luz Dinâmica:** Quando o jogador se encontra no interior da residência, uma das fontes de luz oscila senoidalmente de forma contínua através da variável `luzZ = -20 + Math.cos(now / 1000) * 10`, alterando a direção das sombras tridimensionais em tempo real sobre os móveis.

### C) Transformações Geométricas e Animações 3D
O posicionamento de cada elemento do cenário é controlado por transformações afins tridimensionais encapsuladas na matriz de modelo (`u_model`):
* **Translação, Rotação e Escala:** Aplicadas em ordem estrita para evitar distorções espaciais utilizando álgebra linear de matrizes homogêneas $4 	imes 4$.
* **Animação Cinemática das Portas:** O ângulo das portas e do cofre interpola de maneira suave e progressiva em direção ao valor alvo (`targetAngle`) em cada quadro do laço de renderização, criando o efeito de abertura suave.
* **Animação Orgânica (Animação Esquelética via Matrizes):** O movimento de caminhada do velho e do policial utiliza funções trigonométricas de onda baseadas no tempo real (`now`) para transladar o torso e rotacionar os membros inferiores/superiores de forma alternada, simulando passadas realistas.

### D) Mapeamento de Texturas Avançado e Cores Sólidas
O pipeline de renderização suporta de forma híbrida texturas procedimentais complexas e rendering por coloração sólida:
* **Geração de Texturas Procedimentais:** Como não é permitido o uso de bibliotecas de imagem de alto nível, foi desenvolvido um gerador de texturas que renderiza padrões gráficos (azulejos com rejunte, tábuas de madeira serradas, asfalto com faixas viárias, tapetes ornamentados e folhagens aleatórias) em elementos temporários em memória via HTML5 Canvas 2D, convertendo-os em dados de pixel puros injetados nos buffers de textura do WebGL (`gl.texImage2D`) com suporte a mipmapping automático (`gl.generateMipmap`).
* **Renderização por Cores Sólidas:** O Fragment Shader possui ramificações lógicas controladas por variáveis uniformes (`u_useSolid` e `u_solidCol`). Isso permite desativar a amostragem de mapa de textura para desenhar astros celestes (como o Sol e a Lua com iluminação própria em tom emissivo) e geometrias poligonais de cores estáticas (como a viseira e o fardamento escuro do policial).

### E) Leitor Próprio de Arquivos de Modelos 3D (OBJ Parser)
Um dos maiores pilares de complexidade técnica do projeto é a função assíncrona `carregarModeloOBJ(url)` desenvolvida inteiramente pela equipe:
* **Análise de Fluxo de Texto (Parsing):** O motor lê strings cruas de arquivos `.obj` gerados no Blender, processando linha por linha.
* **Identificadores Identificados:**
    * `v `: Coordenadas espaciais tridimensionais dos vértices $[X, Y, Z]$.
    * `vt`: Coordenadas de mapeamento de textura bidimensional $[U, V]$, invertendo o eixo V para o padrão de leitura correto do WebGL.
    * `vn`: Vetores normais de iluminação $[X, Y, Z]$.
    * `f `: Faces poligonais. O parser reconstrói a geometria realizando a triangulação automática de faces poligonais complexas e mapeando os índices compostos (Vértice / Textura / Normal) para organizar arrays lineares compatíveis com buffers gráficos.
* **Gerenciamento de Materiais (MTL Parser):** O leitor interpreta arquivos de biblioteca de materiais vinculados (`.mtl`), processando tags de cor difusa e mapas de textura (`map_Kd`), gerando sub-buffers e vinculando múltiplos VAOs (*Vertex Array Objects*) para que um único arquivo `.obj` complexo seja desenhado com múltiplos materiais e texturas distintas sequencialmente.

---

##  Arquitetura Estrutural do Código

O arquivo fonte principal está estruturado de forma linear e limpa em JavaScript estruturado, seguindo a lógica estrita das APIs gráficas modernas:

1.  **Inicialização de Contexto:** Captura o elemento `<canvas>` e inicializa o pipeline de execução do `webgl2`.
2.  **Declaração de Shaders:** Compilação em tempo de execução dos Shaders de Vértice (VS) e Fragmento (FS) escritos em linguagem GLSL (OpenGL Shading Language - versão `300 es`).
3.  **Mapeamento de Localização de Uniforms:** Captura das referências de memória na GPU das variáveis de matrizes, vetores de luz e flags de controle.
4.  **Definição de Buffers Primitivos:** Criação de arrays de vértices primitivos em memória de vídeo para objetos cúbicos fundamentais (paredes, pisos e tetos).
5.  **Instanciação dos Motores de Parser:** Chamadas assíncronas de leitura via Promises para carregar dinamicamente `vovo.obj` e `car.obj`.
6.  **Definição do Level Design:** Array estruturado contendo as caixas de colisão tridimensionais, coordenadas espaciais e dimensões físicas de todas as paredes, portas e mobílias da cena.
7.  **Loop Principal de Renderização (`render`):** Executado continuamente através da função nativa `requestAnimationFrame`, calculando o tempo delta (`dt`), processando as físicas de colisão do jogador com o ambiente, atualizando o comportamento de busca por Inteligência Artificial do Velho e executando as chamadas de desenho tridimensional (`gl.drawArrays` e `gl.drawElements`).

---

##  Como Executar o Projeto

Devido às diretivas de segurança padrão do navegador (*Same-Origin Policy*), o carregamento de arquivos externos `.obj` e `.mtl` via chamadas assíncronas `fetch` exige que o arquivo seja executado obrigatoriamente através de um servidor HTTP local.

### Passos para Execução:
1.  Certifique-se de que o arquivo principal `HerancaRoubada.html` e os modelos tridimensionais (`vovo.obj`, `car.obj`) e seus respectivos arquivos `.mtl` estejam organizados no mesmo diretório de pastas.
2.  Abra a pasta do projeto no editor de código **VS Code**.
3.  Instale a extensão **Live Server** (caso ainda não a possua instalada).
4.  Clique com o botão direito no arquivo `index.html` e selecione a opção **"Open with Live Server"**.
5.  O jogo abrirá automaticamente em seu navegador padrão no endereço local `http://127.0.0.1:5500`. Certifique-se de dar um clique inicial na tela do jogo para que o ponteiro do mouse seja travado e o áudio/foco de entrada do teclado seja habilitado.