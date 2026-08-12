# 🛡️ Segurança Pública · Municípios do Pará

> Dashboard interativo de análise de segurança pública dos municípios do estado do Pará, com visualização geoespacial, rankings, histórico mensal e análise de vizinhança.

---

## 📸 Visão Geral

Aplicação web **100% client-side** (sem backend) que carrega um GeoJSON municipal e agrega dados de ocorrências criminais diretamente no navegador.

### Principais Recursos:

*   🗺️ **Mapa Coroplético:** Visualização interativa por município com gradiente de cor por indicador.
*   📊 **Gráficos Dinâmicos:** Rankings, série histórica e comparativos mensais usando Chart.js.
*   🔗 **Análise de Vizinhança:** Identificação de epicentros e ilhas de segurança.
*   🧭 **Painel Regional:** Dados agregados por regionais de segurança pública.
*   🔍 **Filtros em Cascata:** Refinamento por regional, cidade, faixa populacional e mês.
*   📋 **Narrativa Automática:** Contextualização textual gerada automaticamente ao selecionar um município.

---

## 🚀 Como Usar

### Pré-requisitos

Nenhuma instalação é necessária, apenas um navegador moderno.

### Executando Localmente

1.  **Clone o repositório:**
    ```bash
    git clone https://github.com/seu-usuario/seu-repositorio.git
    cd seu-repositorio
    ```

2.  **Verifique os arquivos:** Certifique-se de que o arquivo `dados_seguranca_puplica_pa.geojson` está na mesma pasta do `index.html`.

3.  **Inicie um servidor local:** (Obrigatório para carregar o GeoJSON via `fetch`)
    *   **Python:** `python -m http.server 8080`
    *   **Node.js:** `npx serve .`

4.  **Acesse no navegador:** `http://localhost:8080`

> ⚠️ **Não abra o `index.html` diretamente pelo sistema de arquivos (`file://`).** O carregamento do GeoJSON via `fetch` requer um servidor HTTP.

---

## 📁 Estrutura do Projeto

```
projeto/
├── index.html                          # Aplicação completa (HTML + CSS + JS)
└── dados_seguranca_puplica_pa.geojson  # Base de dados geoespacial municipal
```

## 🗂️ Funcionalidades por Aba

### 🗺️ Aba — Visão
*   Mapa coroplético com gradiente de cor por indicador selecionado.
*   Tooltip detalhado ao passar o mouse sobre cada município.
*   Clique no município para ver KPIs e detalhes no painel lateral.
*   Narrativa automática contextualizada.

### 📊 Aba — Ranking
*   Top 20 municípios em gráfico de barras horizontal.
*   Top 50 municípios em lista com barra de progresso relativa.
*   Ordenação por qualquer indicador disponível.

### 📈 Aba — Histórico
*   Série mensal de ocorrências (CVLI, Furto, Roubo).
*   Série mensal de densidades por 100 mil habitantes.
*   Badge indicando filtros ativos.

### 🧭 Aba — Regionais
*   Comparativo entre regionais de segurança pública.
*   Gráfico de barras com linha de média.
*   Tabela detalhada por regional.

### 🔗 Aba — Vizinhança
*   Cálculo de pressão de vizinhança: `valor_município − média_vizinhos`.
*   Mapa recolorido com gradiente azul → branco → vermelho.
*   Tabela completa ordenada por pressão espacial.

---

## 📊 Indicadores Disponíveis

| Indicador | Tipo | Descrição |
| :--- | :--- | :--- |
| **CVLI** | Contagem | Crimes Violentos Letais Intencionais |
| **Furto** | Contagem | Total de furtos registrados |
| **Roubo** | Contagem | Total de roubos registrados |
| **Densidade CVLI** | Taxa | CVLI por 100 mil habitantes |
| **Densidade Furto** | Taxa | Furtos por 100 mil habitantes |
| **Densidade Roubo** | Taxa | Roubos por 100 mil habitantes |

---

## 🔍 Filtros Dinâmicos

Os filtros funcionam em cascata — ao selecionar um, os demais se atualizam automaticamente:

| Filtro | Descrição |
| :--- | :--- |
| 🧭 **Regional** | Filtra por regional de segurança pública |
| 🏙️ **Cidade** | Filtra por município específico |
| 👥 **Faixa Populacional** | Filtra por porte do município |
| 📅 **Mês** | Filtra por mês de ocorrência |

---

## 🔗 Análise de Vizinhança (Pressão Espacial)

A ferramenta calcula automaticamente a pressão de segurança baseada na geografia:

*   🔴 **Epicentro:** Município com índices significativamente maiores que seus vizinhos diretos.
*   🔵 **Ilha de Segurança:** Município com índices menores que a média da sua vizinhança.

---

## 🗺️ Formato Esperado do GeoJSON

Cada `feature` representa um município em um mês específico e deve conter as seguintes propriedades:

```json
{
  "type": "Feature",
  "geometry": { "type": "MultiPolygon", "coordinates": [...] },
  "properties": {
    "municipio": "Belém",
    "regional": "RMS",
    "populacao": 1499641,
    "mes": "2024-01",
    "cvli": 42,
    "furto": 310,
    "roubo": 198,
    "densidade_cvli": 2.8,
    "densidade_furto": 20.7,
    "densidade_roubo": 13.2
  }
}
