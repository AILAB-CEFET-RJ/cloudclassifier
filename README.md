# Visualização de Dados de Nuvens com Arquivos HDF4 (CloudSat 2B-CLDCLASS)

Este repositório apresenta um exemplo de como abrir, explorar e visualizar dados meteorológicos armazenados em arquivos no formato **HDF4**, como os gerados por produtos do satélite **CloudSat**, em especial o produto `2B-CLDCLASS`.

## 📦 Requisitos

- Python 3.8+
- [Miniconda](https://docs.conda.io/en/latest/miniconda.html) ou Anaconda
- Arquivo `.hdf` do produto CloudSat (ex: `2B-CLDCLASS`)

## ⚙️ Passos de Instalação

### 1. Criar e ativar um ambiente Conda

```bash
conda create -n cloudclassifier python=3.11
conda activate cloudclassifier
```

### 2. Instalar dependências via `conda` e `pip`

```bash
# Instala biblioteca HDF4 nativa (inclui headers como hdf.h)
conda install -c conda-forge hdf4

# Instala bibliotecas Python
pip install pyhdf matplotlib numpy
```

---

## 📂 Organização dos Dados

O arquivo `.hdf` usado é um produto 2B-CLDCLASS contendo os seguintes datasets principais:

- `cloud_scenario`: matriz 2D com a classificação de cenários de nuvem ao longo de perfis verticais.
- `CloudLayerTop`: altura do topo de até 10 camadas de nuvem por perfil.
- `CloudLayerBase`: altura da base das mesmas camadas.

---

## 📊 Visualização

O script abaixo realiza a leitura do arquivo HDF4 e exibe a matriz `cloud_scenario` com sobreposição das camadas de nuvem detectadas:

```python
from pyhdf.SD import SD, SDC
import numpy as np
import matplotlib.pyplot as plt

file_path = "./data/2019140132530_69565_CS_2B-CLDCLASS_GRANULE_P1_R05_E08_F03.hdf"

hdf = SD(file_path, SDC.READ)

cloud_scenario = np.array(hdf.select("cloud_scenario")[:])
layer_top = np.array(hdf.select("CloudLayerTop")[:])
layer_base = np.array(hdf.select("CloudLayerBase")[:])

cloud_scenario = np.ma.masked_where(cloud_scenario < 0, cloud_scenario)
layer_top = np.ma.masked_where(layer_top < 0, layer_top)
layer_base = np.ma.masked_where(layer_base < 0, layer_base)

fig, ax = plt.subplots(figsize=(12, 6))
im = ax.imshow(cloud_scenario.T, aspect='auto', cmap='viridis', origin='lower')
fig.colorbar(im, ax=ax, label='Cenário de Nuvem')
ax.set_title("Cenário de Nuvem com Contornos das Camadas")
ax.set_xlabel("Raios (nray)")
ax.set_ylabel("Altura bin (nbin)")

n_layers = layer_top.shape[1]
for i in range(n_layers):
    ax.plot(layer_base[:, i], label=f'Base Camada {i+1}', linestyle='--', linewidth=0.8)
    ax.plot(layer_top[:, i], label=f'Topo Camada {i+1}', linestyle='-', linewidth=0.8)

ax.legend(loc='upper right', fontsize='small', ncol=2)
plt.tight_layout()
plt.show()
```

---

## ✅ Resultado Esperado

- Um gráfico com o perfil vertical de cenários de nuvem (eixo vertical: altitude bin; eixo horizontal: número do perfil).
- Sobreposição das bases e topos das camadas detectadas.

---

## 🧩 Notas

- O formato HDF4 não é compatível com a biblioteca `h5py`. É necessário usar `pyhdf` + `hdf4` nativo.
- Os valores negativos geralmente indicam dados inválidos e são tratados com `np.ma.masked_where`.

---

## 📖 Referências

- [CloudSat Data Products](https://www.cloudsat.cira.colostate.edu/data-products)
- [pyhdf documentation](https://fhs.github.io/pyhdf/)
