# 🖼️ Binarização de Imagens Digitais

> Implementação em Python puro de um pipeline completo para conversão de imagens coloridas em **tons de cinza** (0–255) e **imagens binarizadas** (0 e 255), com três métodos de limiarização. Desenvolvido como projeto acadêmico durante o bootcamp de Machine Learning da plataforma DIO.

---

## 📋 Sumário

- [Sobre o Projeto](#sobre-o-projeto)
- [Fundamentação Teórica](#fundamentação-teórica)
  - [Conversão para Tons de Cinza](#conversão-para-tons-de-cinza)
  - [Binarização por Limiarização](#binarização-por-limiarização)
- [Estrutura do Repositório](#estrutura-do-repositório)
- [Dependências](#dependências)
- [Como Executar](#como-executar)
  - [Google Colab](#google-colab)
  - [Localmente](#localmente)
- [Resultados](#resultados)
- [Relatório Técnico](#relatório-técnico)
- [Licença](#licença)

---

## Sobre o Projeto

Este projeto implementa, **do zero e sem uso de funções prontas de OpenCV ou scikit-image**, as principais etapas do pipeline de binarização de imagens digitais:

1. **Carregamento** da imagem colorida (RGB)
2. **Conversão para tons de cinza** via três métodos distintos
3. **Binarização** via três algoritmos de limiarização
4. **Visualização** comparativa com histograma de intensidade

Toda a lógica de processamento é construída com operações matriciais explícitas sobre arrays **NumPy**, expondo os algoritmos matemáticos subjacentes de forma transparente. O projeto foi desenvolvido e testado no **Google Colab**, aplicado a quatro imagens reais de teste.

---

## Fundamentação Teórica

### Conversão para Tons de Cinza

Uma imagem RGB possui três canais (R, G, B) por pixel, cada um no intervalo [0, 255]. A conversão para escala de cinza colapsa esses três valores em um único valor de intensidade `Y`. Foram implementados três métodos:

#### Luminância — ITU-R BT.601 *(padrão adotado)*

Pondera os canais de acordo com a sensibilidade do sistema visual humano. O verde recebe o maior peso por ser a faixa espectral à qual o olho humano é mais sensível:

```
Y = 0.299·R + 0.587·G + 0.114·B
```

#### Luma — ITU-R BT.709

Revisão do padrão anterior para displays de alta definição (HDTV). Coeficientes ajustados para fósforos modernos:

```
Y = 0.2126·R + 0.7152·G + 0.0722·B
```

#### Média Aritmética

Trata os três canais como igualmente relevantes. Simples e didático, mas não reflete a percepção humana de brilho:

```
Y = (R + G + B) / 3
```

---

### Binarização por Limiarização

A binarização mapeia cada pixel da imagem em cinza para um de dois valores: **0 (preto)** ou **255 (branco)**, com base em um limiar T.

#### 1. Limiar Global Simples

O limiar T é definido manualmente. Cada pixel é classificado por comparação direta:

```
b(i,j) = 255  se  Y(i,j) ≥ T
b(i,j) =   0  se  Y(i,j) < T
```

Eficiente para imagens com histograma bimodal claro, mas sensível à escolha de T e à iluminação da cena.

#### 2. Método de Otsu (1979)

Determina **automaticamente** o limiar T* que maximiza a variância inter-classe entre pixels de fundo e de objeto. Dada a fração de pixels abaixo e acima de T (w_b e w_f) e suas médias (μ_b e μ_f):

```
σ²_B(T) = w_b(T) · w_f(T) · [μ_b(T) − μ_f(T)]²

T* = argmax σ²_B(T),  T ∈ {0, ..., 255}
```

O algoritmo percorre o histograma de forma incremental com complexidade **O(N + 256)**, tornando-o eficiente mesmo para imagens de alta resolução.

#### 3. Limiarização Adaptativa Local

Em vez de um limiar global, cada pixel recebe um limiar local calculado pela média de sua vizinhança de tamanho W×W, subtraída de uma constante C:

```
T(i,j)  = média( vizinhança_W(i,j) ) − C
b(i,j)  = 255  se  Y(i,j) ≥ T(i,j)
           0   caso contrário
```

Ideal para imagens com **iluminação não uniforme** (ex.: documentos fotografados com iluminação lateral). Complexidade **O(N · W²)**.

---

## Estrutura do Repositório

```
image-binarization/
│
├── binarizacao.py              # Script principal com todo o pipeline
├── binarizacao_colab.ipynb     # Notebook Google Colab com execução e resultados
│
├── examples/                   # Imagens de teste utilizadas
│   ├── jackie_chan_1.jpg
│   ├── jackie_chan_2.jpg
│   ├── sabrina_sato_1.jpg
│   └── sabrina_sato_2.jpg
│
├── results/                    # Saídas geradas pelo pipeline
│   ├── *_cinza.png
│   ├── *_binaria_global.png
│   ├── *_binaria_otsu.png
│   ├── *_binaria_adaptativa.png
│   └── *_resultado_completo.png
│
├── docs/
│   └── relatorio_binarizacao.docx   # Relatório técnico-acadêmico completo
│
├── requirements.txt
├── LICENSE
└── README.md
```

---

## Dependências

```
Pillow>=10.0.0
numpy>=1.24.0
matplotlib>=3.7.0
```

---

## Como Executar

### Google Colab

A forma mais rápida de rodar o projeto sem nenhuma instalação local:

**1.** Acesse o notebook diretamente:

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/[usuario]/image-binarization/blob/main/binarizacao_colab.ipynb)

**2.** No Colab, instale as dependências (já incluídas na primeira célula do notebook):

```python
!pip install Pillow numpy matplotlib
```

**3.** Faça upload da imagem desejada pelo painel lateral ou use uma das imagens do repositório:

```python
from google.colab import files
uploaded = files.upload()  # selecione sua imagem
```

**4.** Execute o pipeline completo:

```python
from binarizacao import processar_imagem

resultados = processar_imagem(
    caminho_entrada='sua_imagem.jpg',
    metodo_cinza='luminancia',   # 'luminancia' | 'media' | 'luma'
    limiar_global=128,
    janela_adaptativa=11,
)
```

---

### Localmente

**1. Clone o repositório:**

```bash
git clone https://github.com/[usuario]/image-binarization.git
cd image-binarization
```

**2. Crie e ative um ambiente virtual (recomendado):**

```bash
python -m venv venv
source venv/bin/activate      # Linux/macOS
venv\Scripts\activate         # Windows
```

**3. Instale as dependências:**

```bash
pip install -r requirements.txt
```

**4. Execute via linha de comando:**

```bash
# Uso básico
python binarizacao.py imagem.jpg

# Especificar método de conversão para cinza
python binarizacao.py foto.png --metodo luma

# Ajustar limiar global
python binarizacao.py foto.jpg --metodo luminancia --limiar 100

# Janela maior para o método adaptativo
python binarizacao.py foto.jpg --janela 21

# Apenas visualizar, sem salvar arquivos
python binarizacao.py foto.jpg --sem-salvar
```

**5. Ou como módulo Python:**

```python
from binarizacao import (
    rgb_para_cinza_luminancia,
    binarizar_otsu,
    binarizar_adaptativo,
)
import numpy as np
from PIL import Image

img = np.array(Image.open('foto.jpg').convert('RGB'))

cinza          = rgb_para_cinza_luminancia(img)
bin_otsu, T    = binarizar_otsu(cinza)
bin_adapt      = binarizar_adaptativo(cinza, tamanho_janela=11)

print(f'Limiar de Otsu encontrado: {T}')
Image.fromarray(bin_otsu).save('saida_otsu.png')
```

---

## Resultados

O pipeline foi aplicado a quatro imagens reais de teste. Para cada imagem, os três métodos de binarização produziram os seguintes comportamentos:

| Método | Ponto forte | Limitação |
|---|---|---|
| **Limiar Global (T=128)** | Simples e rápido | Sensível a variações de iluminação |
| **Otsu** | Limiar automático, robusto para uso geral | Pode falhar em histogramas não bimodais |
| **Adaptativo Local** | Excelente em iluminação não uniforme | Mais lento; sensível a ruído |

> 💡 Os resultados visuais completos (painéis comparativos com histograma) estão disponíveis na pasta [`results/`](./results/).

---

## Relatório Técnico

O relatório técnico-acadêmico completo do projeto está disponível em:

📄 https://github.com/john-richard13/binariza-o_imagens_py/blob/main/(DIO)relatorio.pdf

O documento cobre: fundamentação matemática dos algoritmos, análise de complexidade computacional, discussão dos resultados e referências bibliográficas (Gonzalez & Woods, Otsu 1979, ITU-R BT.601/BT.709).

---

## Licença

Distribuído sob a licença MIT. Consulte o arquivo [`LICENSE`](./LICENSE) para mais detalhes.

---

<p align="center">
  Desenvolvido como projeto acadêmico durante o bootcamp de Machine Learning da plataforma DIO. Processamento Digital de Imagens (PDI) · 2026
</p>
