# 🔦 O ROUBO - Stealth Horror 3D

**O ROUBO** é um minigame 3D de *stealth horror* desenvolvido inteiramente em **WebGL 2.0** e JavaScript Vanilla (apoiado pela biblioteca `gl-matrix` para operações matemáticas). O jogador assume o papel de um ladrão noturno que precisa arrombar uma casa, roubar três móveis específicos e fugir pela porta da frente. Mas tenha cuidado: o dono da casa está dormindo e qualquer barulho pode acordá-lo!

Este projeto foi desenvolvido como trabalho prático para a disciplina de Computação Gráfica, aplicando diretamente os conceitos fundamentais do pipeline gráfico.

---

## 🎮 Como Jogar

### Controles
* **[W, A, S, D]**: Movimentação do personagem.
* **[Mouse]**: Movimentação da câmera (First-Person View).
* **[F]**: Interagir (Entrar na casa, abrir/fechar portas internas).
* **[E]**: Roubar itens (Aproxime-se das caixas marcadas e aperte E).

### Objetivo
1. Entre na casa pela porta da frente.
2. Navegue pelos cômodos furtivamente.
3. Encontre e roube os **3 caixotes** espalhados pela casa.
4. Fuja pela porta da frente antes que o velho te alcance!

---

## 🛠️ Aspectos Técnicos e Requisitos de CG Aplicados

O projeto não utiliza motores gráficos (como Unity ou Unreal). Todo o ambiente, física e renderização foram construídos do zero, englobando os seguintes tópicos da Computação Gráfica:

### 1. Transformações Geométricas 3D
Uso extensivo de matrizes para manipular objetos no espaço.
* **Translação, Rotação e Escala**: Aplicados através da matriz de *Model* (Model Matrix) na função `desenhaObjeto()`. O corpo do "velhinho", por exemplo, é montado por múltiplas partes relativas à posição base.
* **Hierarquia Visual**: Cálculo de posições e ângulos independentes para membros do corpo e dobradiças das portas (`hinge`).

### 2. Câmera Virtual (Visualização 3D)
Implementação de uma câmera *First-Person* interativa.
* **Matriz de Visualização (View Matrix)**: Construída utilizando a técnica *LookAt*, baseada em `camPos`, `camFront` e `camUp`.
* **Projeção Perspectiva (Projection Matrix)**: Configurada com um FOV (Field of View) de 45 graus para garantir a percepção de profundidade do ambiente.
* **Mouse Look (Yaw/Pitch)**: Cálculo de trigonometria esférica para rotacionar o vetor frontal da câmera conforme o movimento do mouse do jogador (Pointer Lock API).

### 3. Pipeline Gráfico e Shaders (GLSL)
* **Vertex Shader**: Recebe os vértices (`a_pos`), normais (`a_norm`) e coordenadas de textura (`a_tex`). Realiza a multiplicação pela matriz MVP (Model-View-Projection) e calcula as normais para a iluminação.
* **Fragment Shader**: Responsável por desenhar as texturas e calcular a cor final do pixel, descartando pixels transparentes (`discard`).

### 4. Iluminação Dinâmica
Implementação do modelo de reflexão de **Lambert (Iluminação Difusa)** e **Iluminação Ambiente**:
* **Ambiente Externo (Dia)**: Luz clara e direcional superior, com forte iluminação ambiente.
* **Ambiente Interno**: Luz reduzida e sombria. Quando a IA acorda, a cor da luz transiciona para um vermelho de alerta dinamicamente (`u_lightCol`), modificando os uniformes no Fragment Shader.
* Cálculos de Produto Escalar (`dot(norm, lightDir)`) no shader para simular o comportamento da luz interagindo com a superfície dos modelos.

### 5. Texturização Procedural
Ao invés de carregar imagens externas, o projeto usa a **Canvas 2D API** para pintar procedimentalmente todas as texturas do jogo (tijolos, madeira, piso, grama, roupas) na inicialização (`criarTextura()`). Essas texturas são então convertidas em *Mipmaps* e passadas para a memória da GPU (VRAM).

### 6. Sistema de Animação e Cinemática
* **Interpolação de Movimento**: As portas não abrem instantaneamente; elas calculam a diferença entre o ângulo atual e o alvo (`targetAngle`), gerando um movimento suave.
* **Cutscene e Máquina de Estados**: O jogo controla o foco da câmera via código durante a animação do despertar do dono da casa, retirando o controle do jogador temporariamente para fins de narrativa.
* **Animação Procedural**: A caminhada do velho utiliza uma função seno (`Math.sin(now / 150)`) para oscilar braços e pernas conforme ele corre atrás do jogador.

### 7. Física e Detecção de Colisão
* Implementação de colisões *AABB (Axis-Aligned Bounding Box)* expandidas com raio de colisão (`colideComParedes`).
* O jogador e a IA navegam calculando previsões de passos e deslizando pelas bordas de móveis, paredes e portas.

---

## 💻 Estrutura do Código

* **HTML/CSS**: Estrutura da interface de usuário (HUD), menus e crosshair.
* **Shaders**: Blocos escritos em GLSL (`#version 300 es`) para processamento paralelo na GPU.
* **Geometria Base**: Um array único de Float32 com dados de um Cubo (Vértices, UVs e Normais), reutilizado e manipulado pelas matrizes para formar todas as paredes, portas, móveis e personagens.
* **Game Loop**: A função recursiva `render(now)` gerencia a atualização física e o redraw a ~60FPS.

---

## 👥 Autores

* **Hyan Victor Cunha Gomes** ( Desenvolvedor )
* **Yasmin** ( Tester )

*Desenvolvido utilizando WebGL2 e matemática pesada.*