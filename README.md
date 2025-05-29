# logica_fuzzy_APS

````markdown
# Documentação do Sistema de Controle Fuzzy para Risco de Falha de Motor

Este documento descreve passo a passo o código Python que implementa um sistema de controle fuzzy usando a biblioteca **scikit-fuzzy**. O objetivo é estimar o *risco de falha* de um motor com base em variáveis de entrada como temperatura, fator de potência, resistência de isolamento, carga aplicada e desbalanceamento de carga.

---

## 1. Pré-requisitos

Antes de rodar o código, certifique-se de ter instalado:

```bash
!pip install scikit-fuzzy
````

Além disso, as bibliotecas a importar são:

```python
import numpy as np
import skfuzzy as fuzz
from skfuzzy import control as ctrl
import matplotlib.pyplot as plt
```

---

## 2. Definição das Variáveis Antecedentes (Inputs)

Cada variável antecedente representa um *input* para o sistema fuzzy. São criados **objetos** `Antecedent` com:

* **Faixa de valores** (`np.arange`)
* **Nome interno** para referência nas regras

```python
temperatura = ctrl.Antecedent(np.arange(-50,  91,   1),  'temperatura')
fat_p      = ctrl.Antecedent(np.arange(  0.0,  1.01, 0.01), 'fator_potencia')
resist_iso = ctrl.Antecedent(np.arange(1000,12001, 1),  'resistencia_isolamento')
carga_apl  = ctrl.Antecedent(np.arange(  0,   101, 1),  'carga_aplicada')
desbal     = ctrl.Antecedent(np.arange(  0,     11, 1),  'desbalanceamento_carga')
```

* **temperatura**: de –50 °C até 90 °C, passo 1 °C
* **fator\_potencia**: 0.00 a 1.00, passo 0.01
* **resistencia\_isolamento**: 1 000 Ω a 12 000 Ω, passo 1 Ω
* **carga\_aplicada**: 0 % a 100 %, passo 1 %
* **desbalanceamento\_carga**: 0 % a 10 %, passo 1 %

> **Observação:** Ainda faltam as definições de funções de pertinência para cada um desses antecedentes (e.g. “baixo”, “médio”, “alto”). Normalmente usam-se triângulos (`fuzz.trimf`) ou trapézios (`fuzz.trapmf`).

---

## 3. Definição da Variável Consequente (Output)

A variável fuzzy de saída, **risco\_falha**, também é criada como um `Consequent`:

```python
risco_falha = ctrl.Consequent(np.arange(0, 101, 1), 'risco_falha')
```

* **risco\_falha**: 0 a 100 (percentual de risco), passo 1

Assim como nos antecedentes, é necessário definir as funções de pertinência (e.g. “baixo”, “médio”, “alto”).

---

## 4. Configuração das Funções de Pertinência

Exemplo de definição (não incluído no código original) para **temperatura**:

```python
temperatura['baixa']  = fuzz.trimf(temperatura.universe, [-50, -50, 20])
temperatura['media']  = fuzz.trimf(temperatura.universe, [    0,  25, 50])
temperatura['alta']   = fuzz.trimf(temperatura.universe, [   30,  90, 90])
```

Repita analogamente para cada variável (`fator_potencia`, `resistencia_isolamento`, etc.) e para o consequente:

```python
risco_falha['baixo'] = fuzz.trimf(risco_falha.universe, [0, 0, 50])
risco_falha['alto']  = fuzz.trimf(risco_falha.universe, [50,100,100])
```

---

## 5. Definição das Regras Fuzzy

As **regras** relacionam padrões de entrada a graus de risco. Exemplo:

```python
from skfuzzy.control import Rule

regra1 = Rule(temperatura['alta'] & desbal['alto'], risco_falha['alto'])
regra2 = Rule(fat_p['baixa'] & carga_apl['alta'], risco_falha['alto'])
regra3 = Rule(resist_iso['baixa'], risco_falha['alto'])
regra4 = Rule(temperatura['baixa'] & fat_p['alta'], risco_falha['baixo'])
# ... adicionar quantas regras forem necessárias
```

No código original, presume-se que exista uma função chamada `regras_ativas_motor()` que encapsula todas essas regras e retorna um objeto `ControlSystem`.

---

## 6. Construção do Sistema de Controle e Simulação

1. **Criação do ControlSystem**:

   ```python
   sistema_fuzzy = ctrl.ControlSystem([regra1, regra2, regra3, regra4, ...])
   ```
2. **Instanciação do simulador**:

   ```python
   simulador = ctrl.ControlSystemSimulation(sistema_fuzzy)
   ```
3. **Atribuição de entradas**:

   ```python
   simulador.input['temperatura'] = 40
   simulador.input['fator_potencia'] = 0.85
   simulador.input['resistencia_isolamento'] = 8000
   simulador.input['carga_aplicada'] = 70
   simulador.input['desbalanceamento_carga'] = 5
   ```
4. **Computação do resultado**:

   ```python
   simulador.compute()
   ```
5. **Leitura do risco calculado**:

   ```python
   risco = simulador.output['risco_falha']
   print(f"Risco de falha estimado: {risco:.2f}%")
   ```

---

## 7. Visualização das Funções de Pertinência

Para entender melhor as formas de pertinência:

```python
temperatura.view()
risco_falha.view()
plt.show()
```

Isso gera gráficos mostrando como cada rótulo fuzzy (“baixo”, “médio”, “alto”) mapeia o universo de valores.

---

> *Este guia deve servir como base para documentar e expandir o sistema fuzzy de estimativa de risco de falha de motores elétricos. Ajuste as funções de pertinência e as regras conforme a criticidade do seu domínio.*
