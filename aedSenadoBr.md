# Dados Abertos - Senado Brasileiro, CEAPS

Os presentes dados estão disponíveis publicamente no site do governo brasileiro como parte da politica de acesso a dados públicos de forma aberta afim de ter-se transparência nos gastos públicos.


[Senado.leg.br Site dos dados](https://www12.senado.leg.br/transparencia/dados-abertos-transparencia/dados-abertos-ceaps)

## Carregando módulos usados na análise.

Bibliotecas usadas para carregamentos dos dados, cálculos e renderização dos gráficos.

Incluem:

[Numpy](https://numpy.org/)

[Pandas](https://pandas.pydata.org/)

[Matplotlib](https://matplotlib.org/)

[Plotly - Visualização de dados](https://plotly.com/)


```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import plotly.express as px
import re
import csv
import glob
%matplotlib inline

#pd.options.plotting.backend = "plotly"
```

## Carregamento dos Dados


### Sobre os Dados

Diversas inconsistências e dados faltantes em campos. Detalhes como datas e informação de documento faltando, ou data não preenchida ou preenchida de forma incompleta.



### Ano de 2022

Amostra para o ano de 2022


```python
# header=1 -> ignore first line header
df = pd.read_csv('data/despesa_ceaps_2022.csv', delimiter=';', header=1, encoding='iso-8859-1')
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ANO</th>
      <th>MES</th>
      <th>SENADOR</th>
      <th>TIPO_DESPESA</th>
      <th>CNPJ_CPF</th>
      <th>FORNECEDOR</th>
      <th>DOCUMENTO</th>
      <th>DATA</th>
      <th>DETALHAMENTO</th>
      <th>VALOR_REEMBOLSADO</th>
      <th>COD_DOCUMENTO</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2022</td>
      <td>1</td>
      <td>ACIR GURGACZ</td>
      <td>Aluguel de imóveis para escritório político, c...</td>
      <td>004.948.028-63</td>
      <td>GILBERTO PISELO DO NASCIMENTO</td>
      <td>001/22</td>
      <td>03/01/2022</td>
      <td>Despesa com pagamento de aluguel de imóvel par...</td>
      <td>6000</td>
      <td>2173614</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2022</td>
      <td>1</td>
      <td>ACIR GURGACZ</td>
      <td>Divulgação da atividade parlamentar</td>
      <td>26.320.603/0001-64</td>
      <td>INFORMANAHORA</td>
      <td>000000000000310/A</td>
      <td>04/01/2022</td>
      <td>Despesa com divulgação da atividade parlamenta...</td>
      <td>1500</td>
      <td>2173615</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2022</td>
      <td>1</td>
      <td>ACIR GURGACZ</td>
      <td>Divulgação da atividade parlamentar</td>
      <td>13.659.201/0001-47</td>
      <td>LINHA PURPURA FOTO E VIDEO LTDA</td>
      <td>107</td>
      <td>14/01/2022</td>
      <td>Despesa com produção de texto e edição de víde...</td>
      <td>6000</td>
      <td>2173616</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2022</td>
      <td>1</td>
      <td>ACIR GURGACZ</td>
      <td>Divulgação da atividade parlamentar</td>
      <td>23.652.846/0001-01</td>
      <td>ROBERTO GUTIERREZ DA ROCHA M.E.I.</td>
      <td>187</td>
      <td>18/01/2022</td>
      <td>Divulgação da atividade parlamentar</td>
      <td>1000</td>
      <td>2173618</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2022</td>
      <td>1</td>
      <td>ACIR GURGACZ</td>
      <td>Divulgação da atividade parlamentar</td>
      <td>08.941.827/0001-01</td>
      <td>RONDONIA DINÂMICA COM. E SERV. DE INFORMÁTICA ...</td>
      <td>000000000001772/A</td>
      <td>17/01/2022</td>
      <td>Divulgação da atividade parlamentar</td>
      <td>2000</td>
      <td>2173617</td>
    </tr>
  </tbody>
</table>
</div>



### Todos Anos 2008/2022

Carregando lista de arquivos contendo os dados para análise.


```python
datas = {}
for file in sorted(glob.glob('data/*.csv')):
    
    if re.search(r'[0-9]+(.csv)', file):
        print('> load... ', file)
        datas[ re.search(r'[0-9]+', file ).group(0) ] = pd.read_csv(file, delimiter=';', header=1, encoding='iso-8859-1')
#(lambda d, x: [d.pop(i) for i in x] )(datas, ['2008','2009', '2010', '2011','2012'])
```

    > load...  data/despesa_ceaps_2008.csv
    > load...  data/despesa_ceaps_2009.csv
    > load...  data/despesa_ceaps_2010.csv
    > load...  data/despesa_ceaps_2011.csv
    > load...  data/despesa_ceaps_2012.csv
    > load...  data/despesa_ceaps_2013.csv
    > load...  data/despesa_ceaps_2014.csv
    > load...  data/despesa_ceaps_2015.csv
    > load...  data/despesa_ceaps_2016.csv
    > load...  data/despesa_ceaps_2017.csv
    > load...  data/despesa_ceaps_2018.csv
    > load...  data/despesa_ceaps_2019.csv
    > load...  data/despesa_ceaps_2020.csv
    > load...  data/despesa_ceaps_2021.csv
    > load...  data/despesa_ceaps_2022.csv


### Pré-Processamento dos Dados

Concatenando as tabelas de dados para formar um único da dataframe com os dados para análise.


```python
dff = pd.concat(datas, ignore_index=True)

```


```python
dff.loc[0:dff.shape[0],'VALOR_REEMBOLSADO'].replace(r'(\s)', '', regex=True, inplace=True)
dff.loc[0:dff.shape[0],'VALOR_REEMBOLSADO'].replace(r'(\n)','', regex=True, inplace=True)
dff.loc[0:dff.shape[0], 'VALOR_REEMBOLSADO'].replace(r'(\r)', '', regex=True, inplace=True)
dff.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ANO</th>
      <th>MES</th>
      <th>SENADOR</th>
      <th>TIPO_DESPESA</th>
      <th>CNPJ_CPF</th>
      <th>FORNECEDOR</th>
      <th>DOCUMENTO</th>
      <th>DATA</th>
      <th>DETALHAMENTO</th>
      <th>VALOR_REEMBOLSADO</th>
      <th>COD_DOCUMENTO</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2008</td>
      <td>9</td>
      <td>ADA MELLO</td>
      <td>Contratação de consultorias, assessorias, pesq...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>12351,52</td>
      <td>2.008091e+12</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2008</td>
      <td>9</td>
      <td>ADA MELLO</td>
      <td>Locomoção, hospedagem, alimentação, combustíve...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>386,6</td>
      <td>2.008091e+12</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2008</td>
      <td>10</td>
      <td>ADA MELLO</td>
      <td>Contratação de consultorias, assessorias, pesq...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>12351,52</td>
      <td>2.008101e+12</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2008</td>
      <td>10</td>
      <td>ADA MELLO</td>
      <td>Locomoção, hospedagem, alimentação, combustíve...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2610,68</td>
      <td>2.008101e+12</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2008</td>
      <td>11</td>
      <td>ADA MELLO</td>
      <td>Contratação de consultorias, assessorias, pesq...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>12351,52</td>
      <td>2.008111e+12</td>
    </tr>
  </tbody>
</table>
</div>




```python
dff.loc[0:,'VALOR_REEMBOLSADO'].replace( regex=r'(,)', value='.',  inplace=True )
#dff.loc[0:dff.shape[0],'VALOR_REEMBOLSADO'] =dff.loc[0:dff.shape[0],'VALOR_REEMBOLSADO'].replace( regex={r'(,)':'.'} )
#dfs['VALOR_REEMBOLSADO'] = pd.to_numeric(dff['VALOR_REEMBOLSADO'], downcast='float')
dff.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ANO</th>
      <th>MES</th>
      <th>SENADOR</th>
      <th>TIPO_DESPESA</th>
      <th>CNPJ_CPF</th>
      <th>FORNECEDOR</th>
      <th>DOCUMENTO</th>
      <th>DATA</th>
      <th>DETALHAMENTO</th>
      <th>VALOR_REEMBOLSADO</th>
      <th>COD_DOCUMENTO</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2008</td>
      <td>9</td>
      <td>ADA MELLO</td>
      <td>Contratação de consultorias, assessorias, pesq...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>12351.52</td>
      <td>2.008091e+12</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2008</td>
      <td>9</td>
      <td>ADA MELLO</td>
      <td>Locomoção, hospedagem, alimentação, combustíve...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>386.6</td>
      <td>2.008091e+12</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2008</td>
      <td>10</td>
      <td>ADA MELLO</td>
      <td>Contratação de consultorias, assessorias, pesq...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>12351.52</td>
      <td>2.008101e+12</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2008</td>
      <td>10</td>
      <td>ADA MELLO</td>
      <td>Locomoção, hospedagem, alimentação, combustíve...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2610.68</td>
      <td>2.008101e+12</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2008</td>
      <td>11</td>
      <td>ADA MELLO</td>
      <td>Contratação de consultorias, assessorias, pesq...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>12351.52</td>
      <td>2.008111e+12</td>
    </tr>
  </tbody>
</table>
</div>




```python
dff.loc[0:, 'VALOR_REEMBOLSADO'] = pd.to_numeric(dff.loc[0:,'VALOR_REEMBOLSADO'], downcast='float')
#dff.dropna(inplace=True)
#dff.to_csv('data.csv', index=False)
```

    /tmp/ipykernel_5037/1191592795.py:1: FutureWarning:
    
    In a future version, `df.iloc[:, i] = newvals` will attempt to set the values inplace instead of always setting a new array. To retain the old behavior, use either `df[df.columns[i]] = newvals` or, if columns are non-unique, `df.isetitem(i, newvals)`
    


## Tratamento e Limpeza dos Dados (Data Wrangling)

No campo 'VALOR_REEMBOLSADO' trocar virgula por ponto e converter para tipo numérico (float).


```python
df['VALOR_REEMBOLSADO'] = pd.to_numeric( df['VALOR_REEMBOLSADO'].replace(regex=r'(,)', value='.'), downcast='float')


```


```python
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ANO</th>
      <th>MES</th>
      <th>SENADOR</th>
      <th>TIPO_DESPESA</th>
      <th>CNPJ_CPF</th>
      <th>FORNECEDOR</th>
      <th>DOCUMENTO</th>
      <th>DATA</th>
      <th>DETALHAMENTO</th>
      <th>VALOR_REEMBOLSADO</th>
      <th>COD_DOCUMENTO</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2022</td>
      <td>1</td>
      <td>ACIR GURGACZ</td>
      <td>Aluguel de imóveis para escritório político, c...</td>
      <td>004.948.028-63</td>
      <td>GILBERTO PISELO DO NASCIMENTO</td>
      <td>001/22</td>
      <td>03/01/2022</td>
      <td>Despesa com pagamento de aluguel de imóvel par...</td>
      <td>6000.0</td>
      <td>2173614</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2022</td>
      <td>1</td>
      <td>ACIR GURGACZ</td>
      <td>Divulgação da atividade parlamentar</td>
      <td>26.320.603/0001-64</td>
      <td>INFORMANAHORA</td>
      <td>000000000000310/A</td>
      <td>04/01/2022</td>
      <td>Despesa com divulgação da atividade parlamenta...</td>
      <td>1500.0</td>
      <td>2173615</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2022</td>
      <td>1</td>
      <td>ACIR GURGACZ</td>
      <td>Divulgação da atividade parlamentar</td>
      <td>13.659.201/0001-47</td>
      <td>LINHA PURPURA FOTO E VIDEO LTDA</td>
      <td>107</td>
      <td>14/01/2022</td>
      <td>Despesa com produção de texto e edição de víde...</td>
      <td>6000.0</td>
      <td>2173616</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2022</td>
      <td>1</td>
      <td>ACIR GURGACZ</td>
      <td>Divulgação da atividade parlamentar</td>
      <td>23.652.846/0001-01</td>
      <td>ROBERTO GUTIERREZ DA ROCHA M.E.I.</td>
      <td>187</td>
      <td>18/01/2022</td>
      <td>Divulgação da atividade parlamentar</td>
      <td>1000.0</td>
      <td>2173618</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2022</td>
      <td>1</td>
      <td>ACIR GURGACZ</td>
      <td>Divulgação da atividade parlamentar</td>
      <td>08.941.827/0001-01</td>
      <td>RONDONIA DINÂMICA COM. E SERV. DE INFORMÁTICA ...</td>
      <td>000000000001772/A</td>
      <td>17/01/2022</td>
      <td>Divulgação da atividade parlamentar</td>
      <td>2000.0</td>
      <td>2173617</td>
    </tr>
  </tbody>
</table>
</div>



Converter campo 'DATA' do tipo string para tipo Date. Campos de data não preenchidos ou preenchidos em formato incorreto será ignorados mantendo conteúdo antigo sem ser convertido para o tipo _"datetime"_ do Python.


```python
# Converter DATA para tipo Date
df['DATA'] = pd.to_datetime(df['DATA'], dayfirst=True, errors='ignore')
```

### Tabela dos Dados Pré-processada e Tratada


```python
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ANO</th>
      <th>MES</th>
      <th>SENADOR</th>
      <th>TIPO_DESPESA</th>
      <th>CNPJ_CPF</th>
      <th>FORNECEDOR</th>
      <th>DOCUMENTO</th>
      <th>DATA</th>
      <th>DETALHAMENTO</th>
      <th>VALOR_REEMBOLSADO</th>
      <th>COD_DOCUMENTO</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2022</td>
      <td>1</td>
      <td>ACIR GURGACZ</td>
      <td>Aluguel de imóveis para escritório político, c...</td>
      <td>004.948.028-63</td>
      <td>GILBERTO PISELO DO NASCIMENTO</td>
      <td>001/22</td>
      <td>2022-01-03</td>
      <td>Despesa com pagamento de aluguel de imóvel par...</td>
      <td>6000.00</td>
      <td>2173614</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2022</td>
      <td>1</td>
      <td>ACIR GURGACZ</td>
      <td>Divulgação da atividade parlamentar</td>
      <td>26.320.603/0001-64</td>
      <td>INFORMANAHORA</td>
      <td>000000000000310/A</td>
      <td>2022-01-04</td>
      <td>Despesa com divulgação da atividade parlamenta...</td>
      <td>1500.00</td>
      <td>2173615</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2022</td>
      <td>1</td>
      <td>ACIR GURGACZ</td>
      <td>Divulgação da atividade parlamentar</td>
      <td>13.659.201/0001-47</td>
      <td>LINHA PURPURA FOTO E VIDEO LTDA</td>
      <td>107</td>
      <td>2022-01-14</td>
      <td>Despesa com produção de texto e edição de víde...</td>
      <td>6000.00</td>
      <td>2173616</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2022</td>
      <td>1</td>
      <td>ACIR GURGACZ</td>
      <td>Divulgação da atividade parlamentar</td>
      <td>23.652.846/0001-01</td>
      <td>ROBERTO GUTIERREZ DA ROCHA M.E.I.</td>
      <td>187</td>
      <td>2022-01-18</td>
      <td>Divulgação da atividade parlamentar</td>
      <td>1000.00</td>
      <td>2173618</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2022</td>
      <td>1</td>
      <td>ACIR GURGACZ</td>
      <td>Divulgação da atividade parlamentar</td>
      <td>08.941.827/0001-01</td>
      <td>RONDONIA DINÂMICA COM. E SERV. DE INFORMÁTICA ...</td>
      <td>000000000001772/A</td>
      <td>2022-01-17</td>
      <td>Divulgação da atividade parlamentar</td>
      <td>2000.00</td>
      <td>2173617</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>16593</th>
      <td>2022</td>
      <td>12</td>
      <td>ZEQUINHA MARINHO</td>
      <td>Passagens aéreas, aquáticas e terrestres nacio...</td>
      <td>22.052.777/0001-32</td>
      <td>Exceller Tour</td>
      <td>WIXHAI</td>
      <td>2022-12-06</td>
      <td>Companhia Aérea: LATAM, Localizador: WIXHAI. P...</td>
      <td>2893.04</td>
      <td>2191398</td>
    </tr>
    <tr>
      <th>16594</th>
      <td>2022</td>
      <td>12</td>
      <td>ZEQUINHA MARINHO</td>
      <td>Passagens aéreas, aquáticas e terrestres nacio...</td>
      <td>22.052.777/0001-32</td>
      <td>Exceller Tour</td>
      <td>WITOLM</td>
      <td>2022-12-09</td>
      <td>Companhia Aérea: GOL, Localizador: WITOLM. Pas...</td>
      <td>1180.19</td>
      <td>2192272</td>
    </tr>
    <tr>
      <th>16595</th>
      <td>2022</td>
      <td>12</td>
      <td>ZEQUINHA MARINHO</td>
      <td>Passagens aéreas, aquáticas e terrestres nacio...</td>
      <td>22.052.777/0001-32</td>
      <td>Exceller Tour</td>
      <td>THPKVQ</td>
      <td>2022-12-20</td>
      <td>Companhia Aérea: TAM, Localizador: THPKVQ. Pas...</td>
      <td>2671.90</td>
      <td>2192274</td>
    </tr>
    <tr>
      <th>16596</th>
      <td>2022</td>
      <td>12</td>
      <td>ZEQUINHA MARINHO</td>
      <td>Passagens aéreas, aquáticas e terrestres nacio...</td>
      <td>22.052.777/0001-32</td>
      <td>Exceller Tour</td>
      <td>QNN9HX</td>
      <td>2022-12-21</td>
      <td>Companhia Aérea: AZUL, Localizador: QNN9HX. Pa...</td>
      <td>1334.31</td>
      <td>2192244</td>
    </tr>
    <tr>
      <th>16597</th>
      <td>2022</td>
      <td>12</td>
      <td>ZEQUINHA MARINHO</td>
      <td>Passagens aéreas, aquáticas e terrestres nacio...</td>
      <td>22.052.777/0001-32</td>
      <td>Exceller Tour</td>
      <td>WMQWBX</td>
      <td>2022-12-30</td>
      <td>Companhia Aérea: TAM, Localizador: WMQWBX. Pas...</td>
      <td>2250.72</td>
      <td>2193622</td>
    </tr>
  </tbody>
</table>
<p>16598 rows × 11 columns</p>
</div>



## Gastos Para o Ano de 2022


```python
senadores = df['SENADOR'].unique()
df_gastos_sn = { 'Senador':[], 'GastoAnual':[], 'RegsSemDocumento':[], 'RegsSemDetalhamentoDoGasto':[] }
```


```python
for i in senadores:
    df_gastos_sn['Senador'].append(i)
    df_gastos_sn['GastoAnual'].append( df[ df['SENADOR'] == i ]['VALOR_REEMBOLSADO'].sum() )
    df_gastos_sn['RegsSemDocumento'].append( df[ df['DOCUMENTO'].isna() == True][df['SENADOR'] == i ].isna().sum()['DOCUMENTO'] )
    df_gastos_sn['RegsSemDetalhamentoDoGasto'].append( df[ df['DETALHAMENTO'].isna() == True][df['SENADOR'] == i ].isna().sum()['DETALHAMENTO'] )

```

    /tmp/ipykernel_5037/4245188335.py:4: UserWarning:
    
    Boolean Series key will be reindexed to match DataFrame index.
    
    /tmp/ipykernel_5037/4245188335.py:5: UserWarning:
    
    Boolean Series key will be reindexed to match DataFrame index.
    



```python
#ds = pd.DataFrame( { 'Senador': df_gastos_sn['Senador'], 'Gastos': df_gastos_sn['GastoAnual'] })
ds = pd.DataFrame( df_gastos_sn )
ds = ds.sort_values(by=['GastoAnual'], ascending=False)
ds
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Senador</th>
      <th>GastoAnual</th>
      <th>RegsSemDocumento</th>
      <th>RegsSemDetalhamentoDoGasto</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>50</th>
      <td>LUCAS BARRETO</td>
      <td>511319.78</td>
      <td>0.0</td>
      <td>228.0</td>
    </tr>
    <tr>
      <th>90</th>
      <td>TELMÁRIO MOTA</td>
      <td>488693.40</td>
      <td>0.0</td>
      <td>249.0</td>
    </tr>
    <tr>
      <th>64</th>
      <td>MECIAS DE JESUS</td>
      <td>488586.66</td>
      <td>0.0</td>
      <td>127.0</td>
    </tr>
    <tr>
      <th>68</th>
      <td>OMAR AZIZ</td>
      <td>487541.24</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>CHICO RODRIGUES</td>
      <td>486958.05</td>
      <td>0.0</td>
      <td>72.0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>5</th>
      <td>ANTONIO ANASTASIA</td>
      <td>19647.13</td>
      <td>3.0</td>
      <td>15.0</td>
    </tr>
    <tr>
      <th>33</th>
      <td>GUARACY SILVEIRA</td>
      <td>19285.82</td>
      <td>0.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>49</th>
      <td>LEILA BARROS</td>
      <td>10567.64</td>
      <td>12.0</td>
      <td>12.0</td>
    </tr>
    <tr>
      <th>63</th>
      <td>MARIA ELIZA DE AGUIAR E SILVA</td>
      <td>10136.42</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>84</th>
      <td>SAMUEL ARAUJO</td>
      <td>3233.90</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
<p>97 rows × 4 columns</p>
</div>



### 10 Senadores com Maior Gastos 2022


```python
ds[:10].plot(x='Senador', y='GastoAnual', title='Gastos (R$) Senadores Brasileiros 2022', kind='bar')
```


<div>                            <div id="905bf7bb-6221-47b4-8601-22ec621a910d" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("905bf7bb-6221-47b4-8601-22ec621a910d")) {                    Plotly.newPlot(                        "905bf7bb-6221-47b4-8601-22ec621a910d",                        [{"alignmentgroup":"True","hovertemplate":"Senador=%{x}<br>GastoAnual=%{y}<extra></extra>","legendgroup":"","marker":{"color":"#636efa","pattern":{"shape":""}},"name":"","offsetgroup":"","orientation":"v","showlegend":false,"textposition":"auto","x":["LUCAS BARRETO","TELM\u00c1RIO MOTA","MECIAS DE JESUS","OMAR AZIZ","CHICO RODRIGUES","DAVI ALCOLUMBRE","ROG\u00c9RIO CARVALHO","MAILZA GOMES","ELMANO F\u00c9RRER","ELIZIANE GAMA"],"xaxis":"x","y":[511319.78,488693.39999999997,488586.66,487541.24000000005,486958.0500000001,486554.54000000004,479699.27,465899.61,465194.93000000005,444927.92000000004],"yaxis":"y","type":"bar"}],                        {"template":{"data":{"histogram2dcontour":[{"type":"histogram2dcontour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"choropleth":[{"type":"choropleth","colorbar":{"outlinewidth":0,"ticks":""}}],"histogram2d":[{"type":"histogram2d","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmap":[{"type":"heatmap","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmapgl":[{"type":"heatmapgl","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"contourcarpet":[{"type":"contourcarpet","colorbar":{"outlinewidth":0,"ticks":""}}],"contour":[{"type":"contour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"surface":[{"type":"surface","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"mesh3d":[{"type":"mesh3d","colorbar":{"outlinewidth":0,"ticks":""}}],"scatter":[{"fillpattern":{"fillmode":"overlay","size":10,"solidity":0.2},"type":"scatter"}],"parcoords":[{"type":"parcoords","line":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolargl":[{"type":"scatterpolargl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"scattergeo":[{"type":"scattergeo","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolar":[{"type":"scatterpolar","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"scattergl":[{"type":"scattergl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatter3d":[{"type":"scatter3d","line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattermapbox":[{"type":"scattermapbox","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterternary":[{"type":"scatterternary","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattercarpet":[{"type":"scattercarpet","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"pie":[{"automargin":true,"type":"pie"}]},"layout":{"autotypenumbers":"strict","colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"hovermode":"closest","hoverlabel":{"align":"left"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"bgcolor":"#E5ECF6","angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"ternary":{"bgcolor":"#E5ECF6","aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]]},"xaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"yaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"geo":{"bgcolor":"white","landcolor":"#E5ECF6","subunitcolor":"white","showland":true,"showlakes":true,"lakecolor":"white"},"title":{"x":0.05},"mapbox":{"style":"light"}}},"xaxis":{"anchor":"y","domain":[0.0,1.0],"title":{"text":"Senador"}},"yaxis":{"anchor":"x","domain":[0.0,1.0],"title":{"text":"GastoAnual"}},"legend":{"tracegroupgap":0},"title":{"text":"Gastos (R$) Senadores Brasileiros 2022"},"barmode":"relative"},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('905bf7bb-6221-47b4-8601-22ec621a910d');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>


### 10 Senadores com mais ocorrência de falta de Documento comprobatório do gasto na Base de Dados


```python
ds.sort_values(by=['RegsSemDocumento'], ascending=False)[:10].plot(x='Senador', y='RegsSemDocumento', title='10 Sen. Sem Documento de Registro do Gasto', kind='bar')
```


<div>                            <div id="06bd876d-3979-4aad-b7aa-c399dd7cbf5c" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("06bd876d-3979-4aad-b7aa-c399dd7cbf5c")) {                    Plotly.newPlot(                        "06bd876d-3979-4aad-b7aa-c399dd7cbf5c",                        [{"alignmentgroup":"True","hovertemplate":"Senador=%{x}<br>RegsSemDocumento=%{y}<extra></extra>","legendgroup":"","marker":{"color":"#636efa","pattern":{"shape":""}},"name":"","offsetgroup":"","orientation":"v","showlegend":false,"textposition":"auto","x":["FABIANO CONTARATO","HUMBERTO COSTA","FERNANDO BEZERRA COELHO","JOS\u00c9 SERRA","MARIA DO CARMO ALVES","JAQUES WAGNER","RENAN CALHEIROS","FL\u00c1VIO BOLSONARO","ZENAIDE MAIA","ROG\u00c9RIO CARVALHO"],"xaxis":"x","y":[276.0,114.0,84.0,80.0,39.0,38.0,30.0,27.0,25.0,21.0],"yaxis":"y","type":"bar"}],                        {"template":{"data":{"histogram2dcontour":[{"type":"histogram2dcontour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"choropleth":[{"type":"choropleth","colorbar":{"outlinewidth":0,"ticks":""}}],"histogram2d":[{"type":"histogram2d","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmap":[{"type":"heatmap","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmapgl":[{"type":"heatmapgl","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"contourcarpet":[{"type":"contourcarpet","colorbar":{"outlinewidth":0,"ticks":""}}],"contour":[{"type":"contour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"surface":[{"type":"surface","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"mesh3d":[{"type":"mesh3d","colorbar":{"outlinewidth":0,"ticks":""}}],"scatter":[{"fillpattern":{"fillmode":"overlay","size":10,"solidity":0.2},"type":"scatter"}],"parcoords":[{"type":"parcoords","line":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolargl":[{"type":"scatterpolargl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"scattergeo":[{"type":"scattergeo","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolar":[{"type":"scatterpolar","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"scattergl":[{"type":"scattergl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatter3d":[{"type":"scatter3d","line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattermapbox":[{"type":"scattermapbox","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterternary":[{"type":"scatterternary","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattercarpet":[{"type":"scattercarpet","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"pie":[{"automargin":true,"type":"pie"}]},"layout":{"autotypenumbers":"strict","colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"hovermode":"closest","hoverlabel":{"align":"left"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"bgcolor":"#E5ECF6","angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"ternary":{"bgcolor":"#E5ECF6","aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]]},"xaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"yaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"geo":{"bgcolor":"white","landcolor":"#E5ECF6","subunitcolor":"white","showland":true,"showlakes":true,"lakecolor":"white"},"title":{"x":0.05},"mapbox":{"style":"light"}}},"xaxis":{"anchor":"y","domain":[0.0,1.0],"title":{"text":"Senador"}},"yaxis":{"anchor":"x","domain":[0.0,1.0],"title":{"text":"RegsSemDocumento"}},"legend":{"tracegroupgap":0},"title":{"text":"10 Sen. Sem Documento de Registro do Gasto"},"barmode":"relative"},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('06bd876d-3979-4aad-b7aa-c399dd7cbf5c');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>



```python
# Número de campos não preenchidos para senadora 'Zenaide Maia'. Neste trecho é possível contar quantos senadores não
# preencheram o campo para o documento do gasto declarado.
#df[ df['DOCUMENTO'].isna() == True][df['SENADOR'] == 'ZENAIDE MAIA'].isna().sum()
```


```python

```
