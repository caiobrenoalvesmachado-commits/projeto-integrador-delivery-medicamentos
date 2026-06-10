#  Sistema de Otimização Logística para Entrega de Medicamentos
## Documentação Técnica Detalhada - MVP Final (Entrega 3)

**Curso:** Sistemas de Informação - FAESA  
**Disciplina:** Projeto Integrador III  
**Professor Orientador:** Howard Cruz Roatti  
**Integrantes:** Alexandre Ferreira Placencia, Caio Breno Alves Machado, Miguel Coelho, Joao Pedro Rosa  

---

## 1. Visão Geral da Arquitetura do Sistema
O MVP foi arquitetado seguindo o modelo cliente-servidor integrado a um pipeline de Ciência de Dados. O sistema coleta as coordenadas geográficas de pedidos e farmácias, processa os dados estruturados no banco relacional e aplica algoritmos de Machine Learning para fornecer inteligência preditiva ao entregador.

┌─────────────────┐       ┌─────────────────┐       ┌────────────────────────┐
│   PostgreSQL    │ ────> │  Pandas / Scipy │ ────> │   Scikit-Learn (K-ML)  │
│ (Banco de Dados)│       │ (Processamento) │       │ (Cluster de Demanda)   │
└─────────────────┘       └─────────────────┘       └────────────────────────┘
│
▼
┌────────────────────────────────────────────────────────────────────────────────┐
│             Módulo do Entregador (Interface Interativa em HTML/Folium)         │
│       - Renderização de Mapas de Calor de Alta Densidade (Zonas Quentes)       │
└────────────────────────────────────────────────────────────────────────────────┘

---

## 2. Modelagem de Dados (PostgreSQL)
A persistência de dados foi estruturada utilizando o **PostgreSQL**, garantindo a integridade referencial necessária para o histórico de trajetórias, controle de estabelecimentos e carimbos de data/hora (*timestamps*).

### DDL (Data Definition Language) das Tabelas Principais:

```sql
-- Tabela de Farmácias Parceiras
CREATE TABLE tb_farmacias (
    id_farmacia SERIAL PRIMARY KEY,
    nome_estabelecimento VARCHAR(150) NOT NULL,
    latitude DECIMAL(10, 8) NOT NULL,
    longitude DECIMAL(11, 8) NOT NULL,
    data_cadastro TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tabela de Pedidos / Demandas Farmacêuticas
CREATE TABLE tb_pedidos (
    id_pedido SERIAL PRIMARY KEY,
    id_farmacia INT REFERENCES tb_farmacias(id_farmacia),
    latitude_entrega DECIMAL(10, 8) NOT NULL,
    longitude_entrega DECIMAL(11, 8) NOT NULL,
    valor_entrega DECIMAL(6, 2) NOT NULL,
    status_pedido VARCHAR(30) DEFAULT 'Pendente',
    data_hora_pedido TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tabela de Histórico de Trajetórias (Módulo do Entregador)
CREATE TABLE tb_trajetorias_entregador (
    id_posicao SERIAL PRIMARY KEY,
    id_entregador INT NOT NULL,
    latitude_atual DECIMAL(10, 8) NOT NULL,
    longitude_atual DECIMAL(11, 8) NOT NULL,
    timestamp_posicao TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```
---

## 3. Algoritmos de Ciência de Dados e Machine Learning
Como requisito do perfil de computação e análise do curso, o backend processa as informações utilizando técnicas de análise geoespacial e aprendizado de máquina:

Algoritmo K-Means (Clusterização): Utilizado para identificar automaticamente os agrupamentos geográficos com maior densidade de pedidos em tempo real. O algoritmo converge as coordenadas de latitude e longitude, calculando os centroides que representam os Hotspots (Zonas Quentes) de demanda. O modelo define pontos estratégicos onde o parceiro logístico maximiza suas chances de receber chamadas.

Biblioteca Folium / Renderização HTML: Utilizada para construir a interface de visualização do entregador por meio de mapas interativos de calor baseados na intensidade de pedidos por região geográfica, permitindo suavização visual e alta usabilidade para dispositivos móveis.
