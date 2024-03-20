# Linear-regression-code
## Analise dos dados exclusivos de proteomica por regressao linear

## Importar pacotes 
import pandas as pd
import statsmodels.api as sm
from sklearn.linear_model import LinearRegression
import statsmodels.api as sm

# Dados do MO

## Dados do Excel para um DataFrame
dados_excel = pd.read_excel('regressao_linear.xlsx', sheet_name = 'MO', index_col=0) 
print (dados_excel)

## Loop para calcular a regressão linear e resultados
resultados_regressao = []
for i, gene1 in enumerate(dados_excel.index):
    for j, gene2 in enumerate(dados_excel.index[i+1:], start=i+1):
        x = dados_excel.loc[gene1]
        y = dados_excel.loc[gene2]

        # Adicionando uma constante para o termo linear
        X = sm.add_constant(x)

        # Ajustando o modelo de regressão linear
        modelo = sm.OLS(y, X).fit()

        # Coletando os resultados
        coef_angular = modelo.params[1]  # Coeficiente Angular
        coef_linear = modelo.params[0]   # Coeficiente Linear
        r2 = modelo.rsquared             # R^2
        pearson_corr = x.corr(y)         # Coeficiente de Correlação de Pearson
        f_statistic = modelo.fvalue      # F-statistic
        p_value = modelo.f_pvalue        # p-value do F-test
        coef_hypothesis_test = modelo.pvalues[1]  # Teste de Hipóteses dos Coeficientes
        std_error = modelo.bse[1]        # Erro Padrão da Estimativa

        resultados_regressao.append({
            'gene_x': gene1,
            'gene_y': gene2,
            'coef_angular': coef_angular,
            'coef_linear': coef_linear,
            'r2': r2,
            'pearson_corr': pearson_corr,
            'f_statistic': f_statistic,
            'p_value': p_value,
            'coef_hypothesis_test': coef_hypothesis_test,
            'std_error': std_error
        })

## Criar um DataFrame com os resultados
df_resultados = pd.DataFrame(resultados_regressao)

## Definir o número máximo de linhas por aba
max_linhas_por_aba = 600000

## Dividir o DataFrame em partes menores
partes = [df_resultados[i:i+max_linhas_por_aba] for i in range(0, df_resultados.shape[0], max_linhas_por_aba)]

## Criar um escritor Excel
writer = pd.ExcelWriter('resultados_regressao_completoMO.xlsx', engine='xlsxwriter')

## Salvar cada parte em uma aba (sheet) diferente
for i, parte in enumerate(partes):
    parte.to_excel(writer, sheet_name=f'Parte_{i+1}', index=False)

## Fechar o escritor Excel
writer.save()


# Filtrar dados - MO
## pearson = ±0,4 , ±0,5 e ±0,6

## Carregar os dados do arquivo Excel em um DataFrame
df1_resultados = pd.read_excel('resultados_regressao_completoMO.xlsx', sheet_name=None)

## Definir os valores de correlação desejados
valores_correlacao = [0.4, 0.5, 0.6]

## Criar um escritor Excel
writer = pd.ExcelWriter('resultados_filtradosMO2.xlsx', engine='xlsxwriter')

## Loop para processar cada valor de correlação desejado
for valor_correlacao in valores_correlacao:
    # Substituir as vírgulas por pontos na coluna 'pearson_corr' para garantir que Python interprete corretamente os valores numéricos
    for sheet_name, df in df1_resultados.items():
        df['pearson_corr'] = df['pearson_corr'].astype(str).str.replace(',', '.').astype(float)

        # Filtrar os resultados com base na coluna 'pearson_corr'
        resultados_filtrados = df[(df['pearson_corr'] >= valor_correlacao) | (df['pearson_corr'] <= -valor_correlacao)]

        # Salvar os resultados filtrados em uma aba (sheet) diferente
        resultados_filtrados.to_excel(writer, sheet_name=f'{sheet_name}_filtrados_{str(valor_correlacao).replace(".", "_")}', index=False)

## Fechar o escritor Excel
writer.save()


# Dados do CTL

## Carregar os dados do Excel para um DataFrame
dados2_excel = pd.read_excel('regressao_linear.xlsx', sheet_name = 'CTL', index_col=0) 
# Supondo que a primeira coluna seja o índice dos genes
print (dados2_excel)
## Converter variáveis categóricas em variáveis dummy, se necessário
dados2_excel = pd.get_dummies(dados2_excel)

## Loop para calcular a regressão linear e obter mais parâmetros
resultados2_regressao = []
for i, gene1 in enumerate(dados2_excel.index):
    for j, gene2 in enumerate(dados2_excel.index[i+1:], start=i+1):
        x = dados2_excel.loc[gene1]
        y = dados2_excel.loc[gene2]

        # Ajustando o modelo de regressão linear
        modelo = sm.OLS(y, sm.add_constant(x)).fit()

        # Coletando os resultados
        coef_angular = modelo.params[1]  # Coeficiente Angular
        coef_linear = modelo.params[0]   # Coeficiente Linear
        r2 = modelo.rsquared             # R^2
        pearson_corr = x.corr(y)         # Coeficiente de Correlação de Pearson
        f_statistic = modelo.fvalue      # F-statistic
        p_value = modelo.f_pvalue        # p-value do F-test
        coef_hypothesis_test = modelo.pvalues[1]  # Teste de Hipóteses dos Coeficientes
        std_error = modelo.bse[1]        # Erro Padrão da Estimativa

        resultados2_regressao.append({
            'gene_x': gene1,
            'gene_y': gene2,
            'coef_angular': coef_angular,
            'coef_linear': coef_linear,
            'r2': r2,
            'pearson_corr': pearson_corr,
            'f_statistic': f_statistic,
            'p_value': p_value,
            'coef_hypothesis_test': coef_hypothesis_test,
            'std_error': std_error
        })
## Criar um DataFrame com os resultados
df_resultados2 = pd.DataFrame(resultados2_regressao)

## Definir o número máximo de linhas por aba
max_linhas_por_aba = 900000

## Dividir o DataFrame em partes menores
partes = [df_resultados2[i:i+max_linhas_por_aba] for i in range(0, df_resultados2.shape[0], max_linhas_por_aba)]

## Criar um escritor Excel
writer = pd.ExcelWriter('resultados_regressao_completoCTL.xlsx', engine='xlsxwriter')

## Salvar cada parte em uma aba (sheet) diferente
for i, parte in enumerate(partes):
    parte.to_excel(writer, sheet_name=f'Parte_{i+1}', index=False)

## Fechar o escritor Excel
writer.save()


# Filtrar dados - CTL
## pearson = ±0,4 , ±0,5 e ±0,6

## Carregar os dados do arquivo Excel em um DataFrame
df1_resultados = pd.read_excel('resultados_regressao_completoCTL.xlsx', sheet_name=None)

## Definir os valores de correlação desejados
valores_correlacao = [0.4, 0.5, 0.6]

## Criar um escritor Excel
writer = pd.ExcelWriter('resultados_filtradosCTL.xlsx', engine='xlsxwriter')

## Loop para processar cada valor de correlação desejado
for valor_correlacao in valores_correlacao:
    # Substituir as vírgulas por pontos na coluna 'pearson_corr' para garantir que Python interprete corretamente os valores numéricos
    for sheet_name, df in df1_resultados.items():
        df['pearson_corr'] = df['pearson_corr'].astype(str).str.replace(',', '.').astype(float)

        # Filtrar os resultados com base na coluna 'pearson_corr'
        resultados_filtrados = df[(df['pearson_corr'] >= valor_correlacao) | (df['pearson_corr'] <= -valor_correlacao)]

        # Salvar os resultados filtrados em uma aba (sheet) diferente
        resultados_filtrados.to_excel(writer, sheet_name=f'{sheet_name}_filtrados_{str(valor_correlacao).replace(".", "_")}', index=False)

## Fechar o escritor Excel
writer.save()
