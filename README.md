# Stealth Horror: A Invasão Noturna 

**Universidade Estadual do Ceará (UECE)**  
**Disciplina:** Computação Gráfica  
**Professor:** Matheus  
**Desenvolvedores:** Hyan  e Yasmin


---

## 📌 Sobre o Projeto

**"A Invasão Noturna"** é um jogo 3D de *Stealth Horror* em primeira pessoa desenvolvido inteiramente do zero utilizando **WebGL 2.0 puro**, HTML5 Canvas e JavaScript. O projeto dispensa o uso de engines gráficas de alto nível (como Three.js ou Unity), implementando toda a pipeline gráfica, matemática de matrizes, iluminação e leitura de malhas 3D diretamente no código-fonte.

O objetivo do jogador é invadir uma casa durante a noite, roubar 3 objetos de valor (TV, Celular e Colar) e guardá-los no porta-malas do carro de fuga. Mas há um problema: o dono da casa está dormindo, e qualquer passo em falso pode ser fatal.

---

##  Imagens do Jogo

<div align="center">
  <img src="imagem1.png" alt="Gameplay 1" width="32%">
  <img src="imagem2.png" alt="Gameplay 2" width="32%">
  <img src="imagem3.png" alt="Gameplay 3" width="32%">
  <br><br>
  <img src="imagem4.png" alt="Gameplay 4" width="48%">
  <img src="imagem5.png" alt="Gameplay 5" width="48%">
</div>

---

##  Mecânicas de Gameplay

* **Sistema de Stealth (Barulho):** Movimentar-se continuamente gera ruído. Uma barra de tensão no HUD indica o quão perto o Vovô está de acordar. Parar de andar faz a barra esvaziar.
* **Inventário Limitado:** O jogador só pode carregar um item roubado por vez, forçando-o a fazer um percurso de "vai e vem" entre o interior da casa e o carro estacionado no quintal.
* **Timer de Amanhecer:** O cronômetro de 30 segundos não para! Se o cronômetro zerar, o sol nasce e o Vovô acorda instantaneamente.
* **Inteligência Artificial (Máquina de Estados):** O Vovô possui estados lógicos (`DORMINDO`, `ACORDANDO`, `CAÇANDO`). Ao acordar, ele utiliza trigonometria (`Math.atan2`) para rotacionar dinamicamente e encarar o jogador, calculando vetores de direção para persegui-lo pela casa.
* **Limites de IA:** O Vovô está restrito ao interior da casa. Caso o jogador consiga cruzar a porta da frente, ele fica a salvo no gramado.

---

##  Implementações Técnicas (Computação Gráfica)

Este projeto foi construído para aplicar conceitos práticos da disciplina de Computação Gráfica. Destaques do desenvolvimento:

1.  **Leitor de `.OBJ` Customizado:** Foi desenvolvido um *parser* nativo em JavaScript capaz de ler arquivos `.obj` complexos. O algoritmo lê os vértices (`v`), mapeamento UV (`vt`) e normais (`vn`), montando a malha dinamicamente.
2.  **Triangulação Automática:** O *parser* foi programado para identificar polígonos compostos (Quads) no arquivo `.obj` e dividi-los em triângulos renderizáveis pelo WebGL em tempo de execução.
3.  **Suporte a Múltiplos Materiais (`.MTL`):** O código conta com um leitor assíncrono de arquivos `.mtl` que mapeia e aplica múltiplas texturas (PNG) a diferentes grupos de vértices de um único modelo 3D.
4.  **Matemática de Transformações:** Uso extensivo da biblioteca `gl-matrix` para calcular Matrizes de Modelo, Visão e Projeção (MVP), aplicando Translações, Rotações (X, Y, Z) e Escalas independentes para cada objeto da cena.
5.  **Iluminação de Phong Simplificada:** O Fragment Shader implementa cálculo vetorial de luz direcional, combinando níveis de luz Ambiente com luz Difusa baseada na Normal de cada vértice.
6.  **Gerenciamento de Texturas (Discard):** O shader analisa o canal Alpha (transparência) das imagens e utiliza a instrução `discard` no GLSL para não renderizar pixels transparentes.
7.  **Câmera Cinemática (Cutscenes):** O jogo manipula a Matriz de Visão ativamente para roubar o controle do jogador durante o evento do "despertar", calculando o *Pitch* e *Yaw* necessários para forçar a câmera a olhar diretamente para o inimigo.

---

##  Como Executar o Projeto

Como o jogo utiliza requisições assíncronas (Fetch API) para carregar as texturas e os modelos 3D (`.obj` e `.mtl`), as políticas de segurança do navegador (CORS) impedem que o arquivo seja aberto diretamente via duplo-clique.

1. Baixe os arquivos do projeto e certifique-se de que o `index.html`, os modelos (`vovo.obj`, `car.obj`, `Rak_OBJ.mtl`) e todas as imagens `.png` estão no **mesmo diretório**.
2. Abra a pasta no **VS Code**.
3. Instale a extensão **Live Server**.
4. Clique com o botão direito no arquivo `index.html` e selecione **"Open with Live Server"**.

---

##  Controles

* `W, A, S, D` - Movimentação do personagem.
* `Mouse` - Controle de Câmera (Look around).
* `F` - Interagir com portas (Arrombar/Entrar/Sair).
* `E` - Interagir com os itens (Roubar objeto / Guardar no carro).
* `ESC` ou `P` - Pausar o jogo / Liberar o cursor do mouse.